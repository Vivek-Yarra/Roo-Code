# AGENTS.md

This file provides guidance to agents when working with code in this repository.

- Settings View Pattern: When working on `SettingsView`, inputs must bind to the local `cachedState`, NOT the live `useExtensionState()`. The `cachedState` acts as a buffer for user edits, isolating them from the `ContextProxy` source-of-truth until the user explicitly clicks "Save". Wiring inputs directly to the live state causes race conditions.

## Fork & Upstream Sync Setup

This repo is a **personal fork** of `RooCodeInc/Roo-Code` that restores the **Claude Code provider** which was removed upstream in commit `7f854c0dd7ed25dac68a2310346708b4b64b48d9`.

### Repository Structure

| Remote | URL | Purpose |
|---|---|---|
| `origin` | `https://github.com/Vivek-Yarra/Roo-Code.git` | Personal fork (push/pull) |
| `upstream` | `https://github.com/RooCodeInc/Roo-Code.git` | Original repo (fetch updates) |

Local `main` tracks `origin/main`.

### What Was Changed

The upstream PR removed the Claude Code provider entirely (33 files, 7 deleted). We reverted it with:
```
git revert 7f854c0dd7ed25dac68a2310346708b4b64b48d9 --no-edit
```

**Restored files:**
- `src/api/providers/claude-code.ts` — API handler
- `src/integrations/claude-code/oauth.ts` — OAuth authentication
- `src/integrations/claude-code/streaming-client.ts` — Streaming client
- `packages/types/src/providers/claude-code.ts` — Type definitions & model configs
- `webview-ui/src/components/settings/providers/ClaudeCode.tsx` — Settings UI
- `webview-ui/src/components/settings/providers/ClaudeCodeRateLimitDashboard.tsx` — Rate limit UI

**Modified files** (claude-code references restored in):
- `packages/types/src/provider-settings.ts` — provider names, schema, `ANTHROPIC_STYLE_PROVIDERS`
- `src/api/index.ts` — handler switch case
- `src/core/config/ProviderSettingsManager.ts`
- `src/core/webview/ClineProvider.ts`
- `src/core/webview/webviewMessageHandler.ts`
- `src/extension.ts`
- `webview-ui/src/components/settings/ApiOptions.tsx`
- And other UI/type files

### Automated Upstream Sync

A GitHub Action at `.github/workflows/sync-upstream.yml` handles this automatically:
- **Schedule:** Every Monday at midnight UTC
- **Manual trigger:** Available from the Actions tab
- **What it does:** `git fetch upstream && git rebase upstream/main && git push --force-with-lease`
- **On conflict:** Creates a GitHub issue with label `upstream-sync` and resolution instructions
- **Deduplication:** Won't create duplicate issues if one is already open

### Manual Sync (when needed)

```bash
git fetch upstream
git rebase upstream/main
# Resolve conflicts if any
git push origin main --force-with-lease --no-verify
```

Note: `--no-verify` is needed because the husky pre-push hook blocks pushes to `main`. This is safe for a personal fork.

### Key Commits on Top of Upstream

1. `47ad91c49` — `Revert "feat: remove Claude Code provider (#10883)"`
2. `104e9e9ed` — `ci: add automated upstream sync workflow`

### Known Pre-existing Issues (not from our changes)

- `@roo-code/web-evals` build fails — missing module resolution in Next.js (`./api.js`, `./cloud.js`, etc.)
- `@roo-code/cloud` test failure — `StaticSettingsService` schema error message mismatch
- Node version warning — repo wants `20.19.2`, local has `24.12.0`
