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

## Not tested

- `/connect` itself, which is interactive. Both runs used `ENGY_API_KEY` instead.
- The OpenCode 2 custom variant example. It follows OpenCode 2's docs (`models.mdx`, "Variants"), but has not been run.
- Windows paths. The paths shown are the Linux and macOS defaults from OpenCode's docs.

## Cost

About 20 requests to engy in all, a fraction of a cent.
