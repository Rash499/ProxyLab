# Phase 05 — Squid Forward Proxy

## Recorded design
Squid listens on TCP 3128 and is intended for clients on 192.168.6.0/24. ACL rules and the host firewall are intended to restrict access. Squid is a forward proxy, not an Nginx upstream.

## Inspection
```bash
sudo systemctl status squid --no-pager -l
sudo ss -lntp | grep ':3128' || true
sudo grep -nE '^[[:space:]]*(acl|http_access|http_port)' /etc/squid/squid.conf
```

For functional tests, use one permitted destination and one deliberately denied destination; capture the client command, status and relevant Squid log line. Never test or configure an open proxy.

Earlier sessions inspected service status, listener and ACLs. Exact raw output was not fully retained in this repository package, so re-run the commands and add current output before marking this phase freshly revalidated.

## Detailed notes
- [Squid ACL troubleshooting and validation](troubleshooting.md)

## Detailed notes
- [Squid ACL troubleshooting and validation](troubleshooting.md)
