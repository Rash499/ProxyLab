# Phase 08 — systemd Hardening and Recovery

## Recorded settings
Backend 1's inspected drop-ins included NoNewPrivileges=yes, PrivateDevices=yes, ProtectSystem=strict, kernel-protection settings, RestrictSUIDSGID=yes, and a write allowance for /home/debian/.gunicorn.

Both backend services were later recorded with:
```text
NoNewPrivileges=yes
PrivateTmp=no
```

PrivateTmp=no is intentional in the documented lab after PrivateTmp=yes contributed to startup problems with the current AppArmor policy. Do not claim private /tmp isolation is enabled.

## Inspect both units
```bash
sudo systemctl cat proxylab.service
sudo systemctl cat proxylab-backend2.service
sudo systemctl show proxylab.service proxylab-backend2.service -p Id -p ExecStart -p NoNewPrivileges -p PrivateTmp -p PrivateDevices -p ProtectSystem -p ReadWritePaths -p ActiveState
sudo systemd-analyze security proxylab.service --no-pager
```

A historical Gunicorn error said there was no usable temporary directory. The workaround used /home/debian/.gunicorn/tmp owned by debian:debian with mode 700, and backend 1 used --worker-tmp-dir there.

Recorded post-reboot validation reported all five services active, HTTPS health and products returned HTTP 200, and the profile was in enforce mode. Security scores are indicators, not guarantees; inspect each backend separately.

## Detailed notes
- [systemd hardening and Gunicorn temp-directory incident](troubleshooting-evidence.md)
