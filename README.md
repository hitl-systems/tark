# tark

A very small model that turns a request and a set of tool definitions into tool calls in one pass. It does not generate text: an encoder reads the request together with the tool schemas, picks which tools to call and how many times, and copies each argument from the request (or picks it from a list, or computes a date). Because it decodes nothing token by token, a call costs a single forward pass.

This repository holds the model file and its card. It contains no source code and no runtime.

## The file

| | |
|---|---|
| `tark-v2.tarkpk` | 3,235,842 bytes (3.24 MB), format TARKPK04 |
| sha256 | `d8ab143660351a19f87cc1c193eccdadd69de5c5de9d898285d7491c9ef2c069` |
| parameters | about 7.8 million |
| peak memory | about 7.6 MB on a CPU runtime (measured on a 150-request sample) |
| latency | tens of milliseconds to a fraction of a second per request on one CPU thread, depending on schema size |

The file is self-contained: weights, tokenizer, value lexicon, and the confidence thresholds that decide whether a call should be acted on.

## What it is for

Short, single-turn requests against a fixed set of typed tools: device actions, app intents, slot filling for a known set of intents. It is trained for that and does it well. It is not a general function-calling model.

## Results on public datasets

Every number below comes from my own conversion of the dataset into one request, a list of tool schemas and gold calls, scored by exact or slot-level match. These are not official leaderboard entries, and the protocols differ from the official ones. "Forced call" means the model always emits its single best call; "shipped" means it may decline to call.

**Datasets whose training split was used for training** (so these are in-domain):

| dataset | metric | shipped | forced call |
|---|---|---|---|
| SNIPS, gold tool offered | strict field F1 | 0.435 | 0.615 |
| SNIPS, all 7 tools offered | strict field F1 | 0.512 | 0.609 |
| Schema-Guided Dialogue (test, 1,813 turns) | slot F1 | 0.192 | 0.559 |

**Datasets not used in training** (the model has not seen these schemas):

| dataset | n | metric | shipped | forced call |
|---|---|---|---|---|
| BFCL v4 single-turn, official AST checker | 2,338 calls | exact | 0.100 | n/a |
| MASSIVE en-US, gold tool offered | 2,974 | strict field F1 | 0.085 | 0.127 |
| MASSIVE en-US, 8 tools offered | 2,974 | strict field F1 | 0.083 | 0.084 |
| NexusRaven API evaluation | 318 | exact call | 0.097 | 0.129 |
| API-Bank level 1 / level 2 | 399 / 135 | exact call | 0.023 / 0.015 | 0.040 / 0.030 |
| MultiWOZ 2.2 test, turn-level slots | 4,953 | slot F1 | 0.113 | 0.127 |
| When2Call test (call or not) | 3,652 | decision accuracy | 0.654 | n/a |

## Limits

- **It does not generalise to schemas it was not trained on.** The second table says so plainly. On BFCL it makes no call at all for about a third of the requests that need one and gets the call count wrong for another fifth.
- **Arguments must be in the request.** About 70% of argument values in typical function-calling data appear word for word; the rest (rewritten numbers, structured values) are mostly missed. NexusRaven CVE/CPE scores 0 of 78.
- **No dialogue state.** It reads a request and, optionally, some context text. It was not trained to track a conversation, and API-Bank and MultiWOZ scores reflect that.
- **The call/no-call confidence is not a guarantee.** The thresholds in the file were fitted to hold wrong actions to about 1% on held-out in-domain data. On a separate held-out test they came in near or slightly above 1% for some tool families. On unseen schemas the confidence is not calibrated at all: on When2Call it is no better than always declining.
- **It cannot ask a follow-up question or explain a refusal.** It either calls or does not.

## Training data and licences

Trained on a mix of: the xLAM function-calling 60k set (CC BY 4.0), ToolACE (Apache-2.0), DroidCall (Apache-2.0), the Mobile Actions set (CC BY), SNIPS (CC0), TinyAgent (MIT), and the Schema-Guided Dialogue set (**CC BY-SA 4.0**). It was distilled from a larger encoder model trained on part of the same data. NexusRaven (CC BY-NC 4.0) was used for evaluation only and is not in the training data.

**The licence for the model file has not been chosen.** Whether the share-alike condition of the Schema-Guided Dialogue data extends to the trained weights is unresolved; do not redistribute under a different licence until that is settled.

## Status

Version 2. The numbers above were produced on the file in this repository.
