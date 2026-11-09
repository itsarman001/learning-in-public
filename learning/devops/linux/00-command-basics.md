# Command Basics

## Command Structure

Generally, a command has three parts:

```
Name + Options + Arguments
```

Example:

```
cat -n file.txt        (short form)
cat --number file.txt  (long form)
 |     |         |
Name Option   Argument
```

- `-n` and `--number` are two ways of passing the **same** option to `cat` (numbers each line of output):
  - `-n` → short form
  - `--number` → full/long name
- `file.txt` is the **argument**.

There can be an unlimited number of options and arguments passed to a command (in practice). There *is* a maximum, but it's imposed by the operating system, not by the shell/command itself.

> **Options and arguments are case-sensitive.**

### Summary

- `Command = command name + options + arguments`
- Options are case-sensitive
- Options can be combined (grouped)
- Option short form: `cat -n abc.txt`
- Option long form: `cat --number abc.txt`

---

## The `man` Command

Used to read the manual for any command.

In the **SYNOPSIS** section you'll find:

```
command_name options arguments
```

Example — `cp`:

```
cp [OPTION] [-T] SOURCE DEST
    -------      ------
    optional     required
```

- Wrapped in `[ ]` → **optional**
- **Underlined** → **required**
