# Word lists

One plain-text file per language, `<lang>.txt`, UPPERCASE, one word per line. `src/words.ts` loads the
one the current locale asks for — a new language is a new file here, not a code change.

`uk-spell.txt` sits beside it: `<board spelling> <true spelling>`, one pair per line, only for the words
where the two differ (`МЯТА М'ЯТА`). Ukrainian only, and only the explain popup reads it.

## en.txt

ENABLE (Enhanced North American Benchmark Lexicon), **public domain** — released by Alan Beale / Mendel
Cooper with no restrictions, which is why it and not TWL/Collins is in here: the official tournament
lexicons are licensed works and shipping one in a commercial game needs a deal with Hasbro/Mattel.

- Upstream: `https://raw.githubusercontent.com/dolph/dictionary/master/enable1.txt` (172 823 words)
- Here: uppercased, `a-z` only, **length 2..9** → 105 241 words, 954 KB

Regenerate (PowerShell):

```powershell
$all = Get-Content enable1.txt
$all | Where-Object { $_.Length -ge 2 -and $_.Length -le 9 -and $_ -cmatch '^[a-z]+$' } |
  ForEach-Object { $_.ToUpper() } | Set-Content assets/words/en.txt -Encoding ascii
```

The 9-letter ceiling is a deliberate trade: it drops ~67k long words that a 7-tile rack can almost never
reach, at the cost of rejecting the rare monster built through several tiles already on the board. Raise
it (or drop the filter) if that ever shows up in playtesting.

## uk.txt

Derived from the **hunspell** distribution of [brown-uk/dict_uk](https://github.com/brown-uk/dict_uk),
`hunspell-uk_UA_6.8.0.zip` → `uk_UA.dic`.

> **License: MPL 1.1.** This matters. dict_uk's *dictionary data* (the full 6.5M-word-form corpus,
> `words_spell.txt`, `lemmas.txt`) is **CC BY-NC-SA** — non-commercial, unusable for a CrazyGames
> release. Only the hunspell distribution is MPL 1.1, which permits commercial use as long as the notice
> travels with the file. Keep this section with `uk.txt`; don't swap the source for a "bigger" list
> without re-reading its license.

- Upstream: `https://github.com/brown-uk/dict_uk/releases/download/v6.8.0/hunspell-uk_UA_6.8.0.zip`
  (`uk_UA.dic`, 352 081 stems + `uk_UA.aff`, 5 692 suffix rules)
- Here: the nouns of that file under the rules below → **46 985 words, 0.70 MB**, plus
  `uk-spell.txt` (1 376 lines, the words the tiles spell differently)

```powershell
node tools/mkuk.mjs <path>\uk_UA.aff <path>\uk_UA.dic   # writes both files and prints the tile tables
node tools/mkdefs.mjs --refold uk                       # re-keys the gloss shards onto the new list
```

### The rules

1. **A NOUN, IN THE SINGULAR, AND NOTHING ELSE.** `КІТ`, never `КОТИ`/`КОТА`/`КОТАМИ`; no `ЧИТАТИ`, no
   `ГАРНИЙ`, no `ШВИДКО`, no `АДЖЕ`, no `ФІ`. It is the rule Ukrainian word games are actually played by,
   and the only one short enough that a player never has to ask whether something counts — if you can't
   say "one of those", it isn't a word here. It is also the only rule that ends the argument this list
   kept starting: every abbreviation, interjection, particle and half-conjugated verb hunspell carries is
   gone by construction rather than by a hand list chasing them one at a time.
   Read off the affix PARADIGM: the flag tables in `tools/mkuk.mjs` were classified from one
   representative expansion per flag, a noun keeps the `.dic` spelling (which IS the nominative singular),
   and the affix expansion is now read for nothing but the tile-frequency cross-check the script prints.
2. **Indeclinable nouns come off a hand list** (`INDECLINABLE` in `tools/mkuk.mjs`, ~150 words). They are
   the one kind of noun the paradigm can't vouch for: hunspell gives `кіно` no flags because there is
   nothing to inflect — and it gives `адже`, `вгорі` and `жене` no flags either, so "flagless" means
   "no paradigm", not "noun". Shape doesn't separate them (measured: a vowel-ending rule let 4 122 words
   in, hundreds of them particles and present-tense verbs), so the list is written down instead. It holds
   the common ones — `КІНО`, `КАФЕ`, `ТАКСІ`, `МЕТРО`, `БЮРО`, `ЄВРО`, `ШОУ` — and the rare indeclinables
   (`АДАЖІО`, `БОРДО`, `ГУАНО`) are simply not in the game. Every entry is asserted against the `.dic`,
   so a typo fails the build; add to it freely.
3. **Folded to what the tiles can spell**: `Ґ→Г`, `Ї→І`, apostrophe dropped. `М'ЯТА` is `МЯТА` on the
   board, `ҐРУНТ` is `ГРУНТ`, `ЇЖАК` is `ІЖАК`. The true spelling of every folded word is in
   `uk-spell.txt`, and the explain popup shows that one.
4. **Length 2..9**, lowercase Ukrainian letters only (no hyphens, no Latin, no digits).
5. **A junk list** (`JUNK`, `VERB_FORM` in `tools/mkuk.mjs`) — abbreviations filed like words (`КГ`,
   `ГРН`, `СМС`, `ФБ`), letter pairs no Ukrainian word can be (`ИТ`, `ЕЕ`, `ШШ`), and the present tense
   of `бити`/`вити`/`пити` (`б'є`, `вб'ються`) that dict_uk files as flagless lemmas. Rule 1 now takes
   almost all of it — the list is kept as the belt to that pair of braces, and because `VERB_FORM` still
   has to spare the five borrowed nouns shaped like it (`інтерв'ю`, `круп'є`, `олів'є`).

What this costs is measured, not guessed. `npm run sim` counts the legal words an average hand can spell
standalone: the bare `uk_UA.dic` stems gave 19.8, the full affix expansion 42.8, dictionary-form-any-POS
33.8, and nouns-only **20.3** on a rack of 8, against English's 48.7 on a rack of 7. A tenth of hands read
4 or fewer. Two things pay that back, neither of them here:

- at the RACK — Ukrainian deals **8 tiles** (`SETS.uk.rack` in `src/board.ts`);
- at the MOVE — `SETS.uk.split` lets a turn drop tiles in more than one place, which does not change how
  many words a hand knows but does change how many tiles it spends: 5.05 a turn on one word against 6.46
  across two (`npm run sim`, last column; English reads 5.27 → 6.53).

If the game plays too tight, the rack is the lever — and it is one number in `src/board.ts`: measured
against this list, a rack of **9** reads 31.4 words per hand (p10 8, 5.60 → 7.49 tiles a turn) against
the 20.3 (p10 4) a rack of 8 reads. It was rejected once, when the list still held every part of speech
and nine tiles read 54 — that argument does not survive the list being less than half the size.

The tile counts are NOT the lever: re-derived against a narrower list they measure worse, because a
frequency table over whole words over-counts `И` and starves `К` and `М`. `SETS.uk` stays as it is;
`tools/mkuk.mjs` prints the fresh derivation each run as a cross-check, not as an instruction.

What the rule does not catch: plural-only nouns (`ЩІ`, `АЗИ`) whose `.dic` form is the plural, since the
paradigm cannot tell them from a noun that has a singular somewhere else. There are a few dozen.

Three things this file's consumers rely on — **keep them true if you regenerate it**:

- **Sorted, and sorted by `.sort()`'s order.** `src/words.ts` bisects this text in place instead of building
  a Set (which costs several times the heap the raw text does), so the file must be sorted by UTF-16 code unit — which for
  Cyrillic is *not* the alphabet: Є and І sort before А. `tests/board.test.ts` samples the file and fails if
  a line it contains can't be found.
- **Every letter in it has a tile.** `SETS.uk` in `src/board.ts` came from the letter frequency of the full
  inflected list, allocated over 100 tiles, with the values that frequency banded backwards; it was *kept*
  when the list narrowed, because re-deriving it there measures worse (25.2 words per hand against 26.0) —
  a frequency table over whole words wants ten `И` tiles and starves `К` and `М`. `tools/mkuk.mjs`
  prints the fresh derivation as a cross-check. What the test enforces is the tile-per-letter rule, not the
  arithmetic.
- **`uk-spell.txt` folds back onto it.** Every key is a word in `uk.txt`, and every value folds to its key —
  otherwise the popup tells the player a word is written a way the board could never have accepted.
