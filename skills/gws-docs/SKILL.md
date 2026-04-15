---
name: gws-docs
version: 1.0.1
description: "Read and write Google Docs."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws docs --help"
---
## documents
```bash
gws docs <resource> <method> [flags]
gws docs documents get --params '{"documentId":"<ID>"}' # Get the latest version of a document
gws docs documents create --params '{"title":"<TITLE>"}' # Create a blank document
gws docs documents batchUpdate --params '{"documentId":"<ID>"}' --json '{"requests":[...]}' # Apply updates to a document
gws docs +write --document <ID> --text <TEXT>
gws docs --help # Browse resources and methods
gws schema docs.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
