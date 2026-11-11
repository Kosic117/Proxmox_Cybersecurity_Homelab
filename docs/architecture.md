
# Network Architecture

```mermaid
flowchart TB
    Remote[Remote Client] --> TS[Tailscale Overlay]
    TS --> Gateway[Debian Gateway]
    Gateway --> Lab[Isolated Lab Network]
    Lab --> MS[Metasploitable 2]
```
  
