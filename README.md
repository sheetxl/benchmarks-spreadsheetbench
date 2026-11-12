# SheetXL on SpreadsheetBench

The complete results of SheetXL's run on [SpreadsheetBench](https://spreadsheetbench.github.io/) Verified-400: every output workbook, every prompt, every model turn and tool call, and the evaluator's result file.

**388/400 = 97.00%** as the leaderboard grades it. **384/400 = 96.00%** under the benchmark's public evaluator, unmodified.

Write-up: <https://www.sheetxl.com/benchmarks/spreadsheetbench>

## The run

| | |
| --- | --- |
| Date | 2026-11-01 |
| Agent | Claude Code, headless (`claude -p`) |
| Model | `claude-opus-5-5` |
| Spreadsheet engine | SheetXL, through its MCP server and the `sheetxl` skill |
| Strategy | One solve turn per task. No verify turn, no retries, no best-of-N |
| The agent sees | `instruction`, `instruction_type`, `answer_position`, the input workbook |
| The agent never sees | `answer_sheet`, `data_position`, any expected answer |
| Recalculation before grading | None. Outputs carry the values SheetXL computed |
| SheetXL build | `d7ea466456`, built from source on the machine that ran the benchmark |

The logs are exactly as submitted to the benchmark's maintainers, unedited. Paths in them are that machine's.

## What is here

```text
verified_400/
  README.md                  the run's own notes, and every full run of this harness with its score
  eval_single_sheetxl.json   the public evaluator's results, unmodified evaluator
  outputs/{id}-output.xlsx   the graded workbook of each task
  logs/{id}/prompt.md        the prompt exactly as sent
  logs/{id}/transcript.jsonl the model's outputs and every tool call and result
  logs/{id}/sheetxl-log/     every SheetXL call: full source, status, duration
  harness/prompt.md          the prompt template
  harness/rules.md           the rules the prompt includes
  harness/skill/             the sheetxl skill the agent loads
```

To check the score, run the benchmark's `evaluation/evaluation.py` over `verified_400/outputs/` and compare with `eval_single_sheetxl.json`.

## Two numbers

The public evaluator raises an exception on four tasks for every submission, because it cannot parse their `answer_position`:

| Task | `answer_position` | What breaks |
| --- | --- | --- |
| 283-32 | `Sheet3'!A:G,'Sheet4'!A:G` | a whole-column range, and a missing leading quote |
| 130-9 | `'b2b, sez, de'!A5:V10` | the sheet name contains commas, and the evaluator splits on `,` |
| 45944 | `G4:G6, G11:G13, G20:G22` | the space after each comma |
| 49300 | `\xa0'Sheet1'!C2:C3` | a leading non-breaking space |

With the position read correctly, our outputs pass all four. That is the difference between 384 and 388. Other published entries count the same tasks the same way: GRID's published outputs score 361 under the unmodified evaluator and 365, its leaderboard score, with these four read correctly.

These four tasks put the public evaluator's ceiling at 396/400. One more, 13284, cannot be passed by any output as shipped: its expected answer was computed from data that is not in the input.

## Published results compared

| Agent | Score | Published |
| --- | --- | --- |
| **SheetXL** | **97.00%** | this repository |
| Tetra (DealGlass) | 94.25% | [evaluation logs and results](https://github.com/arthursolwayne/spreadsheet-agents) |
| GPT for Excel (Talarian) | 92.50% | [write-up](https://autosheet.com/blog/gpt-for-excel-on-spreadsheetbench) |
| GRID Agent | 91.25% | [outputs, transcripts, evaluator results](https://github.com/GRID-is/spreadsheetbench-results), [runner](https://github.com/GRID-is/sheetbench-runner) |
| Leni | 91.25% | [outputs, running log, results](https://github.com/arnabdastidar/leni-spreadsheetbench-submission) |

Thank you to each of them for publishing. If we have missed a published result, open an issue and we will add it.

## Tokens and time

Per task, averaged over the 400 tasks, against GRID's published `results.json`:

| | SheetXL | GRID Agent | |
| --- | --- | --- | --- |
| Model | Claude Opus 5.5 | Claude Sonnet 5 | |
| Output tokens | 2,939 | 9,978 | 3.4× fewer |
| Input tokens, cached included | 219k | 439k | 2.0× fewer |
| Tool calls | 9.4 | 27.2 | 2.9× fewer |
| Time | 30 s | 130 s | 4.3× faster |

Medians: 2,606 output tokens, 8 tool calls and 27 seconds for SheetXL; 7,262, 23 and 97 for GRID. The full run took 76 minutes at three tasks at a time and about $140 at list price.

## The twelve misses

Five are ours:

| Task | First difference | What happened |
| --- | --- | --- |
| 469-9 | `H2`: expected 4543.19, got empty | The sheet does not label debits and credits. We took negative amounts as debits; the expected answer takes positive amounts |
| 524-31 | `E7`: expected empty, got `#N/A` | Seven of 53 rows match no category, and we left the lookup's `#N/A` in them |
| 56786 | `C4`: expected 6.1667, got empty | Our rolling average covers the entries before each row; the expected answer includes the row itself |
| 59129 | `E2`: expected 20, got 21 | One employee's last day is January 1 and we counted them in January. Eleven of twelve months match. The task passed in earlier runs |
| 423-16 | no output | The engine ran out of memory and the task timed out at 20 minutes. A SheetXL bug |

In the other seven, the task and its expected answer point in different directions:

| Task | First difference | The task says | The expected answer |
| --- | --- | --- | --- |
| 44017 | `AD14`: expected 55198.08, got 39540.40 | Apply the increases "to the base rate in column W" | Uses a different column for each month, Q through AB |
| 45738 | `L13`: expected 0, got 1160 | The input already holds 1160 in `L13` | Computes 0 |
| 118-50 | `C2`: expected `TAVERING`, got `TABORING` | `answer_position` starts at C2 | Starts at C1 |
| 13284 | `E5`: expected `Sonia`, got empty | "Only consider rows where both the Start (column C) and End (column D) values are populated" | Matches rows where End is blank |
| 22-47 | `G2`: expected `HASSONA`, got `HASSAN` | Order the names by a helper list, keep their original order within it, and "sort only column H" | Whole rows sorted by column H; the helper list is not used |
| 534-26 | `I1`: expected empty, got 2020-12-01 | Its own `Result` sheet shows December 2020 in `I1` | `I1` is empty |
| 3002 | `G7`: expected `Single`, got `Multiple` | Its own example labels row 7 "Multiple" | "Single" |

GRID's [task notes](https://github.com/GRID-is/spreadsheetbench-results) record 0 passes in 87 attempts on 44017, 0 in 86 on 45738 and 0 in 36 on 118-50, across every model it ran.

## Every full run

This was the ninth full run of the 400 with this harness. The prompt and the skill changed between runs. All nine are listed with their scores, builds and prompt hashes in [verified_400/README.md](verified_400/README.md).
