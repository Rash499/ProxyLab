# Phase 03 — Flask API and Gunicorn

## Endpoints
- GET /health
- GET /products
- GET /products/<id>
- POST /products

Application path: /opt/proxylab/app.

| Backend | Bind | Service |
|---|---|---|
| 1 | 127.0.0.1:5000 | proxylab.service |
| 2 | 127.0.0.1:5001 | proxylab-backend2.service |

## Health checks
```bash
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:5000/health
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:5001/health
```

Recorded outputs:
```json
{"backend":"backend-1","service":"ProxyLab API","status":"healthy"}
{"backend":"backend-2","service":"ProxyLab API","status":"healthy"}
```
Both returned HTTP 200 in retained evidence.

## Gunicorn temporary directory issue
A historical startup error reported no usable temporary directory among /tmp, /var/tmp, /usr/tmp, and /opt/proxylab/app. The documented workaround created /home/debian/.gunicorn/tmp with owner debian:debian and mode 700; backend 1 used --worker-tmp-dir /home/debian/.gunicorn/tmp. Inspect the current unit before changing it.

Never publish /etc/proxylab/proxylab.env. Keep backends bound to loopback and avoid returning internal database exceptions to API clients.

## Detailed notes
- [API validation, error handling and incidents](api-validation-and-incidents.md)
- [Flask/Gunicorn troubleshooting](troubleshooting.md)
