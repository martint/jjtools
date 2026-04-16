# jjtools

A small collection of utility scripts for [Jujutsu](https://github.com/jj-vcs/jj).

- [`jj-exec`](#jj-exec) — run a command on each revision in a revset
- [`jj-untrack`](#jj-untrack) — untrack tracked files that match `.gitignore` patterns

Make the scripts available on your `PATH`, or add them as `jj` aliases (see each section).

---

## jj-exec

`jj-exec` behaves a bit like `git rebase --exec`. It walks a revset from older revisions
to newer ones, does `jj edit` on each revision, and runs a user-provided command at each
step.

If the command fails for any revision, `jj-exec` stops immediately and leaves the working
copy on the failing revision. If the command succeeds for every revision, `jj-exec` restores
the working copy to whatever `@` was when the script started.

### Usage

```bash
jj-exec -r '<revset>' -- <command>
```

Examples:

```bash
jj-exec -r 'foo::bar' -- make test
jj-exec -r 'root()..@' -- ./lint.sh
jj-exec -r 'main..@' -- bash -lc 'cargo test'
```

Notes:

- Revisions are visited in older-to-newer order.
- The revset must resolve to at least one revision.
- The command is run exactly as provided after `--`.
- On success, the original working copy is restored.
- On failure, the script exits non-zero and leaves you at the failing revision.

### Add as a `jj` alias

```toml
[aliases]
exec = ["util", "exec", "--", "jj-exec"]
```

Then:

```bash
jj exec -r 'foo::bar' -- make test
```

---

## jj-untrack

`jj-untrack` reads every `.gitignore` in the workspace, finds files that jj is currently
tracking but that match a gitignore pattern, and calls `jj file untrack` on them. This is
useful after adding new entries to `.gitignore` for files that were previously tracked.

### Usage

```bash
jj-untrack [--dry-run] [PATH...]
```

Examples:

```bash
jj-untrack                    # scan the whole working copy
jj-untrack --dry-run          # preview without making changes
jj-untrack src/ tests/        # only consider these paths
```

Notes:

- All `.gitignore` files under the workspace root are processed (excluding
  `.jj/` and `.git/`). Each pattern applies only to files under the directory
  containing its `.gitignore`, matching standard gitignore scoping.
- Supports common gitignore syntax: globs, anchored patterns (`/foo`), directory
  patterns (`build/`), and negation (`!keep.me`).
- Only files that `jj file list` reports as tracked are candidates — untracked
  files are left alone.

### Add as a `jj` alias

```toml
[aliases]
untrack-ignored = ["util", "exec", "--", "jj-untrack"]
```

Then:

```bash
jj untrack-ignored --dry-run
```

---

## Help

Each script has built-in `--help`:

```bash
jj-exec --help
jj-untrack --help
```

## Installing

Clone this repo and put it on your `PATH`:

```bash
export PATH="/path/to/jjtools:$PATH"
```
