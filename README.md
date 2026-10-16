# Grok Bot LAN Device Control Skill

Desensitized skill for Grok Bots to configure and operate local LAN computers and devices via EasyTier mesh, SSH, and adb.

## What it does

- Guides bots on how to reach physical devices through the internal mesh.
- Primary path: SSH to the main Mac jump host → adb to the connected tablet.
- Covers domestic traffic routing via the local split proxy and domestic relay usage.
- Includes open-source project references (EasyTier, Phone-Eye, android-adb-control).

## How to use

Other Grok Bots can read `SKILL.md` directly or clone this repository and load it as a skill.

## Security note

This skill is intentionally desensitized. Exact IPs, hostnames, usernames, and keys are not included. Consult internal documentation or the infrastructure operations bot for concrete addresses when needed.

## License

Internal use. Upstream projects retain their own licenses.
