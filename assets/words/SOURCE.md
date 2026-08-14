# Word lists

One plain-text file per language, `<lang>.txt`, UPPERCASE, one word per line. `src/words.ts` loads the
one the current locale asks for — a new language is a new file here, not a code change.

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
  (`uk_UA.dic`, 352 081 entries)
- Here: the STEM column only (drop the `/flags` suffix), lowercase Ukrainian letters only, **length 2..9**
  → 85 179 words, 1.3 MB

Regenerate (node):

```js
// node mkuk.mjs uk_UA.dic assets/words/uk.txt
import { readFileSync, writeFileSync } from 'node:fs';
const OK = /^[а-щьюяєіїґ]{2,9}$/; // no apostrophes, hyphens, proper nouns or Latin
const w = new Set();
for (const line of readFileSync(process.argv[2], 'utf8').split('\n').slice(1)) {
  const stem = line.split('/')[0].trim();
  if (OK.test(stem)) w.add(stem.toUpperCase());
}
writeFileSync(process.argv[3], [...w].sort().join('\n') + '\n', 'utf8');
```

Two ceilings worth knowing:

- **Base forms only.** A hunspell stem is the dictionary form, so this list holds `КІТ` but not `КОТА`,
  `ЧИТАТИ` but not `ЧИТАЮ`. That matches the usual Ukrainian house rule (називний однини / інфінітив).
  Expanding every inflection means running the affix rules in `uk_UA.aff` and would blow the asset past
  tens of MB — do it only if playtesting says the base-form rule is too tight.
- **No apostrophes.** `М'ЯСО` can't be spelled with tiles at all, so those words are filtered out rather
  than left in to be permanently unplayable.
