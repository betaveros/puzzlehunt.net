---
layout: page
title: "Client-Side Answer Checker: About"
---

[« back](/checker)

### What is this?

This is a simple way to provide an answer checker for a standalone puzzlehunt puzzles.  Enter the correct answer and an optional title and click Generate, and you'll get a link you can share that will allow others to guess answers and tell them if they got it right. If you got a link that says "Check your answer", just guess answers and the website will tell you if you got it right.

### How are correct answers determined?

Standard puzzlehunt conventions: both the correct answer and any guesses are considered as a string of letters A-Z by changing lowercase letters to uppercase and stripping all other characters. So if the correct answer were "EXAMPLE", any of "example", "Ex Ample", "eXaMpLe", and "e:x:ample!!" would be considered correct, but "EXAMPLES" would be considered incorrect.

(It's easy to make a link that has no correct answer if you mess with the URL. Not sure why you'd do that.)

### What's changed recently?

<ins><strong>2022-02-19</strong>: Titles are now base64-encoded in URLs, so they should survive URL percent-encoding fiddling, e.g. from URL shorteners. (Old links that worked should keep working.) (I pushed this on 01-11, but GitHub Pages failed to deploy for some mysterious reason, and I didn't notice.)</ins>

<ins><strong>2022-03-02</strong>: Simplified the main page's interface and moved all this text to a separate page.</ins>

### Why is this a thing?

In typical puzzlehunts, the web server stores the correct answer and checks submitted guesses against it to tell solvers whether their guesses are correct. Because only the server has the correct answer, only the server is capable of doing this checking, so it can impose guess limits or rate limits to discourage brute-forcing or random guessing. However, hosting dynamic content (in which any computation occurs or any secrets are stored on the server) on the Internet is expensive and finicky compared to hosting static content. On the other hand, if you naïvely put the checker into the web page itself, the web page will have to contain the correct answer to check against, and any enterprising individual can just look at the source code and extract the answer that way.

Without a server, there's no way to keep any secrets or persistent state across answer checks (that can't be tampered with by the user) (unless, I don't know, we decide to make Intel SGX or some analogous secure enclave stuff available to the browser one day? But that would be an absurd amount of effort), so there's no way to enforce a persistent guess limit. However, we can use cryptography to impose a rate limit of sorts, by making each answer check computationally intensive. Specifically, we use a [key derivation function](https://en.wikipedia.org/wiki/Key_derivation_function#Password_hashing) just like best-practices password hashing — even more specifically, we use Richard Moore's [scrypt-js](https://github.com/ricmoo/scrypt-js), a pure-JavaScript implementation of Colin Percival's [scrypt](https://www.tarsnap.com/scrypt.html). However, we don't use particularly high security settings because this shouldn't be a high-security setting.

(As a consequence, this checker requires JavaScript. There's no way around it, really.)

### Why don't you support feature X?

I wanted to keep things simple. See e.g. [Choices are Bad](https://blog.beeminder.com/choices/), from Beeminder. URLs are versioned so we can keep things backwards-compatible, though. But if you do have a use case for a client-side answer checker that requires stronger cryptographic guarantees, or (e.g.) a different policy for canonicalizing answers, I'm happy to consider it.

Or just take this code and change it. There's really not that much.

### Why is the puzzle label used in the hash?

The label is used in the hash in order to somewhat discourage a "meddler-in-the-middle" attack of the following form: You publish a puzzle using an answer checker URL, labeled to indicate your authorship and perhaps that the puzzle comes from a competitive setting. An adversary who can't solve your puzzle modifies your puzzle and the answer checker URL to remove your label and then republishes the modified puzzle and checker under their own name. Or, in the competitive setting, they can send the modified puzzle and checker URL to an innocent solver who is unaware of the puzzle's original context and trick them into solving it for the adversary. Including the label in the hash prevents these attacks, at the cost of making relabeling URLs harder for legitimate purposes. However, even with the label used in the hash, the adversary can rehost this checker or create a checker that proxies answer checks to it in a way that embeds the label used in the hash but doesn't surface it to the solver. Or they can just withhold the answer checker from the solver altogether and manually proxy any answers the solver wishes to check.

### Does any of this matter?

Probably not, but occupational hazard. You still probably shouldn't use this checker in any setting with actual stakes where competitive integrity is actually important. I mean, you can use this anywhere you think it fits, but don't sue me if somebody breaks the checker because I missed a different protocol break, or set the scrypt parameters incorrectly, or anything else.

Let me copy in the liability incantation from the [Blue Oak Model License](https://blueoakcouncil.org/license/1.0.0): As far as the law allows, this software comes as is, without any warranty or condition, and no contributor will be liable to anyone for any damages related to this software or this license, under any kind of legal claim.

### What license is this code?

scrypt-js is © 2016 Richard Moore, released under the MIT License.

The surrounding JavaScript code is released under CC0, although attribution is appreciated, and I would request that alternative implementations change the constant used in the salt (`"puzzlehunt.net/checker#"`). <a rel="license" href="http://creativecommons.org/publicdomain/zero/1.0/"><img src="http://i.creativecommons.org/p/zero/1.0/88x31.png" style="border-style: none;" alt="CC0" /></a>
