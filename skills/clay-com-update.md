---
name: update
description: Keep Clay up to date. Use when the user asks to update or upgrade Clay, the plugin, or the `clay` CLI; when `clay update` reports the CLI is pinned / managed by the plugin; or to check whether a newer version is available. In the plugin, the CLI is pinned by the plugin, so updating means updating the plugin.
allowed-tools: Bash
---

# Keeping Clay up to date

The plugin **bundles and pins** a specific `clay` CLI version. Because of that pin,
`clay update` **cannot** self-update here: it detects the plugin-managed binary and
tells you to update the plugin instead. So for plugin users, "update the CLI" means
**update the plugin** — that moves the pin, and the newer CLI is fetched on next use.

## 1. Check what you're on, and whether a newer version exists

```bash
clay --version        # reports <cliVersion>+<commit>
clay update --check   # reports { updated, latestVersion, message }
```

`clay update --check` reports the latest published CLI version and a `message`. In the
plugin, that message is:

> "This binary is managed by the Clay agent plugin and is pinned to a specific version.
> Update the plugin to change the CLI version."

Seeing that message is expected — it's the signal to update the plugin (step 2), not an
error. Compare `latestVersion` against the `<cliVersion>` from `clay --version`: if
they differ, an update is available.

## 2. Update the plugin

The marketplace is named `clay-plugins` and the plugin is `clay`. Pick your harness:

### Claude Code

Refresh the marketplace, then update the plugin:

```bash
claude plugin marketplace update clay-plugins
claude plugin update clay@clay-plugins
```

Then have the user run `/reload-plugins` (or restart Claude Code) so the new version
loads. If `clay --version` still shows the old version after that, the marketplace
clone was stale — the explicit `claude plugin marketplace update clay-plugins` above
force-refreshes it; as a last resort, uninstall and reinstall the `clay` plugin.

### Codex

Refresh the marketplace snapshot, then restart the Codex session:

```bash
codex plugin marketplace upgrade clay-plugins
```

`codex plugin marketplace upgrade` pulls the latest plugin version; restart Codex so
the running session picks it up.

### Cursor

Cursor is UI-driven — tell the user to do this (there's no reliable CLI path):

- Open **Customize** in the sidebar, find the **Clay** plugin, and click **Refresh**.
- With Auto Refresh enabled, Cursor picks up new commits atomically on its next refresh
  cycle; a manual **Refresh** forces it.
- If the content still looks stale, uninstall and reinstall the Clay plugin.

## 3. Verify

Updating the plugin only moves the pinned CLI version (`bin/cli-version`); the plugin's
`bin/clay` shim then downloads and caches that binary on the next `clay` call. That
fetch is the same on every harness (`bin/` is shared). What differs is whether the
`clay` on your PATH resolves to the **updated** plugin:

- **Claude Code** adds the plugin's `bin/` to PATH and repoints it to the new version
  automatically, so the next `clay` call runs the updated shim.
- **Codex / Cursor** use the `~/.local/bin/clay` forwarder the `setup` skill wrote.
  The current forwarder re-resolves the newest bundled launcher on every call, so a
  plugin update is picked up automatically once you restart the agent — no need to
  repoint it. One exception: an older, hard-coded forwarder (from before setup wrote
  the self-resolving kind) execs a stale plugin-version path and keeps running the
  old pinned CLI. If `clay --version` stays on the old version after updating and
  restarting, re-run the `setup` skill once to replace it.

Then re-check — the first `clay --version` is also what pulls the newly-pinned binary:

```bash
clay --version
clay update --check
```

The `<cliVersion>` should now match the `latestVersion` from `clay update --check`.

## Authoritative details

The CLI help text is a machine-readable spec written for you to read:

```bash
clay update --help
```
