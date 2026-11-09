# File System Operations

## `ls`, `pwd`, `cd`

- **`pwd`** — Print Working Directory
- **`cd`** — Change Directory
- **`ls`** — List directory contents
  - `ls -a` / `ls --all` → list directory contents *including hidden files*

### Combining (grouping) options

Options can be merged for shorthand, e.g.:

```
ls -at     # sort by time, and show hidden files
```

### Directory references

- `.`  → current directory
- `..` → parent (previous) directory
- `/`  → root of the filesystem
- `~`  → home directory

---

## Timestamps

| Command    | Shows            | Updated when...                                              |
|------------|------------------|---------------------------------------------------------------|
| `ls -l`    | Modification time | The **contents** of the file are changed                     |
| `ls -lc`   | Change time        | The file is **moved, renamed**, or its **permissions** change |
| `ls -lu`   | Access time        | The file has been **accessed** by any means (terminal, app, etc.) |

---

## Making Directories and Files

### `mkdir` — create directories

```bash
mkdir folder_name              # create a single folder
mkdir -p parent/child          # create nested folders without errors
mkdir -p parent/{child1,child2}
mkdir -p a/b/c a/{b1,b2}
```

- `-p` creates parent directories as needed, and won't error if they already exist.

### `touch` — create files

```bash
touch a.txt b.txt
```

- `touch` updates **both** access time and modification time of an existing file — unless you use:
  - `-a` → change **only** access time
  - `-m` → change **only** modification time

### `stat`

```bash
stat <entity>
```

Shows detailed status/metadata of a file or directory (timestamps, size, permissions, etc.)

### Files/folders with spaces in the name

Wrap the name in quotes:

```bash
touch "a b"
mkdir "a b"
```
