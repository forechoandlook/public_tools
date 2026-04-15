---
name: gws-keep
version: 1.0.1
description: "Google Keep: Manage notes and attachments."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws keep --help"
---
## Commands
```bash
gws keep <resource> <method> [flags]
### notes
gws keep notes list --params '{"pageSize":<N>,"filter":"<FILTER>"}' # List notes (supports filter by trashed, create_time, etc.)
gws keep notes get --params '{"name":"notes/<ID>"}' # Get a note by name
gws keep notes create --json '{"title":"<TITLE>","content":"<TEXT>"}' # Create a new note
gws keep notes delete --params '{"name":"notes/<ID>"}' # Delete a note (permanent, cannot undo)
### notes.permissions
gws keep notes permissions batchCreate --params '{"parent":"notes/<ID>"}' --json '{"requests":[{"role":"WRITER","email":"<EMAIL>"}]}' # Add collaborators to a note
gws keep notes permissions batchDelete --params '{"parent":"notes/<ID>"}' --json '{"requests":[{"role":"WRITER","email":"<EMAIL>"}]}' # Remove collaborators from a note
### media
gws keep media download --params '{"name":"notes/<NOTE_ID>/attachments/<ATTACHMENT_ID>"}' # Download an attachment
## Discovering Commands
gws keep --help # Browse resources and methods
gws schema keep.<resource>.<method> # Inspect required params, types, and defaults
```
