# Architecture

## End-to-end lab topology

This is the main system diagram, including the application path and the separately hosted backup/recovery path. The backup transfer and checksum verification are recorded; restore validation remains pending.

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
    DB -->|"mysqldump"| BK["Backup set<br/>SQL dump + config archive<br/>SHA256SUMS"]
  end
  S --> Internet["Permitted external destinations"]
  subgraph Recovery["Backup VM — 192.168.6.143"]
    DST["Received backup files"]
    VERIFY["Checksum verification<br/>SQL: OK · Configs: OK"]
    RESTORE["Isolated restore<br/>PENDING"]
    DST --> VERIFY
    VERIFY -. "restore not yet validated" .-> RESTORE
  end
  BK --> DST
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

## Backup and recovery architecture

This detailed view expands the backup portion of the main topology. It represents the **recorded ProxyLab file-transfer path**, separate from the Restic repository used in the backup-platform lab. The solid path was transferred and checksum-verified; the dotted recovery path is planned and remains pending validation.

```mermaid
flowchart LR
  subgraph Source["ProxyLab source VM — 192.168.6.141"]
    APP["Application + MariaDB"]
    SNAP["Timestamped backup directory<br/>/var/backups/proxylab/20261020_130223"]
    SQL["proxylab.sql"]
    CFG["proxylab-configs.tar.gz"]
    MANIFEST["SHA256SUMS"]
    APP --> SNAP
    SNAP --> SQL
    SNAP --> CFG
    SNAP --> MANIFEST
  end
  SNAP -->|"SSH / SCP<br/>dedicated key"| DEST["Backup VM — 192.168.6.143"]
  subgraph Destination["Verified transfer destination"]
    FILES["/home/debian/proxylab-backups/20261020_130223<br/>files recorded with mode 600"]
    CHECK["sha256sum -c SHA256SUMS<br/>proxylab.sql: OK<br/>proxylab-configs.tar.gz: OK"]
    FILES --> CHECK
  end
  DEST --> FILES
  CHECK -. "next step — restore not yet validated" .-> RESTORE["Isolated restore workspace"]
  RESTORE -. "validate after restore" .-> VALIDATE["Restore SQL into test DB<br/>check schema and known rows<br/>check config permissions<br/>test application health"]
```

### Backup/recovery interpretation

- **Recorded:** backup artifacts were transferred from `192.168.6.141` to `192.168.6.143` over SSH/SCP using a dedicated key.
- **Recorded:** `sha256sum -c SHA256SUMS` returned `proxylab.sql: OK` and `proxylab-configs.tar.gz: OK`.
- **Destination caveat:** verified files were in `/home/debian/proxylab-backups/20261020_130223`, not `/backup/restic-repo`. Do not interpret this diagram as proof that the transferred files are Restic snapshots.
- **Pending:** restore the SQL dump to a separate test database, inspect/extract the config archive in an isolated directory, validate data and permissions, test application recovery, and record elapsed time. Dotted arrows denote planned work, not a passed recovery test.
- A checksum match proves integrity against the checksum manifest; it does not prove database consistency, application recovery, whole-VM recovery, immutability, or off-site protection.

## Boundaries

- Nginx is the client-facing reverse proxy; backends bind to loopback.
- Squid is a separate outbound forward-proxy path.
- MariaDB is recorded on loopback, not directly exposed to the LAN.
- AppArmor and systemd are distinct process-confinement/isolation layers.
- Diagrams describe documented configuration, not a live network scan.
