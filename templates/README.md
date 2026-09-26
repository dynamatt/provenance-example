# Templates — illustrative only

No template engine exists yet (this whole repo is a design-validation
exercise ahead of implementation, Detailed Design §7 / High-Level Design
§4.6). The file below shows what a Requirement template is expected to look
like once built: Go `html/template` syntax, layout-only, with graph
traversal, markdown conversion, and heading depth delegated to registered
template functions rather than expressed in the template itself.
