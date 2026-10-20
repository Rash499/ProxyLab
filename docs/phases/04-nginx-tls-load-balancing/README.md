# Phase 04 — Nginx, TLS and Load Balancing

## Recorded upstream
```nginx
upstream proxylab_backend {
    server 127.0.0.1:5000;
    server 127.0.0.1:5001;
}
```

Recorded site behavior: port 80 redirects to HTTPS; port 443 terminates TLS using /etc/nginx/ssl/proxylab.crt and /etc/nginx/ssl/proxylab.key; TLS protocols set to TLSv1.2 and TLSv1.3; requests proxy to the upstream with Host, X-Real-IP, X-Forwarded-For, and X-Forwarded-Proto headers.

## Inspect and validate
```bash
sudo sed -n '1,160p' /etc/nginx/sites-enabled/proxylab
sudo nginx -t
sudo ss -lntp | grep -E ':(80|443)\b'
```

## Recorded load balancing
```bash
for i in $(seq 1 20); do
  printf 'Request %02d: ' "$i"
  curl -k --max-time 5 -sS https://127.0.0.1/health || echo "REQUEST FAILED"
  echo
done
```
Both backend identities appeared in the retained 20-request output; one pair of consecutive requests hit backend 1. A later six-request test alternated backend identities.

## TLS caveat
Notes describe a self-signed certificate with CN 192.168.6.141, valid 9 Oct 2026–9 Oct 2027. curl -k bypasses trust validation. Test chain and hostname separately without -k. The shown upstream uses default round-robin behavior; no active health-check directive was recorded, so do not claim active health checking.

## Detailed notes
- [Nginx/TLS diagnostics and known limitations](nginx-tls-diagnostics.md)
- [Nginx troubleshooting and 502 investigation](troubleshooting.md)
