# Evidence, Status and Limitations

| Area | Recorded status | Limitation / next step |
|---|---|---|
| Nginx upstream | Validated | Both loopback backends were in the inspected config. |
| HTTPS /health | Passed in recorded test | Used curl -k; TLS trust/hostname not validated. |
| HTTPS /products | HTTP 200, four seeded records | curl -k does not validate TLS. |
| Direct backends | Passed | Ports 5000 and 5001 returned expected backend identity. |
| Load balancing | Both identities observed | A 20-request transcript included consecutive backend-1 responses; later six-request test alternated. |
| AppArmor profile | Enforce mode observed | Bounded log query only; does not prove no future denials. |
| AppArmor parser | Command returned without error | -Q does not load the profile. |
| systemd hardening | Partial/settings-specific | Both services later recorded NoNewPrivileges=yes, PrivateTmp=no; inspect each unit. |
| Squid/UFW/SSH | Previously inspected | Re-capture exact current output for final validation. |
| Backup transfer | Checksums matched | Transfer was to backup VM user's home directory. |
| Full restore | Pending full demonstration | Perform isolated SQL/config restore and application checks. |
| Backend failure/recovery | Pending | Controlled backend stop is not recorded as executed. |
| TLS trust/hostname | Pending | Test using intended hostname and trusted CA without -k. |

## Recorded health responses
```json
{"backend":"backend-1","service":"ProxyLab API","status":"healthy"}
{"backend":"backend-2","service":"ProxyLab API","status":"healthy"}
```

The products endpoint was recorded as HTTP 200 with four seeded records: Laptop (id 1), Keyboard (id 2), Mouse (id 3), Monitor (id 4).

## Backup record
- Source: /var/backups/proxylab/20261020_130223/
- Destination host: 192.168.6.143
- Destination: /home/debian/proxylab-backups/20261020_130223/
- Files: proxylab.sql, proxylab-configs.tar.gz, SHA256SUMS
- Source and destination checksums: both files reported OK.
- Files were not recorded as placed in /backup/restic-repo.
- A successful copy and checksum match proves transfer integrity, not application recovery.

Dates and results reflect the retained project notes. Re-run checks before claiming current live state.
