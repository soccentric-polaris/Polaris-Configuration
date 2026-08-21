# Polaris-Configuration

## Project Summary

- Polaris-Configuration is a Rust workspace for the on-device WebConfig
  service used to configure Polaris embedded Linux boards. The
  implementation is split between `crates/webconfig-engine`, which owns
  configuration state, schemas, manifests, appliers, monitoring, and alert
  behavior, and `crates/webconfigd`, which exposes the HTTP daemon. The
  `frontend/` directory is a vanilla-JavaScript interface that renders
  forms from JSON schemas, while `schemas/` and `board-manifests/` supply
  board-aware configuration inputs. `meta-webconfig/` packages the daemon,
  static assets, schemas, manifests, and hardened service integration for
  Yocto.

- The design uses a small single-threaded Tokio service with Axum and rustls,
  using JSON and ring files instead of a database. The daemon surface
  includes authentication, role-aware access, CSRF protection, TLS, section
  APIs, and server-sent events. Network changes have a provisional
  commit/confirm workflow intended to reconnect safely or roll back after a
  timeout. Real and mock backends are represented so host-side tests can
  use a fake sysroot rather than hardware. Board manifests and example
  schemas are concrete configuration inputs; documentation explains offline
  vendoring and cross-target builds.

- The root Makefile is the operational contract: `make build`, `release`,
  `test`, `lint`, `fmt`, `vendor`, `cross`, `run`, `run-mock`, and `check`
  are declared. Automated evidence is visible in
  `crates/webconfigd/src/tests.rs`; the remaining behavior should be
  treated as partially covered until the full Rust test suite is run. The
  project is an implemented service with integration and deployment risk
  still requiring device-level confirmation.

## Feature Inventory

| # | Name / Description | Implemented? | Test Coverage? | Manual Test |
|---:|---|---|---|---|
| 1 | Rust workspace and pinned release profile (`Cargo.toml`, `rust-toolchain.toml`) | Implemented | `make build` and `make lint` declared | [ ] Manual verification required |
| 2 | Configuration engine and persistent state | Implemented | `crates/webconfigd/src/tests.rs`; broader coverage not confirmed | [ ] Manual verification required |
| 3 | Axum WebConfig daemon | Implemented | HTTP tests present in daemon test module | [ ] Manual verification required |
| 4 | TLS via rustls | Implemented | No focused TLS integration file found | [ ] Manual verification required |
| 5 | Authentication, roles, and CSRF controls | Implemented | No complete security matrix found | [ ] Manual verification required |
| 6 | Schema-driven frontend rendering | Implemented | No frontend test suite found | [ ] Manual verification required |
| 7 | Board manifest selection (`polaris.json`, `generic.yaml`) | Implemented | No manifest matrix test found | [ ] Manual verification required |
| 8 | Network commit/confirm rollback workflow | Implemented | No dedicated rollback test found | [ ] Manual verification required |
| 9 | Monitoring, alerts, and SSE updates | Partial | HTTP test evidence exists; event coverage unclear | [ ] Manual verification required |
| 10 | Real and fake sysroot backends | Implemented | `make test` and `make run-mock` declared | [ ] Manual verification required |
| 11 | Yocto `meta-webconfig` packaging | Implemented | No BitBake integration test found | [ ] Manual verification required |
| 12 | Offline vendor and cross-build workflow | Implemented | `make vendor`, `make cross` declared | [ ] Manual verification required |
| 13 | Release artifact/checksum workflow | Partial | Script exists; output not independently verified | [ ] Manual verification required |

## Validation

```bash
make check
make run-mock
```

Browser behavior, TLS certificate deployment, rollback under network loss, and Yocto installation remain manual checks.
