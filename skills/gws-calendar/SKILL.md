---
name: gws-calendar
version: 1.0.1
description: "Google Calendar: Manage calendars, ACL, and settings."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws calendar --help"
---
## Commands
```bash
gws calendar <resource> <method> [flags]
### Helper Commands
gws calendar +insert --summary '<TITLE>' --start '<ISO_TIME>' --end '<ISO_TIME>' # Create a new event
gws calendar +insert --summary '<TITLE>' --start '<ISO_TIME>' --end '<ISO_TIME>' --attendee '<EMAIL>' # Create with attendee
gws calendar +agenda # Show upcoming events
gws calendar +agenda --today # Show today's events
gws calendar +agenda --week # Show this week's events
gws calendar +agenda --days <N> # Show N days ahead
gws calendar +agenda --calendar '<NAME>' # Filter by calendar
### calendarList
gws calendar calendarList list # List calendars on user's calendar list
gws calendar calendarList get --params '{"calendarId":"<ID>"}' # Get a calendar from user's list
gws calendar calendarList insert --json '{"id":"<ID>"}' # Add existing calendar to user's list
gws calendar calendarList update --params '{"calendarId":"<ID>"}' --json '{}' # Update calendar on user's list
gws calendar calendarList patch --params '{"calendarId":"<ID>"}' --json '{}' # Update calendar on user's list (patch)
gws calendar calendarList delete --params '{"calendarId":"<ID>"}' # Remove calendar from user's list
### calendars
gws calendar calendars get --params '{"calendarId":"<ID>"}' # Get calendar metadata
gws calendar calendars insert --json '{"summary":"<NAME>"}' # Create a secondary calendar
gws calendar calendars update --params '{"calendarId":"<ID>"}' --json '{}' # Update calendar metadata
gws calendar calendars patch --params '{"calendarId":"<ID>"}' --json '{}' # Update calendar metadata (patch)
gws calendar calendars delete --params '{"calendarId":"<ID>"}' # Delete a secondary calendar
gws calendar calendars clear --params '{"calendarId":"<ID>"}' # Clear all events from primary calendar
### acl
gws calendar acl list --params '{"calendarId":"<ID>"}' # List ACL rules for a calendar
gws calendar acl get --params '{"calendarId":"<ID>","ruleId":"<ID>"}' # Get an ACL rule
gws calendar acl insert --params '{"calendarId":"<ID>"}' --json '{"role":"<ROLE>","scope":{"type":"<TYPE>","value":"<VALUE>"}}' # Create an ACL rule
gws calendar acl update --params '{"calendarId":"<ID>","ruleId":"<ID>"}' --json '{}' # Update an ACL rule
gws calendar acl patch --params '{"calendarId":"<ID>","ruleId":"<ID>"}' --json '{}' # Update an ACL rule (patch)
gws calendar acl delete --params '{"calendarId":"<ID>","ruleId":"<ID>"}' # Delete an ACL rule
### freebusy
gws calendar freebusy query --json '{"timeMin":"<ISO>","timeMax":"<ISO>","items":[{"id":"<CALENDAR_ID>"}]}' # Check free/busy for calendars
### settings
gws calendar settings list # List all user settings
gws calendar settings get --params '{"setting":"<NAME>"}' # Get a user setting
### colors
gws calendar colors get # Get color definitions for calendars and events
### channels
gws calendar channels stop --json '{"id":"<CHANNEL_ID>","resourceId":"<RESOURCE_ID>"}' # Stop watching a channel
## Discovering Commands
gws calendar --help # Browse resources and methods
gws schema calendar.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
