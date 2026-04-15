---
name: gws-sheets
version: 1.0.1
description: "Google Sheets: Read and write spreadsheets."
metadata:
  openclaw:
    category: "productivity"
    requires:
      bins: ["gws"]
    cliHelp: "gws sheets --help"
---
## Commands
```bash
gws sheets <resource> <method> [flags]
### Helpers
gws sheets +read --spreadsheet '<ID>' --range '<RANGE>' # Read values from a range (e.g. Sheet1!A1:D10)
gws sheets +append --spreadsheet '<ID>' --values 'a,b,c' # Append a single row
gws sheets +append --spreadsheet '<ID>' --json-values '[["a","b"],["c","d"]]' # Append multiple rows
### spreadsheets
gws sheets spreadsheets get --params '{"spreadsheetId":"<ID>"}' # Get a spreadsheet by ID
gws sheets spreadsheets create --params '{"title":"<TITLE>"}' # Create a blank spreadsheet
gws sheets spreadsheets batchUpdate --params '{"spreadsheetId":"<ID>"}' --json '{"requests":[...]}' # Apply updates to a spreadsheet
gws sheets spreadsheets getByDataFilter --params '{"spreadsheetId":"<ID>"}' --json '{}' # Get spreadsheet with data filters
### spreadsheets.values
gws sheets spreadsheets values get --params '{"spreadsheetId":"<ID>","range":"<RANGE>"}' # Get values from a range
gws sheets spreadsheets values update --params '{"spreadsheetId":"<ID>","range":"<RANGE>","valueInputOption":"RAW"}' --json '{"values":[["a","b"],["c","d"]]}' # Set values in a range
gws sheets spreadsheets values batchGet --params '{"spreadsheetId":"<ID>","range":["<R1>","<R2>"]}' # Get values from multiple ranges
gws sheets spreadsheets values batchUpdate --params '{"spreadsheetId":"<ID>","valueInputOption":"RAW"}' --json '{"data":[{"range":"<R>","values":[["a","b"]]}]}' # Set values in multiple ranges
gws sheets spreadsheets values batchClear --params '{"spreadsheetId":"<ID>"}' --json '{"ranges":["<R1>"]}' # Clear values in ranges
gws sheets spreadsheets values append --params '{"spreadsheetId":"<ID>","range":"<R>","valueInputOption":"RAW"}' --json '{"values":[["a","b"]]}' # Append values to a range
gws sheets spreadsheets values clear --params '{"spreadsheetId":"<ID>"}' --json '{"range":"<R>"}' # Clear a range
### spreadsheets.sheets
gws sheets spreadsheets sheets copyTo --params '{"spreadsheetId":"<ID>","sheetId":<N>}' --json '{"destinationSpreadsheetId":"<ID>"}' # Copy a sheet to another spreadsheet
### spreadsheets.developerMetadata
gws sheets spreadsheets developerMetadata get --params '{"spreadsheetId":"<ID>","metadataId":<N>}' # Get developer metadata by ID
gws sheets spreadsheets developerMetadata search --params '{"spreadsheetId":"<ID>"}' --json '{"dataFilters":[{"developerMetadataLookup":{}}]}' # Search developer metadata
## Discovering Commands
gws sheets --help # Browse resources and methods
gws schema sheets.<resource>.<method> # Inspect required params, types, and defaults
gws auth setup
gws auth login
```
