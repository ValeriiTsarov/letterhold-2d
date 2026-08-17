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
  (`uk_UA.dic`, 352 081 stems + `uk_UA.aff`, 5 782 suffix rules)
- Here: **every inflected form the affix rules generate**, lowercase Ukrainian letters only, **length 2..9**
  → 800 950 words, 13.5 MB (2.2 MB over gzip)

Base forms alone made the game close to unplayable: the list held `КІТ` but not `КОТА`, `ЧИТАТИ` but not
`ЧИТАЮ`, so a 7-tile hand averaged 19.8 legal words against English's 45.6. Expanding the affixes takes it
to 41.0 — parity — which is why the Ukrainian hand is 7 tiles like everyone else's.

The `.aff` is unusually easy to expand: it has **only `SFX` blocks**, single-character flags, no `PFX`, no
continuation flags, no `NEEDAFFIX`/`CIRCUMFIX`/compounding. So the whole expander is:

```js
// node mkuk.mjs uk_UA.aff uk_UA.dic assets/words/uk.txt
import { readFileSync, writeFileSync } from 'node:fs';
const OK = /^[а-щьюяєіїґ]{2,9}$/; // no apostrophes, hyphens, proper nouns or Latin
const rules = new Map(); // flag -> [{strip, add, re}]
for (const line of readFileSync(process.argv[2], 'utf8').split(/\r?\n/)) {
  if (!line.startsWith('SFX ')) continue;
  const [, flag, strip, add, cond] = line.split(/\s+/);
  if (strip === 'Y' || strip === 'N') continue; // block header, not a rule
  if (!rules.has(flag)) rules.set(flag, []);
  rules.get(flag).push({
    strip: strip === '0' ? '' : strip,
    add: add === '0' ? '' : add,
    re: !cond || cond === '.' ? null : new RegExp(cond + '$'),
  });
}
const w = new Set();
const keep = (s) => { if (OK.test(s)) w.add(s.toUpperCase()); };
for (const line of readFileSync(process.argv[3], 'utf8').split(/\r?\n/).slice(1)) {
  const s = line.trim();
  const slash = s.indexOf('/');
  const stem = (slash < 0 ? s : s.slice(0, slash)).trim();
  if (!stem) continue;
  keep(stem); // no NEEDAFFIX in this .aff, so every stem is itself a word
  for (const flag of slash < 0 ? '' : s.slice(slash + 1).trim()) {
    for (const r of rules.get(flag) ?? []) {
      if (r.re && !r.re.test(stem)) continue;
      if (r.strip && !stem.endsWith(r.strip)) continue;
      keep(stem.slice(0, stem.length - r.strip.length) + r.add);
    }
  }
}
writeFileSync(process.argv[4], [...w].sort().join('\n') + '\n', 'utf8');
```

Three things this file's consumers rely on — **keep them true if you regenerate it**:

- **Sorted, and sorted by `.sort()`'s order.** `src/words.ts` bisects this text in place instead of building
  a Set (800k strings costs ~137 MB of heap), so the file must be sorted by UTF-16 code unit — which for
  Cyrillic is *not* the alphabet: Є, І, Ї sort before А and Ґ after Я. `tests/board.test.ts` samples the
  file and fails if a line it contains can't be found.
- **The tile set is derived from this file**, not from feel: `SETS.uk` in `src/board.ts` is its letter
  frequency allocated over 100 tiles, and the point values are that frequency banded backwards. A new word
  list means recomputing both.
- **No apostrophes.** `М'ЯСО` can't be spelled with tiles at all, so those words are filtered out rather
  than left in to be permanently unplayable. Same for `Ґ`-words in practice: `Ґ` has no tile (0.04% of all
  letters), so its 0.27% of the list needs a wildcard.
