# Security Assessment

This file expands the ProxyLab security assessment.

```mermaid
flowchart LR
  C[Client] --> N[Nginx]
  N --> B1[Backend 1]
  N --> B2[Backend 2]
  B1 --> DB[(MariaDB)]
  B2 --> DB
  C --> S[Squid]
```
