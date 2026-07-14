# Day 3: Shell Basics — grep, find, pipes & redirection

## Concepts

### Redirection
- `>`  — overwrite file with output
- `>>` — append output to file
- `<`  — feed file in as input
- `2>` — redirect stderr (errors) separately from stdout
- `&>` — redirect both stdout and stderr together

Separating stdout/stderr matters in real ops work — you often want
error output isolated from normal output during log triage.

### Pipes (`|`)
Feeds the output of one command in as the input of the next.
This is the core idea behind almost every useful one-liner in shell:
chain small, single-purpose commands together instead of writing one
big complicated command.

### grep — search text
| Flag | Meaning |
|------|---------|
| `-i` | case-insensitive |
| `-r` | recursive (search directories) |
| `-n` | show line numbers |
| `-v` | invert match (show non-matching lines) |
| `-c` | count matching lines instead of printing them |
| `-E` | extended regex |

### find — search the filesystem
- `find /path -name "*.txt"` — match by filename pattern
- `find /path -type f -mtime -1` — regular files modified in the last day
- `find /path -size +10M` — files larger than 10MB
- Can combine with `-exec` to act on results directly

### Combo tools
`wc`, `sort`, `uniq`, `cut`, `head`/`tail` — used together with pipes to
summarize data, e.g. counting, deduplicating, or slicing fields out of
structured text like logs.

## Hands-on drill

Generated a fake `app.log` with 100 lines (50 ERROR + 50 INFO) spread
across 5 fake servers and ~50 hours of timestamps, using a for-loop and
`date -d "-$i hours"`.

### Commands run and what they do

```bash
grep "ERROR" app.log | head -5
# preview first 5 ERROR lines

grep -c "ERROR" app.log
# count of ERROR lines -> 50

grep "server3" app.log | grep "ERROR"
# chained grep: filter to server3 lines, then filter those to ERROR only

find ~/shell-lab -name "*.log"
# find all .log files in the shell-lab directory

find ~/shell-lab -type f -mtime -1
# find files modified in the last 24 hours

cat app.log | grep "ERROR" | cut -d' ' -f1 | sort | uniq -c
# pipeline: filter ERRORs -> extract date field -> sort -> count
# occurrences per date. Classic log-summarizing one-liner.
```

## Key learnings

- `uniq -c` only collapses **adjacent** duplicate lines — `sort` must
  come before it, or duplicates that aren't next to each other won't
  be counted together.
- `cut -d' ' -f1` splits a line on spaces and returns only the first
  field — useful for pulling out a single column (like a date) from
  structured log lines.
- Chaining greps (`grep X file | grep Y`) is a simple, readable way to
  apply multiple filters instead of one complex regex.
- `find` searches the filesystem (names, types, dates, sizes) — different
  job from `grep`, which searches file *contents*.

## Takeaway

The `filter | extract | sort | count` pattern
(`grep | cut | sort | uniq -c`) is one of the most common shell
one-liners in real SRE/ops work for summarizing logs at a glance,
without writing a script.
