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
A checksum match proves the copied files match the checksum manifest. It does not prove a database or application restore works. The files were placed in the backup VM user's home directory, not inside `/backup/restic-repo`; this is not an independent off-host/off-site copy.

## Remaining restore validation
- [ ] Inspect archive contents with `tar -tzf proxylab-configs.tar.gz`.
- [ ] Extract to an isolated directory.
- [ ] Restore SQL to a separate test database.
- [ ] Verify tables and known records.
- [ ] Validate config paths and permissions without printing secret contents.
- [ ] Test application recovery in isolation.
- [ ] Record commands, output, cleanup, elapsed time and exact recovery scope.
