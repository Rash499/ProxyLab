# Phase 06 — SSH and Firewall Controls

## Intended access policy
- TCP 22 — SSH administration
- TCP 80 — HTTP redirect
- TCP 443 — HTTPS
- TCP 3128 — Squid restricted to the lab subnet
- Gunicorn ports 5000/5001 and MariaDB 3306 — loopback only

This is a summary of the documented design; inspect live rules before relying on them.

## Read-only checks
```bash
sudo ufw status verbose
sudo ss -lntup
sudo sshd -T | grep -Ei 'passwordauthentication|permitrootlogin|pubkeyauthentication|allowusers'
```

Keep an existing SSH session open when changing access rules. Confirm console/recovery access before applying changes that could lock you out. Never publish private key contents. Project notes report key-only SSH and firewall restrictions; recapture current effective settings before final sign-off.

## Detailed notes
- [Firewall and SSH troubleshooting](troubleshooting.md)
