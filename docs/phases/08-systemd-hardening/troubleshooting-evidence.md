# systemd hardening and Gunicorn runtime troubleshooting

## Useful commands

```bash
sudo systemctl cat proxylab.service
sudo systemctl cat proxylab-backend2.service
sudo systemctl status proxylab.service --no-pager -l
sudo journalctl -u proxylab.service -n 60 --no-pager
sudo systemctl show proxylab.service proxylab-backend2.service -p Id -p ExecStart -p NoNewPrivileges -p PrivateTmp -p PrivateDevices -p ProtectSystem -p ReadWritePaths -p ActiveState
sudo systemd-analyze security proxylab.service --no-pager
```

## Incident: no usable temporary directory

Recorded error:
```text
FileNotFoundError: [Errno 2] No usable temporary directory found in ['/tmp', '/var/tmp', '/usr/tmp', '/opt/proxylab/app']
proxylab.service: Main process exited, code=exited, status=255/EXCEPTION
```

AppArmor logs also showed temporary-file creation denials during the earlier failure.

## Recorded remediation

```bash
sudo mkdir -p /home/debian/.gunicorn/tmp
sudo chown debian:debian /home/debian/.gunicorn/tmp
sudo chmod 700 /home/debian/.gunicorn/tmp
sudo tee /etc/systemd/system/proxylab.service.d/tempdir.conf > /dev/null <<'EOF'
[Service]
ExecStart=
ExecStart=/opt/proxylab/app/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 --worker-tmp-dir /home/debian/.gunicorn/tmp app:app
EOF
sudo systemctl daemon-reload
sudo systemctl reset-failed proxylab.service
sudo systemctl restart proxylab.service
systemctl is-active proxylab.service
curl -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:5000/health
sudo namei -l /home/debian/.gunicorn/tmp
```

Recorded result: service active; backend 1 returned healthy JSON and HTTP 200. The dedicated temp directory is owner-only (mode 700).

## Hardening caveat
A historical drop-in contained `PrivateTmp=yes`; later effective checks showed `PrivateTmp=no` for both backends. Inspect the current effective configuration before describing controls. `NoNewPrivileges=yes`, `PrivateDevices=yes`, `ProtectSystem=strict`, and `ReadWritePaths` should be reported only where the effective unit confirms them. Backend 2 must be checked independently.

## Repeatable troubleshooting sequence
1. Read `systemctl status` and recent journal.
2. Confirm effective `ExecStart`, runtime path and permissions.
3. Inspect AppArmor profile attachment and recent denials.
4. Change one setting at a time.
5. Run `systemctl daemon-reload`, restart only the affected unit and repeat the direct health check.
