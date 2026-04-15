---
name: gws-drive
description: "Google Drive: Manage files, folders, and shared drives."
metadata:
  version: 0.22.6
  openclaw:
    category: "productivity"
    requires:
      bins:
        - gws
    cliHelp: "gws drive --help"
---
```bash
gws drive <resource> <method> [flags]
### about
gws drive about get # Get user, Drive, and system capability info
```
### command
```bash
gws drive files list --params '{"q":"<QUERY>"}' # List files (supports search query)
gws drive files get --params '{"fileId":"<ID>"}' # Get file metadata or content
gws drive files create --params '{}' --json '{}' # Create a file
gws drive files copy --params '{"fileId":"<ID>"}' --json '{}' # Copy a file
gws drive files update --params '{"fileId":"<ID>"}' --json '{}' # Update file metadata or content
gws drive files delete --params '{"fileId":"<ID>"}' # Permanently delete a file
gws drive files download --params '{"fileId":"<ID>"}' # Download file content
gws drive files export --params '{"fileId":"<ID>","mimeType":"<MIME>"}' # Export Google doc to MIME type
gws drive files generateIds --params '{"count":<N>}' # Generate file IDs for create/copy
gws drive files emptyTrash # Permanently delete all trashed files
gws drive files listLabels --params '{"fileId":"<ID>"}' # List labels on a file
gws drive files modifyLabels --params '{"fileId":"<ID>"}' --json '{}' # Modify labels on a file
gws drive files generateCseToken # Generate CSE token for CSE files
gws drive files watch --params '{"fileId":"<ID>"}' --json '{}' # Subscribe to file changes
### +upload Upload files to Google Drive with automatic MIME type detection and metadata.
gws drive +upload ./report.pdf --params '{"parents":["<FOLDER_ID>"]}' # Upload a file with automatic metadata
gws drive +upload ./data.csv --name 'Sales Data.csv'
### drives
gws drive drives list # List shared drives
gws drive drives get --params '{"driveId":"<ID>"}' # Get shared drive metadata
gws drive drives create --params '{"name":"<NAME>"}' # Create a shared drive
gws drive drives update --params '{"driveId":"<ID>"}' --json '{}' # Update shared drive metadata
gws drive drives delete --params '{"driveId":"<ID>"}' # Permanently delete a shared drive
gws drive drives hide --params '{"driveId":"<ID>"}' # Hide shared drive from default view
gws drive drives unhide --params '{"driveId":"<ID>"}' # Restore shared drive to default view
### permissions
gws drive permissions list --params '{"fileId":"<ID>"}' # List permissions on a file/drive
gws drive permissions get --params '{"fileId":"<ID>","permissionId":"<ID>"}' # Get a permission by ID
gws drive permissions create --params '{"fileId":"<ID>"}' --json '{"role":"<ROLE>","type":"<TYPE>"}' # Create a permission
gws drive permissions update --params '{"fileId":"<ID>","permissionId":"<ID>"}' --json '{}' # Update a permission
gws drive permissions delete --params '{"fileId":"<ID>","permissionId":"<ID>"}' # Delete a permission
### comments
gws drive comments list --params '{"fileId":"<ID>"}' # List comments on a file
gws drive comments get --params '{"fileId":"<ID>","commentId":"<ID>"}' # Get a comment by ID
gws drive comments create --params '{"fileId":"<ID>"}' --json '{"content":"<TEXT>"}' # Create a comment
gws drive comments update --params '{"fileId":"<ID>","commentId":"<ID>"}' --json '{}' # Update a comment
gws drive comments delete --params '{"fileId":"<ID>","commentId":"<ID>"}' # Delete a comment
### replies
gws drive replies list --params '{"fileId":"<ID>","commentId":"<ID>"}' # List replies to a comment
gws drive replies get --params '{"fileId":"<ID>","commentId":"<ID>","replyId":"<ID>"}' # Get a reply by ID
gws drive replies create --params '{"fileId":"<ID>","commentId":"<ID>"}' --json '{"content":"<TEXT>"}' # Create a reply
gws drive replies update --params '{"fileId":"<ID>","commentId":"<ID>","replyId":"<ID>"}' --json '{}' # Update a reply
gws drive replies delete --params '{"fileId":"<ID>","commentId":"<ID>","replyId":"<ID>"}' # Delete a reply
### revisions
gws drive revisions list --params '{"fileId":"<ID>"}' # List revisions of a file
gws drive revisions get --params '{"fileId":"<ID>","revisionId":"<ID>"}' # Get revision metadata or content
gws drive revisions update --params '{"fileId":"<ID>","revisionId":"<ID>"}' --json '{}' # Update a revision
gws drive revisions delete --params '{"fileId":"<ID>","revisionId":"<ID>"}' # Delete a revision
### changes
gws drive changes list --params '{"pageToken":"<TOKEN>"}' # List changes for user/shared drive
gws drive changes getStartPageToken # Get starting pageToken for changes
gws drive changes watch --params '{"pageToken":"<TOKEN>"}' --json '{}' # Subscribe to changes
### accessproposals
gws drive accessproposals list --params '{"fileId":"<ID>"}' # List access proposals on a file
gws drive accessproposals get --params '{"fileId":"<ID>","accessProposalId":"<ID>"}' # Get access proposal by ID
gws drive accessproposals resolve --params '{"fileId":"<ID>","accessProposalId":"<ID>"}' --json '{"resolution":"<APPROVE|DENY>"}' # Approve or deny a proposal
### approvals
gws drive approvals list --params '{"fileId":"<ID>"}' # List approvals on a file
gws drive approvals get --params '{"approvalId":"<ID>"}' # Get an approval by ID
### apps
gws drive apps list # List installed apps
gws drive apps get --params '{"appId":"<ID>"}' # Get a specific app
### operations
gws drive operations get --params '{"name":"<OPERATION_NAME>"}' # Get state of a long-running operation
### channels
gws drive channels stop --json '{"id":"<CHANNEL_ID>","resourceId":"<RESOURCE_ID>"}' # Stop watching a channel
## Discovering Commands
gws drive --help # Browse resources and methods
gws schema drive.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
