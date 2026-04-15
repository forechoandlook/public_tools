---
name: gws-events
version: 1.0.1
description: "Google Workspace Events: Subscribe to and manage Workspace event notifications."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws events --help"
---
## Commands
```bash
gws events <resource> <method> [flags]
### subscriptions
gws events subscriptions create --params '{}' --json '{"targetResource":"//chat.googleapis.com/spaces/<ID>","eventTypes":["<TYPE>"]}' # Create a Workspace subscription
gws events subscriptions list # List all subscriptions
gws events subscriptions get --params '{"name":"subscriptions/<ID>"}' # Get subscription details
gws events subscriptions patch --params '{"name":"subscriptions/<ID>"}' --json '{}' # Update/renew a subscription
gws events subscriptions reactivate --params '{"name":"subscriptions/<ID>"}' # Reactivate a suspended subscription
gws events subscriptions delete --params '{"name":"subscriptions/<ID>"}' # Delete a subscription
### tasks
gws events tasks get --params '{"name":"tasks/<ID>"}' # Get task state
gws events tasks cancel --params '{"name":"tasks/<ID>"}' # Cancel a task
gws events tasks subscribe --params '{"name":"tasks/<ID>"}' # Stream task update events
### tasks.pushNotificationConfigs
gws events tasks pushNotificationConfigs create --params '{"parent":"tasks/<ID>"}' --json '{"pushEndpoint":"<URL>"}' # Set push notification config for a task
gws events tasks pushNotificationConfigs list --params '{"parent":"tasks/<ID>"}' # List push notification configs for a task
gws events tasks pushNotificationConfigs get --params '{"parent":"tasks/<ID>","name":"pushNotificationConfigs/<ID>"}' # Get a push notification config
gws events tasks pushNotificationConfigs delete --params '{"parent":"tasks/<ID>","name":"pushNotificationConfigs/<ID>"}' # Delete a push notification config
### message
gws events message stream --params '{"task":"tasks/<ID>"}' # Stream task update events until task is complete
### operations
gws events operations get --params '{"name":"<OPERATION_NAME>"}' # Get state of a long-running operation
### Helpers
gws events +subscribe --target '//chat.googleapis.com/spaces/<ID>' --event-types '<TYPE>' --project '<PROJECT>' # Subscribe and stream events as NDJSON
gws events +subscribe --subscription 'projects/<P>/subscriptions/<SUB>' --once # Pull events once and exit
gws events +renew --name 'subscriptions/<ID>' # Renew/reactivate a subscription
gws events +renew --all --within 2d # Renew all subscriptions expiring within 2 days
## Event Types
- `google.workspace.chat.message.v1.created` — Chat message created
- `google.workspace.chat.message.v1.deleted` — Chat message deleted
- `google.workspace.chat.membership.v1.changed` — Chat membership changed
- `google.workspace.drive.file.v1.changed` — Drive file changed
- `google.workspace.calendar.event.v1.changed` — Calendar event changed
## Discovering Commands
gws events --help # Browse resources and methods
gws schema events.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
