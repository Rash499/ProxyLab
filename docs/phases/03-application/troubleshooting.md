# Flask and Gunicorn troubleshooting

## Direct health checks

```bash
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:5000/health
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:5001/health
sudo journalctl -u proxylab.service -n 50 --no-pager
sudo journalctl -u proxylab-backend2.service -n 50 --no-pager
```

Recorded output from the direct checks:
```text
{"backend":"backend-1","service":"ProxyLab API","status":"healthy"}
HTTP 200
{"backend":"backend-2","service":"ProxyLab API","status":"healthy"}
HTTP 200
```

## Issue: missing MariaDB development utility

Historical install error: `mariadb_config: command not found`.

Recorded fix:
```bash
sudo apt install -y libmariadb-dev libmariadb-dev-compat build-essential
```
Then rerun the connector install inside the project's virtual environment.

## Issue: address already in use

A manually started Flask process may occupy a port already assigned to systemd-managed Gunicorn.

```bash
sudo ss -lntp | grep -E ':5000|:5001'
sudo lsof -i :5000
systemctl status proxylab.service --no-pager -l
```

Identify the process before stopping it. Avoid killing arbitrary PIDs; prefer stopping the confirmed duplicate process and restarting the intended unit.

## API test notes
Recorded tests returned HTTP 400 for missing name, negative price and malformed JSON; valid temporary product creation returned HTTP 201; missing product returned HTTP 404. A temporary security-test row was deleted and the four seeded rows remained. Database exception details were logged server-side and clients received a generic HTTP 500 body in a mocked error test.
