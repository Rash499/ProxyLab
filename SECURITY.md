# Security and Responsible Use

## Scope
ProxyLab is an educational lab. Keep it isolated to a trusted network. Do not expose Squid as an open proxy or publish infrastructure details you consider private.

## Never commit
- /etc/proxylab/proxylab.env or real DB_PASSWORD values
- /etc/nginx/ssl/proxylab.key or any private TLS key
- SSH private keys, including proxylab_backup_ed25519
- SQL dumps, backup archives, Restic repository data, tokens, credentials, or unredacted sensitive logs

## Sanitized environment template
```dotenv
DB_HOST=127.0.0.1
DB_PORT=3306
DB_USER=proxyapp
DB_PASSWORD=<set-locally-do-not-commit>
DB_NAME=proxylab
```

This is a placeholder template only.

## TLS caveat
Project notes describe a self-signed certificate with CN 192.168.6.141, valid 9 October 2026 through 9 October 2027. curl -k bypasses trust-chain and hostname verification. Do not report curl -k as certificate validation.

This is an educational project, not a production security guarantee.
