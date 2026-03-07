---
title: "Testing Web Apps from End-to-End in Go" 
---

When I started my first Go project last fall -- a web service backed by PostgreSQL -- getting a test suite setup was my first priority. Since I had no clue how to write Go, I figured tests would be the best way to tell whether my code worked.

Setting up a full integration test suite, running against a local test database, ended up being a big project in itself. Frameworks like Rails provide this out of the box, but the Go community eschews monolithic frameworks and embraces small, focused libraries. I've grown to appreciate that a lot, but it created a much steeper learning curve.

This post shares how I write end-to-end integration tests in Go, with very little code. My tests run against a local PostgreSQL server, and make real HTTP calls to ensure that my services work as a whole.

<!--more-->

## TL;DR

If you prefer to pick apart code instead of English, I've put together [a complete working example on GitHub](https://github.com/nerdyc/testable-golang-web-service). Use it to learn, or as a template for your own projects.

Here's the gist of how it works:

* The `test` sub-package contains shared test setup and helpers.

* The `test.Env` type manages setting up and tearing down global test state, like the connection to PostgreSQL.

* PostgreSQL is running locally (I use [Postgres.app](http://postgresapp.com)), and located via the `DATABASE_URL` environment variable.

* The [`migrate`](https://github.com/mattes/migrate) package by [Matthias Kadenbach](https://github.com/mattes) is used to create a clean database before each test.

* Go's [`net/http/httptest`](https://golang.org/pkg/net/http/httptest/) package provides a test HTTP server that runs the API code.

* Test requests are made using standard HTTP.

The rest of this post will go into more detail, and show how to write clear concise tests.

## Creating The Test Environment

The very first challenge I faced was setting up all the dependencies shared by my tests: 

* A clean PostgreSQL test database.

* An HTTP server running my code.

* An API client to make requests to the test HTTP server.

While I could figure out how to do each of these separately, I had no idea how to share this code easily between tests. Go's testing package is very minimal, and doesn't provide much support for setting up and tearing down test state. In other languages, I put shared code like this in a `TestCase` subclass of some sort. But there's nothing like that in Go.

After much wrangling and refactoring, I ended up with a solution I really enjoy. All global setup is performed by a shared function, `test.SetupEnv()`, which returns a `test.Env` type. The `test.Env` provides all global test state, such as the database and the API client.

Each test starts in a very common, predictable form (e.g. boilerplate):

```go
func Test_GetContactByEmail(t *testing.T) {
  env := test.SetupEnv(t)
  defer env.Close()
  // ...
}
```

The call `env.Close()` cleans up after the test. Without it, tests would get polluted and fail unexpectedly. Nobody likes that.

## Creating the Test Database

The `test.SetupEnv()` function is actually quite small (about 20 lines of code). The vast majority of that is spent setting up the database:

Here's how that works:

* The test database is located via the `DATABASE_URL` environment variable.

* The database is cleaned and migrated using the [`migrate`](https://github.com/mattes/migrate) package.

* Database migrations are written in pure SQL, located in the `db/migrations` directory.

The code is actually short enough to reproduce here:

```go
// SetupDB initializes a test database, performing all migrations.
func SetupDB(t *testing.T) *service.Database {
  databaseUrl := os.Getenv("DATABASE_URL")
  require.NotEmpty(t, databaseUrl, "DATABASE_URL must be set!")

  allErrors, ok := migrate.ResetSync(databaseUrl, "./db/migrations")
  require.True(t, ok, "Failed to migrate database %v", allErrors)

  db, err := sql.Open("postgres", databaseUrl)
  require.NoError(t, err, "Error opening database")

  return &service.Database{db}
}
```

## Setting Up a Test

Most tests require some setup specific to that test. For example, before fetching a Contact from the API can be tests, a Contact needs to be setup first.

The `test.Env` type is a great place to put helpers for setting up things like this. For example:

```go
func Test_GetContactByEmail(t *testing.T) {
  env := test.SetupEnv(t)
  defer env.Close()

  // SETUP:
  env.SetupContact("alice@example.xyz", "Alice Zulu")
  
  // TEST: ...
  
  // VERIFY: ...
}
```

Take a lot of care, and write good test helpers. Whenever I find myself avoiding writing a test, it's usually because setting up the test is too difficult. Good test helpers make all the difference.

I also divide my tests into three distinct steps -- "Setup", "Test", and "Verify". This makes it obvious what code is setting up the test, and what code is running the test.

## Making API Requests

Making HTTP requests is the only way to test that a web service works end-to-end. The first tests I wrote used the `net/http` package directly to make requests. They quickly became repetitive and verbose. Each test was like a tiny API client.

So instead of duplicating that API code, I extracted it into a reusuable API Client. It's set it up as part of the `test.Env`, like any other global. A side benefit is that this client could be used outside tests as well!

Using the `Client` interface, making an API request works like this:

```go
func Test_GetContactByEmail(t *testing.T) {
  env := test.SetupEnv(t)
  defer env.Close()

  // SETUP:
  env.SetupContact("alice@example.xyz", "Alice Zulu")
  
  // TEST: When the contact doesn't exist
  contact, err := env.Client.GetContactByEmail("bob@example.xyz")
  
  // VERIFY: ...
}
```

Any other test or test helper can now reuse the same API code!

## Verifying API Responses

After making a request, we want to assert that the response is what we expect.

The Go `testing` package doesn't provide any helpers for making test assertions. You're expected to use an `if` statement, and then call `testing.Fail()`. That's too austere for my taste. I found it made my tests redundant and hard to read.

I highly recommend the [`testify`](https://github.com/stretchr/testify) package. It provides many useful assertions that save you time and code. It's excellent.

Here's an example of a full test using a `test.Env` and `testify` assertions:

```go
func Test_GetContactByEmail(t *testing.T) {
  env := test.SetupEnv(t)
  defer env.Close()

  // SETUP:
  env.SetupContact("alice@example.xyz", "Alice Zulu")
  
  // TEST: When the contact doesn't exist
  contact, err := env.Client.GetContactByEmail("bob@example.xyz")

  // VERIFY: 404 Not Found returned
  require.Error(t, err)
  require.IsType(t, service.ErrorResponse{}, err)

  assert.Equal(t, http.StatusNotFound, err.(service.ErrorResponse).StatusCode)

  assert.Nil(t, contact)
}
```

Notice that there are two types of assertions. Some begin with `require`, while others begin with `assert`. Those that start with `require` will stop the test immediately if the assertion fails. This is incredibly useful when later assertions depend on earlier ones passing, as in the example.

## Running the Tests

Tests are run via `go test`, just like any other Go project. The only difference is that we also need to provide the `DATABASE_URL`: 

```bash
DATABASE_URL="postgres://username@localhost:5432/service_test?sslmode=disable" go test
```

## That's It!

And that's all there is to it. The hardest part was discovering how to organize test code to allow me to write clear, concise tests. Hopefully this post helps you do the same, without all the struggled that I went through.

[The sample project on GitHub](https://github.com/nerdyc/testable-golang-web-service) will give you a clearer picture than any blog post. There are even more little tips in there. Please let me know if this has helped you!
