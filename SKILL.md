---
name: config-guard
description: "Prevent OpenClaw config changes from crashing the gateway. Auto-backup, schema validation, critical field checks, and auto-rollback. Use before any config.apply, config.patch, or openclaw.json edit."
metadata:
  openclaw:
    emoji: "🛡️"
---

# Config Guard

Validates and protects `openclaw.json` edits. Prevents gateway crashes from bad config by enforcing backup → validate → apply → verify on every change.

**Common failure modes without this skill:**
- Model names with dots instead of hyphens (`claude-sonnet-4.5` vs `claude-sonnet-4-5`)
- Unknown top-level keys (`auth`, `fallbacks`) crashing the gateway
- Config changes silently wiping channel or auth sections
- Missing required fields (`browser.profiles.color` must be hex)

## What Config Guard Does

### Before config changes:
1. **Auto-backup** — `cp openclaw.json` to timestamped backup
2. **JSON syntax check** — catches malformed JSON before it hits the gateway
3. **Schema validation** — checks required fields, correct types, valid values
4. **Semantic checks** — catches AI-specific mistakes (model name format, unknown keys, placeholder values, missing required fields)
5. **Critical field diff** — warns if Telegram channel, auth tokens, or tool deny lists changed

### After config changes (with `--restart`):
6. **Gateway health check** — polls gateway for up to 30s
7. **Auto-rollback** — if gateway doesn't recover, restores backup automatically

See [CHECKS.md](CHECKS.md) for the full validation checklist.

## Usage

### As a script (recommended):
```bash
# Validate current config
bash scripts/config-guard.sh check

# Validate → backup → apply → verify
bash scripts/config-guard.sh apply --restart

# Show what changed vs last backup
bash scripts/config-guard.sh diff

# Emergency rollback
bash scripts/config-guard.sh rollback
```

### As a git hook:
```bash
cp scripts/pre-config-hook.sh /path/to/repo/.git/hooks/pre-commit
```

## ⛔ CRITICAL: DO NOT USE `gateway config.patch` DIRECTLY

`config.patch` **bypasses all validation**. One wrong model name and the gateway crashes — you go offline with no remote fix.

**Use `safe-config-patch.sh` instead:**

```bash
# ✅ CORRECT: Validates before applying
bash scripts/safe-config-patch.sh '{"agents":{"defaults":{"model":"anthropic/claude-opus-4-5"}}}'

# ❌ WRONG: No validation, will crash on bad input
gateway config.patch raw='{"agents":{"defaults":{"model":"claude-opus-4.5"}}}'
```

## Mandatory Workflow

1. **Use the safe wrapper** (validates → backs up → applies → verifies):
   ```bash
   bash /path/to/config-guard/scripts/safe-config-patch.sh '<json-patch>'
   ```

2. **Never guess config field names or values.** If unsure:
   - Use `gateway config.schema` to check valid fields
   - Use `gateway config.get` to see current values
   - Change ONE field at a time

3. **For complex changes**, validate first:
   ```bash
   bash /path/to/config-guard/scripts/config-guard.sh check ~/.openclaw/openclaw.json
   ```

4. **If gateway dies**, rollback:
   ```bash
   bash /path/to/config-guard/scripts/config-guard.sh rollback
   ```

## Install

```bash
clawdhub install config-guard
# or copy the scripts/ directory to your workspace
```

## Requirements

- `python3` (for JSON parsing and validation)
- `curl` (for gateway health checks)
- `bash` 4+
