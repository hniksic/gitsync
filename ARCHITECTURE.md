# gitsync Architecture

## Overview

gitsync establishes a bidirectional communication channel with a remote host over a single
SSH connection. It transfers only the minimal data needed to synchronize the remote
working tree with the local one:

1. Missing commits (as a git bundle)
2. The target HEAD reference
3. List of new files in the diff (so remote can remove stale previously synced uncommitted copies)
4. Uncommitted changes (as a binary diff)

## Communication Architecture

gitsync uses named pipes (FIFOs) to create bidirectional communication over SSH:

```
                      loc2rem (FIFO)
   LOCAL stdout ─────────────────────────────▶ SSH stdin (remote script)

                      rem2loc (FIFO)
   LOCAL stdin  ◀───────────────────────────── SSH stdout (remote script)

   stderr: passes through directly for status messages
```

The local process redirects its stdin/stdout to the pipes after launching SSH in the
background, allowing the two sides to exchange data synchronously.

## Data Flow

```
    LOCAL                                                 REMOTE (via SSH)
    ─────                                                 ────────────────

    ┌─────────────────┐                                   ┌─────────────────┐
    │ Create pipes:   │                                   │                 │
    │ loc2rem,rem2loc │──────── SSH connection ──────────▶│ cd $rel_topdir  │
    └─────────────────┘                                   └────────┬────────┘
                                                                   │
                                                                   ▼
                                                          ┌─────────────────┐
                         ┌────────────────────────────────│ git rev-list    │
                         │      rev-list: SHAs of         │ @{u}^..HEAD     │
    ┌────────────────┐   │    commits remote has          └─────────────────┘
    │ Find common    │◀──┘   (reverse chronological)
    │ ancestor via   │        e.g. "abc123 def456 ..."
    │ git merge-base │
    └───────┬────────┘
            │
            ▼
    ┌─────────────────┐       base64-encoded bundle       ┌─────────────────┐
    │ git bundle      │      (or empty if up-to-date)     │ base64 -d       │
    │ create          │──────────────────────────────────▶│ git fetch       │
    │ $base..HEAD     │                                   │ (from bundle)   │
    └───────┬─────────┘                                   └────────┬────────┘
            │                                                      │
            ▼                                                      ▼
    ┌─────────────────┐          new HEAD SHA             ┌─────────────────┐
    │ git rev-parse   │──────────────────────────────────▶│ git reset --hard│
    │ HEAD            │          e.g. "abc123"            │ $new_head       │
    └───────┬─────────┘                                   └────────┬────────┘
            │                                                      │
            ▼                                                      ▼
    ┌─────────────────┐     base64-encoded file list      ┌─────────────────┐
    │ git diff        │      (or empty if none)           │ base64 -d       │
    │ --name-only     │──────────────────────────────────▶│ rm -f each file │
    │ --diff-filter=A │                                   │ (if exists)     │
    └───────┬─────────┘                                   └────────┬────────┘
            │                                                      │
            ▼                                                      ▼
    ┌─────────────────┐     base64-encoded binary diff    ┌─────────────────┐
    │ git diff        │      (or empty if clean)          │ base64 -d       │
    │ --binary HEAD   │──────────────────────────────────▶│ git apply       │
    └─────────────────┘                                   └─────────────────┘
```

## Protocol

The protocol is a synchronous 5-step exchange over a single SSH connection:

### Step 1: Remote announces state

```
REMOTE → LOCAL:
"sha1 sha2 sha3 ..."   # rev-list from @{u}^ to HEAD (reverse chronological)
```

The remote sends all commits between its upstream and HEAD. The local side iterates
through these to find the first commit that exists in local history (via `git
merge-base`), which becomes the base for the bundle.

### Step 2: Local sends missing commits

```
LOCAL → REMOTE:
"R0lGODlh..."          # base64-encoded git bundle (or empty line if none needed)
```

If there are commits to transfer, local creates a git bundle containing `$base..HEAD` and
sends it base64-encoded. The remote decodes it and runs `git fetch` from the bundle file.

### Step 3: Local sends target HEAD

```
LOCAL → REMOTE:
"abc123def456..."      # full SHA of the desired HEAD
```

The remote runs `git reset --hard` to this commit, updating its HEAD and clearing any
existing uncommitted changes.

### Step 4: Local sends list of new files

```
LOCAL → REMOTE:
"c3JjL25ldy4uLg==" # base64-encoded newline-separated file list (or empty line if none)
```

Before sending the diff, local identifies files that will be created (`git diff -z
--name-only --diff-filter=A HEAD`) and sends the list. The remote removes these files if
they exist (which git reset --hard won't do if you were previously uncommitted),
preventing "already exists in working directory" errors when applying the diff.

### Step 5: Local sends uncommitted changes

```
LOCAL → REMOTE:
"ZGlmZiAt..."          # base64-encoded binary diff (or empty line if clean)
```

If there are local uncommitted changes (staged or unstaged), they're captured with `git
diff --binary HEAD`, base64-encoded, and sent to the remote. The remote applies them with
`git apply`.
