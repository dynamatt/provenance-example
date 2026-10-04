# Templates

How `provenance export website` renders this repository (Detailed Design §7).
Each file here replaces Provenance's built-in version of the same thing;
anything not provided falls back to the built-in, so a project only writes
templates for what it wants to look different.

| File | Replaces | Receives |
|---|---|---|
| `Requirement.tmpl` | The built-in page for every `Requirement` (a table of fields, then the body) | One entity: `.ID`, `.Type`, `.Title`, `.Body`, and every field and incoming link as a PascalCase accessor (`.Statement`, `.ParentRequirement`, `.ChildRequirements`) |
| `requirement-checklist.tmpl`, `risk-summary.tmpl` | Nothing: named presentation templates, used only where a query block asks for them (`DOC/DOC-0002.md`) | The same as a type template |
| `_layout.tmpl` | The layout around every page | `.Title`, `.Root`, `.Component`, the page content via `{{template "content" .}}`, and git stamps: `.GitSHA`, `.ContentHash`, `.LastChangedSHA`, `.Revisions` |
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
