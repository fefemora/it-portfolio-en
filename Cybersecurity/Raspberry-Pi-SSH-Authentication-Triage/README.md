# SSH Authentication Triage on Raspberry Pi

![Blue Team](https://img.shields.io/badge/Blue_Team-Authentication_Triage-059669?style=flat-square) ![OpenSSH](https://img.shields.io/badge/OpenSSH-journalctl-14283F?style=flat-square) ![Lab](https://img.shields.io/badge/Lab-Verified-64748B?style=flat-square)

[← Cybersecurity](../README.md) · [Raspberry Pi DNS lab](../../Networking/Raspberry-Pi-DNS-Lab/README.md) · [Versión en español](https://github.com/fefemora/it-portfolio-es/blob/main/Cybersecurity/Raspberry-Pi-SSH-Authentication-Triage/README.md)

While administering a Raspberry Pi 3 Model B from Windows with PuTTY, I mistyped my SSH password. I reviewed the OpenSSH journal and compared the rejected attempts with a subsequent successful login. I wanted to determine whether the successful login belonged to the same connection or a new one.

## Journal review

On November 25, 2026, I reviewed the 30 most recent SSH service entries on Raspbian 13 (Trixie). I used ISO output to preserve the local time and the UTC−03:00 offset.

```bash
sudo journalctl -u ssh.service -n 30 -o short-iso --no-pager
```

Excerpt transcribed from the PuTTY screenshot. I replaced the username and source IPv6 address with consistent placeholders and joined lines that PuTTY had wrapped.

```text
2026-11-25T16:32:41-03:00 pi3-lab sshd-session[2830]: Failed password for <lab-user> from <client-ipv6> port 57461 ssh2
2026-11-25T16:32:50-03:00 pi3-lab sshd-session[2830]: Failed password for <lab-user> from <client-ipv6> port 57461 ssh2
2026-11-25T16:32:55-03:00 pi3-lab sshd-session[2830]: Connection closed by authenticating user <lab-user> <client-ipv6> port 57461 [preauth]
2026-11-25T16:33:18-03:00 pi3-lab sshd-session[2833]: Accepted password for <lab-user> from <client-ipv6> port 63839 ssh2
```

## Timeline

| Local time | Evidence | Source port / PID |
|---|---|---|
| 16:32:41 and 16:32:50 | Two password rejections | `57461` / `2830` |
| 16:32:55 | Connection closed before authentication completed | `57461` / `2830` |
| 16:33:18 | Password accepted and session opening confirmed by PAM in the next entry | `63839` / `2833` |

## Analysis and disposition

I correlated the entries by time, username, source IPv6 address, source port, and `sshd-session` PID. The two rejections share a port and PID with the closure marked `[preauth]`. The successful login came 23 seconds after the closure, with the same username and IPv6 address and a different port and PID. The sequence shows that authentication succeeded on a new connection.

I classified this episode as legitimate lab activity because I could relate it to my password entry error and later PuTTY login. The journal establishes the authentication sequence. Attribution to the operator depends on the lab context, and the review is limited to the selected events.
