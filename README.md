# jj-exec

`jj-exec` is a small utility for Jujutsu that behaves a bit like `git rebase --exec`.
It walks a revset from older revisions to newer ones, does `jj edit` on each revision, and
runs a user-provided command at each step.

If the command fails for any revision, `jj-exec` stops immediately and leaves the working
copy on the failing revision. If the command succeeds for every revision, `jj-exec` restores
the working copy to whatever `@` was when the script started.

## Usage

Run the script directly:

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

## Add as a `jj` alias

First, make sure `jj-exec` is available on your `PATH`.

Then open your user config with the officially supported jj command:

```bash
jj config edit --user
```

Paste this into the config file that opens:

```toml
[aliases]
exec = ["util", "exec", "--", "jj-exec"]
```

If you want to see which config files jj uses, run:

```bash
jj config path --user
```

Then you can run:

```bash
jj exec -r 'foo::bar' -- make test
jj exec -r 'main..@' -- cargo test
```

## Help

To see the built-in usage text:

```bash
jj-exec --help
```
