# Word explanations

One folder per language, `<lang>/<2 letters>.json`, each file a flat `{"FORM": "gloss"}` map. `src/defs.ts`
fetches ONE shard the first time a word in it is tapped — see its header for why nothing here is bundled.

## uk/

Derived from the **Ukrainian Wiktionary** dump (`{{=uk=}}` section → `Значення` → first sense), with the
lemma→form mapping taken from the same hunspell pair `assets/words/uk.txt` was expanded from.

> **License: CC BY-SA 4.0** (uk.wiktionary text, also dual-licensed GFDL). Commercial use is fine — unlike
> the CC BY-NC-SA dictionary data `words/SOURCE.md` warns about — but two conditions travel with it:
> **attribution**, which is why the popup carries a `Вікісловник · CC BY-SA` line (`i18n.ts`), and
> **share-alike**, which applies to these shard files as an adaptation of that text. Keep them plain data
> files in the repo; don't fold the glosses into the bundle where they'd be indistinguishable from code.

- Upstream: `https://dumps.wikimedia.org/ukwiktionary/latest/ukwiktionary-latest-pages-articles.xml.bz2`
  (16 MB → 187 MB of XML, 73 184 pages) + `hunspell-uk_UA_6.8.0.zip` (see `words/SOURCE.md`)
- Here: **366 shards, 66 537 forms, 7.4 MB on disk** — the fattest is `БА.json` at 366 KB, **29 KB gzipped**

**Coverage is 8.3% of `uk.txt`, and that is the ceiling of the source, not of the extractor**: only 15 439
Ukrainian lemmas in the whole dump have a usable sense (41 695 pages carry a `{{=uk=}}` block, but most are
phrases, proper nouns or grammar-only stubs). Skewed the right way, at least — the shorter the word the
likelier it has a gloss, and short words are what gets played:

| letters | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|
| covered | 30% | 30% | 26% | 19% | 14% | 10% | 7.5% | 5.5% |

A second source is the only way up (СУМ is copyright; `dict_uk`'s own lemma data is CC BY-NC-SA and
unusable commercially). Until then the popup says "немає пояснення" for roughly five words in six.

Regenerate (PowerShell — the dump is not in the repo):

```powershell
$t = "$env:TEMP\ukwikt"; New-Item -ItemType Directory -Force $t | Out-Null
iwr 'https://dumps.wikimedia.org/ukwiktionary/latest/ukwiktionary-latest-pages-articles.xml.bz2' -OutFile "$t\d.xml.bz2"
iwr 'https://github.com/brown-uk/dict_uk/releases/download/v6.8.0/hunspell-uk_UA_6.8.0.zip' -OutFile "$t\h.zip"
& 'C:\Program Files\7-Zip\7z.exe' x "$t\d.xml.bz2" -o"$t" -y
Expand-Archive "$t\h.zip" -DestinationPath "$t\h" -Force
node tools/mkdefs.mjs "$t\d.xml" "$t\h\uk_UA.aff" "$t\h\uk_UA.dic" uk
```

Two things the runtime relies on — **keep them true if you regenerate**:

- **Keys are FORMS, uppercase, and every one of them is a line in `words/<lang>.txt`.** `defs.ts` looks up
  the word straight off the board with no lemmatizer, so a key the word list doesn't have is dead weight
  and a form filed under the wrong prefix is invisible. `tests/defs.test.ts` checks both.
- **The filename is the key's first two letters** — Cyrillic filenames, percent-encoded on the wire.
