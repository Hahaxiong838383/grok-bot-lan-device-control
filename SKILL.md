---
name: lan-device-control
description: "Configure and operate local LAN computers and devices via EasyTier mesh, SSH, and adb. Use when a Grok Bot needs to control or access physical machines in the internal mesh network."
type: workflow
lifecycle: active
---

# LAN Device Control — Operate Mesh Network Devices

Use this skill when a Grok Bot needs to access or control physical computers/devices in the internal EasyTier mesh.

Once the Grok Bot computer (cloud computer) joins the EasyTier mesh, it can reach and control other computers and devices on the same mesh via SSH, adb, or other protocols, as long as those nodes are online and properly configured.

Primary path today: SSH to the main Mac jump host → adb to the connected tablet. Desktop computer use on the Mac is not enabled.

## Recommended Computer Use Solutions (Desktop)

Current mesh path uses SSH + adb for the tablet. For full desktop computer use on Mac or Linux nodes, use the following recommended open-source projects. Always grant permissions on the target machine itself (cannot be done purely over SSH).

### macOS (Recommended)

| Project | Repo | Why Recommended | Key Evidence |
|---------|------|-----------------|--------------|
| opencode-computer-use | https://github.com/virajshoor/opencode-computer-use | Native Swift + CoreGraphics + AXUIElement; MCP for OpenCode | Swift helper uses CGEvent for input and AXUIElement for UI tree; 17 tools including screenshot, click, drag, accessibility tree |
| open-codex-computer-use | https://github.com/iFurySt/open-codex-computer-use | Cross-platform MCP alternative to Codex Computer Use | Accessibility-first; macOS permission agent; supports macOS/Linux/Windows |
| Codeg (codeg-computer-helper) | https://github.com/spacering-net/codeg | Official integrated solution with isolated helper | Architecture docs: helper runs cua-driver; macOS permissions granted to helper.app only |

### Linux (Recommended)

| Project | Repo | Why Recommended | Key Evidence |
|---------|------|-----------------|--------------|
| computer-use-linux | https://github.com/agent-sh/computer-use-linux | Best Wayland + AT-SPI coverage | AT-SPI + Wayland Portals + ydotool; supports GNOME, KDE, Hyprland, niri, i3, COSMIC |
| linux-computer-use | https://github.com/tak-uukti/linux-computer-use | Lightweight X11 solution | AT-SPI 2 + xdotool + scrot; ~471-line Python bridge; X11 only |

### Usage Notes for Bots

1. Prefer semantic accessibility APIs (AX on macOS, AT-SPI on Linux) over pure coordinate clicking when possible.
2. On macOS, Accessibility + Screen Recording must be granted to the helper/app on the Mac GUI.
3. On Linux, enable toolkit-accessibility and ensure AT-SPI is running; Wayland prefers portals, X11 uses xdotool.
4. After joining the mesh, the cloud computer can SSH to the target node and launch the local computer-use MCP/helper if installed.
5. Do not enable computer use on production nodes without explicit approval.

## Open-Source Projects Used

| Project | Purpose | Official Link |
|---------|---------|---------------|
| EasyTier | Decentralized mesh VPN | https://github.com/EasyTier/EasyTier |
| Phone-Eye | AI vision + UI-tree control over adb | https://github.com/boheastill/phone-eye |
| android-adb-control | Agent skill for observe-act-verify on Android | https://github.com/robbin/android-adb-control |
| Android platform-tools | Official adb | https://developer.android.com/tools/releases/platform-tools |

Docs: https://easytier.cn

## Quick Start

1. Confirm the cloud computer is joined to the EasyTier mesh.
2. Once on the mesh, the cloud computer can SSH or otherwise control other online nodes on the same mesh.
3. For tablet control: SSH to the main Mac jump host, then run adb commands **on that Mac**.
4. For domestic traffic: use the local split proxy on the cloud computer.
5. Never modify production configs or restart services without explicit approval.

## Mesh Control Model

After the Grok Bot computer joins the EasyTier mesh:
- It becomes a peer on the mesh network.
- It can directly reach other online mesh nodes (Macs, other computers, exit nodes) by their mesh IPs.
- Typical controls: SSH for command-line access, adb (via a Mac jump host) for Android devices, or other services exposed on the mesh.
- Domestic relay helps maintain stable connectivity when direct P2P is limited.
- All control actions still require proper keys, permissions, and approval for sensitive operations.

### Practical Steps for Bots
1. Confirm EasyTier is running on the cloud computer (`/workspace/easytier/start.sh`).
2. Test connectivity to the target mesh IP with a longer timeout.
3. Use the designated SSH key for the main Mac jump host.
4. For other nodes, follow the same pattern if keys and services are available.
5. Prefer domestic relay when direct links show high latency or packet loss.

| Role | Typical Function | Notes |
|------|------------------|-------|
| Cloud computer | EasyTier client | Start script under `/workspace/easytier/` |
| Domestic relay | EasyTier relay (HK or similar) | Used for stable connectivity when direct P2P is limited |
| Main Mac jump host | SSH entry + adb host | Primary path for tablet control |
| Domestic exit node | Proxy exit | Used for domestic traffic from the cloud computer |
| Backup node | Secondary jump / exit | Often less stable |

- Cloud computer does **not** auto-start EasyTier on boot.
- Prefer connecting via domestic relay when direct links are unstable; do not enable latency-first unless authorized.
- SSH handshake can be slow; set ConnectTimeout 20–30.

## Primary Control Path: Mac Jump Host + Tablet

### SSH to Main Mac
Use the designated key and the mesh IP of the main Mac jump host.  
Set ConnectTimeout to 20–30 seconds.  
After SSH, all further adb and local scripts run on the Mac.

### ADB on Tablet
All adb commands must run **on the Mac**, not from the cloud computer.  
Use the existing wrapper scripts on the Mac after SSH.

Recommended tools:
- Phone-Eye (https://github.com/boheastill/phone-eye) — vision + UI-tree fusion via MCP
- android-adb-control (https://github.com/robbin/android-adb-control) — observe-act-verify skill

Typical flow:
1. Screenshot or UI dump (`uiautomator dump` or Phone-Eye `phone_look`).
2. Identify target (prefer UI tree bounds over pure coordinates).
3. Perform one human-like action (tap/swipe with small offset and press duration 80–150ms).
4. Verify result with a fresh dump or screenshot.
5. Repeat only after verification.

### Computer Use on Tablet
- Prefer structured UI tree + vision fusion when available.
- Install confirmation dialog has a short timeout — prefer handling it locally on the Mac when possible.
- Before releasing the tablet: confirm orientation and proxy settings are clean.
- Only one bot should occupy the tablet at a time; announce occupy and release.

## Domestic Exit Proxy

Cloud computer runs a local split proxy (typically on 8788).
- Domestic domains/sites are routed through the designated domestic exit node.
- Primary exit: the main domestic exit node (proxy port usually 7890).
- Backup exit: secondary node (same port pattern), less reliable.
- Overseas traffic can go direct or via other paths.
- Split proxy mainly supports HTTPS.
- Always verify the proxy is healthy (e.g. curl a domestic site through 8788) before heavy domestic scraping or login-sensitive tasks.

### Usage Notes for Bots
- Prefer the local split proxy for any domestic traffic originating from the cloud computer.
- If the primary exit is down, fall back to the backup exit and report the failure.
- Do not hardcode exit IPs; query internal docs or the infrastructure bot for current addresses.

## Limitations (Current State)

- **No desktop computer use on the main Mac** (remote desktop off, no click-automation tools installed).
- Native computer-use agents (e.g. Codex) require the desktop app running **on the Mac itself** with Screen Recording and Accessibility permissions granted via the GUI (cannot be done purely over SSH).
- Cloud computer restart drops the mesh until EasyTier is manually restarted.
- Known health-check lock issue after EasyTier restart (requires authorized fix).

## Configuration Checklist for Other Bots

1. Ensure EasyTier is running on the cloud computer.
2. Verify SSH key and connectivity to the main Mac jump host.
3. For tablet work: only one bot at a time; announce occupy and release.
4. Route domestic sites through the local split proxy.
5. Do not store passwords, tokens, or keys in chat or logs.
6. Escalate restarts, config changes, or permission grants for approval.
7. When using Phone-Eye or similar, confirm vision endpoint and device serial are correctly set.

## Common Issues

| Symptom | Likely Cause | Action |
|---------|--------------|--------|
| Cannot SSH to Mac | Mesh down or high latency | Check EasyTier process, retry with longer timeout |
| adb device offline | USB disconnected or adb server issue | SSH to Mac and check `adb devices` |
| Domestic site blocked | Proxy down | Check local split proxy and exit node |
| Install dialog timeout | Remote latency | Handle confirmation locally on the Mac |
| Phone-Eye vision fails | Vision server or serial misconfigured | Check env vars and local vision endpoint |

## References

Detailed node lists, exact IPs, hostnames, and current health status live in internal overview documents (not duplicated here for security).  

Related skills: android-adb-control, xiaomi-adb-install, Phone-Eye.

Consult the infrastructure operations bot or internal docs for the latest concrete addresses and status when needed.

For upstream usage details, always refer to the official repositories listed above.
