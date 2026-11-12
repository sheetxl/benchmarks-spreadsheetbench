# SheetXL — SpreadsheetBench Verified-400

Run `20261031-213401-v1-400-sub5c-2`, 2026-11-01. Model `claude-opus-5-5` through headless Claude Code (`claude -p`); SheetXL (engine and xlsx IO) through its MCP server and the `sheetxl` skill. Strategy `single`: one solve turn per task, no verify turn.

- SheetXL build: `d7ea4664560c83da007375aa251e3339d40452a6`
- Harness: `36948678dbf25eccf4cd90b1cacfea5d9b054fbf`

## Score

**388/400 = 97.00%**, graded as the published Verified-400 entries are graded: the shipped evaluator, with the four tasks whose `answer_position` it cannot parse graded on the corrected position (283-32, 130-9, 45944, 49300; upstream issues #33, #36). Those four raise inside the shipped evaluator for every submission. Leni's results file names this mode `official-with-metadata-tolerance`, and grid.is's published score counts the same tasks. Our outputs pass all four.

Under the shipped evaluator alone (`evaluation/evaluation.py` `compare_workbooks`, unmodified, one test case per task): 384/400 = 96.00%, in `eval_single_sheetxl.json`.

## Full runs

Every full Verified-400 run of this harness, in order; the submitted run is the last. Earlier runs used earlier prompts and builds.

| date | run | model | SheetXL build | prompt sha256 (12) | verify turn | passed / graded (of tasks) |
| --- | --- | --- | --- | --- | --- | --- |
| 2026-10-21 | `20261020-222733-v1-400-full` | claude-opus-5-5 | `ba379fe0ed` | `a7a23d8b060e` | yes | 367 / 393 (of 400) |
| 2026-10-22 | `20261022-123727-v1-400-full-2` | claude-opus-5-5 | `7a1038cc52` | `be3fd2544403` | yes | 374 / 399 (of 400) |
| 2026-10-26 | `20261026-191456-v1-400-full-3` | claude-opus-5-5 | `a0b8bc978b` | `72cc67d266a5` | yes | 376 / 400 (of 400) |
| 2026-10-29 | `20261029-185434-v1-400-full-4` | claude-opus-5-5 | `284052b84a` | `49440617bb0f` | yes | 378 / 400 (of 400) |
| 2026-10-30 | `20261029-232517-v1-400-sub-c3-28` | claude-opus-5-5 | `ea1033893c` | `49440617bb0f` | no | 380 / 400 (of 400) |
| 2026-10-30 | `20261030-014123-v1-400-sub2-19` | claude-opus-5-5 | `ea1033893c` | `dfa1461ce70b` | no | 383 / 400 (of 400) |
| 2026-10-30 | `20261030-092634-v1-400-sub3-1` | claude-opus-5-5 | `ea1033893c` | `fa9c54a0d8e1` | no | 384 / 400 (of 400) |
| 2026-10-30 | `20261030-104426-v1-400-sub4-2` | claude-opus-5-5 | `ea1033893c` | `fa9c54a0d8e1` | no | 383 / 400 (of 400) |
| 2026-11-01 | `20261031-213401-v1-400-sub5c-2` | claude-opus-5-5 | `d7ea466456` | `fa9c54a0d8e1` | no | 384 / 399 (of 400) |

## Contents

- `eval_single_sheetxl.json` — the evaluator results.
- `outputs/{id}-output.xlsx` — the graded workbook of each task.
- `logs/{id}/` — the inference log: `prompt.md` (the prompt exactly as sent, with the task fields filled in), `transcript.jsonl` (the model's outputs and every tool call and result, stream-json; the skill text appears where the agent loads it), `sheetxl-log/*.jsonl` (every SheetXL call with its full source, status and duration). The run had no verify turn.
- `harness/` — the prompt template (`prompt.md` includes `rules.md`), which hashes to the run manifest's `promptTemplateSha256`, and `skill/`, the `sheetxl` skill the agent loads. Both are the harness's, not the benchmark's.

## Notes

- The agent sees the task's `instruction`, `instruction_type` and `answer_position`, and the input workbook; never `answer_sheet`, `data_position` or a golden file.
- Outputs carry their computed values; no recalculation pass runs before grading.
- Every task ran once, in this run. Full runs lists every complete run of this harness.
- No output (scored 0): 423-16.
