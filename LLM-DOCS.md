# Docs for agents: llms.txt and markdown pages

This PR also proposes two files for coding agents and LLM tools, next to the page for people:

- [`llms.txt`](llms.txt), meant for `https://engy.ai/llms.txt`: a short index of engy's docs.
- [`docs/opencode.md`](docs/opencode.md), meant for `https://engy.ai/docs/opencode.md`: the OpenCode section as plain markdown.

All facts below were checked on 2026-10-24.

## Why

**Agents read docs to set clients up.** A request like "set up OpenCode to use engy" makes a coding agent fetch engy's docs. Today it finds:

| URL | Status |
|---|---|
| `https://engy.ai/llms.txt` | 404 |
| `https://engy.ai/llms-full.txt` | 404 |
| `https://engy.ai/docs.md` | 404 |
| `https://engy.ai/api/v1/llms.txt` | 200, but it covers only the read-only account API (balance, usage, prices), not how to connect a client |
| `https://engy.ai/docs` | 200, HTML |

So the only source for client setup is the HTML page. It is 104,669 bytes, of which about 25,000 characters are visible text; the rest is markup, scripts and styles an agent has to strip, and every client's section comes along when only one is needed.

**llms.txt is the place agents look.** [llmstxt.org](https://llmstxt.org) (Jeremy Howard, first published September 2024, v2 August 2026) proposes a markdown file at `/llms.txt` with an H1, a one-paragraph summary in a blockquote, and H2 sections listing links to markdown pages, so an agent can read the index and fetch only the page it needs. It also proposes a markdown copy of each page at the same URL with `.md`, linked from the HTML with `rel="alternate" type="text/markdown"`, and `rel="describedby"` pointing to the llms.txt. Anthropic (`docs.anthropic.com/llms.txt`), OpenAI (`platform.openai.com/docs/llms.txt`) and OpenCode (`opencode.ai/llms.txt`) publish one, and so does Decision Models (`decisionmodels.io/llms.txt`), another inference host on models.dev. The proposal says Chrome's Lighthouse checks for one in its agentic-browsing audits; we have not checked that ourselves.

**The two files are small.** `llms.txt` is 2,041 bytes and `docs/opencode.md` 2,351 bytes, against 104,669 for the HTML page.

## What the files do differently from the HTML page

- **No tabs.** An agent cannot click the OpenCode 1 / OpenCode 2 switch. `docs/opencode.md` states both versions side by side and says to run `opencode --version` first, because the config shape and the way a variant is picked differ.
- **Exact strings in code.** Provider id `engy`, variable `ENGY_API_KEY`, model ids `engy/<id>`, file paths and commands, so an agent can copy them.
- **A table instead of prose** for the variants each model has, which is what an agent has to get right.
- **Points to the live model list** (`GET https://api.engy.ai/v1/models`, no key needed, nine models today) instead of only naming models, since ids change.
- **Facts an agent would otherwise get wrong:** the Anthropic base URL has no `/v1`; `none` and `minimal` turn thinking off except on GLM-5.3 and GLM-5.3-Flash; some clients need the `engy/` prefix.

## Proposed vs existing URLs

`llms.txt` links to a markdown page per section of engy.ai/docs (`/docs/opencode.md`, `/docs/claude-code.md`, …). Those pages do not exist yet; only `docs/opencode.md` is drafted here. The links under "Optional" exist today: the account API's `llms.txt` and `openapi.json`, `/pricing` and `/status`.

If engy prefers one file over many, the same content can go in a single `https://engy.ai/llms-full.txt` (as Decision Models does), with `llms.txt` linking to it.

## How the content was checked

The OpenCode facts in `docs/opencode.md` are the same as in the HTML section and are covered by [`VERIFICATION.md`](VERIFICATION.md). The other facts in `llms.txt` come from engy's own pages: the endpoints and the `sk-engy-` / `rk-engy-` key prefixes from engy.ai/docs and engy.ai/docs/agent-api, the effort tiers and the GLM-5.3 exception from "Reasoning effort", and the one-line description from the `<meta name="description">` of engy.ai/docs.
