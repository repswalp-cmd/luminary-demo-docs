# vendor-brief — UAI Integration Documentation Briefing

Produces the complete documentation set for a UAI vendor integration:
1. `<Vendor>_Bugs.xlsx` — Bug registry, field-level examples, and reproduction steps
2. `UAI_Integrations_Playbook.docx` — Fills the vendor's placeholder section with use cases, talk track, objections, and qualification guide
3. `assets_of_interest_tracker.docx` — Adds any relevant AOI finding or scenario entries
4. Git commit + push everything to `repswalp-cmd/luminary-demo-docs`

Run as: `/vendor-brief <Vendor Name>`
Example: `/vendor-brief CrowdStrike Falcon`

---

## Context and File Paths

**Docs folder (all files live here):**
`/Users/rrepswal/Library/CloudStorage/OneDrive-InfobloxInc/Documents/MockAPIs/API/luminary-demo-docs/fixes/`

**Key files:**
- `UAI_Integrations_Playbook.docx` — multi-vendor SE playbook
- `assets_of_interest_tracker.docx` — AOI finding status + change log
- `<Vendor>_Bugs.xlsx` — one per vendor, created by this skill

**Git repo:** `repswalp-cmd/luminary-demo-docs` (private)
Working dir for git commands: `/Users/rrepswal/Library/CloudStorage/OneDrive-InfobloxInc/Documents/MockAPIs/API/luminary-demo-docs/`

**Colour palette (use consistently across all Excel and Word output):**
- Navy header: `1B3A5C` / White text: `FFFFFF`
- Green (available today): `1B5E20` bg, `E8F5E9` light
- Teal (future/post-fix): `006064` bg, `E0F2F1` light
- Red (blocked/bug): `C62828` bg, `FDECEA` light
- Amber (warning): `FFF3CD`
- Slate (section dividers): `37474F`
- Row alternating: `EEF3F8` / `FFFFFF`

---

## Vendor Catalog

| Vendor argument | Integration # in Playbook | Mock API URL |
|---|---|---|
| Microsoft Intune | 1 | https://bwauy2kxvq.us-east-1.awsapprunner.com |
| CrowdStrike Falcon | 2 | https://iviyrvztrr.us-east-1.awsapprunner.com |
| Juniper Mist | 3 | https://grq3hipzpu.us-east-1.awsapprunner.com |
| Cisco Meraki | 4 | https://4ndm7ee4pz.us-east-1.awsapprunner.com |
| Jamf Pro | 5 | https://qceaa5a7pv.us-east-1.awsapprunner.com |
| Tenable | 6 | https://9rdis2evgu.us-east-1.awsapprunner.com |
| ServiceNow | 7 | https://6rm6inhcea.us-east-1.awsapprunner.com |
| Aruba EdgeConnect | 8 | https://bbkvcuavhc.us-east-1.awsapprunner.com |
| VMware vCenter | 9 | https://j6ntvkhgtb.us-east-1.awsapprunner.com |
| Ordr | 10 | https://dm6pi7e2mj.us-east-1.awsapprunner.com |

---

## Step-by-Step Instructions

### STEP 1 — Determine the vendor

The vendor name comes from `$ARGUMENTS`. If no argument is given, use `AskUserQuestion` to
ask which vendor to brief. Match to the catalog table above to get the Integration # and Mock URL.

### STEP 2 — Research phase (do this before writing anything)

Before generating any output, gather facts about this vendor's UAI integration:

**2a. Check the mock API's debug/requests endpoint**
```
GET <mock-api-url>/debug/requests
```
This shows the exact HTTP requests UAI has made — endpoints, `$select` field lists, query params.
Parse out:
- The list call: path, `$select` fields, `$top`, any filters
- The detail call (if any): path, `$select` fields
- Any non-200 responses

**2b. Call the mock API directly**
Get a sample asset (use the token endpoint if needed — check the vendor's CLAUDE.md for auth).
Fetch 1–3 representative assets including at least one with notable field values.

**2c. Check what UAI actually stores**
Use the Infoblox CSP MCP tools (`mcp__infoblox-csp__infoblox-portal_make_get_request`) to fetch
a live UAI asset for this vendor. Compare mock fields vs UAI stored values field by field.
Note every field where mock returns a value but UAI stores null, empty, 0, or wrong value.

**2d. Check the vendor's CLAUDE.md**
Read `/Users/rrepswal/Library/CloudStorage/OneDrive-InfobloxInc/Documents/MockAPIs/API/mock-<vendor-slug>-api/CLAUDE.md`
for: fleet composition, auth pattern, known issues, field notes.

**2e. Check the Intune_Bugs.xlsx as a reference template**
Read `fixes/Intune_Bugs.xlsx` to understand the exact format expected for all three sheets.
Mirror the column widths, colour scheme, font sizes, and row heights.

### STEP 3 — Create `<Vendor>_Bugs.xlsx`

Create the file at:
`/Users/rrepswal/Library/CloudStorage/OneDrive-InfobloxInc/Documents/MockAPIs/API/luminary-demo-docs/fixes/<VendorName>_Bugs.xlsx`

Use `openpyxl`. Do NOT use merged cells — use plain cells only (merged cells corrupt the file
on save). Use `PatternFill`, `Font`, `Alignment`, `Border` from `openpyxl.styles`.
After saving, reload the file and verify all sheets exist with non-empty cell counts before
proceeding. If a sheet is missing or empty, the save failed — fix and retry.

**Sheet 1 — Bug Registry** (red tab `C62828`)

13 columns with these exact headers and approximate widths:
Bug ID (9) | Title (38) | Connector (18) | Component (20) | Severity (11) | Status (11) |
Description (52) | Root Cause (48) | Fields Affected (32) | Mock Returns (32) |
UAI Stores (22) | Workaround / Notes (40) | Date Found (13)

- Header row: navy `1B3A5C` bg, white text, bold, size 9
- Bug IDs: `BUG-001`, `BUG-002`, etc.
- Severity row tints: High → `FDECEA`, Medium → `FFFDE7`, Low → `EEF3F8`
- Severity text colours: High `C62828`, Medium `E65100`, Low `558B2F`
- Status text colours: Open `C62828`, Known `1565C0`, Won't Fix `4A148C`
- Freeze row 3 (below title banner + header), enable auto-filter on header row
- Row height: 90pt for data rows

One row per bug discovered in Step 2c. If no bugs are found (UAI stores everything correctly),
add a single informational row: `BUG-000 · No gaps found — all fields stored correctly`.

**Sheet 2 — Examples** (orange tab `E65100`)

Show 1–2 real devices from the Luminary fleet for this vendor.
Columns: Bug ID | Field Name (API) | UAI Fetches? ($select) | Mock API Returns ✅ | UAI Stores ❌ | Business Impact | Severity | Device Name | Asset State

- Section A header (teal `006064`): a noncompliant / interesting device
- Section B header (green `1B5E20`): a compliant / working field as proof the pipeline works
- Mock Returns column: green text `1B5E20` (correct data)
- UAI Stores column: red text `C62828` (wrong/missing data)
- Row height: 110pt

Use real device hostnames from the Luminary fleet (format: `lsys-<site>-<type>-<num>`).
Use real field values — fetch them from the mock API in Step 2b.

**Sheet 3 — How to Reproduce** (blue tab `1565C0`)

3 columns: Step# (6) | Action Label (30) | Detail / Command (88)

7 parts:
1. What You Need — mock URL, credentials/API key, tool (curl/Postman), UAI portal access, example device
2. Get Auth Token — copy-paste curl command for the vendor's auth flow
3. Reproduce the most impactful bug — exact curl, expected mock response (green bg), what UAI shows (red bg)
4. Reproduce the second most impactful bug
5. Reproduce a third bug or field gap
6. Verify what DOES work — a field that UAI correctly stores (green bg proof)
7. Root cause summary + evidence package for UAI engineering

- Code/command rows: dark bg `1E272E`, Courier New font, green text `A5D6A7`
- Mock-returns rows: light green bg `E8F5E9`
- UAI-stores (wrong) rows: light red bg `FDECEA`
- Freeze row 3

### STEP 4 — Update `UAI_Integrations_Playbook.docx`

Open the file, find the vendor's placeholder section (e.g. "INTEGRATION #2  ·  CROWDSTRIKE FALCON").
Replace the placeholder paragraph with the full content — same 4-subsection structure as
Integration #1 (Microsoft Intune):

**Section N.1 — Customer Use Cases**
- Sub-section A: Use cases available today (what the mock serves + UAI stores correctly)
  Format each as: UC-N · Name | PROBLEM | WHAT UAI ADDS | CUSTOMER QUESTION ANSWERED
  Green header `1B5E20`, light green value bg `E8F5E9`
- Sub-section B: Use cases unlocked once UAI bugs are fixed (reference bug IDs from Sheet 1)
  Teal header `006064`, teal light value bg `E0F2F1`

**Section N.2 — Sales Talk Track**
- Pre-demo check: setup steps + ⚠️ what to avoid (amber bg `FFF3CD`)
- Steps to demo today: numbered steps with exact navigation path + "SAY:" script line
  Slate header `37474F`, alternating row bg `EEF3F8`
- Do Not Attempt Today: fields that will show blank (red bg `FDECEA`)
- Future demo angles: post-fix story (teal bg `E0F2F1`)

**Section N.3 — Discovery Questions & Objection Handling**
- 3–4 discovery questions framed as customer pain
- 3–4 objections with responses, including one for any blank/missing fields in the demo

**Section N.4 — Qualification Guide**
- Persona → primary use cases table
- Strong fit signals
- Weak fit signals
- Close line

Use `python-docx`. Do NOT use merged cells. Tables with `'Table Grid'` style. Set column widths
via `row.cells[n].width = docx.shared.Inches(x)` on the first data row. Use `set_cell_bg()`
with a tcPr/shd element (same pattern as existing content). Font size 9pt for data, 10–11pt
for section headers.

### STEP 5 — Update `assets_of_interest_tracker.docx`

Open the tracker. Decide whether this vendor warrants:

**Table 0 (Finding Status):** Add a new row if there is a concrete AOI scenario for this vendor
(e.g. a specific count of at-risk devices, a specific compliance or threat finding). Use the
same 6-column format: # | Finding | Target | Status | Mocks Involved | Date.
Status colours: READY `1B5E20`, NEAR-READY `E65100`, BLOCKED `C62828`.
If no concrete AOI scenario is ready yet, skip this table — do not add placeholder rows.

**Table 2 (Change Log):** Add a SCENARIO or ENRICHMENT block if applicable:
- SCENARIO block (purple header `4A0072`): new AOI demo scenario enabled by this vendor
- ENRICHMENT block (green header `1B5E20`): field enrichment done in the mock generator
Format mirrors existing blocks (SCENARIO-001, SCENARIO-002, ENRICHMENT-001).
If nothing changed in the mock data for this vendor yet, skip.

Do not add a talk track table — that lives in `UAI_Integrations_Playbook.docx`.
Do not add a "Related Documents" row — one already exists at the bottom of Table 0.

### STEP 6 — Verify all files are correct

Before committing:
```python
# Verify Excel
import openpyxl
wb = openpyxl.load_workbook('<Vendor>_Bugs.xlsx')
assert wb.sheetnames == ['Bug Registry', 'Examples', 'How to Reproduce'], f"Missing sheets: {wb.sheetnames}"
for ws in wb.worksheets:
    nc = sum(1 for row in ws.iter_rows(values_only=True) for c in row if c)
    assert nc > 10, f"Sheet '{ws.title}' looks empty: {nc} cells"
    print(f"  '{ws.title}': {ws.max_row} rows, {nc} cells — OK")
```

If any assertion fails, fix before proceeding.

### STEP 7 — Git commit and push

```bash
cd /Users/rrepswal/Library/CloudStorage/OneDrive-InfobloxInc/Documents/MockAPIs/API/luminary-demo-docs

git add fixes/<Vendor>_Bugs.xlsx fixes/UAI_Integrations_Playbook.docx fixes/assets_of_interest_tracker.docx

git commit -m "Add <Vendor> integration brief: bugs, playbook section, tracker update

fixes/<Vendor>_Bugs.xlsx (new):
  - Bug Registry: <N> bugs found in UAI <Vendor> connector
  - Examples: field-by-field mock vs UAI comparison for <device>
  - How to Reproduce: copy-paste curl commands + UAI portal steps

UAI_Integrations_Playbook.docx:
  - Integration #<N> <Vendor> section filled: use cases, talk track,
    discovery questions, objections, qualification guide

assets_of_interest_tracker.docx:
  - <describe what was added or 'no changes needed'>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"

git push origin main
```

---

## Output to User

After completing all steps, report:

1. **`<Vendor>_Bugs.xlsx`** — N bugs found, sheet cell counts confirmed
2. **`UAI_Integrations_Playbook.docx`** — Integration #N section filled (use cases, talk track, objections, qualification)
3. **`assets_of_interest_tracker.docx`** — what was added (or "no changes needed — no concrete AOI scenario ready yet")
4. **Git** — commit hash + push status

If any step fails, report what failed and what the user needs to do manually.

---

## Quality Rules

- **Never use merged cells in openpyxl** — they silently corrupt files on save. Use plain cells only.
- **Always verify Excel sheets after saving** — reload and count non-empty cells before committing.
- **Use real values** — fetch actual device data from the mock API. Do not fabricate field values.
- **Mirror the Intune format exactly** — column widths, colours, font sizes, row heights. Consistency matters across vendors.
- **Talk track must have an avoid list** — always explicitly list what will show blank/null so the SE doesn't accidentally demo broken fields.
- **Use cases split A/B** — always separate "available today" from "post UAI fix". Never present future capabilities as current.
- **Do not duplicate content** — if something is fully covered in UAI_Integrations_Playbook, do not repeat it in the tracker. Each document has one job.
