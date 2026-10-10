# Templates

How `provenance export website` renders this repository (provenance-ddf DES-0031).
Each file here replaces Provenance's built-in version of the same thing;
anything not provided falls back to the built-in, so a project only writes
templates for what it wants to look different.

| File | Replaces | Receives |
|---|---|---|
| `Requirement.tmpl` | The built-in page for every `Requirement` (a table of fields, then the body) | One entity: `.ID`, `.Type`, `.Title`, `.Body`, and every field and incoming link as a PascalCase accessor (`.Statement`, `.ParentRequirement`, `.ChildRequirements`) |
| `Document.tmpl` | The built-in page for every `Document` | The same, plus `.Citations`: every entity the page cites, in first-citation order, of which it lists the other Documents (*Internal*) and the `Reference` entities (*External*) at the end, each subsection only when it has entries. Empty when the Document is embedded or shown by a query |
| `requirement-checklist.tmpl`, `risk-summary.tmpl` | Nothing: named presentation templates, used only where a query block asks for them (`DOC/DOC-0002.md`) | The same as a type template |
| `_layout.tmpl` | The layout around every page | `.Title`, `.Root`, `.Component`, the page content via `{{template "content" .}}`, git stamps (`.GitSHA`, `.ContentHash`, `.LastChangedSHA`, `.Revisions`) and the page's `.Citations` (this layout doesn't list them; `Document.tmpl` does) |
| `_cite.tmpl` | How every inline `[[ID]]` and `[[ID\|label]]` renders (as its ID, or its label, linked) | The cited entity, as in `.Citations`, with `.CitationIndex`, `.TypeCitationIndex` and `.CitationLabel` (the label, or empty). Shows a `Reference` as its number in the *External* list, `[1]`, and a label as a locator within it, `[3, clause 7]`; anything else as before |
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
