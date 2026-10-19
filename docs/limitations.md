# Limitations

## It does not generalise well to new tool schemas

On BFCL v4 single-turn the model gets 0.100 of the calls that need a call exactly right. Where it fails (shipped setting):

| outcome on those calls | share |
|---|---|
| makes no call at all | about 35% |
| makes the wrong number of calls | about 19% |
| picks the wrong tool | about 1% |
| right tool, a required argument missing | about 11% |
| right tool, a wrong argument value | about 23% |

The "no call" share rises with the number of tools offered (about 31% with one tool, 52% with five to nine). NexusRaven (0.097 exact) and API-Bank (0.023 and 0.015) tell the same story, and on NexusRaven CVE/CPE it gets 0 of 78.

## Arguments must come from the request

Across the BFCL calls, about 71% of argument values appear word for word in the request, and only about 46% of requests have every value in it. The model copies, picks from a list or computes a date; it does not rewrite numbers, build structured objects or derive values. Anything outside that is missed.

## It has no dialogue state

It reads one request and, optionally, some context text. It does not track a conversation across turns. API-Bank and MultiWOZ scores show this: on API-Bank level 2 the model names the right API in 93% of forced-call rows but gets the arguments right in only 3%.

## Its confidence is a hint, not a guarantee

The threshold stored in the file decides when a call is acted on. It was fitted on in-domain data to keep wrong actions near 1%, and on a separate held-out set some tool families came in slightly above that. On tool sets it has not seen the confidence is not calibrated: on When2Call it is no better than never calling a tool (0.654 decision accuracy against 0.645).

## It cannot ask or explain

It either calls or does not. It cannot ask a follow-up question, say why it declined, or refuse in words.

## Size limits of a request

It is built for up to 8 tools per request and 10 arguments per tool; larger inputs are cut down.
