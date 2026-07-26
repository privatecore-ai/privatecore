# Architecture Write-up 1: The Document Ingest Pipeline

*From a phone photo to a validated database row in under a minute — without sensitive data leaking anywhere it shouldn't.*

**Status of this document:** describes the pipeline as designed and currently implemented in the PrivateCore AI reference deployment. Names, hosts, and addresses are generalised; the real deployment uses the same structure.

---

## The problem

A household produces a steady stream of paper: receipts, contracts, official letters, bank statements, warranties. The usual "solutions" either mean manual filing (nobody keeps that up) or shipping every document to a cloud OCR service (unacceptable for a privacy-first platform, and legally delicate for documents that belong to other family members).

The design goal for the ingest pipeline was therefore threefold:

1. **Effortless capture.** Photographing a receipt must be a two-tap action; anything slower dies in practice.
2. **Consent before cloud.** No document belonging to a person is ever processed by a cloud model without that person's recorded, revocable consent — enforced by the pipeline, not by good intentions.
3. **Measurable migration path.** Phase A deliberately uses a cloud vision LLM for extraction quality; Phase B will switch to local vision models. Every call is instrumented so that the switch is a measured decision, not a leap of faith.

## The flow

```
Phone photo
   │  Share Sheet → Shortcut ("AI-relevant? yes/no · whose document?")
   ▼
WebDAV upload  →  /Inbox/AI/  (dedicated upload user, write-only scope)
   │
   ▼  Nextcloud Workflow Engine
POST webhook → ingest service          (HMAC-SHA256 signed)
   │
   ├─ 1. parse owner slug from filename
   ├─ 2. consent check against the consent registry   ← hard gate
   ├─ 3. EXIF strip (GPS + device metadata)           ← defence in depth
   ├─ 4. QR decode + fiscal-signature parse (RKSV)
   ├─ 5. vision LLM: classification  (document type, confidence)
   ├─ 6. vision LLM: type-specific extraction (versioned prompts)
   ├─ 7. cross-check: LLM output vs. cryptographic receipt data
   ├─ 8. schema validation (Pydantic)
   ├─ 9. transactional insert (vendors ⭢ documents ⭢ positions ⭢ tax)
   └─ 10. archive move + push notification
        (any failure → error folder + machine-readable error file)
```

Two hosts are involved, segmented on purpose: the **database host** has no outbound internet at all; the **ingest host** may reach exactly two endpoints (the LLM API and the internal Nextcloud) — enforced with default-drop nftables rules. If the ingest service is ever compromised, it has nowhere interesting to go.

## Design decisions worth defending

### Consent is a database row, not a checkbox

The platform is multi-user: documents belong to individual family members or to the household. The consent registry records, per user and per processing scope, whether cloud processing is permitted. The pipeline checks it **before** any bytes leave the network (step 2) — a document whose owner has no active consent is quarantined with a human-readable reason, and no LLM call happens. Household-level documents (e.g. a shared utility bill) skip the per-person check but are still only exposed to the cloud role through views with anonymised vendor names.

This sounds heavy for a home system. It is the point of the system: the same mechanics that protect a family member's documents are what a GDPR-conscious organisation needs to protect a client's.

### The receipt lies less than the model: cryptographic cross-checking

Austrian receipts carry an RKSV QR code — a signed fiscal record containing date, receipt number, and amounts. The pipeline decodes it *before* the LLM stage and afterwards cross-checks the LLM's extraction against the cryptographically signed values (date within one day, receipt number string-equal after normalisation, total within €0.02). Agreement means the extraction is trustworthy without human review; disagreement flags the document for review instead of silently storing a hallucinated total.

This is the general pattern I would recommend for any LLM extraction pipeline: **find the ground truth that already exists in the input and make the model agree with it.** OCR confidence scores estimate; signatures know.

### Prompts are versioned artifacts

Every LLM call records the model name, a `prompt_version`, token counts, latency, and cost into a `processing_runs` table. Prompts are two-stage (classify first, then extract with a type-specific prompt) and every prompt change bumps the version. The payoff comes in Phase B: the same documents can be re-run through a local vision model with identical prompts, and quality can be compared **per field** against the Phase-A baseline. The privacy trade-off of Phase A (cloud vision on stripped images) is only justified because it produces this baseline.

### Failure paths are first-class

Anything that goes wrong — unparseable filename, missing consent, blurry photo, API failure, schema mismatch — moves the file to an error folder together with a machine-readable error file, and marks the processing run as failed. An unauthenticated webhook call is rejected (HMAC). The acceptance test suite includes deliberately broken inputs, and two of the six end-to-end acceptance criteria are about network segmentation, not features.

## The extraction-proposal contract (C#/.NET)

Between "the model produced JSON" and "the domain tables were updated" sits a deliberately boring, test-first layer: the **extraction proposal**. It exists because LLM extraction output is untrusted input, and because extraction runs may produce multiple variants that need reconciling before commit.

The contract's rules, each of which earned its place:

- **Values are raw strings.** `"3,49"` stays `"3,49"` until the promoter parses it against the field catalogue's declared value kind at commit time. Parsing at the edge means parsing twice and disagreeing once.
- **Slots are `field_code` + `occurrence_index`,** not database IDs — the 15th line item's total is `("position_total", 15)`. New fields are catalogue rows; the contract never changes shape for them.
- **Schema-versioned and forward-compatible from day one.** A newer extractor may add properties; an older promoter must ignore them rather than break.
- **Header fields omit their occurrence index** in the serialised JSON instead of writing `null` — small, but it keeps the payloads honest and the diffs readable.

```csharp
new ProposalField(
    FieldCode: "position_total",
    OccurrenceIndex: 15,          // the 15th line item on the receipt
    Value: "3,49",                // raw string — parsed only at commit time
    ModelConfidence: 0.95m,
    ConflictFlagged: true)        // variants disagreed; majority decided
```

The contract is being implemented test-first in a small .NET library: the xUnit suite is written before the types exist, pinning exactly these behaviours (round-trip fidelity, snake_case JSON congruent with the Postgres world, unknown-property tolerance) so the implementation has no room to drift. It is the part of the pipeline I would expect to survive every future refactoring untouched.

## What I would do differently

- **One WebDAV account per user** instead of a shared upload account with owner slugs in filenames. The slug approach lets one phone capture documents for the whole household, which is how families actually work — but it trades away a clean per-user audit trail at the transport layer. The consent gate compensates, yet the purist in me itches.
- **Retry with backoff before quarantine** for transient API failures. The current design fails fast into the error folder, which is correct for malformed input but wasteful for a 30-second cloud hiccup.
- **PDF ingestion earlier.** Photos were the right first scope, but half of what arrives today is already digital.

## What this is not

This write-up describes architecture and patterns, not a turnkey product. Deployment automation for the ingest stack is part of the project's roadmap (see [ROADMAP.md](../../ROADMAP.md)); the field catalogue, prompts, and reconciliation code are being generalised for publication as the framework matures.

---

*Part of the [PrivateCore AI](../../README.md) documentation series. Next: [MCP Integration](./02-mcp-integration.md) — how the platform's data is exposed to Claude through Model Context Protocol workflows, and what an LLM may never see.*
