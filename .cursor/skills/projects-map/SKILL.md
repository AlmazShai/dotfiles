---
name: projects-map
description: >-
  Resolves which local folder holds a repository or related context by reading
  the workspace project map. Use when asking where a repo, config tree, or
  cross-repo dependency lives, when folder names may not match upstream names,
  or when work spans multiple checkouts under the projects root.
---

# Projects map

## Source of truth

Multi-repo work is rooted at **`~/projects`** (container mount may be `/workspace`).

Before searching the filesystem, read:

**`~/projects/PROJECT_MAP.md`**

That file is the canonical name → path → role mapping for this machine. Prefer it over guessing from upstream remote titles; folder names often differ from repo names.

## How to work

1. Open `~/projects/PROJECT_MAP.md`.
2. Match the user’s need (role, alias, or product area) to a path in the map.
3. If the map links further guides (`AGENTS.md`, README, etc.), read those next for layout quirks.
4. Work inside the mapped path; widen the search only if the map does not cover the request.
5. Honor any always-applied build/git rules for this environment when compiling or committing.

## Do not

- Hard-code or invent paths that are not in `PROJECT_MAP.md`.
- Assume the checkout directory equals the Git remote project name.
- Duplicate the map contents into answers when a path citation is enough.
