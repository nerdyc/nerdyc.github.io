---
title: Emoji Regular Expressions in Swift
date: 2016-05-17
---

A recent Swift project required detecting emoji characters in various bits of text. This is becoming pretty common — everybody ❤️ emoji!

**It was a huge pain in the ass**. Emoji are much trickier to work with than “normal” characters since they use all kinds of Unicode Voodoo. I kept thinking, “WHY ISN’T THERE JUST A TABLE-FLIPPIN’ REGULAR EXPRESSION FOR EMOJI!?!?”

<!--more-->

So I made some, and I put them on GitHub for you! [SwiftEmoji](https://github.com/nerdyc/SwiftEmoji) provides regular expressions for finding emoji in Swift strings. It’s available via Cocoapods, Carthage, and the Swift Package Manager.

But wait! I didn’t want to half-ass this. Emoji are complex, and change over time. How would I know if got them all? What truly defines whether an emoji is emoji? 🤔

Unicode, of course! Instead of hand-coding the regular expressions, [SwiftEmoji](https://github.com/nerdyc/SwiftEmoji) derives them directly from [the Unicode data files that define emoji](http://www.unicode.org/Public/emoji/2.0/) in the first place. This means the expressions are comprehensive, and easy to update in the future.

Enjoy!

---

**UPDATE:** If you're curious how Emoji work, but don't want to read code, I wrote a follow up that dissects [the seedy private lives of Emoji](/blog/2016/05/19/private-lives-of-emoji/). 🕵