# Phase 02 — MariaDB and Least Privilege

## Recorded configuration
- MariaDB 11.8.6
- Database: proxylab
- Application user: proxyapp
- Listener: 127.0.0.1:3306
- Intended grants: SELECT, INSERT, UPDATE, DELETE on proxylab.*
- Seeded records: Laptop, Keyboard, Mouse, Monitor

## Service checks
```bash
sudo apt update
sudo apt install -y mariadb-server mariadb-client
sudo systemctl enable --now mariadb
sudo systemctl status mariadb --no-pager
sudo ss -lntp | grep 3306
```

Example SQL (use a unique strong password locally; never commit it):
```sql
CREATE DATABASE proxylab;
CREATE USER 'proxyapp'@'localhost' IDENTIFIED BY '<choose-a-strong-password-locally>';
GRANT SELECT, INSERT, UPDATE, DELETE ON proxylab.* TO 'proxyapp'@'localhost';
FLUSH PRIVILEGES;
```

The real environment file is /etc/proxylab/proxylab.env and must remain outside Git. Recorded directory/file permissions were root:debian, directory 750 and file 640. The products endpoint returned HTTP 200 with four records in a recorded test; the request used curl -k, so TLS trust was not checked.

## Detailed notes
- [Database validation and troubleshooting](troubleshooting.md)
