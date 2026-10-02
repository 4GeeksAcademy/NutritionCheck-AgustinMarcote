# Changelog

Trying to follow [semantic versioning](https://semver.org/) here — basically MAJOR.MINOR.PATCH, where:
- PATCH = small bug fix, nothing new added
- MINOR = added something new but didn't break anything old
- MAJOR = changed something in a way that would break stuff depending on the old version


## [1.0.2] - 2026-10-01
### Fixed
- "Invalid Barcode" error was returning status code 404 instead of 400. 404 should only be for "product not found," not for a barcode that's just formatted wrong. Copy-pasted this node from the Product Not Found one and forgot to change the code back — classic mistake, but an easy fix.

## [1.0.1] - 2026-09-29
### Fixed
- Webhook path was set to `/healthy-snack` instead of the `/nutrition-check` path the project spec actually asked for. No idea why I named it that originally, probably just typed the first thing that came to mind when I was testing.

## [1.0.0] - 2026-09-23
### Added
- First full working version of the workflow, end to end:
  - Webhook that accepts a barcode
  - Validation for missing/non-numeric barcodes
  - Open Food Facts lookup (geocoding-style GET request)
  - Handling for "product not found" (404 from the API) without crashing
  - Handling for products with no nutrition data ("insufficient data" response)
  - Traffic-light classification for sugar/salt/fat based on the thresholds from the brief
  - Logic that compares the traffic-light verdict against the official Nutri-Score and always goes with whichever one is worse
  - Concern score (0-3, counts how many nutrients are "high")
  - AI-generated verdict message with tone that adapts based on healthy/moderate/unhealthy
  - Final response: plain text for the actual verdict, JSON for every error/info path, with proper status codes

### Known issues at this point
- Salt and fat classification briefly had extra "medium" thresholds that weren't actually in the spec (fixed during testing, before this version — but flagging here since it was a real bug I caught along the way)
- Concern score was originally calculated as a weighted point total instead of a simple 0-3 count of "high" nutrients — also caught and fixed before this version

---

*Not tracking pre-1.0.0 versions since that was just me building the thing for the first time and everything was breaking constantly — didn't feel worth versioning that chaos.*
