---
name: gws-slides
version: 1.0.1
description: "Google Slides: Read and write presentations."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws slides --help"
---
## Commands
```bash
gws slides <resource> <method> [flags]
gws slides presentations get --params '{"presentationId":"<ID>"}' # Get the latest version of a presentation
gws slides presentations create --params '{"title":"<TITLE>"}' # Create a blank presentation
gws slides presentations batchUpdate --params '{"presentationId":"<ID>"}' --json '{"requests":[...]}' # Apply updates to a presentation
### presentations.pages
gws slides presentations pages get --params '{"presentationId":"<ID>","pageObjectId":"<ID>"}' # Get a page in a presentation
gws slides presentations pages getThumbnail --params '{"presentationId":"<ID>","pageObjectId":"<ID>"}' # Generate a thumbnail URL for a page
## Discovering Commands
gws slides --help # Browse resources and methods
gws schema slides.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
