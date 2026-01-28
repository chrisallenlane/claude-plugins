---
name: tea
description: A reference for the Gitea CLI interface (`tea`)
---

# Tea CLI: Issues & Comments

The `tea` CLI interacts with Gitea servers. It auto-detects repository context from `$PWD`.

## Issues

Get all issues:
```bash
tea issues ls --state all
```

Get open issues:
```bash
tea issues ls
```

Get issue details:
```bash
tea issues <number>
```

Create an issue:
```bash
tea issues create --title "Title" --description "$(cat <<'EOF'
Description here.
EOF
)"
```

Close an issue:
```bash
tea issues close <number>
```

Reopen an issue:
```bash
tea issues reopen <number>
```

## Comments

Create a comment:
```bash
cat <<'EOF' | tea comment <number>
Comment here.
EOF
```
