# Backup integrity and disaster-recovery assessment

## Transfer flow

```mermaid
flowchart LR
  SRC["ProxyLab VM<br/>192.168.6.141"] --> B["/var/backups/proxylab/20261020_130223"]
  B -->|"SSH / SCP"| DEST["Backup VM<br/>192.168.6.143"]
  DEST --> D["/home/debian/proxylab-backups/20261020_130223"]
  D --> V["sha256sum -c SHA256SUMS"]
  V --> OK["proxylab.sql: OK<br/>proxylab-configs.tar.gz: OK"]
```

## Recorded transfer incident
The first copy attempt did not complete; the remote directory was empty. Source files were `root:root`, mode 600, so the normal `debian` account could not read them directly. Temporary copies were staged under `/home/debian/proxylab-transfer/`, owner `debian:debian`, mode 600. SCP was retried with the dedicated SSH key and `IdentitiesOnly=yes`. After successful remote verification, temporary staging files were removed and the original source backup remained intact.

## Recorded verification command

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

The remote listing contained the SQL dump, configuration archive and checksum manifest; recorded files had mode 600.

## What this proves and does not prove
A checksum match proves the transferred files match the checksum manifest. It does not prove a database or application restore succeeds. The files were placed in the backup VM user's home directory, not in `/backup/restic-repo`. The Restic repository disk is a virtual disk attached to that VM, so this is not an independent off-host/off-site copy.

## Restore acceptance checklist
- [ ] Inspect archive contents using `tar -tzf proxylab-configs.tar.gz`.
- [ ] Extract into an isolated directory with restrictive permissions.
- [ ] Restore SQL into a separate test database, never the live database.
- [ ] Validate schema and known rows.
- [ ] Review configuration paths and permissions without printing secrets.
- [ ] Test application recovery in isolation.
- [ ] Record commands, outputs, cleanup, elapsed time and whether the exercise is file-, database-, application- or whole-VM recovery.

Never publish the SQL dump, config archive, Restic repository, SSH private key or environment file to this public repository.
