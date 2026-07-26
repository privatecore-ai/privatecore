# PrivateCore AI

**A sovereign edge-AI framework for consumer NAS hardware.**
Local LLMs · ZFS storage · Nextcloud & Paperless integration · 100 % data sovereignty.

> **Project status:** Actively developed. The reference deployment has been running in production for one household since mid-2026, as a nights-and-weekends project.
> Development continues independently of the pending NLnet / NGI Zero Commons Fund review — see [Funding](#funding).

---
## How this relates to existing work

PrivateCore AI builds on established techniques (model cascades, knowledge
distillation, structured RAG) and contributes their integration into a sovereign,
self-hostable appliance. For an honest comparison to prior art, and what is
genuinely our own contribution, see [RELATED-WORK.md](RELATED-WORK.md).

## Mission

The dominant deployment model for "personal AI" today still relies on third-party cloud services. For privacy-conscious users — and for entire classes of professional users (legal, medical, public sector) — that model is fundamentally incompatible with their data-protection obligations and personal preferences.

**PrivateCore AI** addresses this gap by providing a complete, reproducible reference architecture for hosting modern AI workloads on commodity NAS-class hardware. Sensitive data — personal documents, calendars, communications, health and wellness data streams — never leaves the device.

The project is explicitly positioned as a **community-reusable Common**: an open framework, not a product. All deployment automation, configuration guides, and the AI-bridge code are published under a permissive open-source licence.

## What is implemented today

- **Infrastructure:** Proxmox VE / TrueNAS Scale reference setup on consumer NAS hardware (UGreen DXP6800 Pro), ZFS throughout (NVMe pool for VMs, HDD pool for archives and backups), strict per-service network segmentation (the database host has no outbound internet at all).
- **Personal data platform:** PostgreSQL 17 + PostGIS with row-level security, a multi-user data model, and a per-user **consent registry** — no document belonging to a user is ever processed by a cloud model without that user's recorded, revocable consent.
- **Document ingest pipeline:** mobile capture → WebDAV inbox → webhook-driven ingest service (HMAC-authenticated) → EXIF stripping → QR decoding with cryptographic cross-check against Austrian fiscal receipt signatures (RKSV) → vision-LLM classification and type-specific extraction → schema-validated storage. See the [architecture write-up](./docs/architecture/01-document-ingest-pipeline.md).
- **Extraction-proposal contract (C#/.NET):** a schema-versioned, forward-compatible JSON contract that reconciles multi-variant LLM extraction outputs before anything is committed to the domain tables — currently being implemented test-first (xUnit).
- **MCP integration:** the platform's data (files, calendars, task boards) is exposed to Claude through Model Context Protocol workflows, with a tiered privacy model deciding what an LLM may ever see.
- **Benchmark methodology:** every LLM call is recorded with model name, prompt version, token counts, latency, and cost — so the planned migration from cloud vision models to local ones can be measured per field, not guessed.

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

## What is open — and what is not

Everything needed to **reproduce the platform** is (or will be) public in this repository: architecture documentation, deployment automation, configuration guides, the AI-bridge code, and the write-ups in `docs/architecture/`. Code and scripts are Apache-2.0; documentation is CC BY 4.0.

Two things are deliberately **not** part of the open framework: the maintainer's personal data and deployment specifics (redacted or generalised in all published material), and a separate commercial evaluation of Austrian receipt-processing use cases that builds *on top of* the framework. The Common is the framework itself — what anyone builds on it, including the maintainer, is their own.

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

PrivateCore AI has applied for support from the **NLnet Foundation** under the [NGI Zero Commons Fund](https://nlnet.nl/commonsfund/) (application `2026-06-1ac`, submitted May 2026 — resubmission of 2026-06-160 with refined risk section and itemised hardware budget). Decision is expected in Q3 2026. This README will be updated accordingly.

If the application is successful, the published deliverables of the funded work will be released under the Apache-2.0 licence in this repository.

## Contributing

PrivateCore AI is maintained by a single engineer, and both prospective **users** and **co-developers** are explicitly welcome — that is what a Common is for. Open an Issue or a Discussion if you want to reproduce the setup or pick up a roadmap item together. Please note that during the pre-funding phase, response times may be irregular.

Once the funded work begins, the project will commit to a documented issue-response SLA and a monthly development update.

If you have built something adjacent — a NAS-friendly LLM deployment, a privacy-respecting AI agent, a self-hosted vector database integration — please open a Discussion: cross-pollination is explicitly part of the project's goals.

## Licence

Code, scripts, and configuration in this repository are licensed under the **Apache License 2.0** (see [LICENSE](./LICENSE) and [NOTICE](./NOTICE)).

Documentation and write-ups are licensed under **CC BY 4.0** unless otherwise noted.

## Maintainer

Markus Dröscher — Graz, Austria
Contact: opening a GitHub Discussion is preferred over direct email.

---

*PrivateCore AI is not affiliated with NLnet Foundation, the NGI initiative, UGreen, Nextcloud, Paperless-ngx, Proxmox Server Solutions GmbH, or iXsystems (TrueNAS). All trademarks belong to their respective owners.*
