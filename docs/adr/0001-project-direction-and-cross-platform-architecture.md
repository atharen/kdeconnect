# ADR-0001: Project direction and cross-platform architecture

- Status: Accepted
- Date: 2026-08-01
- Decider: Fork maintainer [@atharen](https://github.com/atharen)
- Based on: `1d3c5b2` (tip of `master` from upstream [`cosmic-utils/kdeconnect`](https://github.com/cosmic-utils/kdeconnect); 2026-07-30)

## Context

This repository is a fork of `cosmic-utils/kdeconnect`, a GPL-3.0-only KDE
Connect-compatible implementation written in Rust. The inherited project is
currently oriented around a COSMIC desktop applet and a Linux service. Its
protocol, transport, plugin, persistence, and Linux desktop concerns are
partially combined inside the package named `kdeconnect-core`.

The intended direction of this fork is a personal-device connectivity system
with first-class support for Apple and Android platforms. The existing Rust
implementation provides a useful foundation, but it must be separated from
Linux-specific facilities before it can be shared by iOS, macOS, and Android
clients.

Linux remains valuable for interoperability, development, and validation.
However, this fork will not attempt to displace the mature KDE Connect, GSConnect,
or Valent projects, nor commit to matching their desktop integrations.

Existing Apple-platform implementations also provide relevant prior art.
Soduto is a native Swift KDE Connect-compatible macOS client whose system
integration and protocol behaviour can inform this fork without displacing the
decision to share a Rust protocol engine across platforms.

The project should remain compatible with established KDE Connect clients
where the protocol permits. Project-specific capabilities may be added, but
must not silently change established packet semantics or prevent ordinary KDE
Connect implementations from using supported baseline features.

## Decision drivers

- Share protocol, identity, pairing, transport, and device-state behaviour.
- Improve native platform integration and user experience.
- Avoid independent Swift, Kotlin, and Rust implementations of security- and
  compatibility-sensitive behaviour.
- Make operating-system dependencies explicit.
- Focus effort on Apple platforms and Android while preserving broader
  interoperability.
- Preserve the useful existing Linux implementation on a best-effort basis.
- Retain interoperability with the wider KDE Connect ecosystem.
- Preserve compliance with the inherited GPL-3.0-only license.
- Remain local-first and avoid making a hosted account mandatory for LAN use.

## Decision

### Supported-platform priorities

The project has the following support priorities:

1. iOS and iPadOS
2. macOS
3. Android
4. A portable Rust protocol and connectivity foundation shared by those
   platforms

Linux is a secondary, best-effort platform. The existing Linux daemon, COSMIC
client, and associated integrations will remain in the repository and should
continue to build where reasonably practical, but they will not receive the
same feature, release, or regression priority as the primary platforms.

No dedicated KDE Plasma or GNOME client is planned initially. Contributions
for those environments may be accepted when they preserve the common daemon
and adapter boundaries.

### Shared Rust foundation

Rust will implement the shared KDE Connect protocol and stateful connectivity
foundation. The current `kdeconnect-core` will be incrementally separated into
packages with explicit responsibilities:

- `protocol`: packet types, serialization, framing, identities, capability
  negotiation, protocol versioning, and conformance fixtures.
- `engine`: discovery, connections, TLS, pairing, device lifecycle, transfers,
  and platform-independent plugin state machines.
- `platform-api`: traits and value types through which the engine requests
  operating-system behaviour.
- `ffi`: a stable, versioned command/event boundary for Swift and Android
  clients.

The portable packages must not directly depend on Linux desktop facilities
such as D-Bus, MPRIS, PulseAudio, Wayland, XDG commands, desktop portals, or
desktop notification implementations.

Platform behaviour will be expressed through explicit capabilities and
adapters, including notifications, clipboard access, file access, media
control, contacts, remote input, secure storage, and application lifecycle.
An unavailable platform capability must degrade explicitly rather than be
assumed or emulated through an unrelated entitlement.

### Native clients and adapters

The shared Rust foundation does not require every client or integration to be
implemented in Rust.

- iOS and iPadOS will use Swift and SwiftUI, with UIKit and app extensions
  where Apple provides an appropriate extension point.
- macOS will use SwiftUI and AppKit. XPC services or launch agents may host the
  Rust engine when persistent background operation is appropriate.
- Android will use shared Swift/Skip UI where practical. Native Kotlin or Java
  adapters remain appropriate for services, intents, permissions,
  notification listeners, media sessions, and other Android facilities.
- Linux will retain a Rust daemon and Linux adapters, with the existing COSMIC
  applet as its initial desktop-specific frontend.

Skip is a UI- and application-sharing tool. It is not a requirement that every
Android system API be accessed through shared Swift.

Soduto will be treated as macOS behavioural and integration prior art,
particularly for sharing, secure identity storage, notifications, media,
accessibility, packaging, and compatibility tests. Its independently
implemented Swift protocol engine is not adopted as this project's
architecture.

### Monorepo

The project will remain a monorepo containing the shared Rust crates, native
clients, platform adapters, protocol tests, and packaging metadata. Package and
license boundaries must remain explicit even when code shares a repository.

The intended high-level structure is:

```text
crates/
  protocol/
  engine/
  platform-api/
  ffi/

platform/
  apple/
  android/
  linux/

apps/
  apple/
  android/
  linux/

protocol-tests/
LICENSES/
REUSE.toml
```

This is a target organization, not a requirement to move every file in one
change. The existing implementation should remain executable during the
restructure, and repository history should be preserved.

### Linux policy

The existing Linux work will be integrated into the new boundaries rather
than discarded. It will be divided into a generic Linux daemon, Linux platform
adapters, a documented local IPC surface, packaging, a basic client or CLI,
and the existing COSMIC frontend.

The Linux daemon should prefer freedesktop interfaces and XDG portals where
they provide the necessary behaviour. Wayland functionality such as automatic
clipboard synchronization and remote input may require compositor- or
desktop-specific adapters and is not guaranteed on every session.

Linux maintenance is best-effort:

- Existing functionality should be preserved during the initial restructure.
- Baseline Linux builds and tests should remain in CI where practical.
- COSMIC, KDE Plasma, and GNOME feature parity is not promised.
- Desktop-specific regressions may not receive immediate maintainer attention.
- External maintenance and integration contributions are welcome.
- Unsupported features should be reported as unavailable capabilities.

Users should not assume that this daemon can run alongside KDE Connect,
GSConnect, Valent, or another KDE Connect-compatible daemon in the same user
session. Implementations may compete for discovery ports, identities, and
desktop integrations.

### Upstream relationship

This fork will track two relevant upstream bodies of work:

1. The `cosmic-utils/kdeconnect` Rust project.
2. The wider KDE Connect protocol and official platform implementations.

Upstream changes will be evaluated and integrated on a best-effort basis.
Security fixes, cryptographic and pairing corrections, protocol compatibility
changes, and transport fixes take priority over desktop-specific additions.
Not every upstream change is guaranteed to merge cleanly into the restructured
architecture.

Generally useful fixes developed here should be proposed upstream when they
can be separated from fork-specific product or architecture decisions.

### Compatibility and extensions

Project-specific protocol extensions must:

- Use distinct, documented capability and packet identifiers.
- Be explicitly versioned when their semantics may evolve.
- Be ignored safely by clients that do not implement them.
- Avoid incompatible changes to established KDE Connect packet semantics.
- Preserve useful baseline behaviour with existing clients where practical.

Compatibility should be verified through automated packet fixtures and
behavioural tests covering packet encoding, pairing, identity and certificate
handling, capability negotiation, payload transfer, and unknown extensions.

### Licensing

Inherited Rust code remains GPL-3.0-only. This architecture decision does not
relicense it.

A client that links or embeds the GPL Rust core is distributed as a
GPL-covered combined application, even if individual client-authored files are
also available under a compatible platform-specific license.

Distribution through Apple's App Store may require an additional App Store
distribution permission from every relevant copyright holder of the embedded
GPL code. The KDE Connect iOS project's existing license statement does not
automatically apply to this independent Rust implementation. The exact App
Store exception and contributor policy are deferred to a separate decision.

Copyright, authorship, and SPDX metadata must be preserved during the
restructure.

### Local-first connectivity and iOS background constraints

Pairing, local discovery, and direct encrypted communication should remain
usable without a project-operated account or cloud service whenever the
platform permits it.

iOS does not provide unrestricted background execution for a persistent LAN
protocol connection. The project must not claim otherwise or use background
modes intended for unrelated purposes merely to keep the engine alive.

An eventual iOS design may combine foreground peer-to-peer communication,
system-managed background transfers, visible notifications, best-effort
background pushes, and an optional encrypted rendezvous or wake-up service.
Its privacy, hosting, metadata, entitlement, and failure-mode implications
require a separate ADR.

No Internet-reliant service is introduced by this decision. One may be
introduced only after a dedicated ADR defines its purpose, threat model,
privacy properties, hosting model, and failure behaviour. Any such service
must minimise collected metadata, use end-to-end encryption for content where
possible, disclose when traffic leaves the local network, consider
self-hosting, and degrade gracefully when unavailable. The project's purpose
is to connect personal devices securely and privately; an optional
Internet-facing component must preserve that trust.

## Non-goals

This is a personal project. These support priorities describe the intended
allocation of effort and do not constitute a service-level or maintenance
guarantee.

This decision does not commit the project to:

- Replacing the official KDE Connect desktop implementation.
- Replacing GSConnect, Valent, or Soduto.
- Building dedicated KDE Plasma or GNOME clients.
- Providing identical capabilities on every platform.
- Implementing all native integration in Rust.
- Providing unrestricted background execution on iOS.
- Requiring a hosted account or relay for basic LAN use.
- Relicensing inherited GPL code.
- Implementing all project-specific protocol extensions during the initial
  restructure.

## Consequences

### Positive

- Security- and compatibility-sensitive protocol behaviour is shared.
- Apple and Android clients can retain native system integration.
- Linux provides an executable migration baseline and interoperability target.
- Platform limitations become explicit capabilities rather than hidden
  assumptions.
- Protocol and security fixes can be applied centrally.
- The monorepo permits atomic changes across the core, FFI, clients, and tests.

### Negative

- Separating the current Linux-coupled core will be substantial work.
- A stable asynchronous FFI surface adds design and testing costs.
- iOS cannot provide Android- or desktop-equivalent background connectivity.
- GPL App Store distribution requires additional legal and contributor work.
- Linux desktop integrations may lag behind primary-platform development.
- Cargo, SwiftPM/Xcode, Skip, Gradle, and native packaging increase CI
  complexity.
- Compatibility with independently evolving implementations requires ongoing
  conformance testing.

## Alternatives considered

### Continue as a COSMIC-first Linux application

Continuing as COSMIC-first was not selected as this fork's controlling
direction. The Rust implementation was selected as a practical foundation for
broader platform work, while the existing Linux application remains a
best-effort migration baseline. The upstream project remains independent and
is not expected to adopt this fork's priorities.

### Maintain separate protocol engines in Swift, Kotlin, and Rust

Rejected because protocol, cryptographic, pairing, and compatibility fixes
would be duplicated across implementations. Soduto demonstrates that a native
Swift protocol implementation can provide strong macOS integration, but it is
retained as prior art rather than selected as this project's architecture.

### Implement every client and integration in Rust

Rejected because Apple and Android system integration is more naturally and
reliably implemented through native APIs.

### Require a central service

Rejected as the default because the project is local-first. An optional relay,
rendezvous, or wake-up service may still be introduced by a later decision.

### Immediately implement KDE and GNOME clients

Deferred because mature implementations already exist and Linux desktop
integration is not a primary support target.

## Follow-up decisions

Later ADRs should address:

1. Portable Rust crate boundaries and the migration sequence.
2. FFI technology, ownership, concurrency, and API versioning.
3. iOS background reachability, notifications, and any optional relay.
4. The iOS App Store GPL exception and contributor policy.
5. Android service, lifecycle, and process architecture.
6. The macOS daemon and XPC model, including comparison with Soduto's native
   architecture.
7. The Linux local IPC API and compatibility policy.
8. Protocol-extension namespace and versioning.
9. Device identity, credential storage, and recovery.
10. Privacy, telemetry, and hosted-service policy.
11. Release, signing, packaging, and update channels.

## References

- [KDE Connect desktop implementation](https://github.com/KDE/kdeconnect-kde)
- [KDE Connect iOS license statement](https://github.com/KDE/kdeconnect-ios/blob/master/License.md)
- [GSConnect](https://github.com/GSConnect/gnome-shell-extension-gsconnect)
- [Valent](https://github.com/andyholmes/valent)
- [Soduto](https://github.com/sannidhyaroy/Soduto)
- [Skip](https://github.com/skiptools/skip)
- [Apple: Choosing background strategies for your app](https://developer.apple.com/documentation/BackgroundTasks/choosing-background-strategies-for-your-app)
- [Apple: Local Push Connectivity](https://developer.apple.com/documentation/networkextension/local-push-connectivity)
- [XDG Desktop Portal](https://flatpak.github.io/xdg-desktop-portal/)
