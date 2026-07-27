# 00: How to use this guide

## What this guide assumes

You have written little or no game code. You can use a terminal, edit a text file, and follow a compiler error to a line number. Rust ownership, terminal rendering, save migrations, and game loops will be introduced when they become useful.

Build the chapters in order. Each chapter starts from the previous checkpoint and ends with something you can run or verify.

## The project you will create

Create the Rust repository outside this documentation folder. The examples use:

```text
storyforge-tui/
```

The documentation can stay inside the Obsidian vault. The code repository should have its own Git history so releases, issues, and CI remain focused.

## Chapter rhythm

Every chapter follows the same loop:

1. Read the result and acceptance check.
2. Create a branch.
3. Make the smallest listed change.
4. Run the check immediately.
5. Fix warnings before moving on.
6. Play the new behavior manually.
7. Commit the checkpoint.

Use one branch per chapter while learning:

```powershell
git switch -c guide/03-first-terminal
```

After the checkpoint passes:

```powershell
git add .
git commit -m "Build the first safe terminal loop"
git switch main
git merge --ff-only guide/03-first-terminal
```

If you prefer committing directly to `main` in a private learning repository, that is acceptable. Keep the checkpoint commits.

## Milestones

| Chapters | Milestone |
| --- | --- |
| 00-03 | M0: the terminal application boots and exits safely |
| 04-07 | M1: responsive shell and valid character creation |
| 08-11 | M2: complete playable MVP with save and resume |
| 12-15 | M3: explorable campaign alpha |
| 16-20 | M4: campaign-scale engine beta |
| 21 | M5: native npm release |
| 22 | Post-v1 growth rules |

Tag milestones:

```powershell
git tag -a mvp-0.1.0 -m "Playable MVP"
git log --oneline --decorate -5
```

Do not publish the tag until chapter 21 configures release automation.

## Commands you will use often

```powershell
cargo fmt --all -- --check
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo test --workspace --doc --locked
cargo run -p storyforge-tui -- doctor
cargo run -p storyforge-tui -- validate --pack campaigns/academy-demo
```

During the first dependency setup, `Cargo.lock` may change. After that, use `--locked` in routine checks and CI.

## How to read compiler errors

Start with the first error, not the last. Rust often reports several later errors caused by one missing type or bad borrow.

Use:

```powershell
rustc --explain E0382
```

Replace `E0382` with the code from the compiler.

Do not fix an ownership error by adding `.clone()` everywhere. Ask who should own the value and whether the function only needs `&T` or `&mut T`.

## Error policy

Production paths follow these rules:

- Fallible functions return `Result<T, E>`.
- Library errors use `thiserror`.
- The binary uses `color-eyre` to add context and display a report.
- `?` propagates errors.
- `unwrap` and `expect` are restricted to tests and impossible compile-time fixtures.
- Player-recoverable failures become UI states.

## Documentation policy

- Public crates start with `//!` documentation.
- Public types and functions use `///`.
- `Result` APIs document their error cases.
- Examples in public docs should compile as doc tests.
- Comments explain why a surprising choice exists.
- Work that is not implemented belongs in an issue, not an untracked comment.

## Content policy

Build and release only original `academy-demo` content. Keep the private campaign in:

```text
campaigns-private/wizarding-world-private/
```

Add that directory to `.gitignore` before creating it. Never test release archives by assumption; inspect them.

## When a chapter feels too large

Split your work session, not the architecture. Stop after a passing subsection, commit with a clear message, and continue on the same chapter branch.

Do not start the next chapter while the current acceptance test is failing.

## Acceptance check

Before chapter 01:

- You know where the new code repository will live.
- You understand that this vault folder contains documentation, not the code repository.
- You accept that `academy-demo` is public and the wizarding campaign is local.
- You will keep milestone commits and run the full check suite.

## Suggested commit

No code exists yet, so there is no commit for chapter 00.
