# Ports, Paths and Service Map

| Port or path | Component | Recorded purpose |
|---|---|---|
| 22/tcp | OpenSSH | Administrative access |
| 80/tcp | Nginx | HTTP to HTTPS redirect |
| 443/tcp | Nginx | TLS termination and reverse proxy |
| 3128/tcp | Squid | Forward proxy, intended lab subnet only |
| 127.0.0.1:5000 | Gunicorn / Flask | Backend 1 |
| 127.0.0.1:5001 | Gunicorn / Flask | Backend 2 |
| 127.0.0.1:3306 | MariaDB | Local database |
| /opt/proxylab/app | Application | Flask application |
| /etc/proxylab/proxylab.env | Secret config | Never commit |
| /etc/nginx/ssl/proxylab.crt | TLS certificate | Certificate file |
| /etc/nginx/ssl/proxylab.key | TLS private key | Secret; never commit |
| /var/backups/proxylab/20261020_130223/ | Source backup | Historical backup directory |
| /home/debian/proxylab-backups/20261020_130223/ on 192.168.6.143 | Backup copy | Historical transferred files |
| /backup/restic-repo | Restic repository | Transferred SQL/config files were not recorded as placed here |
