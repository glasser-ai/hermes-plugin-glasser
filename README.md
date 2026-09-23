# Glasser for Hermes Agent

Portable [Agent Plugins v1](https://agent-plugins.org) package that connects
[Hermes Agent](https://github.com/NousResearch/hermes-agent) to
[Glasser](https://glasser.ai)'s hosted MCP server, with a bundled skill that teaches the agent
first sign-in, the seven tools, and the rules that keep spending safe.

Glasser is a data broker: one key covers 1,000+ paid endpoints across providers such as Apollo,
People Data Labs, Hunter, Ahrefs, Semrush, DataForSEO, Serper, Exa, ScrapeCreators and RentCast.
The agent searches the data sources, inspects an endpoint's schema and exact price, runs it, and
pays per call. No signup at each vendor.

## What it adds

| Tool        | Purpose                                                                              |
| ----------- | ------------------------------------------------------------------------------------ |
| `search`    | Search the data sources: runnable paid endpoints, filtered to what the key may run   |
| `inspect`   | One endpoint's input schema, exact price with every charge clause, version, run mode |
| `run`       | Execute an endpoint and charge per call; requires an idempotency key                 |
| `runs_get`  | One run by id: status, output, provider response, charge                             |
| `runs_list` | The workspace's runs, newest first                                                   |
| `runs_stop` | Stop an in-flight run; before dispatch it becomes STOPPED and free                   |
| `balance`   | Balance, held and available; doubles as the auth probe                               |

Plus the `glasser` skill (`skills/glasser/SKILL.md`), loaded on demand through `skills_list` /
`skill_view`.

## Install

Requires Hermes 0.20 or newer and Node.js 18+ on the PATH (the connection runs through
[`mcp-remote`](https://www.npmjs.com/package/mcp-remote), pinned to 0.14.3, which performs the
browser sign-in for clients that speak stdio only).

From the plugin catalog, once listed:

```bash
hermes plugins install glasser
hermes plugins enable glasser
```

From this repository:

```bash
hermes plugins install glasser-ai/hermes-plugin-glasser
hermes plugins enable glasser
```

Start a new session. On the first call the bridge opens glasser.ai in your browser to sign in;
approve it and the token is stored on your machine. The key it resolves to is listed at
[app.glasser.ai/keys](https://app.glasser.ai/keys) and can be revoked there. No key is stored in
this package.

Ask Hermes to use the Glasser `balance` tool to confirm the connection, then try:

- "Using Glasser, find the CTO and VP Engineering at stripe.com with work emails."
- "Do a competitor research report on notion.so with Glasser: firmographics, news, traffic."
- "Research keywords for 'ai agent workflow' in the US market with Glasser."

## How it charges

Every `run` is charged per the endpoint's published price, shown by `inspect` before the call,
and reported back as an exact `charge_usd`. The skill makes the agent name the price and ask
before speculative, looped or bulk runs, and generate an idempotency key for every run so a
retry never pays twice.

## Alternatives

Already have `mcp_servers.glasser` in your `config.yaml`? Use one or the other, not both, or the
tools appear twice. If your Hermes has a shell, the
[Glasser CLI and skill](https://glasser.ai/docs/setup-skill) is the recommended path; use this
package when the runtime cannot shell out or you prefer a one-command install.

## Manage

```bash
hermes plugins disable glasser
hermes plugins remove glasser
hermes plugins update glasser   # catalog updates use reviewed commit pins
```

## Links

- [Glasser MCP server docs](https://glasser.ai/docs/mcp-server)
- [Glasser docs](https://glasser.ai/docs)
- [Hermes plugin catalog](https://hermes-agent.nousresearch.com/docs/plugins)

## License

MIT
