# KDE Connect Linux service

`kdeconnect-service` hosts the current KDE Connect protocol engine for the
Linux implementation. It owns discovery, connections, pairing, device state,
plugin dispatch, and the local IPC interfaces consumed by the COSMIC applet.

The service currently depends on Linux desktop facilities. It is the migration
baseline from which portable protocol and engine crates will be extracted; it
is not itself the future cross-platform boundary. See the
[root architecture map](../../../ARCHITECTURE.md) for the current and intended
system boundaries.

## Run from source

Run these commands from the repository root:

```bash
just build-service
just run-service
```

The complete Linux application can be built and installed with:

```bash
just build
just install
```

The default installation starts the service through D-Bus activation and XDG
autostart on the next login.

## Optional systemd user service

Install and enable the user service with:

```bash
just install-systemd
just enable-service
```

The following helpers operate on that service:

```bash
just status
just logs
just stop
just restart
```

## Debug logging

Install wrappers with full logging for the service and panel applet:

```bash
just install-debug
```

The logs are written to:

- `/tmp/kdeconnect-service.log`
- `/tmp/kdeconnect-applet.log`

Restore the standard installation with `just install`.

## Firewall

KDE Connect discovery and communication use TCP and UDP ports 1714–1764. Allow
that range on the local network when a firewall is enabled.

For UFW:

```bash
sudo ufw allow 1714:1764/udp
sudo ufw allow 1714:1764/tcp
sudo ufw reload
```

For firewalld:

```bash
sudo firewall-cmd --permanent --zone=home --add-service=kdeconnect
sudo firewall-cmd --reload
```

For iptables:

```bash
sudo iptables -I INPUT -i <interface> -p udp --dport 1714:1764 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -I INPUT -i <interface> -p tcp --dport 1714:1764 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -o <interface> -p udp --sport 1714:1764 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -o <interface> -p tcp --sport 1714:1764 -m state --state NEW,ESTABLISHED -j ACCEPT
```

See the
[KDE Connect firewall documentation](https://userbase.kde.org/KDEConnect#Firewall)
for additional distribution-specific guidance.
