# Progress dashboard

Run `kotoba-loop stats` to show your Kanji, Reading, Grammar, and
vocabulary progress. Overall bars appear together at the top, followed by
Kanji and textbook-part breakdowns. Vocabulary is measured against a core
6,000-word target: the tool adds every vocabulary note created on or after the
configured tracking date to the manually estimated baseline.

For now, the file is read from the directory where you run the command. When
working from this repository, edit the included `manual_progress.toml` and run
`uv run kotoba-loop stats` from the repository root.

The file contains:

```toml
vocab_baseline = 4400
vocab_tracking_start = 2026-09-03

reading_i = 0
reading_i_total = 41
reading_ii = 0
reading_ii_total = 29
reading_iii = 0
reading_iii_total = 11
grammar_i = 0
grammar_i_total = 10
grammar_ii = 0
grammar_ii_total = 11
grammar_iii = 0
grammar_iii_total = 5
grammar_iv = 0
grammar_iv_total = 7
grammar_v = 0
grammar_v_total = 3
grammar_vi = 0
grammar_vi_total = 12
```

`vocab_baseline` is the known-word estimate on `vocab_tracking_start`.
Vocabulary notes added to the configured deck from that date onward increase
this count, whether or not they have been reviewed yet.

Each Reading and Grammar count is an independent number of completed sections.
Its matching `_total` value sets the bar's maximum; the overall maximum is the
sum of those section totals. The included file retains Reading totals of I=41,
II=29 (book sections 42–70), III=11, and Grammar totals of I=10, II=11 (book
sections 11–21), III=5 (book sections 22–26), IV=7, V=3, VI=12.
All nine `_total` keys are required.


