# Command Index

Run these on the lab VM after reviewing the impact. Fault-injection commands are intentionally not automated here.

## Service state
```bash
systemctl is-active proxylab.service proxylab-backend2.service nginx mariadb squid
systemctl is-enabled proxylab.service proxylab-backend2.service nginx mariadb squid
systemctl --no-pager -l status proxylab.service
sudo systemctl cat proxylab.service
sudo systemctl cat proxylab-backend2.service
```
Recorded post-reboot status: all five services were reported active.

## Backend and HTTPS health
```bash
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:5000/health
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:5001/health
curl -k --max-time 5 -sS -w '\nHTTP %{http_code}\n' https://127.0.0.1/health
curl -k --max-time 5 -sS -w '\nHTTP %{http_code}\n' https://127.0.0.1/products
```
Note: -k skips TLS certificate verification.

## Nginx
```bash
sudo sed -n '1,160p' /etc/nginx/sites-enabled/proxylab
sudo nginx -t
sudo ss -lntp | grep -E ':(80|443)\b'
```

## AppArmor
```bash
sudo aa-status
sudo apparmor_parser -Q -T /etc/apparmor.d/opt.proxylab.app.venv.bin.gunicorn
sudo journalctl -k --since "10 minutes ago" --no-pager | grep 'apparmor="DENIED"' | tail -10
```
The parser command with -Q checks the profile without loading it.

## systemd hardening
```bash
sudo systemctl show proxylab.service -p ExecStart -p NoNewPrivileges -p PrivateTmp -p PrivateDevices -p ProtectSystem -p ReadWritePaths
sudo systemctl show proxylab-backend2.service -p ExecStart -p NoNewPrivileges -p PrivateTmp -p PrivateDevices -p ProtectSystem -p ReadWritePaths
sudo systemd-analyze security proxylab.service --no-pager
```
Both services were later recorded with NoNewPrivileges=yes and PrivateTmp=no. Do not claim PrivateTmp isolation is enabled.

## Squid, firewall and SSH
```bash
sudo systemctl status squid --no-pager -l
sudo ss -lntp | grep ':3128' || true
sudo grep -nE '^[[:space:]]*(acl|http_access|http_port)' /etc/squid/squid.conf
sudo ufw status verbose
sudo sshd -T | grep -Ei 'passwordauthentication|permitrootlogin|pubkeyauthentication|allowusers'
```

## Backup checksum verification
```bash
cd /var/backups/proxylab/20261020_130223
sha256sum -c SHA256SUMS
```
Recorded output: `proxylab.sql: OK` and `proxylab-configs.tar.gz: OK`, also verified on backup VM 192.168.6.143.

## Failover discovery — read-only, test still pending
```bash
sudo ss -lntp | grep -E '127.0.0.1:(5000|5001)' || true
systemctl list-units --type=service --all --no-legend | grep -Ei 'proxylab|gunicorn' || true
sudo systemctl cat proxylab.service
```
Confirm service-to-port mapping before stopping any service. Keep the SSH session open and restart the stopped backend promptly.
