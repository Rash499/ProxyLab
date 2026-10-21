# Backup transfer troubleshooting and restore evidence

## Recorded backup set
Source: `/var/backups/proxylab/20261020_130223/`
- `proxylab.sql`
- `proxylab-configs.tar.gz`
- `SHA256SUMS`

Destination: `debian@192.168.6.143:/home/debian/proxylab-backups/20261020_130223/`

## Issue and resolution
The first transfer attempt did not complete; the remote directory was empty. Source files were owned by `root:root` with mode 600, so the `debian` account could not read them directly. Temporary copies were staged in `/home/debian/proxylab-transfer/`, owned by `debian:debian`, mode 600. SCP was retried using the dedicated key and `IdentitiesOnly=yes`. After remote checksum verification, temporary copies were removed; the original source backup was preserved.

## Recorded remote verification

```bash
ssh -i /home/debian/.ssh/proxylab_backup_ed25519 \
  -o IdentitiesOnly=yes \
  debian@192.168.6.143 \
  "cd /home/debian/proxylab-backups/20261020_130223 && sha256sum -c SHA256SUMS && ls -lh"
```

Observed:
```text
proxylab.sql: OK
proxylab-configs.tar.gz: OK
```

Remote listing confirmed the archive, SQL dump and checksum manifest. Recorded file modes were 600.

## What checksum validation means
A checksum match proves the copied files match the checksum manifest; by itself it does not prove recovery. The isolated restore was subsequently tested: SQL was restored to `proxylab_restore_test`, the four expected product rows were read using the SELECT-only recovery account, and the recovery API returned HTTP 200 for `/health` and `/products`. The files were placed in the backup VM user's home directory, not inside `/backup/restic-repo`; this is not an independent off-host/off-site copy.

## Completed isolated restore validation
- [x] Inspect/extract the configuration archive in an isolated workspace without exposing secret contents.
- [x] Restore SQL to the separate `proxylab_restore_test` database.
- [x] Verify the `products` table and four expected records: Laptop, Keyboard, Mouse and Monitor.
- [x] Verify SELECT-only grants for `proxyapp_recovery@localhost`.
- [x] Test isolated recovery API `/health` and `/products`; both returned HTTP 200.

## Remaining recovery scope
- [ ] Validate TLS trust and hostname without `curl -k`.
- [ ] Measure recovery duration and define/test RTO/RPO.
- [ ] Test full-VM recovery or production cutover separately if required.
