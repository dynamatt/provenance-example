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

Every pull request and every push to `main` is exported by the *Website*
workflow (`.github/workflows/website.yml`): download the run's `website`
artifact, unzip it and open `index.html`. It builds Provenance from source,
from the first of the pull request's head branch, its base branch and
`main` that exists in the Provenance repository, so a branch here that
needs an unreleased feature is built with the Provenance branch of the
same name.

## Folder structure

Matches the repository layout of provenance-ddf DES-0005:

```
.component              component metadata (code: NEURO)
plugins.lock             active plugins — empty; no third-party plugins used here
schema/                  entity type, enum and record declarations
  enums/                 named enum types (DES-0010)
  records/               records: named row shapes lists use with `of:`
rules/                   rules — one query per file, named by its ID
                         (DES-0016)
templates/               website templates: type pages, named templates, citations,
                         layout, index, stylesheet
scopes/                  standalone query files for `export --scope`
assets/                  images the entities' Markdown shows
.signatures/             append-only e-signature ledger (DES-0040)
USR/ REQ/ DES/ SEV/ OCC/
RSK/ VER/ VAL/ ECO/ DOC/
REF/ CMP/ BLD/ EQP/      entity data, one file per entity (DES-0006)
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
blocks. A verification protocol ([[DOC-0004]]) and its report
([[DOC-0005]]) gather the bench protocols and their evidence into the two
documents a design review signs. Each protocol names its device under test,
the pulse generator ([[CMP-0001]]), and the equipment it needs
([[EQP-0001]]–[[EQP-0003]]); each execution names the build it ran on
([[BLD-0001]], [[BLD-0002]]). One signature record exists for
[[REQ-0001]]'s approval.

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
| `schema/VerificationProtocol.yaml`, `VER/VER-*.md` | A procedure as a `list` of steps, each with a description and acceptance criteria. Steps store no number; the template lays them out as a table and numbers them by position |
| `VER/EVD-*.md` | `step_results`: a result and pass/fail for each step of the tested protocol, row *n* for step *n*. `templates/VerificationEvidence.tmpl` shows each beside its step, read through the `tests` link |
| `schema/Component.yaml`, `CMP/` | A part of the product that can be under test. Not a Provenance component (the repository in `.component`); the name is the device team's |
| `schema/BuildStandard.yaml`, `BLD/` | One identified build of a Component, with its `version` (hardware revision, firmware) and `build_date`: what evidence was actually executed on (`VerificationEvidence.build_tested`) |
| `schema/TestEquipment.yaml`, `EQP/` | Equipment by model, linked from the protocols that need it, so documents list each piece once across all protocols. The units used are a frozen record on the evidence (`Equipment`), each linking back to its TestEquipment |
| `schema/records/Equipment.yaml` | A record: a row shape declared once and used by two lists (`VerificationEvidence` and `ValidationEvidence`). Each row is a unit actually used and links to the TestEquipment it is a unit of; a link in a row has a reverse facet like any other (`used_in`), so DOC-0005 lists each piece of equipment with its units |
| `schema/ValidationEvidence.yaml` | Reuses the `Equipment` record for its own `equipment_used` list |
| `schema/ChangeRequest.yaml` | A deliberate enum-vs-rule-checked-string tradeoff, explained inline |
| `RSK/RSK-0001.md` | `failure_modes` list populated; `row_rating`/`overall_risk_rating` are calculated, never hand-set |
| `DOC/DOC-0001.md` | Query block (`from`/`where`/`order_by`), all four wikilink forms (`[[ID]]`, `[[ID#field]]`, `![[ID]]`) |
| `scopes/approved-requirements.yaml` | A standalone `--scope` query file (`from`/`where`) for `export` |
| `assets/control-loop.svg`, `assets/ecap-response.png`, `assets/bench-setup.jpg` | Images in each supported format, captioned where they are used: two figures and a table in DOC-0001, a photo in EVD-0001; DES-0001 captions an equation |
| `assets/evd-*.svg` | Oscilloscope screenshots attached as evidence inside a step's result: a list row's `text` holds an image and its caption like any body, and the caption numbers in order wherever the evidence is shown (EVD-0001 and EVD-0002, and DOC-0005, which shows both) |
| `schema/Reference.yaml`, `REF/` | External sources as ordinary entities: a standard (ISO 14971), a standard with amendments (IEC 60601-1) and a journal paper, cited with `[[REF-0001]]` in DOC-0001 (once with a clause, `[[REF-0001\|clause 7]]`) and in REQ-0003, which DOC-0001 embeds; `templates/_cite.tmpl` shows them as `[1]`, `[2]`, `[3, clause 7]` |
| `DOC/DOC-0002.md` | A query block selecting two types at once (`from: [Risk, Requirement]`), ordered together by their shared `order` field; results listed by ID (`render: id`) and by one field (`render: field:identifier`), which list entities without citing them |
| `DOC/DOC-0003.md` | A design review checklist: one query block per check the rules make, each empty while the design passes. Sub-queries on a list (equipment rows) and a link (the tested protocol), `^` to read the enclosing entity, `value_of`, `missing`, `not_in`, a `count` aggregation over a link and a reverse link, and a sub-query on `from` to find another requirement with the same title |
| `DOC/DOC-0004.md` | A verification protocol: purpose, the requirements in scope (`exists` on a reverse facet, shown through a named template) and out of scope (`missing`), the device under test and test equipment gathered from all the procedures (a sub-query on a reverse facet, each listed once through a named template), and the approved procedures in full with their numbered steps |
| `DOC/DOC-0005.md` | The verification report: purpose, scope, the builds actually tested and the equipment, gathered from all the executions; every execution in date order with each protocol step beside its result, then a query per thing that would fail the report (a result other than `pass`, a step that did not pass, an approved procedure with no evidence, evidence dated before its build, lapsed calibration), each empty while verification is complete. Cites DOC-0004, which lists it as an *Internal* reference |
| `rules/*.yaml` | Rules written as queries, one per kind of check — see below |
| `.signatures/REQ-0001-2026-08-20.yaml` | Signature record shape (provenance-ddf DES-0040) — proof fields are illustrative placeholders |

## Rules

A rule is a query: `severity`, `message`, and a query block's `from` and
`where`. Every entity, row or link it matches is a violation, and the file
name is the rule's ID (provenance-ddf DES-0016). Each kind of check is
written here at least once:

| Check | File | What it shows |
|---|---|---|
| Reference count | `requirement-verified.yaml`, `protocol-verifies-requirement.yaml` | `aggregation: {type: count}` over a reverse facet and a stored link |
| Required field | `risk-benefit-required.yaml` | `missing` beside a filter |
| Value in set | `eco-change-type-valid.yaml` | `not_in` |
| Comparison across a link | `evidence-after-build.yaml` | A sub-query on a link; `^execution_date` reads the evidence |
| Comparison within an entity | `equipment-calibration-current.yaml`, `validation-equipment-calibration-current.yaml` | A sub-query on a list, reported per row |
| No cycles | `no-requirement-decomposition-cycles.yaml` | `transitive: true` |
| Uniqueness | `requirement-title-unique.yaml` | A sub-query on `from: Requirement`: another requirement whose title equals `^title` |
| Query assertion | `no-implements-deprecated-need.yaml`, `superseded-entities-deprecated.yaml` | Any condition |
| Block language | `block-languages.yaml` | The derived `code_blocks` list |

Unresolved links and links to a disallowed type need no rule: `validate`
checks them itself, with the rest of the built-in checks (provenance-ddf
DES-0048). Signature and frozen-content rules return once signatures and
releases are facts a query can read, and so does the check that evidence
was executed against the protocol as it now stands: evidence records the
protocol's commit (`protocol_commit`), and the protocol has no version
number of its own (provenance-ddf DES-0016, OPEN).

Every entity in this repo currently satisfies every rule — there's no
intentionally-broken example. A few files note in a comment what value
would need to change to trigger the rule (e.g. `EVD-0002.md`'s note on
`BLD-0002`'s `build_date`), so the failure mode is documented without leaving the
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
