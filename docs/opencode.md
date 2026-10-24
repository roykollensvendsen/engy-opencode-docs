# Using engy with OpenCode

engy is in OpenCode's built-in provider list (its catalogue comes from models.dev), so no base URL is needed. Provider id: `engy`. Key variable: `ENGY_API_KEY`. Model ids: `engy/<id>`, with `<id>` from `GET https://api.engy.ai/v1/models`.

Check the version first with `opencode --version`. OpenCode 1.x and 2.x differ in config shape and in how a variant is picked.

## Connect

Both versions: run `opencode`, then `/connect`, pick engy, paste the key; then `/models` to pick a model.

- OpenCode 1.x: `/connect` stores the key in `~/.local/share/opencode/auth.json`. Alternatively `export ENGY_API_KEY=…` before starting `opencode`.
- OpenCode 2.x: models run on a background server, so an environment variable must be set there: `opencode service set env ENGY_API_KEY …`.

## Default model

OpenCode 1.x, `~/.config/opencode/opencode.json` or `opencode.json` in a project:

```json
{ "$schema": "https://opencode.ai/config.json", "model": "engy/glm-5.2" }
```

OpenCode 2.x: the same, in `opencode.jsonc`.

## Reasoning effort (variants)

Variants come from the catalogue and are the same in both versions:

| Model | Variants |
|---|---|
| `glm-5.2` | `high`, `max` |
| `glm-5.3`, `glm-5.3-flash` | `low`, `high`, `max` |
| `deepseek-v4-flash-0731`, `deepseek-v4.1-flash`, `deepseek-v4.1-flash-1017` | `low`, `high`, `max` |
| `qwen3.8-27b` | `low`, `medium`, `xhigh` |
| `kimi-k3`, `qwen3.6-35b-a3b` | none (thinking is on/off only) |

- OpenCode 1.x: `opencode run --model engy/glm-5.2 --variant max "…"`, or `ctrl+t` in the TUI. An unknown variant name is ignored and the model runs at its default.
- OpenCode 2.x: `opencode run --model engy/glm-5.2#max "…"`. An unknown variant name is an error.

## Adding a variant

The value is sent as `reasoning_effort`. Example: a variant that turns thinking off (works on every model except `glm-5.3` and `glm-5.3-flash`).

OpenCode 1.x:

```json
{
  "provider": {
    "engy": {
      "models": {
        "qwen3.8-27b": { "variants": { "off": { "reasoningEffort": "none" } } }
      }
    }
  }
}
```

OpenCode 2.x:

```jsonc
{
  "providers": {
    "engy": {
      "models": {
        "qwen3.8-27b": { "variants": [{ "id": "off", "settings": { "reasoningEffort": "none" } }] }
      }
    }
  }
}
```

Use it as `--variant off` (1.x) or `engy/qwen3.8-27b#off` (2.x).
