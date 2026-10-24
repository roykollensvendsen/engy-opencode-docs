# How the OpenCode section was checked

Every statement in the section, where it comes from, and whether it was run. Checked 2026-10-24 unless a date says otherwise.

## Versions

| | Version | Source |
|---|---|---|
| OpenCode 1 | 1.19.4 | `npm view opencode-ai version` and the GitHub release marked Latest (2026-10-22). This is what `npm install opencode-ai` and the installer give today. |
| OpenCode 2 | 2.0.29 | A pre-release build, installed locally. Its docs live on `anomalyco/opencode` `dev` under `services/www/src/docs/content/`. |

The section defaults to OpenCode 1 because that is what most users run. The OpenCode 2 tab covers what changes.

## Catalogue facts

From engy's entry on models.dev (`providers/engy/` on `anomalyco/models.dev` `dev`, and `https://models.dev/api.json`):

- provider id `engy`, key variable `ENGY_API_KEY`, endpoint `https://api.engy.ai/v1`
- nine models: `deepseek-v4-flash-0731`, `deepseek-v4.1-flash`, `deepseek-v4.1-flash-1017`, `glm-5.2`, `glm-5.3`, `glm-5.3-flash`, `kimi-k3`, `qwen3.6-35b-a3b`, `qwen3.8-27b`
- reasoning effort lists: GLM-5.2 `high`, `max`; GLM-5.3, GLM-5.3-Flash and the DeepSeek models `low`, `high`, `max`; Qwen3.8 `low`, `medium`, `xhigh`; Kimi-K3 and Qwen3.6 have an on/off toggle and no effort list

That GLM-5.3 cannot turn thinking off is from engy's own docs ("Turning thinking off").

## OpenCode behaviour, from its docs and source

| Statement | OpenCode 1.19.4 | OpenCode 2 |
|---|---|---|
| `/connect` then `/models` | `packages/web/src/content/docs/providers.mdx` at tag `v1.19.4` | `services/www/src/docs/content/providers.mdx` |
| key saved in `~/.local/share/opencode/auth.json` | same v1 page, "Credentials" | not stated for OpenCode 2, so the tab leaves it out |
| environment variable | read by the OpenCode process (v1 runs in-process) | must reach the background server: `opencode service set env` (`cli/providers.mdx`, "Environment") |
| default model `"model": "engy/<id>"` | `models.mdx`, "Loading models" | `models.mdx` |
| config shape | `provider.<id>.models.<id>.variants` as an object | `providers.<id>.models.<id>.variants` as a list with `id` |
| picking a variant | `--variant` flag, `ctrl+t` (`variant_cycle`) | `model#variant` |
| variants for engy | built from the catalogue's effort lists (seen in the live listing below) | built from the catalogue (`packages/core/src/variant.ts`): effort list only; a toggle gives no variant on `@opencode/ai/providers/openai-compatible` |

## Live tests

Run with engy's API key against the published catalogue, each OpenCode build installed in a throwaway folder with its own config and data directories.

**OpenCode 1.19.4**

- `opencode models engy` lists all nine models. `--verbose` shows the variants above, exactly.
- `opencode run --model <id>` answered for all nine models.
- `--variant max` on `engy/glm-5.2` answered.
- `--variant high` on `engy/qwen3.8-27b` (not in its list) ran at the default without an error, so the section says unknown names are ignored.
- `engy/glm-5.2#max` failed: the `#` form is OpenCode 2 only.
- `"model": "engy/glm-5.2"` in a project `opencode.json` was used when no `--model` was given.
- The custom variant `"off": { "reasoningEffort": "none" }` on `engy/qwen3.8-27b`: 0 reasoning tokens, against 33 for `xhigh` on the same prompt (`--format json`, step-finish token counts).

**OpenCode 2.0.29**

- `opencode models` lists all nine engy models.
- `engy/glm-5.2#max` answered.
- `engy/qwen3.8-27b#high` was refused with "Variant unavailable", so the tab says an unknown name is an error.
- `"model": "engy/glm-5.2"` in a project `opencode.jsonc` was used when no `--model` was given.
- The custom variant `{ "id": "off", "settings": { "reasoningEffort": "none" } }` on `engy/qwen3.8-27b`: 0 reasoning tokens.

## Agent test

The files for agents were tested by giving a fresh agent (Claude Sonnet, no other context) only `llms.txt` and `docs/opencode.md` and asking it for an OpenCode config with a default model and an `off` variant, plus the commands to run GLM-5.2 at its highest level and Qwen3.8 with `off`, for both versions.

- Round 1: its configs and commands were correct, and the OpenCode 1 ones were run on 1.19.4 as written: the default model was used, `--variant max` answered, `off` gave 0 reasoning tokens, and the added variant showed up next to the built-in ones. It listed what it had to guess: which variant is highest, whether added variants replace the built-in ones, where OpenCode 2 keeps its config, what `opencode run` does, and quoting `#` in the shell. Each was answered in `docs/opencode.md` and the HTML section.
- Round 2, a new agent on the fixed files: the same correct output, without those guesses. Its remaining notes were whether the default model and variants can share a file, whether OpenCode 2 also reads `opencode.json`, and why Qwen3.8 stops at `xhigh`; those are now stated too.

- Control: a fresh agent with only `https://engy.ai/docs` (no OpenCode section today) guessed every OpenCode detail. Its OpenCode 1 config redefined engy as a custom provider and worked on 1.19.4; on 2.0.29 its `--variant` commands failed with `Unrecognized flag: --variant`. See `LLM-DOCS.md`, "Control test".

## Format and layout

- `llms.txt` parses with the reference parser from llmstxt.org (`pip install llms-txt`, version 0.0.7): title `engy`, the summary, details, and the sections Clients (6 links), API features (6) and Optional (4).
- `index.html` was rendered with headless Chromium at 1280 px in light and dark mode and at 390 px. At 390 px long inline code ran off the screen; it now wraps, and only code blocks scroll sideways.

## Not tested

- `/connect` itself, which is interactive. Both runs used `ENGY_API_KEY` instead.
- Windows paths. The paths shown are the Linux and macOS defaults from OpenCode's docs.

## Cost

About 35 requests to engy in all, a fraction of a cent.
