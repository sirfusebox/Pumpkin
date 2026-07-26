# AGENTS.md

## Cursor Cloud specific instructions

Pumpkin is a single-product **Rust Cargo workspace** (a Minecraft server). There is **no database, cache, or other external service** — world/player state is persisted to files on disk. The whole product is the `pumpkin` binary.

The startup update script already runs `git submodule update --init --recursive` (the `pumpkin-plugin-wit` submodule is required for a full build) and `cargo fetch`. The Rust toolchain is pinned to `stable` by `rust-toolchain.toml` and is installed automatically on the first `cargo` invocation.

### Build / lint / test / run

- Build (dev): `cargo build` — first clean build takes ~4 min; output binary is `target/debug/pumpkin`. Prefer debug for iteration; release (`cargo build --release`) is much slower (LTO, `codegen-units = 1`).
- Lint: `cargo clippy --all-targets`. CI (`.github/workflows/rust.yml`) is stricter: it runs clippy with `--all-features` in both debug and release with `RUSTFLAGS="-Dwarnings"`, plus `cargo fmt --check`.
- Test: `cargo test`. CI uses `cargo nextest run` + `cargo test --doc`; `nextest` is **not** preinstalled here, so use `cargo test` (or install nextest if you specifically need it).
- Format: `cargo fmt`.
- Run: `cargo run` (or run the built binary directly).

### Running / testing the server (non-obvious)

- The server reads and writes its config (`pumpkin.toml`), `world/`, `logs/`, and `plugins/` in the **current working directory**. Run it from a scratch directory (e.g. `run/`, which is gitignored) so these generated files don't clutter the repo.
- On first launch it auto-generates `pumpkin.toml` with defaults and listens on `0.0.0.0:25565` (Java + Query). Bedrock (WIP) binds `19132`; RCON binds `25575` but is **disabled by default**.
- By default `[networking.java] online_mode = true` and `encryption = true`, which require Mojang authentication and a licensed account for a client to join. For local/offline testing set `online_mode = false` and `encryption = false` in `pumpkin.toml`.
- No GUI: verify the server over the network. Two easy checks that need no Minecraft client:
  - Server List Ping (status handshake) to `127.0.0.1:25565` returns live server JSON (version, MOTD, player count).
  - Enable RCON (`[networking.rcon] enabled = true` + a `password`) and connect to `127.0.0.1:25575` to run server commands (e.g. `list`, `seed`, `time set day`).
