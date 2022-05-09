---
title: The Private Lives of Emoji
---

I [recently posted](/blog/2016/05/17/emoji-regular-expressions-in-swift/) about the [Emoji regular expressions](https://github.com/nerdyc/SwiftEmoji) I wrote in Swift. While writing that library, I got to know a lot about how Emoji work. Here’s a tell-all guide to what Emoji are like under the hood! 😳

READMORE

### Simple Emoji

Some Emoji are simple. They lead clean, practical lives and don’t more than a single Unicode code point. The Sparkles emoji (✨, `U+2728`) is one of these, just like the capital letter A (`U+0041`).

Unicode places these in the “Emoji_Presentation” category, since these characters are always presented Emoji-style.

### Emoji-Curious

In contrast to the simple Emoji, some characters are Emoji-curious. They’re simple black-and-white characters most of the time, until “Variant Selector 16” (`U+FE0F`) shows up. When that happens, they come out of the Emoji closet.

The Telephone character is one of these. It can be rendered old-school (☎, `U+260E`) or as an Emoji (☎️, `U+260E U+FEOF`). The only difference is the `U+FEOF`. There are way more of these than you’d think. For example, the digits 0 through 9 are all Emoji-curious (e.g. 0️⃣ through 9️⃣)!

Unicode puts these conflicted characters in the “Emoji” category, WHICH IS SUPER CONFUSING. “Emoji” code points are actually not Emoji unless combined with another code point. “Emoji_Curious” is way more apt, IMHO.

### 👋👋🏻👋🏼👋🏽👋🏾👋🏿

Unicode did something similar when it added skin tones to Emoji. Instead of creating separate-but-equal Emoji for each skin tone, they add five skin-tone modifiers — called “Fitzpatrick Modifiers”.

* 👋 (`U+1F44B`)
* 👋🏻 (`U+1F44B U+1F3FB`)
* 👋🏼 (`U+1F44B U+1F3FC`)
* 👋🏽 (`U+1F44B U+1F3FD`)
* 👋🏾 (`U+1F44B U+1F3FE`)
* 👋🏿 (`U+1F44B U+1F3FF`)

To change the skin tone, just change the modifier. Some differences really are just skin-deep 🤔.

### Freak-Flag Emoji

Things get even more freaky-deaky with flag emoji. For example, the US flag (🇺🇸) is expressed as `U+1F1FA U+1F1F8`.

Isn’t that clever!?! 😂

Oh. You didn’t get the joke? Maybe it’d be funnier if you knew that Uruguay’s flag (🇺🇾) is `U+1F1FA U+1F1FE`.

Did you notice that both characters start with `U+1F1FA`? and both countries’ names begin with U? `U+1F1FA` is the “Regional Indicator Symbol U”, `U+1F1F8` is “Regional Indicator Symbol S”, and `` is “Regional Indicator Symbol Y”.

Instead of creating separate characters for each flag, the nerds at Unicode encoded them using the country’s 2-letter IANA country code (US for the United States, and UY is for Uruguay). 

### Polyamorous Emoji

And finally, some emoji *really* like combining with multiple other emoji. For example, 👨‍👩‍👧‍👦 is actually the combination of the following emoji:

* 👨 (`U+1F468`)
* 👩 (`U+1F469`)
* 👧 (`U+1F467`)
* 👦 (`U+1F466`)

But you can’t make 👨‍👩‍👧‍👦 by simply typing in 👨👩👧👦. Each emoji needs to be combined with a Zero-Width Joiner (`U+0200D`). For this reason they’re known as ZWJ sequences.

I don’t know about you, but when I figured this out I immediately thought I could create bespoke emoji, combining all kinds of Emoji. Unfortunately that’s not the case, at least on my Mac. You can’t even change the skin tone or the ordering of people in the family above.

### 👋

And that’s about all there is to being an Emoji! There are a couple other small gotchas, which you can learn about in the source code to [the Emoji regular expression library I wrote](https://github.com/nerdyc/SwiftEmoji).

