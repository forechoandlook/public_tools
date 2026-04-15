---
name: gws-gmail
version: 1.0.1
description: "Gmail: Send, read, and manage emails."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws gmail --help"
---
## Commands
```bash
gws gmail <resource> <method> [flags]
### Helpers
gws gmail +send --to '<EMAIL>' --subject '<SUBJECT>' --body '<TEXT>' # Send an email
gws gmail +send --to '<EMAIL>' --subject '<SUBJECT>' --body '<HTML>' --html # Send HTML email
gws gmail +triage # Show unread inbox summary (sender, subject, date)
gws gmail +triage --query 'from:boss' --max 5 # Filter by Gmail search query
gws gmail +reply --message-id '<ID>' --body '<TEXT>' # Reply to a message
gws gmail +reply-all --message-id '<ID>' --body '<TEXT>' # Reply-all to a message
gws gmail +forward --message-id '<ID>' --to '<EMAIL>' # Forward a message
gws gmail +watch --project '<PROJECT>' # Watch for new emails (streams NDJSON)
gws gmail +watch --project '<PROJECT>' --once # Pull once and exit
### users
gws gmail users getProfile # Get current user's Gmail profile
gws gmail users watch --json '{"labelIds":["INBOX"],"labelFilterAction":"include","topicName":"projects/<P>/topics/<T>"}' # Set up push notification watch
gws gmail users stop # Stop push notifications
### users.messages
gws gmail users messages list # List messages in mailbox (latest 100)
gws gmail users messages list --params '{"userId":"me","q":"is:unread","maxResults":<N>}' # List with search query
gws gmail users messages get --params '{"userId":"me","id":"<ID>","format":"metadata"}' # Get message (format: full, metadata, minimal, raw)
gws gmail users messages send --params '{"userId":"me"}' --json '{"raw":"<BASE64_RFC2822>"}' # Send a raw MIME message
gws gmail users messages delete --params '{"userId":"me","id":"<ID>"}' # Permanently delete a message
gws gmail users messages trash --params '{"userId":"me","id":"<ID>"}' # Move message to trash
gws gmail users messages untrash --params '{"userId":"me","id":"<ID>"}' # Remove message from trash
gws gmail users messages modify --params '{"userId":"me","id":"<ID>"}' --json '{"addLabelIds":[],"removeLabelIds":[]}' # Modify message labels
gws gmail users messages batchDelete --params '{"userId":"me"}' --json '{"ids":["<ID1>","<ID2>"]}' # Batch delete messages
gws gmail users messages batchModify --params '{"userId":"me"}' --json '{"ids":["<ID1>"],"addLabelIds":[],"removeLabelIds":[]}' # Batch modify labels
gws gmail users messages import --params '{"userId":"me","internalDateSource":"<SOURCE>"}' --json '{"raw":"<BASE64>"}' # Import a message (bypasses most scanning)
gws gmail users messages insert --params '{"userId":"me"}' --json '{"raw":"<BASE64>"}' # Insert a message directly (IMAP APPEND style)
### users.messages.attachments
gws gmail users messages attachments get --params '{"userId":"me","messageId":"<ID>","id":"<ATTACHMENT_ID>"}' # Get a message attachment
### users.threads
gws gmail users threads list # List threads in mailbox
gws gmail users threads list --params '{"userId":"me","q":"is:unread"}' # List threads with search query
gws gmail users threads get --params '{"userId":"me","id":"<ID>"}' # Get a thread
gws gmail users threads trash --params '{"userId":"me","id":"<ID>"}' # Move thread to trash
gws gmail users threads untrash --params '{"userId":"me","id":"<ID>"}' # Remove thread from trash
gws gmail users threads delete --params '{"userId":"me","id":"<ID>"}' # Permanently delete thread
gws gmail users threads modify --params '{"userId":"me","id":"<ID>"}' --json '{"addLabelIds":[],"removeLabelIds":[]}' # Modify thread labels
### users.drafts
gws gmail users drafts list --params '{"userId":"me"}' # List drafts
gws gmail users drafts get --params '{"userId":"me","id":"<ID>"}' # Get a draft
gws gmail users drafts create --params '{"userId":"me"}' --json '{"message":{"raw":"<BASE64>"}}' # Create a draft
gws gmail users drafts update --params '{"userId":"me","id":"<ID>"}' --json '{"message":{"raw":"<BASE64>"}}' # Update a draft
gws gmail users drafts send --params '{"userId":"me","id":"<ID>"}' # Send an existing draft
gws gmail users drafts delete --params '{"userId":"me","id":"<ID>"}' # Permanently delete a draft
### users.labels
gws gmail users labels list --params '{"userId":"me"}' # List all labels
gws gmail users labels get --params '{"userId":"me","id":"<ID>"}' # Get a label
gws gmail users labels create --params '{"userId":"me"}' --json '{"name":"<NAME>","labelListVisibility":"<VIS>","messageListVisibility":"<VIS>"}' # Create a label
gws gmail users labels update --params '{"userId":"me","id":"<ID>"}' --json '{"name":"<NAME>"}' # Update a label
gws gmail users labels patch --params '{"userId":"me","id":"<ID>"}' --json '{"name":"<NAME>"}' # Update a label (patch)
gws gmail users labels delete --params '{"userId":"me","id":"<ID>"}' # Delete a label
### users.history
gws gmail users history list --params '{"userId":"me","startHistoryId":"<ID>"}' # List mailbox change history
### users.settings
gws gmail users settings getAutoForwarding --params '{"userId":"me"}' # Get auto-forwarding setting
gws gmail users settings getImap --params '{"userId":"me"}' # Get IMAP settings
gws gmail users settings getPop --params '{"userId":"me"}' # Get POP settings
gws gmail users settings getLanguage --params '{"userId":"me"}' # Get language settings
gws gmail users settings getVacation --params '{"userId":"me"}' # Get vacation responder settings
gws gmail users settings updateAutoForwarding --params '{"userId":"me"}' --json '{"enabled":true,"forwardingAddress":"<EMAIL>"}' # Update auto-forwarding
gws gmail users settings updateImap --params '{"userId":"me"}' --json '{"enabled":true}' # Update IMAP settings
gws gmail users settings updatePop --params '{"userId":"me"}' --json '{"enabled":true}' # Update POP settings
gws gmail users settings updateLanguage --params '{"userId":"me"}' --json '{"displayLanguage":"<LANG>"}' # Update language
gws gmail users settings updateVacation --params '{"userId":"me"}' --json '{"responseSubject":"<SUB>","responseBodyPlainText":"<BODY>","restrictToDomain":true}' # Update vacation responder
### users.settings.filters
gws gmail users settings filters list --params '{"userId":"me"}' # List message filters
gws gmail users settings filters get --params '{"userId":"me","id":"<ID>"}' # Get a filter
gws gmail users settings filters create --params '{"userId":"me"}' --json '{"criteria":{"from":"<EMAIL>"},"action":{"addLabelIds":["<ID>"]}}' # Create a filter
gws gmail users settings filters delete --params '{"userId":"me","id":"<ID>"}' # Delete a filter
### users.settings.delegates
gws gmail users settings delegates list --params '{"userId":"me"}' # List delegates
gws gmail users settings delegates get --params '{"userId":"me","delegateEmail":"<EMAIL>"}' # Get a delegate
gws gmail users settings delegates create --params '{"userId":"me"}' --json '{"delegateEmail":"<EMAIL>"}' # Add a delegate
gws gmail users settings delegates delete --params '{"userId":"me","delegateEmail":"<EMAIL>"}' # Remove a delegate
### users.settings.forwardingAddresses
gws gmail users settings forwardingAddresses list --params '{"userId":"me"}' # List forwarding addresses
gws gmail users settings forwardingAddresses get --params '{"userId":"me","forwardingEmail":"<EMAIL>"}' # Get a forwarding address
gws gmail users settings forwardingAddresses create --params '{"userId":"me"}' --json '{"forwardingEmail":"<EMAIL>"}' # Create a forwarding address
gws gmail users settings forwardingAddresses delete --params '{"userId":"me","forwardingEmail":"<EMAIL>"}' # Delete a forwarding address
### users.settings.sendAs
gws gmail users settings sendAs list --params '{"userId":"me"}' # List send-as aliases
gws gmail users settings sendAs get --params '{"userId":"me","sendAsEmail":"<EMAIL>"}' # Get a send-as alias
gws gmail users settings sendAs create --params '{"userId":"me"}' --json '{"sendAsEmail":"<EMAIL>","displayName":"<NAME>"}' # Create a send-as alias
gws gmail users settings sendAs update --params '{"userId":"me","sendAsEmail":"<EMAIL>"}' --json '{"signature":"<SIG>"}' # Update a send-as alias
gws gmail users settings sendAs patch --params '{"userId":"me","sendAsEmail":"<EMAIL>"}' --json '{"signature":"<SIG>"}' # Update a send-as alias (patch)
gws gmail users settings sendAs delete --params '{"userId":"me","sendAsEmail":"<EMAIL>"}' # Delete a send-as alias
gws gmail users settings sendAs verify --params '{"userId":"me","sendAsEmail":"<EMAIL>"}' # Verify a send-as alias
### Gmail Label IDs
# INBOX, STARRED, IMPORTANT, SENT, DRAFT, SPAM, TRASH, UNREAD, CATEGORY_PERSONAL, CATEGORY_SOCIAL, CATEGORY_PROMOTIONS, CATEGORY_FORUMS, CATEGORY_UPDATES
## Discovering Commands
gws gmail --help # Browse resources and methods
gws schema gmail.users.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
