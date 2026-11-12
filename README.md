# KEVO TUNNEL

**A desktop proxy client for Linux and Windows, with optional system-level TUN routing, built around the official [sing-box](https://github.com/SagerNet/sing-box) engine.**

KEVO TUNNEL gives you one place to manage proxy profiles, start or stop a connection, and check what the local client and tunnel are actually doing. Local proxy mode is designed to work without changing system routes. System TUN is an explicit, optional mode that requires the operating system’s normal authorization and networking support.

- **Product site:** [ktunnel.vercel.app](https://ktunnel.vercel.app)
- **Downloads:** [GitHub Releases](https://github.com/DevKevo25/KTunnel/releases)
- **Linux source (private):** [DevKevo25/KEVONET](https://github.com/DevKevo25/KEVONET)
- **Windows source (private):** [DevKevo25/KT_WIN](https://github.com/DevKevo25/KT_WIN)

This public repository is the product and release hub. The Linux and Windows clients are maintained in separate private source repositories; source links require authorized GitHub access.

## Choose a build

Use the [Releases page](https://github.com/DevKevo25/KTunnel/releases) as the source of truth for what is currently available. Download only an asset attached to a published release, and choose the file that matches your operating system and processor architecture.

| Platform | Build | Notes |
| --- | --- | --- |
| Linux x86_64 | AppImage | AppImage is the Linux release format for this build; the source also has DEB/RPM packaging targets. Check the release page to see which packages have actually been published. |
| Windows x64 | NSIS `.exe` installer | Use the Windows installer shown on the Releases page. System TUN has additional administrator and driver requirements. |

A build target is not a promise that a package is currently available. Check the release page before installing; this site and repository do not mirror installers.

## What it does

- **Local proxy mode:** runs a loopback-only mixed HTTP/SOCKS listener backed by sing-box.
- **Optional system TUN:** routes system traffic through the tunnel only when you explicitly enable it and the operating system allows the required changes.
- **Profile management:** add, validate, activate, and remove profiles; import native sing-box JSON or the supported subset of Clash YAML.
- **Connection visibility:** inspect live connections, stream daemon/engine logs, and close active connections from the app.
- **Explicit connection checks:** run an HTTPS egress test through the configured local proxy to check beyond whether a local listener is merely responding.
- **Clear failure behavior:** unsupported Clash rules, proxy groups, or features return a configuration/conversion error rather than being silently treated as supported.

The Linux adapter documents support for standard proxy nodes including VLESS, VMess, Trojan, Shadowsocks, Snell, HTTP, SOCKS5, and SSH. The exact accepted configuration subset can vary by client version; use the validation result shown by the app. Proprietary mobile payload formats such as HTTP Custom/HTTP Injector `.ehi` and Dark Tunnel `.hc` are not sing-box proxy protocols and are not supported by this adapter.

## Install and run

1. Open [GitHub Releases](https://github.com/DevKevo25/KTunnel/releases) and select a published release.
2. Download the Linux AppImage or Windows x64 installer that matches your system.
3. On Linux, make the downloaded AppImage executable if your file manager did not do so, then launch it. On Windows, run the downloaded installer and follow its prompts.
4. Start with **Local proxy** mode. Import a profile you trust, validate it, and use the app’s HTTPS egress check to confirm the configured route.
5. Enable **System TUN** only if you need operating-system-wide routing and have met the platform requirements below.

Keep proxy profiles private. They may contain credentials or server details; do not post them in public issues, screenshots, or logs.

## System TUN requirements

### Linux

- A working `/dev/net/tun` device and a kernel/network setup that permits TUN interfaces and route management.
- A working desktop polkit authentication agent when the app requests administrator authorization.
- The app requests the required privilege for the TUN operation; running the entire desktop GUI as root is not recommended.
- Stop the connection in the app or exit normally so the engine can clean up routes. Force-killing the process can interrupt cleanup.

### Windows

- Launch KEVO TUNNEL with **Run as administrator** before enabling System TUN.
- A working Wintun-compatible adapter/driver is required for the system tunnel.
- Local proxy mode does not require administrator privileges. If TUN is unavailable, use Local proxy mode or install/enable the compatible driver.
- Stop the connection from the app or exit normally so the tunnel adapter and routes can be cleaned up.

## Understanding connection status

A ready local listener means the configured local proxy port responded to a protocol probe while the managed sing-box process was alive. It does **not** prove that the remote proxy server is reachable. TUN readiness additionally checks the expected interface and operating-system routing setup; it does **not** prove that every destination is reachable. Use the separate HTTPS egress check and live connection information for upstream verification.

No client can guarantee the privacy, speed, or availability of a remote proxy server. Choose and configure upstream services carefully.

## Build from source

### Linux

The Linux client is built from the private [KEVONET](https://github.com/DevKevo25/KEVONET) repository. Maintainers with source access need Node.js, Rust, Go, and the native Tauri/Linux development libraries (including WebKitGTK 4.1, GTK, and the platform indicator and X11 development packages). From the repository root:

```bash
npm ci
npm run build:daemon
npm run tauri -- build --bundles appimage
```

The daemon build stages the pinned sing-box sidecar and verifies its checksum. See the Linux repository’s README and workflow for the current system-package list and Go version source.

### Windows

The Windows client is built from the private [KT_WIN](https://github.com/DevKevo25/KT_WIN) repository. Maintainers with source access need Node.js, Go 1.24.x, Rust for `x86_64-pc-windows-msvc`, the Tauri Windows build prerequisites, and NSIS. From the repository root on a Windows build machine:

```powershell
npm ci
npm run dist:windows
```

The NSIS installer is written under `src-tauri/target/x86_64-pc-windows-msvc/release/bundle/nsis/`. The daemon build downloads and verifies its pinned sing-box sidecar before packaging.

## Architecture and local API

Both clients use a Tauri 2 desktop shell with a React/TypeScript interface and a Go daemon that manages sing-box as a child process. The daemon exposes an authenticated localhost REST/SSE API to the desktop shell. All API endpoints require a bearer token except `/api/health`.

Common API areas include status, profile CRUD and activation, engine stop and HTTPS probe, active connections, and streaming logs. These endpoints are intended for the local desktop integration, not as a public network service.

## Source repositories (maintainer access required)

- [KEVONET — Linux client (private)](https://github.com/DevKevo25/KEVONET)
- [KT_WIN — Windows client (private)](https://github.com/DevKevo25/KT_WIN)
- [sing-box — upstream proxy platform](https://github.com/SagerNet/sing-box)

Third-party notices and upstream licensing information are included with the client resources. Review the applicable licenses before redistributing modified builds.
