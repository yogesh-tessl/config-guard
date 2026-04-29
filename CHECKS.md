# Config Guard — Validation Checks

Full list of checks performed by `config-guard.sh check` and `safe-config-patch.sh`:

| Check | What it catches |
|---|---|
| JSON syntax | Malformed JSON, trailing commas, unquoted keys |
| Unknown top-level keys | AI inventing fields like `fallbacks`, `auth` at wrong level |
| Model name format | `claude-sonnet-4.5` → should be `claude-sonnet-4-5` |
| browser.profiles.color | Missing required field, non-hex values |
| commands.native | Missing required fields |
| Placeholder values | `your-token-here`, `sk-xxx` in sensitive fields |
| Empty primary model | No model configured = agent can't think |
| Telegram channel wipe | Silent removal of channel config |
| Auth token removal | Gateway auth accidentally deleted |
| Tool deny list changes | Security-critical tools removed from deny |
