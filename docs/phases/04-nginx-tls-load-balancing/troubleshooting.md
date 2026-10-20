# Nginx and TLS troubleshooting

## Inspect config and listener

```bash
sudo sed -n '1,160p' /etc/nginx/sites-enabled/proxylab
sudo nginx -t
sudo ss -lntp | grep -E ':(80|443)\b'
sudo tail -n 50 /var/log/nginx/proxylab_error.log
sudo tail -n 30 /var/log/nginx/proxylab_access.log
```

Do not reload Nginx if `nginx -t` reports a syntax error.

## Recorded upstream

```nginx
upstream proxylab_backend {
    server 127.0.0.1:5000;
    server 127.0.0.1:5001;
}
```

## HTTP status interpretation
- HTTP root path returned 301: consistent with redirecting HTTP to HTTPS.
- HTTPS root path returned 404: expected because the API has no homepage route.
- HTTPS `/health` and `/products` returned HTTP 200 in recorded tests.
- Nginx returned HTTP 502 during the recorded MariaDB stop experiment because the dependent backend services were stopped.

## 502 troubleshooting
1. Check both service states.
2. Check the 5000/5001 listeners.
3. Test each backend directly.
4. Inspect both Gunicorn journals.
5. Inspect Nginx error log for the exact upstream connect failure.
6. Check MariaDB state and service dependency if both backends are down.

## TLS limitation
The lab certificate was recorded as self-signed, CN 192.168.6.141. Historical tests used `curl -k`, so they prove the endpoint answered but do not prove certificate trust or hostname validation. Complete a test with the correct hostname/IP SAN and trusted CA without `-k`.
