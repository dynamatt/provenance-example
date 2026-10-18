# Templates

How `provenance export website` renders this repository (provenance-ddf DES-0031).
Each file here replaces Provenance's built-in version of the same thing;
anything not provided falls back to the built-in, so a project only writes
templates for what it wants to look different.

| File | Replaces | Receives |
|---|---|---|
| `Requirement.tmpl` | The built-in page for every `Requirement` (a table of fields, then the body) | One entity: `.ID`, `.Type`, `.Title`, `.Body`, and every field and incoming link as a PascalCase accessor (`.Statement`, `.ParentRequirement`, `.ChildRequirements`) |
| `Document.tmpl` | The built-in page for every `Document` | The same, plus `.Citations`: every entity the page cites, in first-citation order, of which it lists the cited `Definition`s (*Definitions*), then the other Documents (*Internal*) and the `Reference` entities (*External*) at the end, each section only when it has entries. Empty when the Document is embedded or shown by a query |
| `VerificationProtocol.tmpl` | The built-in page for every `VerificationProtocol` | The same as `Requirement.tmpl`. Lays out `.Steps` as a table (step, description, acceptance criteria), numbered by a CSS counter in `style.css`, and lists the executions (`.TestedBy`) |
| `VerificationEvidence.tmpl` | The built-in page for every `VerificationEvidence` | The same. Shows the tested protocol's steps (`.Tests.Steps`) in the same table, with Result and Pass/Fail columns from the result recorded at the same position in `.StepResults`; a missing result is marked *Not recorded*, and results beyond the last step are flagged |
| `requirement-checklist.tmpl`, `risk-summary.tmpl` | Nothing: named presentation templates, used only where a query block asks for them (`DOC/DOC-0002.md`) | The same as a type template |
| `device-under-test.tmpl`, `build-tested.tmpl` | Nothing: named templates rendering a Component, or a BuildStandard with the version tested, as one row of the Device Under Test tables in `DOC/DOC-0004.md` and `DOC/DOC-0005.md`. A query block renders each result separately, so each is a one-row table; `style.css` joins consecutive ones into one (shared column widths, first header only) | The same as a type template |
| `equipment-used.tmpl` | Nothing: a named template giving a piece of equipment as one row of `DOC/DOC-0005.md`'s equipment table, with every unit of it the verification evidence used, found through the reverse facet `used_in` of the evidence's equipment rows. Joined into one table like the two above | The same as a type template |
| `equipment-summary.tmpl` | Nothing: a named template giving one line per piece of equipment, for the list in `DOC/DOC-0004.md` | The same as a type template |
| `_layout.tmpl` | The layout around every page | `.Title`, `.Root`, `.Component`, the page content via `{{template "content" .}}`, git stamps (`.GitSHA`, `.ContentHash`, `.LastChangedSHA`, `.Revisions`) and the page's `.Citations` (this layout doesn't list them; `Document.tmpl` does) |
| `_cite.tmpl` | How every inline `[[ID]]` and `[[ID\|label]]` renders (as its ID, or its label, linked) | The cited entity, as in `.Citations`, with `.CitationIndex`, `.TypeCitationIndex` and `.CitationLabel` (the label, or empty). Shows a `Reference` as its number in the *External* list, `[1]`, and a label as a locator within it, `[3, clause 7]`; a `Definition` as its abbreviation or term, or its label; anything else as before |
| `_index.tmpl` | The site's main page | `.Component` and `.Types` (entities grouped by type) |
| `style.css` | The built-in stylesheet | — |

A named template's file name is lower-case kebab-case, so it can't be
mistaken for a type template (`Requirement.tmpl`) or a site override
(`_layout.tmpl`). A query block chooses one for all its results with
`template: <name>`, or per type with `templates: {Requirement: <name>}`;
types it doesn't name keep their usual template.

Other types (Design, Risk, …) have no type template here, so they use the
built-in page — compare `_site/entities/REQ-0001.html` with
`_site/entities/DES-0001.html` after an export.

Templates are Go `html/template`, layout only: Markdown conversion and links
are template functions (`markdown`, `link`, `href`). Headings are relative —
write `<h1>` for the top of the entity and Provenance shifts headings to the
depth the entity is rendered at, keeping their `id` and `class`.
