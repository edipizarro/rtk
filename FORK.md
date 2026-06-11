# Fork notes

This is edipizarro's fork of [rtk-ai/rtk](https://github.com/rtk-ai/rtk) carrying local
patches to the token-savings accounting. Patches live on the `edipizarro`
branch as small, isolated commits so upstream merges stay trivial; `develop` is kept
pristine and tracks upstream.

## Local patches

1. **Honest `git branch` savings baseline** (`src/cmds/git/git.rs`) — rtk implicitly runs
   `git branch -a`; savings were measured against all remote refs the user never asked
   for, inflating `rtk gain` by orders of magnitude on repos with many remote branches.
2. **Real token counting** (`src/core/tracking.rs`, `Cargo.toml`) — `estimate_tokens`
   uses the o200k_base BPE tokenizer (tiktoken-rs) for texts ≥1000 chars; the chars/4
   heuristic remains for small texts (error is negligible there) and as fallback.
3. **Verbatim commands** (`src/cmds/git/git.rs`, `src/cmds/git/gh_cmd.rs`,
   `src/cmds/system/ls.rs`) — wrapped commands run exactly as written: no implicit
   `git branch -a`, no implicit `git log -10`/`--no-merges`, no forced
   `gh run list --limit 10`, no forced `ls -a`. Unbounded `git log` output is capped
   at display time with a visible `[+N more commits]` note instead. Format-only flags
   (`--no-color`, `--format=json`) are kept — they affect parsing, not scope.

## Syncing with upstream

```bash
git fetch upstream
git checkout develop && git merge --ff-only upstream/develop
git checkout edipizarro && git merge develop
cargo build --release && cargo test --release tracking::
```

## Installing the patched binary

```bash
cargo build --release
cp target/release/rtk /opt/homebrew/bin/rtk
```

`brew link --overwrite rtk` restores the Homebrew binary; note `brew upgrade rtk`
silently replaces the patched one.
