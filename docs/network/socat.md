---
title: socat
tags:
  - network
---

## usage

```bash
socat TCP4-LISTEN:8022,fork TCP4:upstreamhost:8022
```

- listen locahost port 8022
- send trafic to upstreamhost:8022
