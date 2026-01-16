# Troubleshooting

Common issues and solutions for the Antigravity Auth plugin.

---

## Quick Fixes

### Auth problems
Delete the token file and re-login:
```bash
rm ~/.config/opencode/antigravity-accounts.json
opencode auth login
```

### "Model not found"
Add this to your `google` provider config:
```json
"npm": "@ai-sdk/google"
```

### Session errors
Type `continue` to trigger auto-recovery, or use `/undo` to rollback.

---

## Gemini CLI Permission Error

When using Gemini CLI models, you may see:
> Permission 'cloudaicompanion.companions.generateChat' denied on resource '//cloudaicompanion.googleapis.com/projects/...'

**Why this happens:** The plugin defaults to a predefined project ID that doesn't exist in your Google Cloud account. Antigravity models work, but Gemini CLI models need your own project.

**Solution:**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create or select a project
3. Enable the **Gemini for Google Cloud API** (`cloudaicompanion.googleapis.com`)
4. Add `projectId` to your account in `~/.config/opencode/antigravity-accounts.json`:

```json
{
  "version": 3,
  "accounts": [
    {
      "email": "you@gmail.com",
      "refreshToken": "...",
      "projectId": "your-project-id"
    }
  ]
}
```

> **Note:** For multi-account setups, add `projectId` to each account.

---

## Safari OAuth Callback Fails (macOS)

**Symptoms:**
- "fail to authorize" after successful Google login
- Safari shows "Safari can't open the page" or connection refused

**Cause:** Safari's "HTTPS-Only Mode" blocks the `http://localhost` callback URL.

**Solutions:**

1. **Use a different browser** (easiest):
   Copy the URL from `opencode auth login` and paste it into Chrome or Firefox.

2. **Temporarily disable HTTPS-Only Mode:**
   - Safari > Settings (⌘,) > Privacy
   - Uncheck "Enable HTTPS-Only Mode"
   - Run `opencode auth login`
   - Re-enable after authentication

3. **Manual callback extraction** (advanced):
   - When Safari shows the error, the address bar contains `?code=...&scope=...`
   - See [issue #119](https://github.com/NoeFabris/opencode-antigravity-auth/issues/119) for manual auth support

---

## Port Already in Use

If OAuth fails with "Address already in use":

**macOS / Linux:**
```bash
lsof -i :8080
kill -9 <PID>
opencode auth login
```

**Windows:**
```powershell
netstat -ano | findstr :8080
taskkill /PID <PID> /F
opencode auth login
```

---

## WSL2 / Remote Development

The OAuth callback requires the browser to reach `localhost` on the machine running OpenCode.

- **WSL2:** Configure port forwarding, or use VS Code's port forwarding
- **SSH:** Use `ssh -L 8080:localhost:8080 user@remote`
- **Headless servers:** See [issue #119](https://github.com/NoeFabris/opencode-antigravity-auth/issues/119) for manual auth

---

## Plugin Compatibility Issues

### @tarquinen/opencode-dcp

DCP creates synthetic assistant messages that lack thinking blocks. **List this plugin BEFORE DCP:**

```json
{
  "plugin": [
    "opencode-antigravity-auth@latest",
    "@tarquinen/opencode-dcp@latest"
  ]
}
```

### oh-my-opencode

Disable built-in auth:
```json
{
  "google_auth": false
}
```

When spawning parallel subagents, multiple processes may hit the same account. **Workaround:** Enable `pid_offset_enabled: true` or add more accounts.

### Other gemini-auth plugins

You don't need them. This plugin handles all Google OAuth.

---

## Migration Guides

### v1.2.8+ (Variants)

v1.2.8+ introduces **model variants** for dynamic thinking configuration.

**Before (v1.2.7):**
```json
{
  "antigravity-claude-sonnet-4-5-thinking-low": { ... },
  "antigravity-claude-sonnet-4-5-thinking-max": { ... }
}
```

**After (v1.2.8+):**
```json
{
  "antigravity-claude-sonnet-4-5-thinking": {
    "variants": {
      "low": { "thinkingConfig": { "thinkingBudget": 8192 } },
      "max": { "thinkingConfig": { "thinkingBudget": 32768 } }
    }
  }
}
```

Old tier-suffixed models still work for backward compatibility.

### v1.2.7 (Prefix)

v1.2.7+ uses explicit `antigravity-` prefix:

| Old Name | New Name |
|----------|----------|
| `gemini-3-pro-low` | `antigravity-gemini-3-pro` |
| `claude-sonnet-4-5` | `antigravity-claude-sonnet-4-5` |

Old names work as fallback, but `antigravity-` prefix is recommended.

---

## Debugging

Enable debug logging:
```bash
OPENCODE_ANTIGRAVITY_DEBUG=1 opencode   # Basic
OPENCODE_ANTIGRAVITY_DEBUG=2 opencode   # Verbose (full request/response)
```

Logs are in `~/.config/opencode/antigravity-logs/`.

---

## E2E Testing

The plugin includes regression tests (consume API quota):

```bash
npx tsx script/test-regression.ts --sanity      # 7 tests, ~5 min
npx tsx script/test-regression.ts --heavy       # 4 tests, ~30 min
npx tsx script/test-regression.ts --dry-run     # List tests
```

---

## Still stuck?

Open an issue on [GitHub](https://github.com/NoeFabris/opencode-antigravity-auth/issues).
