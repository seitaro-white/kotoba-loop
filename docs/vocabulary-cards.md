# Vocabulary cards, commit review and enrichment

## Kanji search and spelling selection

Kanji searches check all spellings supplied for each Jisho entry and display the
first spelling containing the searched kanji, with its corresponding reading.
For example, searching `眼` includes `眼鏡` even when Jisho lists `メガネ` first.
Each dictionary entry contributes at most one result. Adding a word preserves that
selected spelling too: if Jisho's word page displays a different primary spelling,
the front uses the matching **Other forms** reading (for example, `眼鏡【めがね】`
rather than replacing it with `メガネ`). Grouped readings stay visible until all
their kanji are known; kana and other non-kanji text remain plain text.
If the word page returns 404 (as for `善い` and `善く`), the front instead uses
an exact **Other forms** spelling from Jisho search results. Unrelated results
are ignored; conflicting readings across matching entries produce an error.

## Setup and API key

To update to the latest version, upgrade the tool and run setup again so Anki has
the current V3 vocabulary note type and templates:

```bash
uv tool upgrade kanji-vocab-miner
kanji-vocab-miner setup
```

A [DeepSeek API key](https://platform.deepseek.com/) is required when committing
new vocabulary, but not for setup, searching, or reviewing pending words. Export
it in the environment used to launch the tool:

```bash
export KANJI_VOCAB_MINER_LLM__API_KEY="your-api-key"
```

The key is environment-only. **Do not put it in `config.toml`.**

Run `kanji-vocab-miner setup` after installing or upgrading. Setup creates or
safely refreshes the required `MyJapaneseVocabularyV3` note type in the existing
`KanjiVocabMiner-Vocabulary` deck. It does not migrate or regenerate existing
cards. Legacy `MyJapaneseVocabulary` and `MyJapaneseVocabularyV2` notes remain
unchanged in the deck.

New pending words default to **Add on** and **Recall off**. The commit review
screen supports:

- `Up`/`Down`: move between rows
- `Space`: toggle Add for the focused row
- `r`: toggle Recall; enabling Recall also enables Add
- `a`/`n`: enable or disable Add for all rows without erasing Recall choices
- `Enter`: commit Add-enabled rows and discard Add-disabled rows
- `Esc`/`q`: abort while preserving pending rows and Recall choices

Each newly committed V3 note receives a Japanese-to-English recognition card.
Its back shows these sections in order:

1. the primary Jisho dictionary definition;
2. **Nuance** — one English sentence about usage beyond the short definition;
3. **Example** — one short Japanese sentence, with the exact used form bold and
   red (no translation or separate reading); and
4. **Why these kanji** — a word-level explanation of the kanji, okurigana, or
   the fact that no kanji applies.

DeepSeek generates the three enrichment sections automatically during commit.
Generation is grounded in the word's first dictionary definition and first part
of speech. There is no generated-content preview or approval step. The existing
recall card behavior is unchanged: enabling Recall also creates a
Japanese-definition-to-Japanese-word card.

The Japanese definition is fetched synchronously from Kotobank for every note,
including recognition-only notes. Definitions prefer デジタル大辞泉 and fall
back to 精選版 日本国語大辞典. If the spelling is missing or has no supported
article, sibling spellings from bundled JMdict entries matching the selected
reading are tried (for example, 択ぶ → 選ぶ and 急劇 → 急激). A sibling is accepted
only when its supported dictionary headword explicitly lists the original spelling;
missing or ambiguous matches still fail. Results are not cached. The normalized text,
rendered cue, source name, and canonical source URL are stored on the note.
DeepSeek generation runs concurrently, while successful notes are written to
Anki sequentially in the reviewed order. The CLI shows one **Pushing to Anki…**
activity spinner throughout the batch, then reports the added, already-existing,
and failed counts; it does not show percentages or changing phase labels.

Each word gets at most three DeepSeek attempts, with short exponential backoff
and validation feedback after an invalid response. If definition lookup, furigana
rendering, enrichment, note serialization, or Anki writing fails, that word is
skipped while other successful rows can still be committed. Failed rows remain pending in reviewed order with
their Recall choice for retry. If the API key is missing, all eligible
nonduplicate rows fail clearly at enrichment and remain pending; the tool never
falls back to creating V2 or unenriched V3 notes.

Legacy, V2, and V3 expressions all participate in duplicate detection. Legacy
and V2 notes remain in the deck and continue to participate in furigana sync
when they contain compatible `Front` or `JapaneseCue` fields.

Cue furigana follows the same reviewed-kanji visibility rules as the vocabulary
word. To enable recall later, set the note's `Recall` field to `1` in Anki. To
disable it, clear `Recall`, then run **Tools → Empty Cards** because Anki keeps
previously generated cards until empty cards are removed.

## DeepSeek enrichment settings

Setup creates the default editable prompt at
`~/.config/kanji-vocab-miner/enrichment-prompt.md` only when that file is
absent. Existing prompt edits are never overwritten. Edit this file to change
content guidance for nuance, examples, and kanji explanations. The application
owns the structured response schema, validation, and safe example highlighting;
the editable prompt does not replace that fixed contract.

The prompt path and request concurrency are the only LLM settings in
`~/.config/kanji-vocab-miner/config.toml`:

```toml
[llm]
prompt_path = "~/.config/kanji-vocab-miner/enrichment-prompt.md"
concurrency = 5
```

`concurrency` controls simultaneous DeepSeek generation requests, defaults to
`5`, and accepts values from `1` through `10`. The API key is not a TOML setting.
Set it in the process environment, or put the following in a `.env` file in the
current working directory (the repository `.gitignore` excludes this file):

```dotenv
KANJI_VOCAB_MINER_LLM__API_KEY="your-api-key"
```

An explicitly exported environment variable takes precedence over `.env`. When
running `uv run kanji-vocab-miner` from the repository root, the local `.env`
file is loaded automatically.
