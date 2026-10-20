# Phase 07 — AppArmor Confinement

## Recorded profile
/etc/apparmor.d/opt.proxylab.app.venv.bin.gunicorn

## Checks
```bash
sudo aa-status
sudo apparmor_parser -Q -T /etc/apparmor.d/opt.proxylab.app.venv.bin.gunicorn
sudo journalctl -k --since "10 minutes ago" --no-pager | grep 'apparmor="DENIED"' | tail -10
```

The parser check returned without error, but -Q checks without loading the profile. A later status check showed proxylab-gunicorn in enforce mode; a bounded recent-log query returned no matching denials.

## Historical denial investigation
Historical kernel logs on 20 October contained AppArmor DENIED events for getattr on dev/null under this profile. The base abstraction contained /dev/null rw, but the exact cause was not conclusively established. Later short-window query showed no new matching events; that does not guarantee future absence.

Keep the profile in enforce mode, investigate specific denials, and avoid disabling AppArmor or making broad permission changes to work around service issues.
