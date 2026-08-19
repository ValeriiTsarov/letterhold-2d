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
- Here: the affix expansion under the three rules below → **118 653 words, 1.83 MB**, plus
  `uk-spell.txt` (5 428 lines, the words the tiles spell differently)

```powershell
node tools/mkuk.mjs <path>\uk_UA.aff <path>\uk_UA.dic   # writes both files and prints the tile tables
node tools/mkdefs.mjs --refold uk                       # re-keys the gloss shards onto the new list
```

### The three rules

1. **Dictionary form only, every part of speech** — `КІТ` and `КОТИ` but never `КОТА`/`КОТІВ`/`КОТАМИ`,
   `ЧИТАТИ` but not `ЧИТАЮ`, `ГАРНИЙ` but not `ГАРНОГО`. One sentence a player can hold in their head, and
   the rule Ukrainian word games actually play by. It costs: a 7-tile hand falls from 34.5 playable words
   to 26.0 (English is 48.7), which is why the Ukrainian rack is 8 tiles and not 7. What it buys, besides the rule itself, is glosses — the Wiktionary shards key
   on lemmas, so cutting the list to lemmas takes explanation coverage from 6.4% to 10.0%.
2. **Folded to what the tiles can spell**: `Ґ→Г`, `Ї→І`, apostrophe dropped. `М'ЯТА` is `МЯТА` on the
   board, `ҐРУНТ` is `ГРУНТ`, `ЇЖАК` is `ІЖАК`. This *added* 12 800 apostrophe words that used to be
   filtered out as unplayable and freed the two tiles those letters were squatting on; the true spelling
   of every folded word is in `uk-spell.txt`, and the explain popup shows that one.
3. **Length 2..9**, lowercase Ukrainian letters only (no hyphens, no Latin, no digits).

hunspell carries no part of speech, so "is this a noun, and is this its nominative?" is read off the affix
PARADIGM — the flag tables in `tools/mkuk.mjs` were classified from one representative expansion per flag.
It is not perfect: ~363 oblique forms that dict_uk files as their own flagless lemmas (`ЛЮДЬМИ`, `ОЧЕЙ`)
survive the cut, 0.3% of the list. Comparatives filed as their own lemmas (`ГАРНІШИЙ`) survive too, and
should: they are dictionary headwords.

`npm run sim` is the measurement that decides all of this — legal words an average 7-tile hand can spell
standalone. The bare `uk_UA.dic` stems gave 19.8, the full affix expansion 42.8, the nominative-noun
compromise 34.5, and these rules 26.0 against English's 48.7. That gap is the price of the one-rule
lexicon, and it is paid back at the RACK, not here: Ukrainian deals **8 tiles**, which reads 44.0 — English's
seven, near enough (`SETS.uk.rack` in `src/board.ts`; 9 tiles reads 67.0 and is not a game). The tile counts
were the other candidate and are not the answer — re-derived against this list they measure *worse* (25.2),
because a lemma list still over-counts `И`, so `SETS.uk`'s tables stay as they are.

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
