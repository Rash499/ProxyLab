# Security assessment matrix

| Area | Recorded evidence | Status / limitation |
|---|---|---|
| Nginx upstream | Two loopback upstreams inspected | Validated at test time |
| HTTPS /health and /products | HTTP 200 responses recorded | Tests used `curl -k`; certificate trust/hostname not proven |
| Load balancing | Both backend identities observed; six later requests alternated | Controlled backend failure/recovery still pending |
| API input validation | Invalid inputs HTTP 400; valid insert HTTP 201; missing product HTTP 404 | Recorded test; temporary row was removed |
| DB error disclosure | Mocked DB error returned generic HTTP 500 | Does not represent the separate live DB outage |
| MariaDB outage | Dependent Gunicorn units stopped and Nginx returned 502; services recovered after restart | systemd dependency behavior recorded |
| AppArmor | Enforce mode observed; later 10-minute query had no matching denial | Historical /dev/null denial root cause not conclusively established |
| systemd hardening | Backend 1 score 4.6 at one point; settings inspected | Recheck both effective units; later `PrivateTmp=no` |
| Gunicorn temp directory | Dedicated owner-only temp path resolved startup failure | Recorded fix validated |
| Squid ACLs | Service/listener/ACL inspected earlier | Capture fresh complete output and functional allow/deny tests |
| Firewall / SSH | Previously inspected | Capture fresh current output |
| Backup transfer | Source/destination SHA-256 checks matched | Transfer integrity only; restore evidence must be scoped accurately |
| TLS trust | Not proven by `curl -k` | Pending test with correct hostname/SAN and trusted CA |
| Backend failover | Intentional backend stop not recorded in latest assessment | Pending; use failover runbook |

## Evidence labels
- **Validated:** test was executed and supporting output retained.
- **Historical inspection:** checked earlier, but complete current transcript is not retained here.
- **Partial:** only some properties were tested.
- **Pending:** not executed or not proven by available evidence.
