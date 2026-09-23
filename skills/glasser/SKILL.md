---
name: glasser
description: >
  Use this skill when a task needs external or paid data the native tools do not cover — person or
  company enrichment, people or company search, web, news, places, scholar or shopping search,
  SEO metrics, social media posts and profiles, US property and rental data, stock quotes — and
  whenever the user says "set up Glasser", "connect Glasser", asks what Glasser costs, or a
  Glasser tool returns an error. Covers first sign-in, the seven MCP tools, and the spending rules.
metadata:
  version: "0.1.0"
---

# Glasser — setup and usage

This plugin connects one remote MCP server, `https://api.glasser.ai/mcp`, through `mcp-remote`
(a local stdio bridge that handles the browser sign-in). Glasser is a broker: one key covers
1,000+ paid endpoints from providers such as Apollo, People Data Labs, Hunter, Ahrefs, Semrush,
DataForSEO, Serper, Exa, ScrapeCreators and RentCast. Each call is charged per the endpoint's
published price and the response is the provider's own output.

The server exposes seven tools, named `mcp__glasser__<tool>` in Hermes:

| Tool        | Purpose                                                                              |
| ----------- | ------------------------------------------------------------------------------------ |
| `search`    | Search the data sources: runnable paid endpoints, filtered to what the key may run   |
| `inspect`   | One endpoint's input schema, exact price with every charge clause, version, run mode |
| `run`       | Execute an endpoint and charge per call; requires an `idempotency_key` you generate  |
| `runs_get`  | One run by id: status, output, provider response, charge                             |
| `runs_list` | The workspace's runs, newest first                                                   |
| `runs_stop` | Stop an in-flight run; before dispatch it becomes STOPPED and free                   |
| `balance`   | Balance, held and available; doubles as the auth probe                               |

## Setup

Node.js 18 or newer must be on the PATH: the bridge runs with `npx`.

1. Call `balance`. If it returns a balance, setup is complete.
2. If the bridge has no token yet, it opens the browser at glasser.ai to sign in (or prints the
   URL when it cannot open a browser). Show the user that URL exactly as printed and wait for
   them to approve. The token is stored on this machine by `mcp-remote`; the key it resolves to
   appears at https://app.glasser.ai/keys and can be revoked there.
3. Call `balance` again. Report the available balance. Do not report key values, workspace names
   or install paths.

Never ask the user to paste a key. If sign-in fails, report the error the bridge printed; a
network failure does not mean the key is invalid. Do not offer paid task prompts while setup is
blocked.

## When to use

- The task needs a capability no native tool or existing integration provides. The user's own
  keys and integrations outrank Glasser: use it to fill gaps, never to route around what they
  already have.
- Do not use it for local files, code, or anything the native web tools already answer.

## Working agreement

1. **Search first, then inspect.** `search` with a plain description of the data you need. Pick
   an endpoint and `inspect` it before the first `run`: the schema says what it takes, and the
   price says what one call costs, per clause.
2. **Name the price before spending.** Say the per-call price and get the user's go-ahead before
   any run that is speculative, looped, or bulk. Volume parameters (limits, page sizes) start
   small.
3. **Generate an `idempotency_key` (a UUID) for every `run`.** Retrying with the SAME key returns
   the original run instead of a second charge; the result carries `replayed: true`. A new key
   is a new charge.
4. **Read status and charge separately.** Every run reports `charge_usd` as an exact decimal
   string. A provider 404 is a COMPLETED run charged per the endpoint's published clauses.
5. **A run that comes back QUEUED or RUNNING** is polled with `runs_get` until it is COMPLETED,
   FAILED or STOPPED. `runs_stop` cancels one that has not been dispatched yet.
6. **Refusals are tool results, not protocol errors.** `insufficient_balance`, `rate_limited`
   with `retry_after_ms`, and policy refusals come back with `isError`; read the message and
   act on it (top up, wait, or pick another endpoint). Do not retry blindly.

## Reporting

Report what the data says, which provider served it, and the total charge for the task. Link
the run URL when the user may want to inspect the raw output.

## Troubleshooting

- **`npx` not found** — install Node.js 18+ and start a new session.
- **Sign-in page never opens** — run the bridge once by hand,
  `npx -y mcp-remote@0.14.3 https://api.glasser.ai/mcp`, open the printed URL, then start a new
  Hermes session.
- **`unauthorized` after a working session** — the token was revoked at app.glasser.ai/keys.
  Delete the cached token under `~/.mcp-auth/` and sign in again.
- **Tools missing from `mcp__glasser__*`** — the package is installed but not enabled:
  `hermes plugins enable glasser`, then start a new session.
