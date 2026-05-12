# `pcat` – Pretty Cat with Many Features

---

## Table of Contents

1. [Introduction](#introduction)
2. [Synopsis](#synopsis)
3. [Description](#description)
4. [Options](#options)
   - [Display & Formatting Options](#display--formatting-options)
   - [Recursion & Directory Filtering Options](#recursion--directory-filtering-options)
   - [Size, Line & Character Limit Options](#size-line--character-limit-options)
   - [Paging & Follow Options](#paging--follow-options)
   - [Miscellaneous Options](#miscellaneous-options)
5. [Special Arguments](#special-arguments)
6. [Pattern Matching & Wildcards](#pattern-matching--wildcards)
7. [Size Unit Syntax – Complete Reference](#size-unit-syntax--complete-reference)
   - [Supported Units](#supported-units)
   - [Floating Point & Decimal Values](#floating-point--decimal-values)
   - [Case Insensitivity](#case-insensitivity)
   - [Default Behaviour](#default-behaviour)
   - [Examples of Valid Size Strings](#examples-of-valid-size-strings)
8. [Include / Exclude Patterns – Detailed Rules](#include--exclude-patterns--detailed-rules)
   - [Multiple Patterns with Commas](#multiple-patterns-with-commas)
   - [Matching Scope](#matching-scope)
   - [Directory‑Specific Include/Exclude](#directoryspecific-includeexclude)
9. [Recursion Behaviour](#recursion-behaviour)
10. [Follow Mode (`-f`) – In‑Depth](#follow-mode--f--indepth)
11. [Paging (`-p`) and Custom Pagers](#paging--p-and-custom-pagers)
12. [Colour Output (`-c`) – Requirements & Fallback](#colour-output--c--requirements--fallback)
13. [Line Numbers (`-n`) & Start Line (`-l`)](#line-numbers--n--start-line--l)
14. [Line Truncation (`-w`)](#line-truncation--w)
15. [Separator (`-S`)](#separator--s)
16. [Reading from Standard Input (`-`)](#reading-from-standard-input--)
17. [Exit Status](#exit-status)
18. [Examples](#examples)
    - [Basic Usage](#basic-usage)
    - [Line Numbers & Colour](#line-numbers--colour)
    - [Recursion & Directory Control](#recursion--directory-control)
    - [Include / Exclude Files](#include--exclude-files)
    - [Size, Line & Character Limits](#size-line--character-limits-1)
    - [Follow Mode & Paging](#follow-mode--paging-1)
    - [Complex Real‑World Examples](#complex-realworld-examples)
19. [Bugs, Limitations & Future‑Proofing](#bugs-limitations--future-proofing)
20. [License](#license)

---

## Introduction

`pcat` (Pretty Cat) is a command‑line utility that displays file contents with enhanced formatting, filtering, and navigation. It was designed to overcome the limitations of standard `cat` while remaining simple to use. `pcat` can recursively traverse directories, include or exclude files by pattern, filter by file size, line count, or character count, output with line numbers and colours, follow a file in real time (like `tail -f`), and page through long outputs.

All filtering is silent – files that do not meet criteria are simply omitted, and binary files are skipped without warnings.

---

## Synopsis

```bash
pcat [OPTIONS] [FILE...]
```

**Examples of valid invocations:**

```bash
pcat file.txt
pcat -n -c *.log
pcat -r . -i "*.py" -x "*test*" --min-size 50k -p
echo "hello world" | pcat -
```

---

## Description

`pcat` reads each file (or standard input) and writes its content to standard output. Before writing, a header line is printed showing the file name, surrounded by a configurable separator (default: `==========`). Optionally, line numbers can be prefixed, and the header and numbers can be coloured.

If recursion (`-r`) is enabled, directories are traversed and all files inside (subject to include/exclude filters) are shown.

Binary files are detected by checking for a null byte in the first 1024 bytes; they are silently skipped – no error or warning is emitted.

All size, line, and character limits are inclusive. For example, `--min-lines 5` requires **at least** 5 lines, so a file with exactly 5 lines is included.

---

## Options

### Display & Formatting Options

| Option | Argument | Default | Description |
|--------|----------|---------|-------------|
| `-n`, `--line-num` | none | off | Show line numbers (starting from 1 for the first displayed line, even if `-l` skips earlier lines). |
| `-c`, `--color` | none | off | Colour the header cyan and the line numbers green. Colours are disabled if `colorama` is not installed (but the script still runs correctly). |
| `-S`, `--separator` | STRING | `==========` | The string used to build the header. The final header becomes `SEPARATOR filename SEPARATOR`. Example: `-S "***"` → `*** file.txt ***`. |
| `-l`, `--start-line` | N | 1 | Start output at line N (the first line of the file is line 1). Lines before N are discarded. |
| `-w`, `--max-len` | N | 0 | Truncate each line after N characters. If a line is longer, it is cut and `…` (U+2026) is appended. `0` means no truncation. |

### Recursion & Directory Filtering Options

| Option | Argument | Default | Description |
|--------|----------|---------|-------------|
| `-r`, `--recursive` | none | off | Traverse directories recursively. Without this, passing a directory produces an error. |
| `-x`, `--exclude` | PATTERN | (none) | Exclude files whose name or full path matches PATTERN (shell wildcards). Can be repeated or comma‑separated. |
| `-i`, `--include` | PATTERN | (none) | **Only** include files that match at least one given PATTERN. If any `-i` is used, files not matching any pattern are excluded. |
| `-xd`, `--exclude-dir` | PATTERN | (none) | When recursing, exclude directories whose name or full path matches PATTERN. The directory itself is not entered. |
| `-id`, `--include-dir` | PATTERN | (none) | When recursing, **only** enter directories that match at least one given PATTERN. If specified, directories not matching any pattern are skipped entirely. |

### Size, Line & Character Limit Options

All limit options are **inclusive** and accept integer values. Size options also accept unit strings (see [Size Unit Syntax](#size-unit-syntax)). A value of `0` (or empty) disables the filter.

| Option | Argument | Default | Description |
|--------|----------|---------|-------------|
| `--min-size` | SIZE | 0 | Minimum file size. Files smaller than this are skipped. |
| `--max-size` | SIZE | 0 | Maximum file size. Files larger than this are skipped. |
| `--min-lines` | N | 0 | Minimum number of lines. Files with fewer lines are skipped. |
| `--max-lines` | N | 0 | Maximum number of lines. Files with more lines are skipped. |
| `--min-chars` | N | 0 | Minimum number of characters (Unicode code points, not bytes). Files with fewer characters are skipped. |
| `--max-chars` | N | 0 | Maximum number of characters. Files with more characters are skipped. |

**Note:** For `--min-chars` and `--max-chars`, the entire file content (after reading as UTF-8 with `errors='replace'`) is counted. Newline characters are included. Binary files are skipped before counting.

### Paging & Follow Options

| Option | Argument | Default | Description |
|--------|----------|---------|-------------|
| `-p`, `--page` | none | off | Pipe the entire output through a pager. Default pager is `less -R` on Unix-like systems, `more` on Windows. |
| `--pager-cmd` | COMMAND | `less -R` / `more` | Override the pager command. The command is split on whitespace. Example: `--pager-cmd "less -R -X"`. |
| `-f`, `--follow` | none | off | Follow a single file (like `tail -f`). Output new lines as they are appended. Ignores all size/line/char limits, include/exclude, recursion, and paging. Only `--max-len` and `--color` (header before following) are respected. Press `Ctrl+C` to stop. |

### Miscellaneous Options

| Option | Description |
|--------|-------------|
| `-h`, `--help` | Show a brief help summary (automatically generated by `argparse`). |
| `--version` | Show version `pcat 2.0`. |

---

## Special Arguments

| Argument | Meaning |
|----------|---------|
| `-` (a single hyphen) | Read from standard input instead of a file. No header is printed. Useful for piping: `echo "text" | pcat -`. |
| `.` (a single dot) | The current working directory. When used with `-r`, it recurses from the current folder. Without `-r`, it produces an error because `.` is a directory. |
| `*` (shell glob) | Not special to `pcat` itself, but the shell expands `*` to all non‑hidden files in the current directory. `pcat` receives the list of expanded names. |

---

## Pattern Matching & Wildcards

Patterns in `-x`, `-i`, `-xd`, `-id` use Python’s `fnmatch` with **case‑sensitive** matching on Linux/Unix (because the underlying file system is case‑sensitive). On Windows/macOS (case‑insensitive file systems), `fnmatch` follows the platform default (usually case‑insensitive). For portability, assume **case‑sensitive**.

Wildcards:

| Wildcard | Meaning |
|----------|---------|
| `*` | matches any sequence of characters (including none) |
| `?` | matches any single character |
| `[seq]` | matches any character in `seq` |
| `[!seq]` | matches any character not in `seq` |

**Examples:**

- `*.log` – matches all files ending with `.log`
- `temp*` – matches files starting with `temp`
- `file?.txt` – matches `file1.txt`, `fileA.txt`, but not `file10.txt`
- `[0-9]*.bak` – matches `.bak` files that start with a digit

Patterns are matched **both against the full path** (as a string) **and against the base filename**. This means `-x "*.tmp"` will exclude a file named `a.tmp` anywhere in the tree, even if the full path is `./subdir/a.tmp`.

---

## Size Unit Syntax – Complete Reference

The `--min-size` and `--max-size` options accept a flexible, human‑readable syntax.

### Supported Units

| Unit (case‑insensitive) | Meaning | Bytes |
|------------------------|---------|-------|
| (no unit) or `b`       | bytes   | 1 |
| `k`, `kb`, `kib`       | kibibytes | 1024 |
| `m`, `mb`, `mib`       | mebibytes | 1024² = 1,048,576 |
| `g`, `gb`, `gib`       | gibibytes | 1024³ = 1,073,741,824 |
| `t`, `tb`, `tib`       | tebibytes | 1024⁴ = 1,099,511,627,776 |

### Floating Point & Decimal Values

You can use decimal numbers. The value is multiplied by the unit’s byte count and then truncated to an integer (floor). Examples:

- `1.5k` → `1.5 * 1024 = 1536` bytes
- `2.3mb` → `2.3 * 1048576 = 2411724` bytes (floor)
- `0.5g` → 536,870,912 bytes

### Case Insensitivity

All units are case‑insensitive: `K`, `kB`, `KiB`, `kib`, `KB` are all recognised as kilobytes. The same applies to `M`, `MB`, `MIB`, `G`, `GB`, `T`, `TB`.

### Default Behaviour

If no unit is given and the string is a plain number (e.g. `500`), it is interpreted as **bytes**.

If the string contains an invalid unit or is unparseable, the size is treated as `0` (meaning no limit). No error is printed.

### Examples of Valid Size Strings

| String | Parsed as | Bytes (approx) |
|--------|-----------|----------------|
| `1024` | 1024 bytes | 1,024 |
| `1k`   | 1 KiB | 1,024 |
| `1.5k` | 1.5 KiB | 1,536 |
| `2M`   | 2 MiB | 2,097,152 |
| `3gib` | 3 GiB | 3,221,225,472 |
| `0.1tb`| 0.1 TiB | 109,951,162,777 |
| `42`   | 42 bytes | 42 |
| `1.`   | 1 byte | 1 |

---

## Include / Exclude Patterns – Detailed Rules

### Multiple Patterns with Commas

You can specify multiple patterns in a single `-x` or `-i` by separating them with **commas** (no spaces required, but spaces are stripped). Examples:

```bash
pcat -x "*.tmp, *.log, *~"
pcat -i "*.py, *.rs, *.go"
```

You can also repeat the option:

```bash
pcat -x "*.tmp" -x "*.log" -x "*~"
```

The two forms can be mixed.

### Matching Scope

- For **file** patterns (`-x`, `-i`): The pattern is compared against the **full path** (as a string, e.g. `./subdir/file.txt`) and against the **base name** (`file.txt`). If either matches, the pattern is considered matched.
- For **directory** patterns (`-xd`, `-id`): Only the directory’s full path and base name are matched. The directory is either excluded (skipped entirely) or included (only entered if it matches at least one include pattern).

### Directory‑Specific Include/Exclude

When recursing, the directory filters are applied **as `os.walk` traverses**. Excluded directories are not entered; included directories (if any `-id` are given) are **only** the ones that match at least one `-id` pattern. If no `-id` is given, all directories are candidates (subject to `-xd`).

This allows you to, for example, only traverse into `src/` and `lib/` while skipping `test/`, `build/`, and any hidden directories:

```bash
pcat -r . -id "src, lib" -xd "test, build, .*"
```

---

## Recursion Behaviour

- When `-r` is **not** used, any directory argument produces an error: `pcat: dir: Is a directory (use -r)`.
- When `-r` is used, directories are traversed using `os.walk`. The traversal respects `-xd` and `-id` to prune branches.
- Hidden files (starting with `.`) are **not** automatically excluded; they are included unless filtered by `-x` or `-i`. To skip hidden files, use `-x ".*"`.
- Symbolic links are followed if `os.walk` follows them (by default it does on Unix). Be careful with cyclic links.

---

## Follow Mode (`-f`) – In‑Depth

The `-f` option (follow) makes `pcat` behave similarly to `tail -f`. It opens a single file and prints new lines as they are written. 

**Smart Start Behavior:**
- If the file **already exists** when you start `pcat`, it seeks to the end and only shows new content (standard behavior).
- If the file **does not exist**, `pcat` will wait for it to appear. Once created, it starts reading from **line 1**, ensuring you see the very first data saved to that file.

**Log Rotation Support:** 
If the file is truncated (e.g., cleared or rotated), `pcat` detects the size change and automatically resets to the beginning of the file to continue following the new data.

**This mode ignores:**
- `-r` (recursion)
- `-x`, `-i`, `-xd`, `-id`
- `--min-size`, `--max-size`, `--min-lines`, `--max-lines`, `--min-chars`, `--max-chars`
- `-l` (start line)
- `-p` (page)
- `--pager-cmd`

**It respects:**
- `-c` (colour – prints a starting message in magenta)
- `-w` (line truncation)
- `-n` (line numbers are **not** printed in follow mode because the offset from start is unknown)

**Note on Buffering:** Standard output buffering can sometimes delay lines from appearing. `pcat` forces an immediate output flush on every new line to ensure true real-time viewing.

If more than one file is given with `-f`, an error is printed and the program exits.

Press `Ctrl+C` to stop following.

---

## Paging (`-p`) and Custom Pagers

When `-p` is used, `pcat` writes its entire output (headers, line numbers, content) to a subprocess’s stdin, typically `less -R`. The `-R` flag preserves ANSI colour codes.

To use a different pager, set `--pager-cmd`. Example:

```bash
pcat -p --pager-cmd "more" largefile.txt
```

The command is split on whitespace, so complex arguments should be quoted. If your pager requires specific arguments, include them:

```bash
pcat -p --pager-cmd "less -R -X -F" *.log
```

Paging is disabled automatically when `-f` is used.

---

## Colour Output (`-c`) – Requirements & Fallback

`pcat` attempts to import `colorama` and call `init(autoreset=True)`. If `colorama` is not installed, the `Fore` and `Style` classes are replaced with dummy versions that produce empty strings. Thus, colours are simply **disabled** and no error is raised.

To install `colorama` (optional):

```bash
pip install colorama
```

When colours are enabled:

- Header: `Fore.CYAN + Style.BRIGHT`
- Line numbers: `Fore.GREEN`
- Follow mode start message: `Fore.MAGENTA`

No other elements are coloured.

---

## Line Numbers (`-n`) & Start Line (`-l`)

Line numbers are printed **before each line**, separated by a space (or coloured green). The numbering always starts at **1** for the first **displayed** line, even if `-l` skips some initial lines.

Example: `pcat -n -l 10 file.txt` will show lines starting from the original line 10, but the first printed line will be numbered `1`.

Line numbers are **not** printed in follow mode (`-f`).

---

## Line Truncation (`-w`)

When `-w N` (with N > 0) is given, each line longer than N characters is cut after the Nth character, and the Unicode ellipsis `…` (U+2026) is appended right after it. This is done **after** reading the file and **before** prepending line numbers.

If N is 0 or negative, no truncation occurs.

---

## Separator (`-S`)

The header is built as:

```
SEPARATOR filename SEPARATOR
```

If the separator string contains spaces or special characters, it is used as‑is. Example:

```bash
pcat -S "--> " file.txt
# Output: --> file.txt -->
```

To suppress the header entirely, use an empty string: `-S ""`. This will produce only the filename on a line by itself (no surrounding separators). 

---

## Reading from Standard Input (`-`)

When the argument `-` is given, `pcat` reads from stdin. No header is printed. 

Because `stdin` is a stream and not a seekable file on disk, file size limits (`--min-size`, `--max-size`) are **ignored**. However, **line and character limits are fully supported!** If the piped input does not meet your `--min-lines` or `--max-chars` requirements, it will be silently filtered out.

Line numbers and truncation also apply normally to stdin.

Example:

```bash
# Only output the web payload if it's over 100 lines long, and truncate width to 80
curl -s https://example.com/ | pcat -n -w 80 --min-lines 100 -
```

You can also combine stdin with regular files, but note that when `-` appears among file arguments, the entire stdin is consumed first, then the files are processed. This behaviour is inherited from Python’s file reading order.

---

## Exit Status

| Code | Meaning |
|------|---------|
| 0 | Success – no errors occurred. |
| 1 | An error occurred: missing file, invalid argument, follow mode without a single file, or a directory without `-r`. |

Errors are printed to stderr (unlike filtered files, which are silently omitted).

---

## Examples

### Basic Usage

```bash
pcat README.md
pcat config.json package.json
pcat *.txt
pcat -                     # type input, press Ctrl+D to end
ls -la | pcat -            # pipe directory listing
```

### Line Numbers & Colour

```bash
pcat -n long_script.py
pcat -c colourful_header.go
pcat -n -c messsage.log
pcat -n -c -l 100 huge.log      # start at line 100, number from 1
```

### Recursion & Directory Control

```bash
pcat -r .                      # everything under current directory
pcat -r src/ --include "*.rs"  # only Rust files in src/
pcat -r . -xd ".git, __pycache__, node_modules"
pcat -r . -id "src, lib" -xd "test"
pcat -r . -i "*.c, *.h"        # all C sources and headers recursively
```

### Include / Exclude Files

```bash
pcat * -x "*.tmp, *.log, DUMP_*"
pcat -r . -x "*test*" -i "*.py"
pcat .bashrc .profile -x ".bashrc"   # exclude .bashrc itself
pcat -r . -i "Makefile"             # only files named Makefile
```

### Size, Line & Character Limits

```bash
pcat --min-size 10k *.log
pcat --max-size 2m --min-lines 5 *.json
pcat --min-size 1.5kb --max-size 1.5mb -r .
pcat --min-lines 100 --max-lines 5000 --min-chars 1000 --max-chars 100000 huge.log
pcat --max-size 100b *               # only tiny files
pcat --min-size 1g --max-size 2g *   # files between 1 and 2 GiB
```

### Follow Mode & Paging

```bash
pcat -f /var/log/syslog
pcat -f -w 200 app.log
pcat -p big_report.txt
pcat -p --pager-cmd "less -R -X" results.csv
pcat -n -c -p --min-size 1m *.sql
```

### Complex Real‑World Examples

```bash
# 1. Show all Python files (recursively) except tests, at least 1KB, with line numbers and colour, paged.
pcat -n -c -r . -i "*.py" -x "*test*" --min-size 1k -p

# 2. Follow a log, truncate long lines, colour the initial message.
pcat -f -w 120 -c /var/log/application.log

# 3. Recursively find all config files (.conf, .ini, .cfg) between 100 bytes and 50KB, exclude backup files.
pcat -r /etc -i "*.conf, *.ini, *.cfg" -x "*.bak, *~" --min-size 100b --max-size 50k

# 4. Only show files with exactly 100 to 200 lines, and at least 5000 characters.
pcat --min-lines 100 --max-lines 200 --min-chars 5000 *.txt

# 5. Recursively process a project, skipping .git and build/, only including .c and .h, with size >= 1KB and <= 100KB.
pcat -r . -xd ".git, build" -i "*.c, *.h" --min-size 1k --max-size 100k
```

---

## Bugs, Limitations & Future‑Proofing

- **Binary detection** reads only the first 1024 bytes; a file that starts with text but contains binary later will still be considered text (which is acceptable).
- **Performance**: For huge files (e.g., >1GB), reading the entire file into memory (`f.read()`) can cause high memory usage. This is a design choice for simplicity; do not use `pcat` on multi‑gigabyte files unless you have sufficient RAM.
- **Character counting** uses Python’s `len(content)` after decoding, which counts Unicode code points. This may differ from byte counts for non‑ASCII text.
- **2038 Problem**: `pcat` does not store timestamps, so it is immune to the Year 2038 issue. It will work in 2052 and beyond.
- **5TB files**: The code supports tebibytes (`t`, `tb`, `tib`), but processing such files is impractical on current hardware. Future users may thank you.
- **No symlink following control**: Symbolic links are followed by `os.walk`. Use `-xd` to skip directories that are symlinks if needed (by name).

---

## License

This project is licensed under the [GNU General Public License v3.0 (GPLv3)](https://www.gnu.org/licenses/gpl-3.0.html).
