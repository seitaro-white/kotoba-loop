# Furigana article reader

Start the localhost-only web reader with Anki Desktop and AnkiConnect running:

```bash
kanji-vocab-miner web
# Open http://127.0.0.1:8766
```

Paste up to 50,000 characters of plain Japanese text. The limit is checked on
submission rather than silently truncating pasted text. The result colours
unknown words using the bundled, unofficial JLPT N5–N1 vocabulary data; a
**Highlight known words** toggle can extend those colours to known vocabulary.
Choose **Study furigana** (default), **All furigana**, or **No furigana**.
Study mode uses a **Target JLPT level** dropdown (N5–N1, default N2). For unknown
vocabulary harder than the target (e.g. N1 at an N2 target), or absent from the
JLPT lists, readings stay visible even when all its kanji are known. For known
vocabulary, and unknown vocabulary at or easier than the target, a reading hides
only when all kanji represented by that ruby group are known. Known vocabulary
with unknown kanji still gets furigana. Names and non-vocabulary tokens retain
the kanji-only rule. Missing readings are never guessed.
The kanji inventory is separate from vocabulary membership. All/No furigana
override study mode regardless of target level.
Runs of blank lines are reduced to one blank line between paragraphs. Below
the article, an unknown-vocabulary list groups dictionary forms from N5 through
N1, followed by unlisted words. Entries show their article frequency and are
sorted by descending frequency within each level; ties retain first-appearance
order. The responsive vocabulary grid shows each level's distinct-word count and
compact frequency badges. Highlighted words have subtle inline outlines, while
known words stay visually quiet until highlighting is enabled. The grouped furigana
controls, target-level dropdown and separate highlighting toggle work without
reprocessing the article. The target level is not persisted; new articles default
to N2.
**Start Over** returns to an empty editor (or the sample text in the explicit demo).

A token is treated as known when its normalized dictionary identity matches an
`Expression` in the configured Anki vocabulary deck or an entry in
`~/.config/kanji-vocab-miner/known_vocab.txt`. The reader reloads both sources
on every submission, along with kanji from the configured kanji deck. The kanji
inventory uses the existing reviewed-card criterion (`-is:new OR flag:1`), not kanji
that merely occur in vocabulary cards. If either Anki inventory cannot be loaded,
the reader blocks the result rather than silently treating it as empty. Real Anki
verification of this new reader path remains pending; offline tests mock all IO.

The reader uses **Sudachi C-mode segmentation** with its core dictionary. Only
kanji-containing vocabulary receives JLPT backgrounds. The reader extends a verb
or adjective's background over common adjacent inflection endings, so `乗った`,
`食べました` and `降って` each have one continuous highlight. This is display-only:
Sudachi still supplies separate tokens such as `食べ / まし / た`; vocabulary
matching, counts and JLPT level come from the lexical token, not its endings.

The bounded display policy includes past/conditional `た`, polite `ます`/`です`,
negative `ない`/`ぬ`, passive/potential `れる`/`られる`, causative `せる`/`させる`,
desiderative `たい`, and connective `て`/`で`. Adjectival negatives such as
`難しくない` are also grouped. Grouping stops at whitespace, punctuation or other
words: `降ってきた` highlights `降って` but leaves the kana continuation neutral.
Noun copulas, standalone particles/auxiliaries and affixes remain separate;
`子供っぽい` still uses `子供 / っぽい`. This is not a general phrase or compound
joining engine and does not affect Anki cue rendering.

Dictionary-form matching handles inflections without marking unrelated kanji
homophones as known. JLPT vocabulary data is unofficial and incomplete; unlisted
words remain uncoloured, including their display-grouped endings.

Parser-tagged person and place names have **dotted underlines** rather than JLPT
backgrounds and are omitted from the unknown-vocabulary list. This is a small use of
Sudachi's existing tags, not comprehensive name recognition. Names and ambiguous
words can have incorrect or missing readings. The reader keeps original text and uses
grouped ruby when reliable kana alignment is unavailable.

## Offline demo (no Anki needed)

```bash
uv sync
uv run python -m kanji_vocab_miner.web --demo
# Open http://127.0.0.1:8766 (use --port 8767 if needed)
```

The demo is clearly labelled and uses explicit mock inventories: known vocabulary
`学校`, `図書館`, `食べる`, `経済的`; known kanji `学校食見日人本大生`.
It does not read your known-vocabulary file or call Anki. This is never an automatic
fallback for a failed live connection.

Open [the standalone native preview](docs/reader-sudachi-preview.html) for all eight
reviewed passages, native boundaries/readings and comparisons with the original human
annotations. Regenerate it offline with:

```bash
uv run python scripts/preview_sudachi_reader.py
```

SudachiPy **0.6.11** and SudachiDict-core **20260428** are pinned together for binary
compatibility, avoiding the currently unstable 0.7 transition. The core dictionary
adds roughly 70 MB compressed to installation. Sudachi code/dictionary package licensing
is Apache-2.0 with additional incorporated-resource notices; retain those notices when
redistributing. See [SudachiDict LEGAL](https://github.com/WorksApplications/SudachiDict/blob/develop/LEGAL).
The reader and Anki Japanese-definition cues share one UI-independent Sudachi
C-mode tokenizer; Fugashi and UniDic-lite are no longer required.

To retain the 50,000-character input limit within Sudachi's byte limits, long articles
are chunked near sentence/newline boundaries. Extremely long unbroken text may be split
inside a word; normalization-expanded input is reduced into smaller chunks if needed.
No source characters are dropped. The same size handling applies to Japanese cues.

## Anki cue tokenization migration

New Japanese-definition cues use the same pinned Sudachi engine/core dictionary.
They retain the original source text, including CR/LF, repeated blank lines and
leading/trailing whitespace: the reader's newline normalization is **not** applied
to stored definitions. Each sense still has its own escaped `<div class="sense">`
block, separated by `<br>`, and ruby visibility still uses Anki's `rt.known` class.
Reader JLPT metadata, vocabulary filters and name styling do not enter card HTML.

Native C-mode boundaries/readings can differ from the former Fugashi/UniDic-lite
output. Representative offline comparisons:

| Text | Former cue analysis | Sudachi cue analysis |
| --- | --- | --- |
| `国家公務員` | `国家 / 公務 / 員` | One ruby group, `こっかこうむいん` |
| `東京都` | `東京 / 都` | One ruby group, `とうきょうと` |
| `食べました` | `食べ / まし / た` | Same boundaries; okurigana remains outside ruby |
| `取り扱う`, `大人` | Kana-aligned / irregular grouped ruby | Same rendering structure |
| `今日` in the comparison phrase `取り扱う 大人 今日 一日` | `こんにち` | `きょう` |

Readings remain contextual, not guaranteed for ambiguous words or names. Missing
and out-of-vocabulary readings remain plain text; for example Sudachi tagged `一`
in that comparison phrase as OOV, so it receives no ruby. Larger ruby groups also
mean their reading stays visible until **all** represented kanji are known. No
custom joining, deinflection, name recognition or extra dictionary is applied.

This affects **future cue generation only**. Existing notes are not regenerated,
card templates are unchanged, and target-word front-card furigana still comes from
Jisho. Regenerating stored cue fields requires a separate explicit decision.

