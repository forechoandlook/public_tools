---
name: gws-tasks
version: 1.0.1
description: "Google Tasks: Manage task lists and tasks."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws tasks --help"
---
## Commands
```bash
gws tasks <resource> <method> [flags]
### tasklists
gws tasks tasklists list # List all task lists
gws tasks tasklists get --params '{"tasklist":"<ID>"}' # Get a specific task list
gws tasks tasklists insert --json '{"title":"<TITLE>"}' # Create a new task list
gws tasks tasklists update --params '{"tasklist":"<ID>"}' --json '{"title":"<TITLE>"}' # Update a task list
gws tasks tasklists patch --params '{"tasklist":"<ID>"}' --json '{"title":"<TITLE>"}' # Update a task list (patch)
gws tasks tasklists delete --params '{"tasklist":"<ID>"}' # Delete a task list
### tasks
gws tasks tasks list --params '{"tasklist":"<ID>"}' # List all tasks in a task list
gws tasks tasks get --params '{"tasklist":"<ID>","task":"<ID>"}' # Get a specific task
gws tasks tasks insert --params '{"tasklist":"<ID>"}' --json '{"title":"<TITLE>"}' # Create a new task
gws tasks tasks update --params '{"tasklist":"<ID>","task":"<ID>"}' --json '{"title":"<TITLE>","status":"<STATUS>"}' # Update a task
gws tasks tasks patch --params '{"tasklist":"<ID>","task":"<ID>"}' --json '{"title":"<TITLE>"}' # Update a task (patch)
gws tasks tasks move --params '{"tasklist":"<ID>","task":"<ID>","parent":"<ID>","previous":"<ID>"}' # Move task to another position/parent
gws tasks tasks delete --params '{"tasklist":"<ID>","task":"<ID>"}' # Delete a task
gws tasks tasks clear --params '{"tasklist":"<ID>"}' # Clear completed tasks from a list
## Discovering Commands
gws tasks --help # Browse resources and methods
gws schema tasks.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
