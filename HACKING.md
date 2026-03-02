# Hacking

## Local Development

To install from a local clone, add the following to `~/.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "claude-swe-workflows@claude-swe-workflows": true
  }
}
```

And register the local directory as a marketplace in `~/.claude/plugins/known_marketplaces.json`:

```json
{
  "claude-swe-workflows": {
    "source": {
      "source": "directory",
      "path": "/path/to/claude-swe-workflows"
    },
    "installLocation": "/path/to/claude-swe-workflows",
    "lastUpdated": "2026-01-01T00:00:00.000Z"
  }
}
```
