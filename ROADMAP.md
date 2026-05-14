# PrivateCore AI — Roadmap

This roadmap describes the planned development of PrivateCore AI as a community-reusable open-source framework for sovereign edge-AI on consumer NAS hardware.

The work is structured around the five tasks of the **NLnet / NGI Zero Commons Fund** proposal `2026-06-160` (submitted May 2026; decision expected Q3 2026).

The timeline below assumes a positive funding decision and a project start in autumn 2026. Independent development on documentation, the public reference architecture, and community foundations begins immediately, regardless of the funding outcome.

---

## Project phases

### Phase 0 · Foundations (pre-funding, ongoing)

Goals:

- Establish the public project (this repository, organisation profile, contribution guidelines).
- Document the reference hardware platform and the deployment-target rationale.
- Document the initial threat model in draft form.
- Set up communication channels (GitHub Discussions; longer-term: Matrix room or Fediverse account).

Deliverables:

- `README.md`, `ROADMAP.md`, `LICENSE`, `NOTICE` — done.
- `CONTRIBUTING.md` — pending.
- Draft architecture diagram — pending.
- Public statement of scope and explicit out-of-scope items (no medical-device positioning, no regulated-domain claims during Phase A).

---

### Phase 1 · NLnet Task 1 — High-Performance Infrastructure Research

**Effort:** 50 hours.
**Status:** Planned (pending funding).

Research and document optimal ZFS configuration for LLM-inference and RAG-indexing workloads on Proxmox VE:

- ARC sizing and tuning for AI workloads (recordsize, compression, primarycache policy).
- Evaluation of L2ARC and special-vdev for the NVMe layer.
- Setup and tuning of TrueNAS Scale as a Proxmox guest with PCIe HBA / disk pass-through.
- I/O benchmark methodology and reproducible test harness.

Deliverables:

- Architecture decision record (ADR) on storage layout.
- Benchmark script and result dataset (CSV + plots) in the repository.
- Configuration playbooks for Proxmox host and TrueNAS guest.

Acceptance criteria:

- Reproducible benchmarks on the reference hardware.
- Documented configuration is applicable to at least one alternative platform (handover to Task 5).

---

### Phase 2 · NLnet Task 2 — AI Integration for Privacy Ecosystems

**Effort:** 100 hours.
**Status:** Planned (pending funding).

Develop the **AI-Bridge** — a connector layer between local LLM runtimes and existing privacy-first applications:

- Backend abstraction supporting Ollama, llama.cpp, and Intel OpenVINO / IPEX-LLM as interchangeable runtimes.
- Integration with **Nextcloud** Assistant (existing extension API).
- Integration with **Paperless-ngx** for document-question-answering via Retrieval-Augmented Generation.
- Local vector-database integration (Qdrant or Chroma; decision documented as an ADR).
- Reference RAG pipeline with documented prompt templates.

Deliverables:

- `ai-bridge` component (Python or Go service) published in the repository.
- Nextcloud app manifest and integration guide.
- Paperless-ngx integration guide.
- Reference flow: "ask your own documents" — fully working on the reference hardware.

Acceptance criteria:

- End-user can ask a question about a Nextcloud document and receive a grounded answer without external network access.
- All components are reproducibly deployable via Ansible.

---

### Phase 3 · NLnet Task 3 — Sovereign Agent Framework

**Effort:** 80 hours.
**Status:** Planned (pending funding).

Generalise the AI-Bridge into a **modular agent framework**:

- Vendor-agnostic ingestion layer for additional personal-data sources: IMAP, calendar (CalDAV / ICS), file exports, activity files (FIT, TCX, GPX), generic JSON/CSV.
- Pluggable connector model so contributors can add new data sources without modifying core code.
- Local web UI for interacting with the personal agent, with per-user authentication and isolation.

Deliverables:

- `connector-framework` component with at least four reference connectors.
- Local web UI prototype.
- Documentation: "How to write a connector in 30 minutes".

Acceptance criteria:

- A non-expert user can extend the system with a new data source by following the connector tutorial.
- Per-user data isolation is verified by a security review checklist.

---

### Phase 4 · NLnet Task 4 — Security & Open-Source Documentation

**Effort:** 45 hours.
**Status:** Planned (pending funding).

Harden the reference architecture and publish a community-grade deployment Common:

- VLAN-segmented network topology with documented reverse-proxy (Caddy or Traefik) and TLS configuration.
- Threat model (STRIDE for the system, LINDDUN for the data flows) published as `docs/threat-model.md`.
- Default `fail2ban` profiles and intrusion-detection baseline.
- Public Ansible playbook and Docker Compose stacks for the complete reference deployment.
- Documented update and patch procedure with rollback path.

Deliverables:

- `deploy/` directory with the full deployment automation.
- `docs/threat-model.md`.
- `SECURITY.md` with disclosure policy and CVE-tracking process.

Acceptance criteria:

- A reviewer who has not contributed to the project can deploy the reference architecture from scratch in under two hours using only the public documentation.

---

### Phase 5 · NLnet Task 5 — Hardware Benchmarking & Validation

**Budget:** € 3 000 itemised hardware (see proposal).
**Status:** Planned (pending funding).

Validate the **hardware-agnostic** claim and produce comparative benchmarks:

- Cross-platform deployment on a second hardware platform with a different chipset.
- Inference benchmarks (latency, tokens-per-second, memory footprint) across iGPU, NPU stick (Coral, Hailo), and CPU-only.
- Power and thermal benchmarks under sustained AI load.
- End-user UI/UX validation on smartphone and tablet form factors.
- Backup-restore validation cycles using external storage.

Deliverables:

- `benchmarks/` directory with raw data, scripts, and a written report.
- Compatibility matrix in the documentation.

Acceptance criteria:

- The reference architecture is verifiably operational on at least two distinct hardware platforms.
- Energy-per-token figures are published for each tested configuration.

---

## Beyond the NLnet scope

The work funded by the NGI Zero Commons Fund delivers the **open-source foundation**. Independently, the maintainer plans a complementary commercial offering (pre-configured appliance, support, branded use-case bundles) that is intended to fund continued maintenance and security updates of the openly published framework. The two layers are kept cleanly separated.

A longer-term Phase B — addressing regulated professional users (legal, medical, public sector) with the corresponding certifications — is **explicitly out of scope** of the Commons Fund proposal and will, if pursued at all, be the subject of a separate funding instrument.

---

## How this roadmap may change

This document reflects the current plan. Concrete deliverables, dependencies, or scope may be adjusted based on:

- The outcome of the NLnet review (scope reduction or acceptance with conditions).
- Community feedback during the early phases.
- Technical findings during Task 1 (storage research) that materially affect later tasks.

All material changes will be tracked in the repository history of this file.

---

*Last updated: 2026-05-14*
