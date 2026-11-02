# Kotoba Loop

This is a very simple CLI tool to help with learning Japanese kanji and vocabulary. It glues together Jisho (an online Japanese dictionary) and Anki (the flashcard program), and designed with a context-based learning approach to Kanji in mind; as new kanji are learned, we find new vocabulary that include those Kanji to learn the readings (rather that painful rote memorization of kun and on readings from kanji alone).

![demo.gif](docs/demo.gif)

## What It Does

The tool allows for searching of either specific kanji or words.
- When a word is searched, it will look up the word on Jisho, display its details, and allow you to add it to your Anki vocabulary deck.
- When a kanji is searched, it will first look up the Kanji on Jisho, then fetch a list of vocabulary words that include that kanji, with details including JLPT level, whether the word already exists in your Anki deck and whether the other kanji in the word are already in your list of reviewed kanji. You can then select which words to add to your Anki vocabulary deck.

The workflow is designed to complement kanji study: as you learn new kanji, you naturally build vocabulary that reinforces those characters.

## Who is it for?
Originally just me!

There are literally hundreds of far more polished/sophisticated tools for learning Japanese available on the web. The use-case/workflow here is very niche to my own learning style and circumstances. You might find this tool useful if:
   - You are reading primarily printed Japanese texts (textbooks and novels) rather than online content, where vocabulary mining tools like Rikaichan, or learning apps like Todai are arguably far more useful/convenient.
   - You already have a decent base of spoken Japanese and are focusing on catching up on reading/writing skills (this may apply to learners who grew up speaking, but not reading/writing, Japanese).
   - You hate Duolingo and would actually like to learn a language properly!


## Installation

Using this tool requires some Anki setup:

1. **Anki Desktop** - Download from [apps.ankiweb.net](https://apps.ankiweb.net/)
2. **AnkiConnect Plugin** - Install from [AnkiWeb](https://ankiweb.net/shared/info/2055492159)
   - In Anki: Tools → Add-ons → Get Add-ons → Enter code `2055492159`. This plugin allows external programs to interact with Anki via a local API.
3. **"All in One Kanji" Deck** - Download from [AnkiWeb](https://ankiweb.net/shared/info/1862058740)
   - This is the recommended kanji deck that the tool expects by default.
   - File → Import → Browse to downloaded `.apkg` file
   - The tool will re-order existing cards from this deck for kanji study, while creating new vocabulary cards in a separate deck, `KotobaLoop-Vocabulary`.
   - The kanji deck name can be changed with `[kanji_deck] name` in `~/.config/kotoba-loop/config.toml` (see [sample-config.toml](sample-config.toml)).

Once this is done (and Anki is running with AnkiConnect enabled), install the tool with [uv](https://docs.astral.sh/uv/):

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

Make sure Anki is running and run `kotoba-loop` to start the interactive session:

**Commands:**
- `n` - Look up the kanji on the card currently open in Anki Reviewer
- `a` - Move the most recently looked-up kanji card to the top of its Anki deck
- `c` - Review pending words (toggle which to add, and whether to create a recall card for each), then commit them to Anki
- `q` - Quit the program
- Type a kanji directly (e.g., `食`) - Search for words containing that kanji and show whether its Anki card has been reviewed
- Type `@word` (e.g., `@食べる`) - Look up a specific word on Jisho
- When in word selection mode, enter numbers or ranges (e.g., `1 3 5` or `1-5`) to select words to add


## Japanese definitions

Each new card also includes a Japanese-language definition from [Kotobank](https://kotobank.jp/), shown on the card with furigana for any kanji you haven't reviewed yet. If a definition can't be found, the word stays pending so you can retry.

## Credits

- [Jisho.org](https://jisho.org/) for the excellent Japanese dictionary API
- [AnkiConnect](https://foosoft.net/projects/anki-connect/) for making Anki automation possible
- [All in One Kanji](https://ankiweb.net/shared/info/1862058740) deck creators (whoever you are!)
