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
**Learned:** Two most basic navigation/reading commands in Linux.

## Level 1 → 2
**Goal:** Read a file named `-` (a single dash) in the home directory.
**Concepts:** Handling filenames that clash with command syntax; relative paths.
**Command used:**
- `cat ./-` — the `./` prefix tells `cat` that `-` is a filename in the current directory, not an option/flag.
**Learned:** A lone `-` is often interpreted as stdin by commands, so `./` (or a full path) is needed to reference it as a file.

## Level 2 → 3
**Goal:** Read a file named `--spaces in this filename--`.
**Concepts:** Handling filenames containing spaces and leading dashes.
**Command used:**
- `cat "./--spaces in this filename--"` — quotes handle the spaces (so the shell reads it as one filename), and `./` ensures the leading `--` is treated as part of the filename rather than command options.
**Learned:** How the shell parses spaces and dashes, and multiple ways to reference awkwardly-named files.

---

*Progress ongoing — updated as I work through the levels.*
