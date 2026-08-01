# Architecture

This document is a map of the repository: what the major pieces do, how they
relate, and where a change belongs. It deliberately describes stable
boundaries rather than implementation details. Architecture decision records
in `docs/adr/` are authoritative when a decision and this map differ.

## Bird's-eye view

The project connects personal devices using the KDE Connect protocol. It
discovers peers, establishes encrypted connections, pairs devices, exchanges
capability-specific packets and payloads, and presents those capabilities
through native platform interfaces.

The current executable system is Linux-specific:

```text
KDE Connect peer
    ↕ UDP discovery, TCP/TLS, packets and payloads
kdeconnect-core
    ↕ AppEvent commands and ConnectionEvent notifications
kdeconnect-service
    ↕ Varlink commands and D-Bus methods/signals
COSMIC applet, settings, and SMS UI
```

The intended system keeps the protocol and stateful connectivity behavior in
shared Rust while native clients and adapters own operating-system behavior:

```text
native client
    ↕ versioned commands and events
FFI boundary
    ↕
shared engine → shared protocol
    ↕ capability traits
platform adapters
```

The current implementation will be separated incrementally; it is not expected
to jump directly from the first diagram to the second.

## Codemap: current implementation

### Shared and transitional Rust code

`crates/kdeconnect-core` contains the current protocol engine. The name is
transitional: the crate still combines portable behavior with Linux desktop
integrations.

- `KdeConnectCore` owns the main asynchronous event loop.
- `protocol` defines packet types, serialization, and wire-level values.
- `transport` performs discovery, TCP/TLS connection handling, identity
  exchange, and payload transport.
- `device` and `pairing` own device lifecycle and pair state.
- `plugin_interface` dispatches packets to implementations under `plugins`.
- `event` defines the current command boundary (`AppEvent`) and notification
  boundary (`ConnectionEvent`).
- `config`, `plugin_config`, contacts, SMS state, and conversation state own
  the current local persistence concerns.

When looking for wire behavior, start in `protocol` and `transport`. When
looking for a KDE Connect capability, start in the matching file under
`plugins`.

### Linux service

`apps/linux/kdeconnect-service` is the long-running Linux process. Its `main`
function starts `KdeConnectCore`, D-Bus interfaces, the Varlink server, and
Linux clipboard integration.

- `dbus_interface` exposes daemon, SMS, contacts, notification, MPRIS, and
  related desktop interfaces.
- `varlink_server` exposes the local command-oriented IPC surface.
- `clipboard` integrates with the Linux desktop clipboard.

The service translates local IPC requests into `AppEvent` values and publishes
`ConnectionEvent` results back to clients.

### Linux IPC libraries

`platform/linux/kdeconnect-varlink` owns the Varlink interface definition,
generated Rust bindings, and socket-address convention shared by service and
clients.

`platform/linux/kdeconnect-dbus-client` owns typed D-Bus proxies and translates
signals into `ServiceEvent` values for the COSMIC client.

Changes to an IPC operation normally touch the service implementation, the
corresponding platform library, and the applet backend together.

### COSMIC application

`apps/linux/cosmic-ext-connect-applet` contains the panel applet, settings
window, and SMS window.

- `main` owns the applet lifecycle and top-level update loop.
- `backend` is the UI-to-service boundary. Commands prefer Varlink and fall
  back to D-Bus; D-Bus also carries signals and MPRIS integration.
- `messages`, `models`, and `ui` define application state and presentation.
- `plugins/sms` contains the dedicated SMS application behavior.
- `portal` contains desktop portal integration.
- `flatpak` and `resources` contain Linux/COSMIC packaging assets.

UI code does not open KDE Connect network connections directly. It operates
through the service boundary.

### Repository support files

The root `Cargo.toml` defines the Rust workspace and shared dependency
versions. The root `justfile` provides build and Linux installation commands.
`docs/adr` contains architecture decision records and their template and
index.

## Main runtime flows

### Discovery and connection

`UdpTransport` discovers peers and exchanges identities. `TcpTransport`
establishes the encrypted connection. `KdeConnectCore` records the active
device and writer, then reports lifecycle changes through `ConnectionEvent`.

### Local command to remote device

The COSMIC UI sends a Varlink or D-Bus request to `kdeconnect-service`. The
service converts it to an `AppEvent`. `KdeConnectCore` and the relevant plugin
produce a protocol packet or payload for the active transport.

### Remote packet to local UI

The transport emits a received packet to `KdeConnectCore`. The plugin registry
dispatches it by packet type. Plugin output becomes a `ConnectionEvent`, which
the service translates to local IPC signals or state consumed by the UI.

## Intended boundaries

The target shared Rust foundation is divided by responsibility:

- `protocol`: packets, framing, identities, capability negotiation, protocol
  versions, and conformance fixtures.
- `engine`: discovery, encrypted connections, pairing, device lifecycle,
  transfers, and platform-independent plugin state machines.
- `platform-api`: traits and value types requested by the engine when an
  operating system must perform an action.
- `ffi`: a stable, versioned command/event boundary for Swift and Android.

Platform implementations belong under `platform/<platform>`. User-facing
applications belong under `apps/<platform>`. Protocol compatibility fixtures
and behavioral tests will belong under `protocol-tests` when introduced.

## Architectural invariants

- Portable crates must not depend on D-Bus, MPRIS, PulseAudio, Wayland, XDG
  commands, desktop portals, or another operating-system implementation.
- Protocol, identity, pairing, transport, and compatibility-sensitive state
  machines have one shared Rust implementation.
- Native clients do not own independent KDE Connect protocol engines.
- Operating-system behavior crosses explicit capability or adapter
  boundaries. An unavailable capability is reported as unavailable.
- UI code communicates through command/event boundaries and does not own
  network transports, pairing state, or credentials.
- Established KDE Connect packet semantics remain compatible. Project-specific
  extensions use distinct, documented, safely ignorable identifiers.
- Pairing and direct encrypted LAN communication remain usable without a
  project-operated account whenever the platform permits it.
- Linux remains an executable migration and interoperability baseline while
  the portable boundaries are extracted.

Existing Linux dependencies inside `kdeconnect-core` are migration constraints,
not precedent for new shared-code dependencies.

## Cross-cutting concerns

### Concurrency and events

The current engine uses Tokio tasks and channels. `AppEvent` carries commands
into the engine, `CoreEvent` coordinates internal work, and `ConnectionEvent`
carries results toward platform clients. The eventual FFI boundary should
preserve command/event semantics without exposing Rust task or channel types.

### Persistence

The current implementation stores device identities, pair state, plugin
configuration, contacts, and messaging state in platform configuration and
data directories. Secure identity storage and recovery are deferred design
work; new platforms should not copy Linux filesystem assumptions.

### Compatibility and security

Protocol parsing, identity exchange, TLS, pairing, and payload transfer are
compatibility- and security-sensitive. Changes in these areas should be tested
against established KDE Connect behavior and should avoid platform-specific
forks of the logic.

### Platform lifecycle

Desktop daemons can maintain long-running connections. Mobile platforms,
especially iOS, cannot assume unrestricted background execution. Lifecycle,
reachability, notification, and optional relay behavior require explicit
platform designs rather than hidden workarounds in the shared engine.

## Where to make a change

| Change | Start here |
| --- | --- |
| Packet type or encoding | `crates/kdeconnect-core/src/protocol.rs` |
| Discovery, TLS, or transport | `crates/kdeconnect-core/src/transport.rs` |
| Pairing or device lifecycle | `pairing.rs` and `device.rs` in `kdeconnect-core` |
| KDE Connect plugin behavior | Matching module under `kdeconnect-core/src/plugins` |
| Linux service API | `kdeconnect-service/src/dbus_interface.rs` or `varlink_server.rs` |
| Linux IPC binding | Matching crate under `platform/linux` |
| COSMIC UI | `cosmic-ext-connect-applet/src` |
| Linux/COSMIC packaging | `cosmic-ext-connect-applet/flatpak` and `resources` |
| Architecture decision | `docs/adr` |

## Further reading

- `docs/adr/0001-project-direction-and-cross-platform-architecture.md`
- `docs/adr/README.md`
- `apps/linux/cosmic-ext-connect-applet/README.md`
- `apps/linux/kdeconnect-service/README.md`
