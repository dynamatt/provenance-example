# NeuroPulse — worked example repository

This is a hand-authored example repository for
[Provenance](https://github.com/dynamatt/provenance), a git-native
design-control tool: a fictional implantable closed-loop neurostimulator,
written to pressure-test the schema, rule and query syntax against
real-shaped content. It is also Provenance's test fixture: its CI pins a
commit of this repository and exports it on every change.

`provenance export website` works against this repo, from release
`v0.1.0-alpha` on: every entity gets a page, Documents are rendered from
their live query blocks and wikilinks, calculated fields are evaluated,
figures, tables and equations are numbered, citations are listed and
numbered, and every page carries the commit and content hash it came from.
The project templates in `templates/` shape the result (see its README).
`provenance verify content` prints the content hash. `validate`, `sign`
and the other commands are not built yet, so `rules/` and `.signatures/`
are written to the design but not yet checked. Everything in `schema/`,
`rules/`, and the entity data files is written exactly as a real project
would author it, and is intended to be literally correct against the design
in [provenance-ddf](https://github.com/dynamatt/provenance-ddf). If you find
a place where it *isn't* — a syntax the design doesn't actually support, a
rule instance that doesn't parse per its own spec — that's a real gap the
design missed, not a mistake to quietly fix.

To see it, with a `provenance` binary on your `PATH`:

```bash
provenance export website            # the whole DHF, into _site/
provenance export website --scope DOC/DOC-0001.md --out _doc
                                     # one document and what it shows
provenance verify content            # the content hash on every page
```

## Folder structure

Matches the repository layout of provenance-ddf DES-0005:

```
.component              component metadata (code: NEURO)
plugins.lock             active plugins — empty; no third-party plugins used here
schema/                  entity type, enum and record declarations
  enums/                 named enum types (DES-0010)
  records/               records: named row shapes lists use with `of:`
rules/                   rule instances — one file per rule, one example per
                         rule type in the fixed library (REQ-0035)
templates/               website templates: type pages, named templates, citations,
                         layout, index, stylesheet
scopes/                  standalone query files for `export --scope`
assets/                  images the entities' Markdown shows
.signatures/             append-only e-signature ledger (DES-0040)
USR/ REQ/ DES/ SEV/ OCC/
RSK/ VER/ VAL/ ECO/ DOC/
REF/                     entity data, one file per entity (DES-0006)
```

`VerificationProtocol` and `VerificationEvidence` both live under `VER/` by
folder convention, but use different ID prefixes (`VER-` / `EVD-`) — folder
placement is a convention, not the source of an entity's type or ID
(provenance-ddf DES-0006).

`Validation` is reduced to evidence only (User Need ↔ Validation Evidence,
no Validation Protocol): it follows the identical
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
| `schema/enums/*.yaml` | Named enum types (provenance-ddf DES-0010) |
| `schema/SeverityLevel.yaml`, `OccurrenceLevel.yaml` | Enum-vs-linked-entity guidance — these need a numeric `score`, so they're entities, not enums |
| `schema/Requirement.yaml` | `link` with `reverse_name` (asymmetric naming, provenance-ddf DES-0011); self-referential link (`parent_requirement`); list of an enum (`verification_methods`, `of: VerificationMethod`) |
| `schema/Design.yaml` | List of a built-in type (`standards`, `of: string`) |
| `schema/Risk.yaml` | `list` field with its row fields declared inline, a nested `calculated` sub-field, plus an entity-level `calculated` field aggregating across it (provenance-ddf DES-0012) |
| `schema/VerificationEvidence.yaml` | `list` field for a frozen point-in-time record (equipment calibration), its rows a named record (`of: Equipment`) |
| `schema/records/Equipment.yaml` | A record: a row shape declared once and used by two lists (`VerificationEvidence` and `ValidationEvidence`) |
| `schema/ValidationEvidence.yaml` | Reuses the `Equipment` record for its own `equipment_used` list |
| `schema/ChangeRequest.yaml` | A deliberate enum-vs-rule-checked-string tradeoff, explained inline |
| `RSK/RSK-0001.md` | `failure_modes` list populated; `row_rating`/`overall_risk_rating` are calculated, never hand-set |
| `DOC/DOC-0001.md` | Query block (`from`/`where`/`order_by`), all four wikilink forms (`[[ID]]`, `[[ID#field]]`, `![[ID]]`) |
| `scopes/approved-requirements.yaml` | A standalone `--scope` query file (`from`/`where`) for `export` |
| `assets/control-loop.svg`, `assets/ecap-response.png`, `assets/bench-setup.jpg` | Images in each supported format, captioned where they are used: two figures and a table in DOC-0001, a photo in EVD-0001; DES-0001 captions an equation |
| `schema/Reference.yaml`, `REF/` | External sources as ordinary entities: a standard (ISO 14971), a standard with amendments (IEC 60601-1) and a journal paper, cited with `[[REF-0001]]` in DOC-0001 (once with a clause, `[[REF-0001\|clause 7]]`) and in REQ-0003, which DOC-0001 embeds; `templates/_cite.tmpl` shows them as `[1]`, `[2]`, `[3, clause 7]` |
| `DOC/DOC-0002.md` | A query block selecting two types at once (`from: [Risk, Requirement]`), ordered together by their shared `order` field; results listed by ID (`render: id`) and by one field (`render: field:identifier`), which list entities without citing them |
| `rules/*.yaml` | One worked example per rule type in the fixed library — see below |
| `.signatures/REQ-0001-2026-08-20.yaml` | Signature record shape (provenance-ddf DES-0040) — proof fields are illustrative placeholders |

## Rule type coverage

Every rule type in the fixed library (provenance-ddf REQ-0035, REQ-0041 to
REQ-0052) has a worked instance here:

| Rule type | File |
|---|---|
| Reference Count | `requirement-verified.yaml`, `protocol-verifies-requirement.yaml` |
| Required Field | `risk-benefit-required.yaml` |
| Field Value In Set | `eco-change-type-valid.yaml` |
| Field Comparison Across Link | `evidence-verified-latest-version.yaml` |
| Field Comparison Within Entity | `equipment-calibration-current.yaml` (expressed as `where`, per provenance-ddf DES-0015) |
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

Building this surfaced four things worth carrying back into the design
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
   into Detailed Design §5's field type taxonomy (now provenance-ddf
   DES-0010). A self-consistency checker run against this repo caught the
   gap immediately — every entity with a body-backed field was failing
   "missing required field" until the schemas declared it.
2. `VerificationProtocol` and `VerificationEvidence` sharing a folder but
   needing distinct ID prefixes wasn't explicitly stated anywhere before —
   now noted in this README and worth a one-line addition to
   provenance-ddf DES-0006 if it comes up again.
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
   could disagree, contradicting "declared once, on its source side" (now
   provenance-ddf DES-0011). Fixed by keeping only `VerificationProtocol.verifies`: the
   link belongs on the artefact written *later*, pointing at what it depends
   on, so a requirement is never edited when verification is added. A
   requirement's `verified_by` is now the derived reverse facet, which is
   exactly what `rules/requirement-verified.yaml` relies on.
