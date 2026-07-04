# Sync Vendor Mock API from Master Sheet

Syncs a vendor mock API repo from the central Luminary Systems master asset sheet,
regenerates seed data, updates all docs, pushes to GitHub, and deploys to App Runner.

> **Automated alternative:** `python tools/deploy_mock.py --vendor <key>` runs steps 3–7
> automatically. Use `--list` to see all vendor keys. Use `--skip-docker` or `--skip-git`
> to run partial steps. This skill is for Claude Code orchestration when more control
> (e.g. README edits) is needed before deploying.

## Vendor Reference

| Vendor | Repo directory | Generator script | ECR repo name | Has GitHub Actions CI/CD |
|---|---|---|---|---|
| ServiceNow | mock-servicenow_v2_api | generate_sn_v2_data.py | mock-servicenow-v2-api | No (CI broken — AWS secrets not set in repo) |
| CrowdStrike | mock-crowdstrike-api | generate_cs_data.py | mock-crowdstrike-api | No |
| Intune | mock-intune-api | generate_intune_data.py | mock-intune-api | No |
| JAMF | mock-jamfpro-api | generate_jamf_data.py | mock-jamfpro-api | No |
| Ordr | mock-ordr-api | generate_ordr_data.py | mock-ordr-api | No |
| Mist | mock-mist-api | generate_mist_data.py | mock-mist-api-v2 | No |
| Meraki | mock-meraki-api | generate_meraki_data.py | mock-meraki-api | No |
| Aruba EdgeConnect | mock-aruba-ec-api | generate_ec_data.py | mock-aruba-ec-api | No |

**AWS config:** region `us-east-1`, account `905418046272`,
ECR base: `905418046272.dkr.ecr.us-east-1.amazonaws.com`

**Paths (absolute):**
- API root: `/Users/rrepswal/Library/CloudStorage/OneDrive-InfobloxInc/Documents/MockAPIs/API`
- Master sheet: `{API root}/luminary-demo-docs/master-sheet/assets_luminary.xlsx`
- Vendor repos: `{API root}/{repo directory}/`

---

## Steps to Follow

### Step 1 — Ask which vendor to update

Ask the user which vendor mock API they want to sync using AskUserQuestion with these options:
- ServiceNow
- CrowdStrike
- Intune
- JAMF
- Ordr
- Mist
- Meraki
- Aruba EdgeConnect

### Step 2 — Verify master sheet exists

Check that `luminary-demo-docs/master-sheet/assets_luminary.xlsx` exists in the API root.
If missing, tell the user to pull the latest `luminary-demo-docs` repo first and stop.

### Step 3 — Verify master sheet is accessible (no copy needed)

All generators use a central/local fallback: they look for the master sheet at
`{API root}/luminary-demo-docs/master-sheet/assets_luminary.xlsx` first, and only fall back
to their local `seed_data/source/assets_luminary.xlsx` copy if the central path is missing.

This means **no manual copy is required** as long as `luminary-demo-docs` is checked out
in the same `API/` folder as the vendor repos. Confirm the central xlsx exists:
```
{API root}/luminary-demo-docs/master-sheet/assets_luminary.xlsx
```
If it's missing, pull the `luminary-demo-docs` repo first. Do not copy the file manually.

### Step 4 — Run the generator

Run: `python3 {vendor repo}/seed_data/{generator script}`

The generator will print which xlsx it's reading (CENTRAL or LOCAL) — confirm it says CENTRAL.
Capture the full output — you'll need the total asset count and per-category/per-site breakdown
to update the README.

### Step 5 — Update the README

Read the vendor repo's `README.md`. Update:
- Total record/asset count (match what the generator printed)
- Per-site breakdown table if present
- Per-category breakdown table if present
- Any "last updated" or date references → set to today's date
- Master sheet source reference → point to `https://github.com/repswalp-cmd/luminary-demo-docs`

Do not rewrite the entire README — only update the numbers and source references.

### Step 6 — Git commit and push to GitHub

**CRITICAL: The Docker image (Step 7) is built from disk, not from git — but App Runner serves
whatever was baked into the image at build time. Always commit and push BEFORE building the image,
so the deployed service matches the git history.**

Stage all changed files in the vendor repo:
```
git add seed_data/raw/ seed_data/generate_*.py README.md
```
Commit with a clear message describing what changed (mention master sheet sync and date).
Push to `origin main`.

**Note — ServiceNow CI is broken**: the `deploy.yml` workflow exists but fails on every run
because `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` secrets are not set in the repo.
Treat ServiceNow the same as other vendors — build and push manually (Step 7).

### Step 7 — Deploy to ECR + App Runner (vendors WITHOUT GitHub Actions CI/CD)

For all vendors including ServiceNow — do this manually (ServiceNow CI/CD is broken):

**7a. Authenticate with ECR:**
```bash
aws ecr get-login-password --profile okta-sso --region us-east-1 \
  | docker login --username AWS --password-stdin 905418046272.dkr.ecr.us-east-1.amazonaws.com
```
(Profile is `okta-sso` — the only configured AWS profile in this environment.)

**7b. Build the image (must be linux/amd64 for App Runner):**

Use an intermediate tag to avoid the zsh `:l` modifier bug (applies to all vendors):
```bash
docker build --no-cache --platform linux/amd64 -t mock-{vendor}-build {vendor repo path}
docker tag mock-{vendor}-build 905418046272.dkr.ecr.us-east-1.amazonaws.com/{ecr-repo-name}:latest
```

**7c. Push to ECR:**
```bash
docker push 905418046272.dkr.ecr.us-east-1.amazonaws.com/{ecr-repo-name}:latest
```

**7d. App Runner auto-deploys** via `AutoDeploymentsEnabled=True` — no manual trigger needed.
Verify the deployment started:
```bash
aws apprunner list-services --profile okta-sso --region us-east-1 \
  --query 'ServiceSummaryList[?contains(ServiceName, `{ecr-repo-name}`)].[ServiceName,Status]' \
  --output table
```

**Meraki-specific note:** The Meraki App Runner service runs in permissive auth mode
(no `MERAKI_API_KEY` env var set) — any non-empty string is accepted as the API key.
In the UAI portal, set the Base URL to the App Runner URL and enter any placeholder key.

**Aruba EdgeConnect-specific note:** UAI portal has three fields — **Username**, **Password**,
and **Orchestrator URL** (not "Base URL"). Use:
- Username: `lsys-ec-admin`
- Password: `Lum1nary@Aruba#2026`
- Orchestrator URL: `https://bbkvcuavhc.us-east-1.awsapprunner.com`
- Skip TLS Verification: **Enabled**
- IPAM Discovery: **Enabled**, Federated Realm: **Default**

### Step 8 — Verify CSP Mock Deployer backend autowiring

After the App Runner service is running, confirm the CSP Mock Deployer backend is correctly
wired for this vendor. Read:

```
/Users/rrepswal/Library/CloudStorage/OneDrive-InfobloxInc/Documents/MockAPIs/csp-mock-deployer/backend/store.py
```

Find the vendor's entry in `VENDOR_CATALOG`. Use this mapping from sync-vendor name → catalog key:

| Sync-vendor name | VENDOR_CATALOG key |
|---|---|
| ServiceNow | ServiceNow |
| CrowdStrike | CrowdStrike |
| Intune | Microsoft Intune |
| JAMF | Jamf Pro |
| Ordr | Ordr |
| Mist | Juniper Mist |
| Meraki | Cisco Meraki |
| Aruba EdgeConnect | Aruba EdgeConnect |

**Check 1 — apprunner_service name matches ECR repo name**

Confirm `provider["apprunner_service"]` in the catalog matches the ECR repo name from the
Vendor Reference table at the top of this skill. A mismatch means `provider.py` will call
`apprunner:DescribeService` on the wrong service and credential fields will come back empty.

**Check 2 — hardcoded endpoint URL is live**

Some vendors have a hardcoded `endpoint` key in `provider` (Intune, Meraki, Mist, Aruba
EdgeConnect). For those, fetch the live App Runner URL:

```bash
aws apprunner list-services --profile okta-sso --region us-east-1 \
  --query 'ServiceSummaryList[?contains(ServiceName, `{ecr-repo-name}`)].[ServiceName,ServiceUrl]' \
  --output table
```

Confirm the catalog `endpoint` (`https://<host>`) matches the live `ServiceUrl`. If they differ,
update `store.py` to use the correct URL and note it for the user.

**Check 3 — credential env var names match the vendor app.py**

List the `env` values for each entry in `provider["fields"]` and verify they match the env var
names actually read by the vendor's `app.py` (e.g. `os.environ.get("INTUNE_CLIENT_ID")`).
A mismatch means `_provider_fields()` fetches the right App Runner service but reads the wrong
env var, returning blank credentials to the UI.

**Report the result** — pass (all three checks green) or flag any discrepancy with the exact
fix needed before the user runs an autowire from the CSP Mock Deployer.

### Step 9 — Report to user

Summarise what was done:
- Master sheet version used (file size or record count)
- Generator output (total records, by site, by category)
- Files changed in the repo
- GitHub push status + Actions URL (if ServiceNow)
- ECR push status and App Runner deployment status
- Step 8 autowiring check result (pass / issues found)
- Any errors encountered
