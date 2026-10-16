---
name: lan-device-control
description: "Configure and operate local LAN computers and devices via EasyTier mesh, SSH, and adb. Use when a Grok Bot needs to control or access physical machines in the internal mesh network."
type: workflow
lifecycle: active
---

# LAN Device Control — Operate Mesh Network Devices

Use this skill when a Grok Bot needs to access or control physical computers/devices in the internal EasyTier mesh.

Primary path today: SSH to the main Mac jump host → adb to the connected tablet. Desktop computer use on the Mac is not enabled.

## Open-Source Projects Used

| Project | Purpose | Official Link |
|---------|---------|---------------|
| EasyTier | Decentralized mesh VPN | https://github.com/EasyTier/EasyTier |
| Phone-Eye | AI vision + UI-tree control over adb | https://github.com/boheastill/phone-eye |
| android-adb-control | Agent skill for observe-act-verify on Android | https://github.com/robbin/android-adb-control |
| Android platform-tools | Official adb | https://developer.android.com/tools/releases/platform-tools |

Docs: https://easytier.cn

## Quick Start

1. Confirm mesh connectivity (EasyTier running on cloud computer).
2. For tablet control: SSH to the main Mac jump host, then run adb commands **on that Mac**.
3. For domestic traffic: use the local split proxy on the cloud computer.
4. Never modify production configs or restart services without explicit approval.

## Network Overview (Generic)

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

### ADB on Tablet
All adb commands must run **on the Mac**, not from the cloud computer.  
Use the existing wrapper scripts on the Mac after SSH.

Typical flow (from Phone-Eye / android-adb-control style):
1. Screenshot or UI dump (`uiautomator dump` or Phone-Eye `phone_look`).
2. Identify target.
3. Perform one human-like action (tap/swipe with small offset and press duration).
4. Verify result.
5. Repeat.

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
- Always verify the proxy is healthy before heavy domestic scraping or login-sensitive tasks.

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
