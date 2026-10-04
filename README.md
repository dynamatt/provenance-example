# NeuroPulse — worked example repository

This is a hand-authored example repository for the (not-yet-built) GxP
design-control platform ("Provenance" in earlier design discussion) — a
fictional implantable closed-loop neurostimulator, used to pressure-test the
schema DSL, rule instance syntax, and query grammar against real-shaped
content before any code is written.

`provenance export website` works against this repo, including the
project templates in `templates/` (see its README); `validate` and `sign`
are not built yet. Everything in `schema/`,
`rules/`, and the entity data files is written exactly as a real project
would author it, and is intended to be literally correct against the design
documents. If you find a place where it *isn't* — a syntax the design docs
don't actually support, a rule instance that doesn't parse per its own
spec — that's a real gap the design missed, not a mistake to quietly fix.

## Folder structure

Matches Detailed Design §1 exactly:

```
.component              component metadata (code: NEURO)
plugins.lock             active plugins — empty; no third-party plugins used here
schema/                  entity type, enum and record declarations
  enums/                 named enum types (Detailed Design §5)
  records/               records: named row shapes lists use with `of:`
rules/                   rule instances — one file per rule, one example per
                         rule type in the fixed library (Requirements Spec §6)
templates/               website templates: Requirement page, layout, index, stylesheet
.signatures/             append-only e-signature ledger (Requirements Spec §11)
USR/ REQ/ DES/ SEV/ OCC/
RSK/ VER/ VAL/ ECO/ DOC/ entity data, one file per entity (High-Level Design §4.3)
```

`VerificationProtocol` and `VerificationEvidence` both live under `VER/` by
folder convention, but use different ID prefixes (`VER-` / `EVD-`) — folder
placement is a convention, not the source of an entity's type or ID
(Detailed Design §1).

`Validation` is reduced to evidence only (User Need ↔ Validation Evidence,
no Validation Protocol): Requirements Spec §2 states it follows the identical
pattern as Verification, so a protocol wouldn't exercise anything new. The
evidence is kept because it shares the `Equipment` record with
Verification Evidence.

## The story this repo tells

A closed-loop stimulator adjusts pulse amplitude based on a sensed neural
response ([[REQ-0001]] → [[DES-0001]]), with a hardware safety ceiling
independent of firmware for the single-fault case ([[REQ-0002]] →
[[REQ-0003]]). A hazard analysis ([[RSK-0001]]) tracks two failure modes
against that hazard. Both requirements are verified by bench protocols
([[VER-0001]], [[VER-0002]]) with recorded execution evidence
([[EVD-0001]], [[EVD-0002]]), and the first user need is validated in a
simulated-use study ([[VAL-0001]]). An ECO ([[ECO-0001]]) records the change that
added a fault-injection step to one protocol. A Document ([[DOC-0001]])
composes several of these into a rendered specification using live query
blocks. One signature record exists for [[REQ-0001]]'s approval.

(Wikilink-style references above are for this README's readability — they
aren't live in a plain Markdown file the way they would be inside an actual
entity's body.)

## What each file demonstrates

| File | Demonstrates |
|---|---|
| `schema/enums/*.yaml` | Named enum types (Detailed Design §5) |
| `schema/SeverityLevel.yaml`, `OccurrenceLevel.yaml` | Enum-vs-linked-entity guidance — these need a numeric `score`, so they're entities, not enums |
| `schema/Requirement.yaml` | `link` with `reverse_name` (asymmetric naming, Detailed Design §6); self-referential link (`parent_requirement`); list of an enum (`verification_methods`, `of: VerificationMethod`) |
| `schema/Design.yaml` | List of a built-in type (`standards`, `of: string`) |
| `schema/Risk.yaml` | `list` field with its row fields declared inline, a nested `calculated` sub-field, plus an entity-level `calculated` field aggregating across it (Detailed Design §5) |
| `schema/VerificationEvidence.yaml` | `list` field for a frozen point-in-time record (equipment calibration), its rows a named record (`of: Equipment`) |
| `schema/records/Equipment.yaml` | A record: a row shape declared once and used by two lists (`VerificationEvidence` and `ValidationEvidence`) |
| `schema/ValidationEvidence.yaml` | Reuses the `Equipment` record for its own `equipment_used` list |
| `schema/ChangeRequest.yaml` | A deliberate enum-vs-rule-checked-string tradeoff, explained inline |
| `RSK/RSK-0001.md` | `failure_modes` list populated; `row_rating`/`overall_risk_rating` are calculated, never hand-set |
| `DOC/DOC-0001.md` | Query block (`from`/`where`/`order_by`), all four wikilink forms (`[[ID]]`, `[[ID#field]]`, `![[ID]]`) |
| `scopes/approved-requirements.yaml` | A standalone `--scope` query file (`from`/`where`) for `export` |
| `assets/control-loop.svg`, `assets/ecap-response.png`, `assets/bench-setup.jpg` | Images in each supported format, captioned where they are used: two figures and a table in DOC-0001, a photo in EVD-0001 (`templates/_captions.yaml`) |
| `DOC/DOC-0002.md` | A query block selecting two types at once (`from: [Risk, Requirement]`), ordered together by their shared `order` field |
| `rules/*.yaml` | One worked example per rule type in the fixed library — see below |
| `.signatures/REQ-0001-2026-08-20.yaml` | Signature record shape (Requirements Spec §11) — proof fields are illustrative placeholders |

## Rule type coverage

Every rule type in Requirements Spec §6 has a worked instance here:

| Rule type | File |
|---|---|
| Reference Count | `requirement-verified.yaml`, `protocol-verifies-requirement.yaml` |
| Required Field | `risk-benefit-required.yaml` |
| Field Value In Set | `eco-change-type-valid.yaml` |
| Field Comparison Across Link | `evidence-verified-latest-version.yaml` |
| Field Comparison Within Entity | `equipment-calibration-current.yaml` (expressed as `where`, per Detailed Design §6) |
| No Cycles | `no-requirement-decomposition-cycles.yaml` |
| Uniqueness | `requirement-title-unique.yaml` |
| Signature Presence | `requirement-approval-signed.yaml` |
| Query Assertion | `no-implements-deprecated-need.yaml`, `superseded-entities-deprecated.yaml` |
| Reference Validity | `cross-references-resolve.yaml` |
| Content Frozen After Release | `content-frozen-requirement.yaml` |
| Block Language | `block-languages.yaml` |

Every entity in this repo currently satisfies every rule — there's no
intentionally-broken example. A few files note in a comment what value
would need to change to trigger the rule (e.g. `EVD-0002.md`'s comment on
`protocol_version`), so the failure mode is documented without leaving the
repo in a permanently non-compliant state.

## What this exercise found

Building this surfaced three things worth carrying back into the design
documents, all applied already:

1. **A genuine gap, not a nitpick: nothing said which field's content lives
   in the file's Markdown body vs. as a frontmatter string.** Every entity
   here has a `text` field (a Requirement's `statement`, a Design's
   `description`, and so on) that was always intended to be the file's main
   prose — but no schema mechanism said so. Resolved by adding a `body:
   true` attribute a `text` field can set (at most one per schema); a
   schema with none uses the body as unstructured freeform content instead
   (the existing pattern for `Document`, and now also `Risk`, which has
   four `text` fields and no single one that should own the body). Written
   into Detailed Design §5's field type taxonomy. A self-consistency
   checker run against this repo caught the gap immediately — every entity
   with a body-backed field was failing "missing required field" until the
   schemas declared it.
2. `VerificationProtocol` and `VerificationEvidence` sharing a folder but
   needing distinct ID prefixes wasn't explicitly stated anywhere before —
   now noted in this README and worth a one-line addition to Detailed
   Design §1 if it comes up again.
3. Writing `ChangeRequest.change_type` as a plain string checked by a rule,
   instead of a named enum, is the first real example of *not* using the
   enum-guidance default — worth keeping as a documented precedent for when
   a project might prefer looser, rule-checked values over a schema-file
   enum declaration.
4. **Found by the first implementation (export, E1.4): the Requirement ↔
   Verification Protocol link was declared on both sides.**
   `Requirement.verified_by` (reverse `verifies`) and
   `VerificationProtocol.verifies` (reverse `verified_by`) described the same
   relationship twice, and both REQ and VER files stored it — two copies that
   could disagree, contradicting Detailed Design §6's "declared once, on its
   source side". Fixed by keeping only `VerificationProtocol.verifies`: the
   link belongs on the artefact written *later*, pointing at what it depends
   on, so a requirement is never edited when verification is added. A
   requirement's `verified_by` is now the derived reverse facet, which is
   exactly what `rules/requirement-verified.yaml` relies on.
