# OverTheWire: Bandit — Write-up

The Bandit wargame teaches Linux command-line basics through a series of levels, each requiring a password, found using terminal skills. Below I document the concepts and commands I learned at each level.

> Note: Passwords/solutions are intentionally omitted per OverTheWire's guidance — this log focuses on the skills and commands, not the answers.

---

## Level 0 → 1
**Goal:** Find the password for the next level, stored in a file in the home directory.
**Concepts:** Listing directory contents, reading files.
**Commands used:**
- `ls` — listed files in the home directory (found a file named `readme`)
- `cat readme` — printed the file's contents to reveal the password
**What I Learned:**
  Two most basic navigation/reading commands in Linux.

## Level 1 → 2
**Goal:** Read a file named `-` (a single dash) in the home directory.
**Concepts:** Handling filenames that clash with command syntax; relative paths.
**Command used:**
- `cat ./-` — the `./` prefix tells `cat` that `-` is a filename in the current directory, not an option/flag.
**What I Learned:**
  A lone `-` is often interpreted as stdin by commands, so `./` (or a full path) is needed to reference it as a file.

## Level 2 → 3
**Goal:** Read a file named `--spaces in this filename--`.
**Concepts:** Handling filenames containing spaces and leading dashes.
**Command used:**
- `cat "./--spaces in this filename--"` — quotes handle the spaces (so the shell reads it as one filename), and `./` ensures the leading `--` is treated as part of the filename rather than command options.
**What I Learned:**
  How the shell parses spaces and dashes, and multiple ways to reference awkwardly-named files.

## Level 3 → 4

**Goal:** Password stored in a hidden file inside the `inhere/` directory.

**Commands used:** `ls -a`, `find`, `cat`

**What I learned:**
- Files beginning with `.` are hidden from `ls`.
- The file `...Hiding-From-You` deliberately mimics the `.` and `..` directory
  entries, so it's easy to overlook.
- `ls -a` reveals hidden files; `find .` also lists them by path.
- Leading dots don't need a `./` prefix for `cat` — that's only required when a
  filename starts with `-`.

## Level 4 → 5

**Goal:** Password stored in the only human-readable file in `inhere/`.

**Commands used:** `file`, `cat`

**What I learned:**
- Filenames starting with `-` break commands, because the shell reads them as
  options (e.g. `file -file00`). Prefixing with `./` fixes it — `file ./*` — or
  use `--` to signal the end of options.
- `file` identifies content by inspecting the actual bytes (magic numbers), not
  the extension or name. `file ./*` narrowed ten files down to the one reporting
  `ASCII text`.
- This ties into the `reset` tip: `cat`-ing a binary file dumps control
  characters that garble the terminal, which `file` helps you avoid.

## Level 5 → 6

**Goal:** Password in a file under `inhere/` that is human-readable, exactly
1033 bytes, and not executable.

**Commands used:** `find`

**What I learned:**
- `find` chains test flags to filter on multiple properties at once:
  `find . -size 1033c ! -executable`
- `-size 1033c` matches an exact byte count. The `c` suffix means bytes —
  without it, the number is read as 512-byte blocks. Other suffixes: `k`
  (kilobytes), `M` (megabytes).
- `!` negates a test, so `! -executable` excludes executable files.
- `find` recurses the entire directory tree, searching every `maybehere*`
  subfolder in one command — much faster than inspecting each by hand.

## Level 6 → 7

**Goal:** Password stored somewhere on the server — owned by user `bandit7`,
owned by group `bandit6`, and 33 bytes in size.

**Commands used:** `find`

**What I learned:**
- `find` can filter by ownership: `-user bandit7` matches the owning user,
  `-group bandit6` the owning group. Combined with `-size 33c` for exact bytes:
  `find / -user bandit7 -group bandit6 -size 33c`
- Searching from `/` means `find` hits directories the user can't read and
  prints "Permission denied" errors to stderr.
- `2>/dev/null` redirects stderr (file descriptor 2) to the null device,
  discarding the error noise so only real results show on stdout.
- Reinforced the distinction between stdout (file descriptor 1) and stderr
  (file descriptor 2) as separate output streams.

## Level 7 → 8

**Goal:** Password stored in `data.txt`, next to the word `millionth`.

**Commands used:** `grep`

**What I learned:**
- `grep` searches *inside* file contents, unlike `find` which filters files by
  their metadata: `grep millionth data.txt`
- `grep pattern file` prints every line matching the pattern. The file had
  thousands of word/value pairs, but only one line contained `millionth`.
- Key distinction: `find` locates files by properties; `grep` locates text
  within files. Both are fundamental and often piped together.

## Level 8 → 9

**Goal:** Password is the only line in `data.txt` that occurs exactly once.

**Commands used:** `sort`, `uniq`, `|` (pipe)

**What I learned:**
- The pipe `|` feeds one command's stdout into the next command's stdin,
  chaining small tools into a filter: `sort data.txt | uniq -u`
- `uniq` only detects duplicates on *adjacent* lines, so `sort` must run first
  to group identical lines together.
- `uniq -u` prints only lines that appear exactly once. Related flags:
  `-d` (only duplicated lines), `-c` (prefix each line with its count).
- This is the core Unix philosophy — small single-purpose tools composed with
  pipes to achieve what neither does alone.

## Level 9 → 10

**Goal:** Password is in `data.txt`, among a few human-readable strings,
preceded by several `=` characters.

**Commands used:** `strings`, `grep`, `|` (pipe)

**What I learned:**
- `strings` scans a binary file and prints only runs of printable characters
  (4+ by default), discarding non-printable bytes — a safe way to read text
  from a binary file without garbling the terminal.
- Piped `strings` into `grep` to filter the extracted text:
  `strings data.txt | grep ===`
- Reinforced tool composition: extract with `strings`, filter with `grep` —
  the same pipe pattern from the previous level applied to binary data.

  ## Level 10 → 11

**Goal:** Password stored in `data.txt` as base64-encoded data.

**Commands used:** `base64`

**What I learned:**
- `base64 -d` decodes base64 data (`-d` = decode; without it, `base64`
  encodes): `base64 -d data.txt`
- Base64 is an *encoding*, not encryption. It represents binary data using 64
  printable ASCII characters (A–Z, a–z, 0–9, `+`, `/`), letting arbitrary bytes
  pass safely through text-only channels.
- It's fully reversible with no key, so it provides zero security — it only
  obscures. Recognisable by its limited character set and trailing `=` padding.

  ## Level 11 → 12

**Goal:** Password in `data.txt`, with all letters rotated 13 positions (ROT13).

**Commands used:** `tr`, `cat`

**What I learned:**
- `tr` (translate) maps characters from one set to another by position:
  `cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'`
- The mapping shifts each letter 13 forward. Uppercase and lowercase are handled
  as separate ranges so case is preserved.
- ROT13 is a Caesar cipher shifting by 13. Since 13 is half of 26, the same
  operation both encodes and decodes — it's its own inverse.
- Like base64, ROT13 is obfuscation, not encryption — no key, no security.
- `tr` also does case conversion, character deletion (`-d`), and squeezing
  repeated characters (`-s`).

  ## Level 12 → 13

**Goal:** Password in `data.txt` — a hexdump of a file compressed through many
nested layers. Reverse the hexdump, then decompress every layer.

**Commands used:** `xxd`, `file`, `gzip`, `bzip2`, `tar`, `mkdir`, `cp`, `mv`

**What I learned:**
- Work in a scratch dir (`mktemp -d` or `mkdir /tmp/<name>`); copy the data in
  and operate on the copy.
- `xxd -r` reverses a hexdump back into raw binary.
- Core loop: `file <data>` to identify the type by magic bytes → apply the
  matching tool → re-inspect. Repeat until `file` reports ASCII text, then
  `cat`.
- The stack ran ~8 layers deep, mixing three formats:
  - gzip → `mv` to `.gz`, then `gzip -d`
  - bzip2 → `mv` to `.bz2`, then `bzip2 -d`
  - tar → `mv` to `.tar`, then `tar xf` (a container, not compression;
    extracts a new file to inspect next)
- gzip/bzip2 refuse to run without the expected extension, so `mv` is required
  each time — even though `file` proves the real type regardless of name. tar
  ignores extensions.
- gzip's `file` output shows `original size modulo 2^32`: a multiple of 512
  (e.g. 20480 = 40×512) hints a tar is underneath; a tiny value (49) signals
  the final payload.
- The "inspect → act → re-inspect" discipline is the real-world method for
  unpacking unknown files in forensics and malware triage.

---

*Progress ongoing — updated as I work through the levels.*
