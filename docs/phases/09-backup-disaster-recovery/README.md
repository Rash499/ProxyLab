# Phase 09 — Backup and Disaster Recovery

## Recorded backup

Source: `/var/backups/proxylab/20261020_130223/`

Files: `proxylab.sql`, `proxylab-configs.tar.gz`, `SHA256SUMS`

Backup destination:
- Host: 192.168.6.143
- Directory: `/home/debian/proxylab-backups/20261020_130223/`
- File ownership/mode recorded: `debian:debian`, `600`
- Temporary source staging directory `/home/debian/proxylab-transfer` was removed after transfer.

## Recorded checksum output

```text
proxylab.sql: OK
proxylab-configs.tar.gz: OK
```

The same results were recorded after transfer to the backup VM.

Remote verification command:

```bash
ssh -i /home/debian/.ssh/proxylab_backup_ed25519 -o IdentitiesOnly=yes debian@192.168.6.143 "cd /home/debian/proxylab-backups/20261020_130223 && sha256sum -c SHA256SUMS && ls -lh"
```

Do not publish the private key or the backup files.

## Isolated database and application recovery validation — 21 October 2026

**Result: database read access and recovery API checks passed for the isolated test instance.** This validates the test database and test application process, not a production cutover or full-VM recovery.

### Recovery process configuration

The recovery Gunicorn master (PID 3375 at the time of inspection) was running from the extracted application directory:

`/home/debian/proxylab-restore-test.TVpkdX/config-inspection/opt/proxylab/app`

The inspected process environment contained:

```text
DB_HOST=127.0.0.1
DB_USER=proxyapp_recovery
DB_NAME=proxylab_restore_test
```

No database password was printed or recorded. The recovery instance listened on `127.0.0.1:5055`; its launcher used the existing application virtual environment's Python and Gunicorn WSGI entry point.

### Least-privilege account

Command:

```bash
sudo mariadb -e "SHOW GRANTS FOR 'proxyapp_recovery'@'localhost';"
```

Observed grants (password hash intentionally omitted):

```text
GRANT USAGE ON *.* TO proxyapp_recovery@localhost
GRANT SELECT ON proxylab_restore_test.* TO proxyapp_recovery@localhost
```

The account had SELECT-only access to the isolated restore database. The password hash from MariaDB output is deliberately not copied into this documentation.

### Direct authenticated database test

Command run interactively; the password was entered at the prompt and was not placed in command history or command arguments:

```bash
mariadb --protocol=TCP --host=127.0.0.1 --user=proxyapp_recovery --password proxylab_restore_test
```

Queries:

```sql
SELECT id, name FROM products ORDER BY id;
SELECT DATABASE(), CURRENT_USER();
```

Observed product rows:

```text
1  Laptop
2  Keyboard
3  Mouse
4  Monitor
```

Observed connection identity:

```text
DATABASE()             CURRENT_USER()
proxylab_restore_test  proxyapp_recovery@localhost
```

This confirms that the dedicated recovery account authenticated and read all four expected product records from the test database. The separate administrative check also confirmed that `proxylab_restore_test.products` exists.

### Recovery API checks

The recovery API on `127.0.0.1:5055` returned HTTP 200 for both `/health` and `/products`. The health response identified the backend as `recovery-test`; the products response contained the four expected records: Laptop, Keyboard, Mouse and Monitor.

These endpoint results, combined with the inspected process environment and direct authenticated SQL query, support that the recovery test instance was configured for `proxylab_restore_test` and could read the expected data.

### Evidence limitations

- This was an isolated database/application recovery test; no production cutover was performed.
- The health endpoint alone does not prove database connectivity; the direct SQL query and products endpoint provide the database-backed evidence.
- The four known rows were checked, but this was not a full row-by-row comparison against a source checksum or a large-data integrity test.
- The exact duration, RTO and RPO were not measured.
- This does not demonstrate full-VM recovery, recovery of the separate backup server, or restoration of every host-level service.
- Keep passwords, password hashes, `DB_PASSWORD`, private SSH keys, TLS private keys, SQL dumps and configuration archives out of public source control.

## Recovery checklist

- [x] Verify the SQL/configuration backup files by SHA-256 before and after transfer.
- [x] Confirm the restored test database contains the expected `products` table.
- [x] Verify the recovery account is restricted to SELECT on `proxylab_restore_test.*`.
- [x] Authenticate as `proxyapp_recovery` and query the four expected product rows.
- [x] Check recovery API `/health` and `/products` responses (HTTP 200).
- [ ] Validate TLS trust and hostname without `curl -k`.
- [ ] Perform and document a controlled production cutover only if separately planned and safe.
- [ ] Measure recovery duration and define/test RTO/RPO.
- [ ] Test full-VM recovery separately if that is a project requirement.

## Detailed notes

- [Backup transfer troubleshooting and restore evidence](transfer-troubleshooting.md)
