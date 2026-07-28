# 02: Create the Rust workspace

## Result

You will create a Git repository with three focused crates:

```text
storyforge-tui/
├── .gitignore
├── Cargo.lock
├── Cargo.toml
├── README.md
├── rust-toolchain.toml
├── campaigns/
│   └── academy-demo/
├── crates/
│   ├── storyforge-content/
│   ├── storyforge-core/
│   └── storyforge-tui/
```

`storyforge-core` has no terminal or filesystem dependency. `storyforge-content` loads and validates campaigns. `storyforge-tui` is the executable.

## Create the repository

Choose a parent folder, then run:

```powershell
cargo new storyforge-tui --vcs git
Set-Location storyforge-tui
Remove-Item -LiteralPath src -Recurse
Remove-Item -LiteralPath Cargo.toml
New-Item -ItemType Directory -Path crates,campaigns | Out-Null
cargo new crates/storyforge-core --lib
cargo new crates/storyforge-content --lib
cargo new crates/storyforge-tui --bin
```

The removal targets are the new repository's generated `src` and `Cargo.toml`, not a parent directory. Confirm `Get-Location` ends with `storyforge-tui` before running the two removal commands.

## Root `Cargo.toml`

Create the workspace-root `Cargo.toml`:

```toml
[workspace]
members = ["crates/*"]
resolver = "3"
default-members = ["crates/storyforge-tui"]

[workspace.package]
version = "0.1.0"
edition = "2024"
rust-version = "1.96"
license = "MIT OR Apache-2.0"
repository = "https://github.com/toonchavez8/storyforge-tui"

[workspace.dependencies]
serde = { version = "1", features = ["derive"] }
thiserror = "2"

[workspace.lints.rust]
future_incompatible = "warn"
missing_docs = "warn"
nonstandard_style = "deny"
unsafe_code = "forbid"

[workspace.lints.clippy]
all = { level = "warn", priority = -1 }
pedantic = { level = "warn", priority = -1 }
redundant_clone = "deny"
large_enum_variant = "warn"
needless_collect = "warn"
```

Use the real repository URL if it differs.

## Crate manifests

`crates/storyforge-core/Cargo.toml`:

```toml
[package]
name = "storyforge-core"
version.workspace = true
edition.workspace = true
rust-version.workspace = true
license.workspace = true
repository.workspace = true

[dependencies]
serde.workspace = true
thiserror.workspace = true

[lints]
workspace = true
```

`crates/storyforge-content/Cargo.toml`:

```toml
[package]
name = "storyforge-content"
version.workspace = true
edition.workspace = true
rust-version.workspace = true
license.workspace = true
repository.workspace = true

[dependencies]
serde.workspace = true
storyforge-core = { path = "../storyforge-core" }
thiserror.workspace = true

[lints]
workspace = true
```

`crates/storyforge-tui/Cargo.toml`:

```toml
[package]
name = "storyforge-tui"
version.workspace = true
edition.workspace = true
rust-version.workspace = true
license.workspace = true
repository.workspace = true

[dependencies]
storyforge-content = { path = "../storyforge-content" }
storyforge-core = { path = "../storyforge-core" }

[lints]
workspace = true
```

## Pin the toolchain

Create `rust-toolchain.toml`:

```toml
[toolchain]
channel = "1.96.0"
components = ["clippy", "rustfmt"]
profile = "minimal"
```

When intentionally upgrading Rust, change this file in its own pull request and run the entire workspace check.

## Ignore generated and private files

Create `.gitignore`:

```gitignore
/target/
.storyforge.local.toml
/wizarding-world-private/
/campaigns-private/
*.log
*.tmp
*.bak
.env
.idea/
.vscode/settings.json
```

Create the public campaign directory:

```powershell
New-Item -ItemType Directory -Path campaigns/academy-demo | Out-Null
```

The last two ignored directory names are defenses against an accidental copy. Do not create either directory. Chapter 19 creates `wizarding-world-private` as a separate sibling Git repository.

## Crate documentation

Replace `storyforge-core/src/lib.rs`:

```rust
//! Deterministic rules and state transitions for Storyforge campaigns.

/// Returns the engine name used in diagnostics.
#[must_use]
pub const fn engine_name() -> &'static str {
    "Storyforge"
}

#[cfg(test)]
mod tests {
    use super::engine_name;

    #[test]
    fn engine_name_should_be_stable() {
        assert_eq!(engine_name(), "Storyforge");
    }
}
```

Replace `storyforge-content/src/lib.rs`:

```rust
//! Campaign loading and validation for Storyforge.

/// Returns the current content schema version.
#[must_use]
pub const fn schema_version() -> u32 {
    1
}
```

Replace `storyforge-tui/src/main.rs`:

```rust
fn main() {
    println!(
        "{} content schema {}",
        storyforge_core::engine_name(),
        storyforge_content::schema_version()
    );
}
```

## Root README

Create a short code-repository `README.md` containing:

- The one-sentence pitch.
- Current milestone.
- `cargo run -p storyforge-tui`.
- The three-crate boundary.
- Public `academy-demo` and sibling private-pack policy.
- The Reconstruction, Fracture, and Convergence arc roadmap.
- License status.

Keep the architecture section consistent with the documentation README in this vault.

## Verify

```powershell
cargo fmt --all
cargo check --workspace --locked
cargo clippy --workspace --all-targets --all-features --locked -- -D warnings
cargo test --workspace --all-features --locked
cargo run -p storyforge-tui
git status --short
```

Expected program output:

```text
Storyforge content schema 1
```

`git status` must contain only the public workspace files created in this chapter.

## Common mistakes

- A member crate forgot `[lints] workspace = true`.
- The executable used an underscore crate name in `cargo run`; package names use hyphens.
- The root still has the generated single-crate `src`.
- A private campaign was copied into the public workspace.

## Acceptance check

- All three crates compile.
- Workspace tests pass.
- Clippy reports no warnings.
- The binary prints engine and schema names.
- `.storyforge.local.toml` and common accidental private-directory names are ignored.
- No private campaign directory exists inside the public repository.

## Suggested commit

```powershell
git add .
git commit -m "Create the Storyforge Rust workspace"
```
