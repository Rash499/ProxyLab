# MariaDB troubleshooting and API verification

## Connection and permission checks

```bash
sudo systemctl status mariadb --no-pager
sudo ss -lntp | grep ':3306'
sudo mariadb -e "SHOW DATABASES LIKE 'proxylab';"
sudo mariadb -e "SELECT User, Host, plugin FROM mysql.user WHERE User='proxyapp';"
sudo mariadb -e "SHOW GRANTS FOR 'proxyapp'@'localhost';"
sudo mariadb proxylab -e "SELECT id,name,price FROM products ORDER BY id;"
```

The intended listener is 127.0.0.1:3306, and application access is through least-privilege account `proxyapp@localhost`, not database root.

## Recorded seed data
The `/products` endpoint returned HTTP 200 with four expected records: Laptop (id 1, 250000.00), Keyboard (id 2, 15000.00), Mouse (id 3, 7500.00), Monitor (id 4, 45000.00).

## Troubleshooting
- **Access denied:** compare account host, current password loaded from the protected environment file, and grants. Do not display the password.
- **Connection refused:** check service state and listener.
- **Unexpected data:** inspect selected database/table and order by ID before making changes.
- **HTTP 500:** inspect backend journal; client response should not contain SQL credentials or internal exception details.

Recorded security test: a mocked DB exception returned HTTP 500 and `{"error":"Internal server error"}`, without exposing the simulated sensitive detail. A separate real MariaDB-stop experiment stopped dependent backends and produced Nginx HTTP 502; the units' `Requires=mariadb.service` dependency explains that behavior. Services were restarted and the endpoints recovered.

Never run a destructive test against the live database. Use a separate test database and remove temporary rows after validation.
