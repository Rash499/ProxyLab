# Firewall and SSH troubleshooting

## Intended exposure

| Port | Intended access |
|---|---|
| 22/tcp | Authorized management source |
| 80/tcp | Lab clients; HTTP redirect |
| 443/tcp | Lab clients; HTTPS |
| 3128/tcp | Approved lab subnet only |
| 5000/5001 | Loopback only |
| 3306 | Loopback only |

## Read-only checks

```bash
sudo ufw status verbose
sudo ss -lntup
sudo sshd -T | grep -Ei 'passwordauthentication|permitrootlogin|pubkeyauthentication|allowusers'
sudo systemctl status ssh --no-pager -l
sudo sshd -t
```

Earlier notes report firewall restrictions and key-based SSH, but the latest security assessment does not contain the full raw output. Re-run the checks before claiming current verification.

## Safe remediation process
1. Keep the current SSH session open and confirm console access.
2. Confirm the management source address before restricting SSH.
3. Validate SSH config with `sshd -t`.
4. Add an intended allow rule before removing an existing rule.
5. Open a second session to verify access before closing the first.
6. Capture final effective settings and firewall rules.

Do not blindly reset firewall rules over SSH; it can lock you out. Never commit private keys or secret-bearing output.
