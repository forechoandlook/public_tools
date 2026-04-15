---
name: gws-workflow
version: 1.0.1
description: "Google Workflow: Cross-service productivity workflows."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws workflow --help"
---
## Commands
```bash
gws workflow +standup-report # Today's meetings + open tasks as a standup summary
gws workflow +meeting-prep # Prepare for next meeting: agenda, attendees, linked docs
gws workflow +email-to-task # Convert a Gmail message into a Google Tasks entry
gws workflow +weekly-digest # Weekly summary: this week's meetings + unread email count
gws workflow +file-announce # Announce a Drive file in a Chat space
```