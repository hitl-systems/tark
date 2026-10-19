# Interface

tark takes one request and a list of tool definitions and returns zero or more calls. It is one forward pass: nothing is decoded token by token.

## Input

One JSON object per request:

| field | meaning |
|---|---|
| `id` | any string; echoed back |
| `ctx` | optional free text shown before the request (a date, earlier turns); may be empty |
| `req` | the user's request |
| `tools` | the tools on offer: each has `name`, `description` and `parameters` (an object with typed `properties`, optional `enum` lists and `required`) |

The model was built for a handful of tools per request (up to 8) and for tools with a modest number of arguments (up to 10 each). Larger sets are cut down.

## Output

| field | meaning |
|---|---|
| `calls` | a list of `{name, args}`; empty when the model decides not to call anything |
| `logc` | the log of the model's confidence in the call (closer to 0 is more confident) |
| `fire` | whether the call should be acted on: true when there is a call and `logc` is above the threshold stored in the model file |

Arguments are filled in one of four ways: copied from the request, picked from an `enum` or other list, computed as a date relative to `ctx`, or left out. The model never writes free text, so a value that is not in the request (a rewritten number, a structured object) is usually missing or wrong.

## Shipped and forced call

Two settings appear throughout the model card:

- **Shipped**: the model's own decision. It may return no call.
- **Forced call**: the model always returns its single best call, whatever its confidence. This shows how well it fills in a call when the call is wanted, and is the better measure on tasks where a call is always expected.

The thresholds that decide `fire` are fitted for a particular family of tools and are stored inside the model file. They are not a guarantee of any error rate, and they are not calibrated for tool sets the model has not seen (see [limitations.md](limitations.md)).
