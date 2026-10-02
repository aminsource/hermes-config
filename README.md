# Hermes portable config

Portable, non-secret Hermes Desktop/Agent assets shared across machines.

This repo is for things that are safe and useful to sync between Hermes installs:

- `desktop-plugins/` — Hermes Desktop plugins, including `iransansweb-font`
- `skills/` — custom/user skills
- `skins/` — Hermes skins
- `dashboard-themes/` — dashboard/desktop theme YAMLs

Do **not** commit private runtime state or secrets:

- `.env`
- `auth.json`
- `state.db`
- `sessions/`
- `logs/`
- `cache/`
- `cron/`
- `memories/`

## Install on another machine

First install Hermes normally on the target machine and run it once so `~/.hermes/` exists.

Then clone this repo:

```bash
git clone git@github.com:aminsource/hermes-config.git ~/hermes-config
```

Back up any existing local Hermes folders, then link this repo into Hermes:

```bash
mkdir -p ~/.hermes

for d in desktop-plugins skills skins dashboard-themes; do
  if [ -e "$HOME/.hermes/$d" ] && [ ! -L "$HOME/.hermes/$d" ]; then
    mv "$HOME/.hermes/$d" "$HOME/.hermes/$d.bak.$(date +%Y%m%d-%H%M%S)"
  elif [ -L "$HOME/.hermes/$d" ]; then
    rm "$HOME/.hermes/$d"
  fi
  ln -s "$HOME/hermes-config/$d" "$HOME/.hermes/$d"
done
```

Restart Hermes Desktop, or use **Cmd/Ctrl+K → Reload desktop plugins**.

## Pull updates on another machine

```bash
cd ~/hermes-config
git pull
```

Then reload desktop plugins if a plugin changed.

## Push changes from a machine

```bash
cd ~/hermes-config
git status
git add desktop-plugins skills skins dashboard-themes README.md .gitignore
git commit -m "Update Hermes config"
git push
```

## Notes

- This repo intentionally does not sync provider tokens, OAuth sessions, chat history, memories, or logs.
- API keys and OAuth login must be configured separately on each machine with `hermes setup`, `hermes model`, or `hermes auth add ...`.
- If you add machine-specific absolute paths inside a skill or plugin, they may need editing on the other machine.

---

# راهنمای فارسی

این repo برای انتقال بخش‌های امن و قابل‌حمل Hermes بین چند سیستم است.

مواردی که sync می‌شوند:

- پلاگین‌های Desktop مثل `iransansweb-font`
- skillها
- skinها
- themeهای dashboard/desktop

مواردی که نباید داخل git بروند:

- کلیدهای API و secretها
- فایل‌های OAuth مثل `auth.json`
- history چت‌ها و `state.db`
- sessionها، logها، cache، memoryها

## نصب روی سیستم دیگر

اول Hermes را روی سیستم مقصد نصب کن و یک بار اجرا کن تا پوشه `~/.hermes/` ساخته شود.

بعد:

```bash
git clone git@github.com:aminsource/hermes-config.git ~/hermes-config
```

بعد این دستور را بزن تا پوشه‌های Hermes به repo وصل شوند:

```bash
mkdir -p ~/.hermes

for d in desktop-plugins skills skins dashboard-themes; do
  if [ -e "$HOME/.hermes/$d" ] && [ ! -L "$HOME/.hermes/$d" ]; then
    mv "$HOME/.hermes/$d" "$HOME/.hermes/$d.bak.$(date +%Y%m%d-%H%M%S)"
  elif [ -L "$HOME/.hermes/$d" ]; then
    rm "$HOME/.hermes/$d"
  fi
  ln -s "$HOME/hermes-config/$d" "$HOME/.hermes/$d"
done
```

بعد Hermes Desktop را restart کن یا از **Cmd/Ctrl+K → Reload desktop plugins** استفاده کن.

## گرفتن تغییرات جدید

```bash
cd ~/hermes-config
git pull
```

## فرستادن تغییرات جدید

```bash
cd ~/hermes-config
git status
git add desktop-plugins skills skins dashboard-themes README.md .gitignore
git commit -m "Update Hermes config"
git push
```
