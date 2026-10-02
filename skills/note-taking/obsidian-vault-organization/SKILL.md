---
name: obsidian-vault-organization
description: Use when organizing notes in the user's HoomHoom vault.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux]
metadata:
  hermes:
    tags: [Obsidian, Notes, Vault, Organization]
    related_skills: [obsidian]
---

# Obsidian Vault Organization

Use this with the `obsidian` skill when the task is not just creating text, but deciding where a note belongs in the user's vault.

## When to Use

Use this when creating, moving, or filing Obsidian notes for the user and the destination needs to be chosen inside the HoomHoom vault structure.

## Procedure

1. Resolve the concrete vault path before file operations. When the user names `/home/hooman/Documents/Obsidian` and asks for the note to be in `HoomHoom`, use `/home/hooman/Documents/Obsidian/HoomHoom` as the working vault root.
2. Inspect the existing structure with `search_files(target="files")` before choosing a destination. Prefer existing class folders and index notes over creating new top-level folders.
3. Put the note in the closest subject folder, then update that folder's index note with a wikilink when one exists.
4. Keep filenames descriptive and class-level, not dated incident logs, unless the user explicitly asks for a journal/daily note.
5. Report the final path and any index note changed; do not replay the full note unless asked.

## Placement rules

- Technical Linux/system configuration notes belong under `Resources/Technology/` and should be linked from `Resources/Technology/Technology.md` unless the vault structure clearly provides a better home.
- General capture or unresolved items belong in `Inbox/` only when no appropriate subject folder is clear.
- Prefer Persian link aliases in indexes when the surrounding index is Persian or bilingual.

## Pitfalls

- Do not leave new notes in the outer `/home/hooman/Documents/Obsidian` folder when the user asks for `HoomHoom`; that outer directory contains the vault container and support files, while the user's organized notes live under `HoomHoom`.
- Do not create an orphan note without linking it from the relevant index when an index exists; Obsidian navigation depends on those local index notes.
