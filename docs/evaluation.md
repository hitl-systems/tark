# How the results were produced

Every number in the model card comes from a conversion of a public dataset into the model's input format: one request, a list of tools and gold calls. The conversions are mine, not the dataset authors', so none of these is an official leaderboard entry. Rows with a gold call that the model could not possibly produce are still counted.

"Shipped" and "forced call" are defined in [interface.md](interface.md).

## Slot-filling datasets

Scored as micro-averaged F1 over tuples. In the strict variant a tuple is (tool name, slot, normalised value), so choosing the wrong tool loses all of that row's slots; in the slot variant the tool name is ignored. Values are lower-cased, whitespace-collapsed, stripped of surrounding quotes and punctuation, and numbers are canonicalised.

| dataset | rows | conversion | metric |
|---|---|---|---|
| SNIPS | 700 test queries | each of the 7 intents is a tool; slots are string arguments. Two settings: only the gold tool offered, or all 7 offered | strict F1 |
| Schema-Guided Dialogue | 1,813 test turns | one row per user turn and active frame; the tools are the intents of that service; no dialogue history is given | slot F1 (it is not joint goal accuracy) |
| MASSIVE, en-US | 2,974 test utterances | each of the 60 intents is a tool; slots are string arguments; when a slot repeats in one utterance only the first occurrence is kept; two settings: gold tool only, or the gold tool plus 7 randomly chosen others (fixed seed) | strict F1 |
| MultiWOZ 2.2 | 4,953 test user turns | one row per user turn and active service; gold is the slot values updated in that turn (not the accumulated dialogue state); the previous 4 turns are given as context; values marked "dontcare" are dropped | slot F1 (it is not joint goal accuracy) |

## Function-calling datasets

Scored by exact call match: the tool name and every argument must match after the same value standardisation BFCL uses.

| dataset | rows | conversion |
|---|---|---|
| BFCL v4, single-turn | 2,338 calls that need a call | rows joined to the official BFCL files and scored with the official AST checker (bfcl-eval 2026.3.23); the pooled figure in the card adds the irrelevance rows, where abstaining is correct |
| NexusRaven API evaluation | 318 queries in 4 sub-benchmarks | the sub-benchmark's APIs are the tools, capped at 8 per query (the gold tool is always kept), so the larger tool sets are easier than in the original setup |
| API-Bank, level 1 and level 2 | 399 and 135 | the dialogue so far is given as context; level 2 offers the retrieval tool, the APIs returned earlier in the dialogue and the gold API; 8 level-1 rows have a gold argument the offered schema does not contain |

## When2Call

3,652 test questions with the tools given in each item. The correct behaviour is one of: call a tool (1,295), ask a follow-up question (1,062), or say the question cannot be answered with these tools (1,295). tark can only call or not call, so the two non-call behaviours are both scored as "no call". The metric is the call-or-not decision: accuracy, and precision, recall and F1 for "call". Baselines: never calling scores 0.645 accuracy; always calling scores 0.524 F1 for "call".

## Confidence intervals

Where the card or the scorer gives an interval it is a 95% bootstrap over rows (1,000 resamples, fixed seed).
