# Developer notes

## Anki implementation boundaries

Within `kanji_vocab_miner/anki/`, `connect.py` owns HTTP and collection access;
its supported legacy add helpers remain unchanged. `note_types.py` owns stable
legacy/V2/V3 field order, model names, templates and CSS. `model_setup.py` creates
models or refreshes only templates/CSS after validating exact field order;
`setup.py` at package level retains wizard and prerequisite presentation.
`furigana_sync.py` migrates stored furigana and updates `Front`/`JapaneseCue`
together by note ID across compatible models, leaving definitions untouched.
`vocab_notes.py` remains the pure current V3 payload builder; application commit
orchestration stays in `vocab_commit.py`.

Internal model definitions use immutable `NoteTypeDefinition` and `CardTemplate`
dataclasses rather than Anki-shaped dictionaries. Stored-note maintenance reads
`VocabNoteFurigana`; each editable fragment produces a `FuriganaChange` with named
HTML and change-status attributes. Serialization remains at the service edge:

```text
NoteTypeDefinition + CardTemplate -> model_setup -> AnkiConnect model payload
notesInfo record -> VocabNoteFurigana -> FuriganaChange -> updateNote field mapping
```

Google-style docstrings describe attributes, parameters, return values and
relevant failures. Existing validated Pydantic models are retained; simple
strings and genuine field-name mappings are not wrapped just for uniformity.
Legacy add helpers retain their executable behavior; their annotations and
docstrings describe optional inputs and per-word failure handling.

Inventory policies are unchanged: reader loads are strict, while reviewed-kanji
CLI loads are best-effort and include `flag:1` cards as well as non-new cards.
Furigana maintenance retains that best-effort policy: a failed reviewed-kanji
load is treated as empty knowledge, which can reveal previously hidden ruby.
HTTP timeout policy and this maintenance failure policy are separate follow-ups,
not behavior changes in the boundary refactor.

## Tests

```bash
uv sync
uv run pytest
```

The default suite uses mocked HTTP services and blocks accidental live Requests
calls. It needs neither a running Anki instance nor Jisho/DeepSeek access.
Dictionary tests use bundled local data or a miniature SQLite fixture.

Live Anki checks are opt-in: `uv run pytest -m integration`. Run them only against
a suitable test collection: some create/delete a test deck or temporarily change
an existing card's due value. They are not part of the default suite.

