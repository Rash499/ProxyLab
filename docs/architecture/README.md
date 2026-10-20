# Architecture

## End-to-end lab topology

```mermaid
flowchart TB
  Client["LAN clients<br/>192.168.6.0/24"] -->|"HTTP / HTTPS"| FW["Host firewall"]
  Client -->|"Forward proxy :3128"| FW
  Admin["Administrator"] -->|"SSH :22 / public key"| FW
  subgraph VM["Debian 13 ProxyLab — 192.168.6.141"]
    FW --> N["Nginx :80 / :443<br/>HTTP redirect + TLS termination"]
    FW --> S["Squid :3128<br/>ACL-controlled forward proxy"]
    N --> B1["Gunicorn / Flask<br/>127.0.0.1:5000"]
    N --> B2["Gunicorn / Flask<br/>127.0.0.1:5001"]
    B1 --> DB[("MariaDB<br/>127.0.0.1:3306")]
    B2 --> DB
    AA["AppArmor profile"] -. confines .-> B1
    AA -. confines .-> B2
    SD["systemd hardening"] -. isolates .-> B1
    SD -. isolates .-> B2
  end
  S --> Internet["Permitted external destinations"]
```

## Reverse proxy request flow

```mermaid
sequenceDiagram
  participant C as Client
  participant N as Nginx
  participant G as Gunicorn/Flask
  participant D as MariaDB
  C->>N: HTTP request :80
  N-->>C: 301 redirect to HTTPS
  C->>N: HTTPS request :443
  N->>G: Proxy to selected upstream :5000 or :5001
  G->>D: Query application data
  D-->>G: Result
  G-->>N: API response
  N-->>C: HTTPS response
```

## Forward proxy flow

```mermaid
flowchart LR
  C["Lab client"] -->|"TCP 3128"| S["Squid"]
  S --> ACL{"ACL decision"}
  ACL -->|Allowed| W["External destination"]
  ACL -->|Denied| D["Access denied"]
```

## Boundaries
- Nginx is the client-facing reverse proxy; backends bind to loopback.
- Squid is a separate outbound forward-proxy path.
- MariaDB is recorded on loopback, not directly exposed to the LAN.
- AppArmor and systemd are distinct process-confinement/isolation layers.
- Diagrams describe documented configuration, not a live network scan.
