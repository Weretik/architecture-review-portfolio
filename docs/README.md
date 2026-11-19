# Architecture documents

The architecture I build and review against. It is the same for both backends and both frontends that I run in production. Names of clients and business modules are left out.

| Document | What it covers |
| --- | --- |
| [c4.md](c4.md) | C4 context and container diagrams: users, outside systems and the deployable parts |
| [module-map.md](module-map.md) | How the system is split into modules and layers, and how the parts connect |
| [dependency-rules.md](dependency-rules.md) | What may depend on what, and which automatic check enforces each rule |
| [adr/](adr/) | Key architecture decisions: what was chosen, what was rejected and why |
| [runbook-api-5xx.md](runbook-api-5xx.md) | How I diagnose server errors in production, from the cheapest check to the most expensive |
