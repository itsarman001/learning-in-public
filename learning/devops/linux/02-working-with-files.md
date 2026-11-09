# Working with Files

## The `file` Command

`file <filename>` ignores the file's **extension** and instead inspects the file's **actual content** to determine its real type.

Example:

```bash
touch file.mp4        # actually contains a Python script
file file.mp4
# → file.mp4: Python script, ASCII text executable
```

So `file` reports what the content *actually is*, regardless of what the extension claims.

---

## Nano Text Editor

**Nano** is a simple, easy-to-use text editor that operates within a terminal window.

- If the file **does not exist**, nano creates and opens it.
- If the file **exists**, nano opens it for editing.

### Basic Shortcuts

All keyboard shortcuts use **Ctrl** (shown as `^` in nano's help menu):

| Shortcut       | Action                              |
|----------------|------------------------------------|
| `Ctrl + O`     | Write (save) the current file to disk |
| `Ctrl + X`     | Close the file buffer and exit nano   |
| `Alt + G`      | Go to a specific line number or column |
| `Ctrl + ↑`     | Jump to start of the text             |
| `Ctrl + ↓`     | Jump to end of the text               |
| `Ctrl + A`     | Jump to start of the current line     |
| `Ctrl + E`     | Jump to end of the current line       |

### Workflow Example

```bash
nano myfile.txt
# → Opens nano (creates file if it doesn't exist)
# → Type or edit content
# → Ctrl + O → Save
# → Ctrl + X → Exit
```
