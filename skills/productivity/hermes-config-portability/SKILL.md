---
name: hermes-config-portability
description: "Use when syncing portable Hermes config via Git."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes, config, portability, git, sync, symlinks, backup]
    related_skills: [hermes-agent, hermes-desktop-customization]
---

# Hermes Config Portability

Use this when the user wants to move or sync Hermes customizations between Hermes installs, especially via Git. The target shape is a small repo of portable user-authored assets symlinked into `HERMES_HOME`, not a Git repo containing the whole `~/.hermes` runtime state.

## When to Use

- The user wants to reuse Hermes Desktop plugins, skins, dashboard themes, or skills on another Hermes install.
- The user asks whether to put Hermes settings in Git or wants a portable sync workflow.
- The user wants a backup that excludes credentials, sessions, memories, logs, caches, and install files.

## Procedure: create a portable Git repo

1. **Load `hermes-agent` first for current path/profile rules.** Resolve the active Hermes home from `$HERMES_HOME` when available; otherwise use `~/.hermes`. Never modify another profile's directories unless the user explicitly asks.
2. **Create a separate repo outside Hermes runtime state**, for example:
   ```bash
   mkdir -p ~/hermes-config
   cd ~/hermes-config
   git init
   ```
3. **Move only portable asset directories into the repo and symlink them back:**
   ```bash
   for d in desktop-plugins skills skins dashboard-themes; do
     if [ -L "$HOME/.hermes/$d" ]; then
       echo "$d already symlinked"
     elif [ -d "$HOME/.hermes/$d" ]; then
       mv "$HOME/.hermes/$d" "$HOME/hermes-config/$d"
       ln -s "$HOME/hermes-config/$d" "$HOME/.hermes/$d"
     else
       mkdir -p "$HOME/hermes-config/$d"
       ln -s "$HOME/hermes-config/$d" "$HOME/.hermes/$d"
     fi
   done
   ```
   If any destination already exists, stop and inspect instead of overwriting.
4. **Add a defensive `.gitignore` before committing:**
   ```gitignore
   # secrets / auth
   .env
   .env.*
   auth.json
   *.key
   *.pem
   *.p12
   *.pfx

   # Hermes private/runtime state
   state.db
   state.db-*
   sessions/
   logs/
   cache/
   cron/
   memories/
   *.db
   *.sqlite
   *.sqlite3

   # Hermes skill-manager runtime metadata
   skills/.curator_*
   skills/.usage*
   skills/.locks/
   skills/.hub/
   skills/.bundled_manifest

   # OS/editor noise
   .DS_Store
   Thumbs.db
   *.swp
   *.tmp
   ```
5. **Add a short README** explaining that the repo is for portable non-secret Hermes assets and that credentials, auth tokens, memories, sessions, logs, and runtime databases must not be committed.
6. **Scan before commit for accidental secrets and runtime files.** At minimum check that `.env`, `auth.json`, key files, and databases are absent from the repo. Text hits for words like `token` inside documentation are not automatically secrets; distinguish instructions from live credential files.
7. **Commit locally once clean:**
   ```bash
   git add .
   git commit -m "Initial portable Hermes config"
   git status --short
   ```
   Add a remote only after the user provides the target URL.

## Procedure: install the repo on another machine

1. Install or update Hermes normally on the destination; do not copy the `hermes-agent` source tree as configuration.
2. Clone the portable repo:
   ```bash
   git clone <remote-url> ~/hermes-config
   ```
3. Back up existing destination asset directories, then symlink the repo copies:
   ```bash
   mv ~/.hermes/desktop-plugins ~/.hermes/desktop-plugins.bak 2>/dev/null || true
   mv ~/.hermes/skills ~/.hermes/skills.bak 2>/dev/null || true
   mv ~/.hermes/skins ~/.hermes/skins.bak 2>/dev/null || true
   mv ~/.hermes/dashboard-themes ~/.hermes/dashboard-themes.bak 2>/dev/null || true

   ln -s ~/hermes-config/desktop-plugins ~/.hermes/desktop-plugins
   ln -s ~/hermes-config/skills ~/.hermes/skills
   ln -s ~/hermes-config/skins ~/.hermes/skins
   ln -s ~/hermes-config/dashboard-themes ~/.hermes/dashboard-themes
   ```
4. Keep credentials and auth machine-local. Re-run provider auth or copy secrets only through a secure, intentional channel.
5. Run `hermes doctor` and reload Hermes Desktop plugins if Desktop customizations do not appear immediately.

## Pitfalls

- **Do not put the whole `~/.hermes` directory in Git.** It contains credentials, OAuth tokens, chat/session history, memories, logs, caches, runtime databases, and install state that are private or machine-specific.
- **Do not commit `config.yaml` wholesale unless the user explicitly wants that and it has been reviewed.** It often contains absolute paths and machine-specific settings; use a redacted `config.example.yaml` for shared defaults.
- **Do not sync `~/.hermes/hermes-agent` as user configuration.** It is the application checkout/install; install Hermes on each machine and sync only user-authored portable assets.
- **Keep skill-manager metadata out of Git.** Directories such as `skills/.hub`, `skills/.locks`, and files such as `skills/.usage*` and `skills/.curator_*` are runtime bookkeeping, not reusable workflow content.
- **Stop on destination collisions.** If the repo already has a directory or the target Hermes directory is already a symlink, inspect and merge deliberately instead of overwriting user work.
