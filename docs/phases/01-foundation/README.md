# Phase 01 — Foundation and Topology

## Objective
Build an isolated Debian 13 lab to demonstrate reverse-proxy and forward-proxy patterns with host security controls.

## Recorded components
- Debian 13 VM: 192.168.6.141/24
- Gateway/DNS: 192.168.6.2
- Nginx: TCP 80/443
- Squid: TCP 3128
- Gunicorn/Flask: loopback ports 5000 and 5001
- MariaDB: loopback port 3306
- UFW, OpenSSH, AppArmor, systemd

See [architecture diagrams](../../architecture/README.md).

## Inventory commands
```bash
ip -brief address
ip route
sudo ss -lntup
systemctl is-active nginx squid mariadb proxylab.service proxylab-backend2.service
```

Nginx is the inbound web entry point; Squid is the separate outbound forward proxy. Re-run inventory commands for fresh evidence.

## Detailed notes
- [Foundation troubleshooting and diagnostic sequence](troubleshooting.md)
