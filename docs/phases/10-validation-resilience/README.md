# Phase 10 — Security Validation and Resilience

## Recorded API checks
```bash
echo "=== Backend 1 ==="
curl --max-time 5 -sS http://127.0.0.1:5000/health
echo
echo "=== Backend 2 ==="
curl --max-time 5 -sS http://127.0.0.1:5001/health
echo
echo "=== HTTPS through Nginx ==="
for i in $(seq 1 6); do curl -k --max-time 5 -sS https://127.0.0.1/health || echo "FAILED"; echo; done
echo "=== Recent service errors ==="
sudo journalctl -u proxylab.service --since "10 minutes ago" --no-pager -p err
```

Recorded outputs: direct backends returned their healthy JSON with HTTP 200; initial HTTPS requests succeeded and showed both backend identities. HTTPS checks used `-k`, so TLS trust was not checked.

## Controlled backend failure and recovery — completed 21 October 2026

**Result: passed for both single-backend service-stop scenarios.**

| Test | Observed result |
|---|---|
| Baseline | Nginx, MariaDB and both backends active; ports 443, 5000 and 5001 listening; `nginx -t` successful; HTTPS `/health` and `/products` returned HTTP 200. |
| Stop backend 1 (`proxylab.service`) | Backend 1 became inactive; backend 2 remained active; 8/8 health requests returned HTTP 200 from `backend-2`; products endpoint returned HTTP 200 and all four products. |
| Restart backend 1 | Both services active; 8/8 health requests returned HTTP 200 and reached both backend identities; products endpoint returned HTTP 200. Gunicorn logs showed backend 1 listening on port 5000. |
| Stop backend 2 (`proxylab-backend2.service`) | Backend 2 became inactive; backend 1 remained active; 8/8 health requests returned HTTP 200 from `backend-1`; products endpoint returned HTTP 200 and all four products. |
| Restart backend 2 and final validation | Nginx, MariaDB and both backend services active; both backend ports listening; `nginx -t` successful; 10/10 health requests returned HTTP 200 and reached both identities; products endpoint returned HTTP 200 and all four products. |

### Evidence and limitations
- The complete command-by-command transcript and observed results are in [Backend failure/recovery runbook](backend-failover-runbook.md).
- Direct requests to the stopped backend port were not captured; its direct-failure response is not claimed as verified.
- The test demonstrates observed failover for controlled systemd stops, not every application-crash scenario or production-grade HA. The inspected upstream does not have an active-health-check directive.
- The check of `/var/log/nginx/error.log` returned no lines, but the virtual-host config specifies `/var/log/nginx/proxylab_error.log`; use the configured path in future checks.
- HTTPS testing used `curl -k`; certificate trust and hostname verification remain pending.
- No formal recovery-time measurement or RTO/RPO validation was performed.

## Isolated database/application recovery — 21 October 2026

**Result: recovery API and restricted database access verified for the isolated test instance.**

- Recovery Gunicorn master PID 3375 was configured with `DB_HOST=127.0.0.1`, `DB_USER=proxyapp_recovery`, and `DB_NAME=proxylab_restore_test`; its working directory was the extracted app under `/home/debian/proxylab-restore-test.TVpkdX/config-inspection/opt/proxylab/app`.
- The grant check showed SELECT-only access on `proxylab_restore_test.*` for `proxyapp_recovery@localhost`.
- Interactive TCP authentication as that account succeeded. `SELECT id, name FROM products ORDER BY id;` returned IDs 1–4: Laptop, Keyboard, Mouse and Monitor. `SELECT DATABASE(), CURRENT_USER();` returned `proxylab_restore_test` and `proxyapp_recovery@localhost`.
- The recovery API at `127.0.0.1:5055` returned HTTP 200 from both `/health` and `/products`; the products endpoint returned the four expected records.

This confirms the isolated restored test database is readable through the least-privilege account and that the recovery API responds. It does **not** prove production cutover, full-VM recovery, TLS certificate trust/hostname validation, or measured RTO/RPO. The SQL dump and configuration archive remain sensitive artifacts and must not be committed.

## Detailed notes
- [Security assessment evidence, commands and results](security-assessment.md)
- [Controlled backend failure/recovery runbook and recorded results](backend-failover-runbook.md)
- [Backup and disaster-recovery restore evidence](../09-backup-disaster-recovery/README.md)

TLS trust/hostname verification remains pending; test without `-k` using the intended hostname and trusted CA.
