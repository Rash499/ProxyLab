# Phase 09 — Backup and Disaster Recovery

## Recorded backup
Source: /var/backups/proxylab/20261020_130223/
Files: proxylab.sql, proxylab-configs.tar.gz, SHA256SUMS

Backup destination:
- Host: 192.168.6.143
- Directory: /home/debian/proxylab-backups/20261020_130223/
- File ownership/mode recorded: debian:debian, 600
- Temporary source staging directory /home/debian/proxylab-transfer was removed after transfer.

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

## Important limitation
The files were copied into the backup VM user's home directory, not recorded as being placed inside /backup/restic-repo. Checksum match proves transfer integrity, not successful application restoration or whole-VM recovery.

## Remaining restore work
- [ ] Restore SQL dump to a separate test database.
- [ ] Validate schema and known records.
- [ ] Extract configuration archive to an isolated directory.
- [ ] Validate expected files and permissions without exposing secrets.
- [ ] Test application recovery from restored artifacts.
- [ ] Record exact commands, outputs, cleanup and recovery time.
- [ ] State whether this is file-, database-, application- or full-VM recovery.

## Detailed notes
- [Backup transfer troubleshooting and restore evidence](transfer-troubleshooting.md)

## Detailed notes
- [Backup transfer troubleshooting and restore evidence](transfer-troubleshooting.md)
