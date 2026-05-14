# PrivateCore AI

**A sovereign edge-AI framework for consumer NAS hardware.**
Local LLMs · ZFS storage · Nextcloud & Paperless integration · 100 % data sovereignty.

> **Project status:** Pre-alpha — public repository established for transparency.
> Implementation work scheduled to begin pending the outcome of the NLnet / NGI Zero Commons Fund review (decision expected Q3 2026).

---

## Mission

The dominant deployment model for "personal AI" today still relies on third-party cloud services. For privacy-conscious users — and for entire classes of professional users (legal, medical, public sector) — that model is fundamentally incompatible with their data-protection obligations and personal preferences.

**PrivateCore AI** addresses this gap by providing a complete, reproducible reference architecture for hosting modern AI workloads on commodity NAS-class hardware. Sensitive data — personal documents, calendars, communications, health and wellness data streams — never leaves the device.

The project is explicitly positioned as a **community-reusable Common**: an open framework, not a product. All deployment automation, configuration guides, and the AI-bridge code are published under a permissive open-source licence.

## Architecture overview

PrivateCore AI is an **infrastructure-first** project. Rather than building yet another chat front-end, it provides the missing layer that makes existing privacy-first software run reliably on real consumer hardware:

```
┌──────────────────────────────────────────────────────────────┐
│                   Consumer NAS hardware                      │
│  ┌────────────────────────────────────────────────────────┐  │
│  │            Proxmox VE  (host, on internal SSD)         │  │
│  │                                                        │  │
│  │   ZFS pool (NVMe)         PCIe pass-through            │  │
│  │      │                          │                      │  │
│  │      ▼                          ▼                      │  │
│  │  VM disks               TrueNAS Scale  →  HDDs         │  │
│  │   ├─ Nextcloud              (RAIDZ1)                   │  │
│  │   ├─ Paperless-ngx                                     │  │
│  │   ├─ Local LLM runtime         │                       │  │
│  │   ├─ Vector DB (RAG)           ▼                       │  │
│  │   └─ AI-Bridge          NFS / SMB shares               │  │
│  └────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────┘
```

Core technology choices:

- **Virtualisation:** Proxmox VE
- **Storage:** ZFS (NVMe pool for VMs, HDD pool via TrueNAS Scale for archives and backups)
- **AI runtime:** abstraction layer over Ollama / llama.cpp / Intel OpenVINO / IPEX-LLM
- **Vector database:** Qdrant or Chroma (decision deferred to Task 2)
- **Integrations:** Nextcloud (Assistant connector), Paperless-ngx (RAG ingestion), Immich (optional photo search)
- **Deployment automation:** Ansible + Docker Compose; Terraform where useful

See [ROADMAP.md](./ROADMAP.md) for the detailed milestone plan.

## Reference hardware

The initial reference platform is the **UGreen DXP6800 Pro** with an Intel Core i5-1235U (10C / 12T, Intel Iris Xe Graphics with 80 EUs), DDR5 SODIMM memory, and an NVMe + HDD storage mix. The deployment automation, however, is explicitly designed to be **hardware-agnostic**: any x86_64 system with virtualisation support, sufficient memory, and ZFS-capable storage should be a viable target. Validation on a second platform is part of the project scope (Task 5).

## Roadmap

See [ROADMAP.md](./ROADMAP.md).

The project is structured around five milestones aligned with the NLnet / NGI Zero Commons Fund proposal (application code `2026-06-1ac`):

1. **High-performance infrastructure research** — ZFS-cache tuning for LLM/RAG workloads on Proxmox.
2. **AI integration for privacy ecosystems** — AI-Bridge for Nextcloud and Paperless-ngx; local vector-DB pipeline.
3. **Sovereign agent framework** — vendor-agnostic ingestion layer; secure local UI.
4. **Security & open-source documentation** — network hardening, threat model, publishable Ansible/Docker deployment.
5. **Hardware benchmarking & validation** — cross-platform validation of the hardware-agnostic claim.

## Funding

PrivateCore AI has applied for support from the **NLnet Foundation** under the [NGI Zero Commons Fund](https://nlnet.nl/commonsfund/) (application `2026-06-160`, submitted May 2026). Decision is expected in Q3 2026. This README will be updated accordingly.

If the application is successful, the published deliverables of the funded work will be released under the Apache-2.0 licence in this repository.

## Contributing

PrivateCore AI is a young project maintained by a single engineer. Contributions, issue reports, and design discussions are welcome — please note that during the pre-funding phase, response times may be irregular.

Once the funded work begins, the project will commit to a documented issue-response SLA and a monthly development update.

If you have built something adjacent — a NAS-friendly LLM deployment, a privacy-respecting AI agent, a self-hosted vector database integration — please open a Discussion: cross-pollination is explicitly part of the project's goals.

## Licence

Code, scripts, and configuration in this repository are licensed under the **Apache License 2.0** (see [LICENSE](./LICENSE) and [NOTICE](./NOTICE)).

Documentation and write-ups are licensed under **CC BY 4.0** unless otherwise noted.

## Maintainer

Markus Dröscher — Graz, Austria
Contact: opening a GitHub Discussion is preferred over direct email during the pre-alpha phase.

---

*PrivateCore AI is not affiliated with NLnet Foundation, the NGI initiative, UGreen, Nextcloud, Paperless-ngx, Proxmox Server Solutions GmbH, or iXsystems (TrueNAS). All trademarks belong to their respective owners.*
