<h1 align="center">Kotoba Loop</h1>

<p align="center">
  A toolkit for Japanese kanji and vocabulary study, built around Anki.
</p>

<p align="center">
  <img alt="Python 3.12" src="https://img.shields.io/badge/python-3.12-blue">
  <img alt="MIT license" src="https://img.shields.io/badge/license-MIT-green">
  <img alt="Built with uv" src="https://img.shields.io/badge/built%20with-uv-6c47ff">
</p>

<p align="center">
  <a href="#features">Features</a> ·
  <a href="#who-is-it-for">Who is it for?</a> ·
  <a href="#installation">Installation</a> ·
  <a href="#usage">Usage</a> ·
  <a href="#credits">Credits</a>
</p>

<p align="center">
  <img src="docs/demo.gif" alt="Kotoba Loop demo" width="700">
</p>

Kotoba Loop is a toolkit for Japanese vocabulary study. The core underlying concept is a reinforcing "loop" between new Kanji and new words - learning a new Kanji unlocks new words containing said character, which then unlocks other characters, and so on and so on.

The toolkit glues together (fairly haphazardly!) various Japanese dictionaries, Anki for flashcards, LLMs for flashcard generation and TUI & web interfaces for review. It's something that's evolved out of my learning experiences and does not represent a polished product (yet).

## Features

### Kanji-to-Vocab & vice versa

TUI interface linked to Anki. When a new Kanji is learnt, relevant vocabulary is surfaced by difficulty level (JLPT) that contains both the new Kanji and already reviewed Kanji to add to reviews. This reduces the cognitive load of studying new vocabulary - only one new Kanji is being learned, whilst the other(s) should already be familiar.

For example, the user already knows 生 (life), 校 (school) and 水 (water). When they learn 学 (study), the vocabulary 学生 (student) and 学校 (school) are surfaced, since each pairs the new kanji with one they know. The user's reviewed kanji and vocabulary are saved in Anki, forming an ever-growing profile of what they know.

### Adaptive furigana

Based on that profile, furigana is applied only to unknown words/characters on Anki flashcards and the web-reader and updated constantly - users will gradually see less and less furigana over characters as they advance.

### Article reader

Web reader for Japanese texts, which adds adaptive furigana and highlights words based on JLPT difficulty level.

<p align="center">
  <img src="docs/images/reader.png" alt="Article reader showing adaptive furigana and JLPT-coloured words" width="700">
</p>

### LLM enrichment

LLM generation of vocabulary flashcards in Anki, complete with nuance, example sentences and composition notes.

> [!IMPORTANT]
> Committing new vocabulary requires a DeepSeek API key. See [docs/vocabulary-cards.md](docs/vocabulary-cards.md) for setup.

## Who is it for?

Originally just me!

There are literally hundreds of far more polished/sophisticated tools for learning Japanese available on the web. The use-case/workflow here is quite particular to my own learning style and circumstances. However, you might find this tool useful if:

   - You already have a decent base of spoken Japanese and are focusing on catching up on reading/writing skills (this may apply to learners who grew up speaking, but not reading/writing, Japanese).
   - You are happy for studying to be "hard" and don't mind grinding flashcards.
   - You hate Duolingo and would actually like to learn a language properly.

## Installation

Using this tool requires some Anki setup:

1. **Anki Desktop**: download it from [apps.ankiweb.net](https://apps.ankiweb.net/).
2. **AnkiConnect plugin**: in Anki, go to Tools → Add-ons → Get Add-ons and enter code `2055492159` ([AnkiWeb page](https://ankiweb.net/shared/info/2055492159)). The plugin lets external programs talk to Anki through a local API.
3. **"All in One Kanji" deck**: download it from [AnkiWeb](https://ankiweb.net/shared/info/1862058740), then import the `.apkg` with File → Import. This is the kanji deck the tool expects by default.

The tool re-orders cards in the kanji deck for kanji study and creates new vocabulary cards in a separate deck, `KotobaLoop-Vocabulary`. To use a different kanji deck, set `[kanji_deck] name` in `~/.config/kotoba-loop/config.toml` (see [sample-config.toml](sample-config.toml)).

> [!NOTE]
> Anki must be running with AnkiConnect enabled whenever you install, run `setup`, or use the tool.

Once this is done, install the tool with [uv](https://docs.astral.sh/uv/):

```bash
uv tool install git+https://github.com/seitaro-white/kotoba-loop.git
kotoba-loop setup
```

`setup` creates the `KotobaLoop-Vocabulary` deck and the `MyJapaneseVocabularyV3` note type in Anki. This installs `kotoba-loop` globally so you can run it from any directory. If you don't have uv installed, follow the [uv installation guide](https://docs.astral.sh/uv/getting-started/installation/).

To update to the latest version (re-running `setup` refreshes the Anki note type and templates):

```bash
uv tool upgrade kotoba-loop
kotoba-loop setup
```

## Usage

### Interactive Mode

Make sure Anki is running and run `kotoba-loop` to start the interactive session. A typical loop: review a kanji card in Anki, press `n` to look up its kanji, pick the vocabulary you want, then `c` to commit it as new cards.

| Command | What it does |
| --- | --- |
| `n` | Look up the kanji on the card currently open in Anki Reviewer |
| `a` | Move the most recently looked-up kanji card to the top of its Anki deck |
| `c` | Review pending words (toggle which to add, and whether to create a recall card for each), then commit them to Anki |
| `q` | Quit the program |
| a kanji, e.g. `食` | Search for words containing that kanji and show whether its Anki card has been reviewed |
| `@word`, e.g. `@食べる` | Look up a specific word |
| numbers or ranges, e.g. `1 3 5` or `1-5` | In word selection mode, select words to add |

### Article reader

With Anki running, start the web reader and open <http://127.0.0.1:8766>:

```bash
kotoba-loop web
```

Paste Japanese text to see adaptive furigana and JLPT-coloured vocabulary. See [docs/reader.md](docs/reader.md) for the options and an offline demo that doesn't need Anki.

## Japanese definitions

Each new card also includes a Japanese-language definition from [Kotobank](https://kotobank.jp/), shown on the card with furigana for any kanji you haven't reviewed yet. If a definition can't be found, the word stays pending so you can retry.

## Credits

- [Jisho.org](https://jisho.org/) for the excellent Japanese dictionary API
- [Kotobank](https://kotobank.jp/) for the Japanese-language definitions shown on cards
- [AnkiConnect](https://foosoft.net/projects/anki-connect/) for making Anki automation possible
- [All in One Kanji](https://ankiweb.net/shared/info/1862058740) deck creators (whoever you are!)
