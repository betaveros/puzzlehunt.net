---
layout: default
title: Tools
short_title: Tools
---
## Word/Phrase Query Tools

Tool | Corpus | Search pattern support | Semantic search
- | - | - | -
[Nutrimatic](https://nutrimatic.org/) | mined from Wikipedia + Wiktionary | custom regexes w/ anagramming and ([buggily?](https://nutrimatic.org/usage.html#syntax_anagram)) rearranging arbitrary patterns | no
[OneLook](https://onelook.com/) (+ [Reverse Dictionary](https://onelook.com/reverse-dictionary.shtml)) | synthesis of dictionaries | globs* | reverse dictionary
[Qat](https://quinapalus.com/cgi-bin/qat) ([about](https://www.quinapalus.com/qat.html)) | choice/union of dictionaries | custom regexes w/ multiple word constraints, anagramming, letterbanks | "qategories"
[MoreWords](https://www.morewords.com/) | Scrabble-ish dictionary | globs* | no
[RhymeZone](https://www.rhymezone.com/) | dictionary | globs* | no
[CrosswordNexus Wikipedia Regex Search](https://crosswordnexus.com/wiki/) | Wikipedia/Wiktionary titles | Perl regexes w/ backreferences | no
[Lou Hevly's Regex Dictionary](https://www.visca.com/regexdict/) | American Heritage Dictionary 4th | Perl regexes w/ backreferences | no
[Mystery Hunter's Word Search](http://thewordsword.com/) | choice of dictionaries/Wiki titles | Perl regexes w/ backreferences? | no
[NPL's Base Finding Tools](http://wiki.puzzlers.org/dokuwiki/doku.php?id=solving:bases) | choice of dictionaries | single-letter wildcards | no
[OneAcross](http://www.oneacross.com/) | ? | single-letter wildcards | crossword clue database

\* "globs" is shorthand for the "any single letter" wildcard and "any number of letters" wildcard, the two simplest components of [glob patterns in programming](https://en.wikipedia.org/wiki/Glob_%28programming%29).

## Anagramming

- [Free Online Anagram Solver](https://anagram-solver.net/)
- [Internet Anagram Server](http://wordsmith.org/anagram/)
- [OneAcross Anagram Search](http://www.oneacross.com/anagrams/)

## Codes and Ciphers

### Printable code sheets

Table | A=1 Bases | Morse | Braille | Semaphore | ASCII | Others
- | -
[Puzzled Pint (PDF)](http://www.puzzledpint.com/files/2415/7835/9513/CodeSheet-201912.pdf) | 2, 3, 10, 16 | yes | yes | yes | | Pigpen, NATO
[Eric Harshbarger (PDF)](http://www.ericharshbarger.org/epp/code_sheet.pdf) | 2, 10, Roman | yes | yes | yes | yes | ASL, phone, ICS flags, Scrabble, Dvorak, Dancing Man, Gold Bug; also includes digits 0–9
[Netninja.com (PDF)](https://netninja.com/2012/12/02/moleskine-code-sheet/) | 2, 3, 8, 10, 16 | yes | yes | yes | yes | good "reverse" Morse/semaphore lookup

### Other References

- Wikipedia articles for the usual suspects: [Braille](https://en.wikipedia.org/wiki/Braille), [Flag semaphore](https://en.wikipedia.org/wiki/Flag_semaphore), [International maritime signal flags](https://en.wikipedia.org/wiki/International_maritime_signal_flags), [Morse code](https://en.wikipedia.org/wiki/Morse_code), [Pigpen cipher key](https://en.wikipedia.org/wiki/Pigpen_cipher#/media/File:Pigpen_cipher_key.svg), [Spelling alphabets](https://en.wikipedia.org/wiki/Spelling_alphabet)
- [Best ASCII Table](https://bestasciitable.com/)
- [PhoneSpell](https://www.phonespell.org/)
- [HandSpeak](https://www.handspeak.com/)

### Tools

- [Corby's Decoders](http://flystrip.com/corby/code/test.html) (Morse, Braille, semaphore)
- [cryptii](https://cryptii.com/) (a lot of ciphers, encodings (e.g. base64), and modern cryptography (e.g. AES block ciphers))
- [CyberChef](https://gchq.github.io/CyberChef/) (a lot of ciphers, encodings, modern crypto, and parsing modern formats (e.g. parse Protobuf))
- [dCode](https://www.dcode.fr/en) (a lot of ciphers, some math and simple cryptoanalysis tools)
- [monkey](http://www.npinsker.me/puzzles/monkey) (Morse, Braille, semaphore, amino acids)
- [quipqiup](https://www.quipqiup.com/) (substitution ciphers)
- [Rumkin Cipher Tools](http://rumkin.com/tools/cipher/) (a lot of classical ciphers)
- [tools.qhex.org](https://tools.qhex.org/) (basic encodings, some counting cryptoanalysis tools, wordplay commands, automagic extraction, word search, some grid puzzle and polyomino assembly solvers)
- [solvertools](https://github.com/rspeer/solvertools) (Python library including ciphers, wordplay, some types of automatic extraction, and a "cromulence" metric — whether something "looks like" an answer)
- For your phone:

  - iOS: [Puzzle Sidekick](https://apps.apple.com/us/app/puzzle-sidekick/id678644111)
  - Android: [Puzzlehunt Assistant](https://play.google.com/store/apps/details?id=cz.absolutno.sifry&hl=en); [Puzzle Pal](https://play.google.com/store/apps/details?id=com.sqisland.android.puzzlepal)
  - Windows Phone: [Puzzle Tools](https://www.microsoft.com/zh-tw/p/puzzle-tools/9nblggh0jpch?rtc=1&activetab=pivot:overviewtab)

## Infrastructure

- [xword.group](https://xword.group/)
- [Crossword Parser](http://www.npinsker.me/puzzles/crossword)

## Other Lists I Stole From

- [Puzzled Pint resources](http://www.puzzledpint.com/resources/)
- [MIT Mystery Hunt: Puzzle Tools](https://web.mit.edu/puzzle/www/tools.html)
- [Microsoft College Puzzle Challenge: Puzzle Tools and Resources](https://www.collegepuzzlechallenge.com/PuzzleTools.aspx)
