# 01: Set up the development environment

## Result

You will have a verified Rust, Node, npm, and Git toolchain. You will also know which commands diagnose a missing component.

The examples were verified with:

```text
rustc 1.96.0
cargo 1.96.0
node 26.3.0
npm 11.17.0
git 2.54.0.windows.1
```

Use the current stable Rust toolchain if these versions have moved forward. The repository will pin its chosen version in chapter 02.

## Install Rust

Install Rust through `rustup` from <https://rustup.rs/>.

On Windows, use the MSVC toolchain and install the Visual Studio Build Tools if `rustup` requests them. Select the Desktop development with C++ workload.

Verify:

```powershell
rustup show active-toolchain
rustc --version
cargo --version
```

The Windows result should contain:

```text
stable-x86_64-pc-windows-msvc
```

Add the required components:

```powershell
rustup component add rustfmt clippy
cargo fmt --version
cargo clippy --version
```

## Install Node and npm

Node is not part of the game runtime. It is used to test the final npm installer and `npx` experience.

Install the current Node LTS or newer stable release from <https://nodejs.org/>.

Verify:

```powershell
node --version
npm --version
```

## Install Git

Install Git from <https://git-scm.com/>.

Verify:

```powershell
git --version
git config --get user.name
git config --get user.email
```

If identity is empty, set your own name and email:

```powershell
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Use the email connected to the account that will host the repository.

## Editor

Visual Studio Code with rust-analyzer is the easiest documented path:

1. Install Visual Studio Code.
2. Install the `rust-analyzer` extension.
3. Enable format on save for Rust.
4. Keep the integrated terminal set to PowerShell on Windows.

Do not install the older `rust-lang.rust` extension beside rust-analyzer.

## Verify a compiler round trip

Use a temporary directory outside the future repository:

```powershell
$sandbox = Join-Path $env:TEMP "storyforge-rust-check"
cargo new $sandbox
Set-Location $sandbox
cargo run
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings
cargo test
Set-Location ..
```

Delete the temporary directory through File Explorer after the checks if you no longer need it.

## Terminal checks

The game needs:

- At least 80 columns and 24 rows.
- ANSI escape support.
- Raw input support.
- UTF-8 for the standard theme.

Windows Terminal, recent PowerShell terminals, iTerm2, Terminal.app, and common Linux terminals are suitable. The game will include ASCII and no-color fallbacks.

## Troubleshooting

### `link.exe` is missing

Install the Visual Studio Build Tools with MSVC and a Windows SDK, then restart the terminal.

### `cargo` is not recognized

Restart the terminal after `rustup`. Check that `%USERPROFILE%\.cargo\bin` is in `PATH`.

### PowerShell blocks a script

Do not weaken the machine-wide execution policy for this project. Use signed installers or invoke the specific approved script according to its documentation.

### Unicode borders look broken

Choose a terminal font with box-drawing characters. Cascadia Mono is a good Windows default. The game will still provide ASCII borders.

## Acceptance check

Run:

```powershell
rustc --version
cargo fmt --version
cargo clippy --version
node --version
npm --version
git --version
```

Every command must exit successfully.

## Suggested commit

There is no project repository yet. Continue to chapter 02.

