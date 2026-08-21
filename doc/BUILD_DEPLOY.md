# Polaris-Configuration Build and Deployment

## Purpose and scope

Polaris-Configuration is the Rust `webconfigd` daemon and workspace. Its Makefile is the single entry point for host builds, release builds, tests using fake sysroot behavior, linting, cross-target parity, vendoring, release tarballs, and development runs.

## 1. Prerequisites

- Linux, GNU Make, curl, and a working network for initial toolchain installation.
- Rustup with the pinned `rust-toolchain.toml` channel `1.97.1`, `rustfmt`, `clippy`, and targets `x86_64-unknown-linux-gnu`, `aarch64-unknown-linux-gnu`, and `armv7-unknown-linux-gnueabihf`.
- `fuser` for `make stop`; target-specific backend behavior may require a real Linux host.

## 2. Setup

```bash
make toolchain
```

This idempotent target installs rustup if missing, installs the pinned toolchain/components, and adds all three cross targets. It requires access to `sh.rustup.rs` and `static.rust-lang.org`.

## 3. Build and compile

```bash
make build                  # workspace debug build
make release                # pinned release profile
make cross                  # release build for all declared targets
```

The Makefile reads the channel from `rust-toolchain.toml` and invokes the configured Cargo. `make cross` is CI-parity compilation, not proof that a target device can boot or run the daemon.

## 4. Test and lint

```bash
make test                   # cargo test --workspace --all-targets
make lint                   # clippy -D warnings and rustfmt --check
make fmt                    # format in place
make check                  # build, test, and lint
```

Tests use fake sysroot behavior and do not require hardware.

## 5. Packaging and artifacts

```bash
make vendor                 # regenerate vendor/ after dependency changes
make release-artifacts     # tools/release-artifacts.sh and SHA-256 output
```

Build products are in `target/`. Release artifacts package source and vendored crate tarballs for the `webconfigd.bb` recipe. `make vendor` changes the vendored tree and should only be used deliberately.

## 6. Deployment, install, and run

```bash
make run                    # real host sysroot, port WEBCONFIG_PORT (8443)
make run-mock               # temporary fake sysroot, no hardware
WEBCONFIG_PORT=9443 make run
WEBCONFIG_PORT=8443 make stop
```

`make run` reads live `/proc` and uses real system backends; use it only on an appropriate host and expect persistent development state in `./data`. `make run-mock` is the safe no-hardware validation path. No remote/cloud deployment target is declared.

## 7. Cleanup

```bash
make clean                  # cargo clean, removes target/
rm -rf data                 # optional manual reset of first-boot dev state
```

Removing `data` resets the daemon's development database and certificates; this is intentionally outside the Makefile.

## 8. Verification caveats

Cross compilation and fake-sysroot tests do not validate Renesas hardware, systemd/update backends, certificates, or live `/proc` values. Before installation, manually verify the target image integration, service permissions, port availability, first-boot flow, and persistence. Credentials and production deployment are outside this repository's Makefile.