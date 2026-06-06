# How PrivateCore AI relates to existing work

PrivateCore AI does not propose a new machine-learning method. It builds on
established techniques and contributes their integration, operating point and
packaging for an audience that has no off-the-shelf access to them today.

We think being explicit about this is a feature, not a weakness: it tells
prospective users and co-developers exactly what is proven and what is our own
engineering contribution.

## Building blocks (state of the art)

- **Model cascades / routing** — handling most queries with a small model and
  escalating only the hard tail to a larger one. See FrugalGPT
  (arXiv:2305.05176), RouteLLM, and cascade-routing (ICLR 2025).
- **Knowledge distillation** — specialising a small "student" model from a
  larger "teacher" model is the standard way to build capable on-device models.
- **Grounding LLMs in structured data** — structured RAG, text-to-SQL and
  relational grounding to keep answers tied to real records.
- **Cloud-edge collaboration** for cost, privacy and trustworthiness is itself
  an active research area (survey arXiv:2510.13890).

## What is different here

- **Operating point:** the cascade runs on-premise on consumer NAS hardware; the
  large model is the only component that may touch the cloud.
- **Local continual specialisation:** the small local expert grows from the
  cascade's own escalations on the user's private data, without data leaving the
  device.
- **Deterministic source of truth:** facts live in a relational database; the LLM
  is confined to ingest and an agent layer, so outputs are auditable and the
  system cannot hallucinate facts that exist in the database.
- **Measured, not asserted:** efficiency is reported as a rising local share on
  real data.
- **Target group:** small businesses with sensitive data and little or no
  in-house IT (craft businesses are our first focus), via a one-click,
  self-hostable appliance. Two value axes: data sovereignty, and cost
  consolidation (one self-hosted box instead of a stack of cloud subscriptions).

## References

- FrugalGPT — https://arxiv.org/abs/2305.05176
- A Unified Approach to Routing and Cascading for LLMs (ICLR 2025) — https://files.sri.inf.ethz.ch/website/papers/dekoninck2024cascaderouting.pdf
- Survey: Collaborating Small and Large Language Models (Cloud-edge Privacy, Trustworthiness) — https://arxiv.org/abs/2510.13890
- On-Device LLM Personalization — https://arxiv.org/abs/2311.12275
- Survey: LLM-based Text-to-SQL (TKDE 2025) — https://github.com/DEEP-PolyU/Awesome-LLM-based-Text2SQL
