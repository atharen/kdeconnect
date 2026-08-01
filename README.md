# KDE Connect

An experimental, local-first personal-device connectivity project based on
the KDE Connect protocol.

This repository is a fork of
[`cosmic-utils/kdeconnect`](https://github.com/cosmic-utils/kdeconnect). It is
being reorganized around a shared Rust protocol and connectivity foundation
with native clients for Apple and Android platforms.

## Status

The existing executable implementation is the inherited Linux service and
COSMIC desktop applet. They remain the working migration baseline while the
shared code is separated from Linux-specific integrations.

The intended platform priorities are:

1. iOS and iPadOS
2. macOS
3. Android
4. The portable Rust foundation shared by those platforms

Linux support is retained on a best-effort basis. The project is under active
restructuring and does not yet provide Apple or Android applications.

## Repository guide

- `crates/` contains shared Rust code. `kdeconnect-core` is the current,
  transitional crate and still includes Linux-specific behavior.
- `apps/` contains user-facing applications and executable services, grouped
  by platform.
- `platform/` contains platform integration libraries and local IPC bindings.
- `docs/adr/` records significant architecture and project decisions.

Start with [`ARCHITECTURE.md`](ARCHITECTURE.md) for the system map and intended
boundaries. The accepted direction is recorded in
[`ADR-0001`](docs/adr/0001-project-direction-and-cross-platform-architecture.md).

## Current Linux implementation

- [COSMIC applet, settings, SMS UI, and Flatpak instructions](apps/linux/cosmic-ext-connect-applet/README.md)
- [Linux service, activation, logging, and network setup](apps/linux/kdeconnect-service/README.md)

## License

The project is distributed under GPL-3.0-only. See [`LICENSE`](LICENSE).
