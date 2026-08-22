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
- Here: **425 shards, 16 149 forms, 3.08 MB on disk** — the fattest is `КО.json` at 82 KB
  (the dump alone gave 347 shards / 5 593 forms / 0.71 MB; the rest is uk.wikipedia, below)

### Hand-written: `tools/defs-manual-uk.json`

951 short words neither source explains **at all** — uk.wiktionary has no page for `сік`, `ЩУР`, `ГЛЕК`,
`ПУД`. Picked from the 3–4 letter words missing a gloss, minus the everyday ones nobody needs explained: a
gloss earns its place only where a player would ask "what IS that?". Written by hand, so **not** Wiktionary
text and not CC BY-SA — the popup's attribution line is a small over-claim on these 951 of 15 266.
`writeShards()` merges the file into the shards on every run (full build and `--refold` both), last, so a
hand-written gloss also overrides the dump. Keys must be board spelling and be in `words/<lang>.txt` —
anything else is skipped with a warning.

Keys are the BOARD spelling, folded like `uk.txt` (`ҐРУНТ` is filed as `ГРУНТ`) — the popup then shows the
true spelling from `words/uk-spell.txt` beside the gloss. Words that only exist with an apostrophe (`М'ЯТА`)
have no gloss yet: they were filtered out of the old word list, so the dump was never read for them. A full
run against a fresh dump picks them up; `--refold` cannot invent them.

**The Wiktionary dump alone tops out at 11.9% of `uk.txt`, and that is the ceiling of the source, not of the
extractor**: only 15 439 Ukrainian lemmas in the whole dump have a usable sense (41 695 pages carry a
`{{=uk=}}` block, but most are phrases, proper nouns or grammar-only stubs). It was 6.4% while `uk.txt`
still held every inflected form — cutting the list to dictionary forms threw away words the dump never had
a lemma for. A second source was the only way up (СУМ is copyright; `dict_uk`'s own lemma data is CC BY-NC-SA
and unusable commercially), and that source is uk.wikipedia — see below. Together with the hand-written file they give **34.4%**,
skewed the right way: the shorter the word the likelier it has a gloss, and short words are what gets played.

| letters | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|
| dump only | 28% | 28% | 24% | 20% | 15% | 11% | 7.9% | 5.8% |
| + wikipedia + manual + derived | 100% | 81% | 79% | 68% | 49% | 39% | 31% | 8% |

9-letter words stay at 6% on purpose: `--wiki` only asks about 3–8 letters, which is what a rack can lay.

### Second source: `tools/defs-wiki-uk.json` (uk.wikipedia lead sentences)

Measured before it was built: of the words the dump has no lemma for, 2.3% have a **Wiktionary** page (so
the extractor was not the bottleneck) and 62.3% have a **Wikipedia** one. `node tools/mkdefs.mjs --wiki uk`
asks the MediaWiki API for the intro of every such word (20 titles per request, queried with the TRUE
spelling from `words/uk-spell.txt`) and keeps the first sentence. Same CC BY-SA as Wiktionary, so the
attribution line in Settings credits both.

What it drops, because an encyclopedia answers a different question than a dictionary: leads about a place,
a person, a film, a band, a deity or a myth (`entityRe()`), stubs under 15 characters, and articles the
redirect walked away from (`БІОГЕН` → "Біогенні речовини"). Disambiguation leads are cut at the second
mention of the headword. 8 974 glosses survive; 1 678 entities and 7 stubs do not.

Every API response, hit or miss, is cached in `%TEMP%\mkdefs-wiki-uk.json` (~29 000 titles, ~40 MB), and
the output is rebuilt from the whole cache each run. So tuning a filter and re-running costs nothing and
touches no network. Delete that file only if you want to spend two hours re-fetching.

`writeShards()` merges the wiki file as a **fallback**: where the dump has a sense, the dump wins — it is
shorter and it is about the word, not about the thing.

Regenerate (PowerShell — the dump is not in the repo):

```powershell
$t = "$env:TEMP\ukwikt"; New-Item -ItemType Directory -Force $t | Out-Null
iwr 'https://dumps.wikimedia.org/ukwiktionary/latest/ukwiktionary-latest-pages-articles.xml.bz2' -OutFile "$t\d.xml.bz2"
iwr 'https://github.com/brown-uk/dict_uk/releases/download/v6.8.0/hunspell-uk_UA_6.8.0.zip' -OutFile "$t\h.zip"
& 'C:\Program Files\7-Zip\7z.exe' x "$t\d.xml.bz2" -o"$t" -y
Expand-Archive "$t\h.zip" -DestinationPath "$t\h" -Force
node tools/mkdefs.mjs "$t\d.xml" "$t\h\uk_UA.aff" "$t\h\uk_UA.dic" uk
```

When the WORD LIST changes and the dump hasn't, there is no reason to download 187 MB of XML again:

```powershell
node tools/mkdefs.mjs --refold uk   # re-key the shards here onto the current uk.txt, drop what's gone
node tools/mkdefs.mjs --wiki uk     # the same, plus top up from uk.wikipedia (cached; add a number to cap it)
```

Two things the runtime relies on — **keep them true if you regenerate**:

- **Keys are FORMS, uppercase, and every one of them is a line in `words/<lang>.txt`.** `defs.ts` looks up
  the word straight off the board with no lemmatizer, so a key the word list doesn't have is dead weight
  and a form filed under the wrong prefix is invisible. `tests/defs.test.ts` checks both.
- **The filename is the key's first two letters** — Cyrillic filenames, percent-encoded on the wire.
