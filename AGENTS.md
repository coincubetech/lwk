# AGENTS.md - Guidelines for Coding Agents

This file contains guidelines for agentic coding agents working in the LWK (Liquid Wallet Kit) repository.

## Instruction Priority and Conflict Resolution

When multiple instructions apply, follow this priority order:
1. Safety, security, and data integrity
2. Correctness and testability
3. Repository conventions (style, structure, commit format)
4. Performance and speed of execution

If instructions conflict, follow the higher-priority rule and document the tradeoff briefly in your final message.

## Project Overview

LWK is a Rust workspace containing libraries for Liquid wallets. It consists of multiple crates:
- `lwk_wollet` - Watch-only wallets based on CT descriptors
- `lwk_signer` - Signing operations
- `lwk_jade` / `lwk_ledger` - Hardware wallet integrations
- `lwk_bindings` - UniFFI bindings for Python/Kotlin/Swift/C#
- `lwk_common` - Shared utilities
- `lwk_cli` - Command line interface
- `lwk_wasm` - WebAssembly bindings
- and others

## Build Commands

```bash
# Build the entire workspace
cargo -q build

# Build specific crate
cargo -q build -p lwk_wollet

# Cross-compile for WASM
cargo -q check --target wasm32-unknown-unknown -p lwk_wollet
```

## Test Commands

```bash

# Run all tests, including integration tests that spawn executables and Docker containers
cargo -q test

# Run all unit tests, much faster; every unit test should run in less than a second
cargo -q test --lib

# Run tests for specific package
cargo -q test -p lwk_wollet

# Run a single test
cargo -q test -p lwk_wollet test_name_here

# Run bindings tests
cargo -q test -p lwk_bindings --features foreign_bindings

# Build tests without running
cargo -q test --no-run
```

## Lint Commands

```bash
# Format code
cargo -q fmt

# Check formatting
cargo -q fmt --check

# Run clippy
cargo -q clippy --all-features --all-targets -- -D warnings

# Security audit
cargo audit --deny yanked

# Generate documentation
cargo -q doc --no-deps -p lwk_wollet --all-features
RUSTDOCFLAGS="-D warnings --cfg docsrs" cargo +nightly -q doc --all-features --no-deps
```

## Just Commands

The project uses `just` for complex workflows, some examples include:

```bash
just --list                           # List all recipes
just build-bindings-lib              # Build liblwk.so
just python-test-bindings            # Test Python bindings
just swift                           # Build Swift framework (macOS only)
just android                         # Build Android libs
just kotlin                          # Generate Kotlin bindings
just mdbook                          # Build documentation
```

if you're adding a complex flow, please add a just command.

## Code Style Guidelines

### Rust Version
- Rust toolchain: **1.85.0** (specified in `rust-toolchain.toml`)
- Edition: **2021**

### Imports
- Group imports: std lib → external crates → workspace crates → local modules
- Use `use crate::` for local module imports
- Re-export commonly used types at crate root (see `lwk_wollet/src/lib.rs`)

### Error Handling
- Use `thiserror` for error enums with `#[derive(thiserror::Error, Debug)]`
- Use `#[error(transparent)]` for wrapping other errors
- Use `#[from]` for automatic conversion
- Return `Result<T, Error>` from public functions
- Deny `unwrap()` in non-test code: `#![cfg_attr(not(test), deny(clippy::unwrap_used))]`

### Naming Conventions
- Types: `PascalCase` (e.g., `WolletDescriptor`, `TxBuilder`)
- Functions/Methods: `snake_case` (e.g., `full_scan_with_electrum_client`)
- Constants: `SCREAMING_SNAKE_CASE` (e.g., `LIQUID_SOCKET`)
- Feature flags: `snake_case` (e.g., `foreign_bindings`, `test_emulator`)

### Documentation
- Require docs on public items: `#![warn(missing_docs)]`
- Use `//!` for module-level documentation
- Document all public functions with examples where appropriate
- Mark feature-gated items: `#[cfg_attr(docsrs, feature(doc_cfg))]`

### Type Patterns
- Use `pub type` aliases for common types (e.g., `pub type BlindingPublicKey = elements::secp256k1_zkp::PublicKey`)
- Prefer `&str` over `&String` in function parameters
- Use builder patterns for complex construction (e.g., `TxBuilder`, `WolletBuilder`) and for constructor with large/unknown number of parameters

### Commit Messages
Format: `context: <description>`
- Context: crate/directory name or `ci`/`fix`/`feat`/`docs`/`refactor`
- For crates, do not include `lwk_` for example `lwk_wollet:` use just `wollet:`
- Breaking changes: append `!` after context
- Title: max 50 chars, imperative mood, no period
- Body: blank line after title, explain "why", use bullet points

### Feature Flags
- Default features should be minimal but functional
- Group related features logically
- Use `#[cfg(feature = "...")]` for conditional compilation
- Document feature requirements in crate README

### Testing
- Unit tests in `#[cfg(test)]` modules
- Integration tests in `tests/` directory
- Use `lwk_test_util` for shared test utilities
- E2E tests require Docker environment (Jade emulator, etc.)
- Possible panics, unchecked index array access, unwraps, are allowed in testing code.
- Do not suggest safety improvements (e.g., bounds checks, error handling) for these in tests unless they cause compile/run failures.

## Additional Resources
- CI Configuration: `.gitlab-ci.yml` (GitLab) and `.github/workflows/` (GitHub)
- Nix development: `nix develop .` for reproducible environment

## Shared startup and integration branches (owner instruction, 2026-11-04)

This repository belongs to COINCUBE. Before substantive work, read the owning
workspace's AGENTS.md and required company context, then this repository's
instructions. Load `tirith-tech/agent-ops` from one verified current reviewed
`main` commit, reading committed AGENTS.md, README.md, contracts/task.md,
docs/task-routing.md, workflows/delivery.md, workflows/pr-review-repair.md and
workflows/batched-ci.md. Read its owning company profile when available; never
substitute another company's brain. Use an existing verified checkout or fetch
the committed files from GitHub without switching another worker's checkout.
Record the full policy SHA and startup receipt in the task's existing issue/PR.
A link alone does not prove the policy was read. If unavailable, report that
before dependent work.

Robert adopted integration-branch batching here. Verify the current primary
branch (currently `master`). Honor an assigned PR target; otherwise reuse a
matching integration branch only after checking scope and integrator ownership,
or create `feature/<topic>` from current primary for multi-slice work. Create
component task branches/worktrees from it and target their PRs there. Choose
automatically; ask only for ambiguous targets, scope or conflicting ownership.
Small standalone fixes may use one draft PR to primary. Never retarget unrelated
active PRs. Record base/head SHAs, chosen targets and one integration owner.

Component PRs use local checks and independent exact-head review. Final PRs to
`master`, `redesign` stay draft while assembling the batch; local validation,
finding dispositions and independent exact-head acceptance allow ready, which
starts applicable hosted CI. Hosted checks must pass before the existing merge
gate. Missing, skipped or unscheduled CI is not passed. Preserve all company
merge/deployment authority, protection and specialist-review requirements.
This adoption changes branch routing and CI scheduling only. For this workflow
it supersedes older instructions that require CI before the ready transition.

PRs targeting the long-standing `redesign` branch also run CI. Do not rename it
or assume unrelated work belongs there.
