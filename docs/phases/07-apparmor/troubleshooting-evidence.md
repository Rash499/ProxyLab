# AppArmor troubleshooting evidence

## Profile
- Profile name: `proxylab-gunicorn`
- File: `/etc/apparmor.d/opt.proxylab.app.venv.bin.gunicorn`

## Commands

```bash
sudo grep -Rnl 'profile proxylab-gunicorn' /etc/apparmor.d 2>/dev/null || true
sudo find /etc/apparmor.d -maxdepth 3 -type f \( -iname '*proxylab*' -o -iname '*gunicorn*' \) -print
sudo aa-status | grep -A 2 -B 2 'proxylab-gunicorn'
sudo apparmor_parser -Q -T /etc/apparmor.d/opt.proxylab.app.venv.bin.gunicorn
sudo journalctl -k --since "10 minutes ago" --no-pager | grep 'apparmor="DENIED"' | tail -10
```

## Historical issue

Kernel audit logs showed two `getattr` denials for `dev/null` under `proxylab-gunicorn` on 20 October 2026. The base abstraction contained `/dev/null rw,`; a later check showed `/proc/<pid>/root/dev/null` present. The precise root cause was not proven.

Recorded output from a later status check included:
```text
6 processes are in enforce mode.
0 processes are in complain mode.
```
The subsequent bounded DENIED query returned no matching output.

## Interpretation
- Historical entries remain evidence of a past denial.
- A later empty log query means no matching entries were found in that window only.
- `apparmor_parser -Q -T` checks the profile without loading it.
- Earlier troubleshooting observed complain-mode activity; later audit showed enforce mode.

## Safe investigation steps
1. Capture the exact denial timestamp, operation, path, profile and PID.
2. Check the current profile and process attachment.
3. Reproduce a representative request.
4. Inspect only the relevant path and abstraction.
5. Apply the narrowest evidence-based rule, validate syntax and reload intentionally.
6. Repeat health tests and inspect a fresh kernel-log window.

Do not disable AppArmor or broaden permissions based on a stale denial alone.
