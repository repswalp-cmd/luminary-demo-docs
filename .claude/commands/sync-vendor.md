# Sync Vendor Mock API from Master Sheet

Syncs a vendor mock API repo from the central Luminary Systems master asset sheet,
regenerates seed data, updates all docs, pushes to GitHub, and deploys to App Runner.

## Vendor Reference

| Vendor | Repo directory | Generator script | ECR repo name | Has GitHub Actions CI/CD |
|---|---|---|---|---|
| ServiceNow | mock-servicenow_v2_api | generate_sn_v2_data.py | mock-servicenow-v2-api | Yes |
| CrowdStrike | mock-crowdstrike-api | generate_cs_data.py | mock-crowdstrike-api | No |
| Intune | mock-intune-api | generate_intune_data.py | mock-intune-api | No |
| JAMF | mock-jamfpro-api | generate_jamf_data.py | mock-jamfpro-api | No |

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

### Step 2 — Verify master sheet exists

Check that `luminary-demo-docs/master-sheet/assets_luminary.xlsx` exists in the API root.
If missing, tell the user to pull the latest `luminary-demo-docs` repo first and stop.

### Step 3 — Copy master sheet into vendor repo

Copy the master sheet to `{vendor repo}/seed_data/source/assets_luminary.xlsx`.

### Step 4 — Run the generator

Run: `python3 {vendor repo}/seed_data/{generator script}`

Capture the output — you'll need the total asset count and per-category/per-site breakdown
to update the README.

### Step 5 — Update the README

Read the vendor repo's `README.md`. Update:
- Total record/asset count (match what the generator printed)
- Per-site breakdown table if present
- Per-category breakdown table if present
- Any "last updated" or date references → set to today's date (2026-07-01)
- Master sheet source reference → point to `https://github.com/repswalp-cmd/luminary-demo-docs`

Do not rewrite the entire README — only update the numbers and source references.

### Step 6 — Git commit and push to GitHub

Stage all changed files in the vendor repo:
```
git add seed_data/ README.md
```
Commit with a clear message describing what changed (mention master sheet sync and date).
Push to `origin main`.

For **ServiceNow**: the GitHub Actions `deploy.yml` workflow triggers automatically on push.
It builds the Docker image and pushes to ECR. App Runner picks it up via AutoDeploymentsEnabled.
Tell the user the push has triggered CI/CD and they can monitor at:
`https://github.com/repswalp-cmd/{repo}/actions`

### Step 7 — Deploy to ECR + App Runner (vendors WITHOUT GitHub Actions CI/CD)

Skip this step for ServiceNow (handled by GitHub Actions in Step 6).

For CrowdStrike, Intune, and JAMF — do this manually:

**7a. Authenticate with ECR:**
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 905418046272.dkr.ecr.us-east-1.amazonaws.com
```

**7b. Build the image (must be linux/amd64 for App Runner):**
```bash
docker build --no-cache --platform linux/amd64 \
  -t 905418046272.dkr.ecr.us-east-1.amazonaws.com/{ecr-repo-name}:latest \
  {vendor repo path}
```

**7c. Push to ECR:**
```bash
docker push 905418046272.dkr.ecr.us-east-1.amazonaws.com/{ecr-repo-name}:latest
```

**7d. App Runner auto-deploys** via `AutoDeploymentsEnabled=True` — no manual trigger needed.
Verify the deployment started:
```bash
aws apprunner list-services --region us-east-1 \
  --query 'ServiceSummaryList[?contains(ServiceName, `{ecr-repo-name}`)].[ServiceName,Status]' \
  --output table
```

### Step 8 — Report to user

Summarise what was done:
- Master sheet version used (file size or record count)
- Generator output (total records, by site, by category)
- Files changed in the repo
- GitHub push status + Actions URL (if ServiceNow)
- ECR push status and App Runner deployment status
- Any errors encountered
