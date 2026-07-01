# Luminary Systems — Demo Assets Documentation

Central reference repository for the **Luminary Systems UAI Demo Dataset** used across all Infoblox mock API integrations.

## Contents

### `master-sheet/`
| File | Description |
|---|---|
| `assets_luminary.xlsx` | Master asset dataset — source of truth for all vendor mock APIs |

The master sheet contains **2,293 assets** across 19 categories and 6 global sites (San Francisco, New York, London, Amsterdam, Singapore, Bangalore). Any change to the dataset must be made here first, then propagated to each vendor repo.

### `analysis/`
| File | Description |
|---|---|
| `assets_luminary_analysis.md` | Full anomaly analysis report — all 19 categories |
| `assets_luminary_analysis.docx` | Same report in Word format |

Covers duplicate IPs, hostname/category mismatches, printer manufacturer anomalies, site coverage gaps, and informational notes on IoT, clinic, mobile, and SD-WAN sources.

### `fixes/`
| File | Description |
|---|---|
| `data_fixes_log.docx` | Log of all data fixes applied to the master sheet |

Documents 8 fixes applied on 2026-07-01 including duplicate IP resolution, hostname standardization, printer rebalancing, and new firewall records.

---

## Vendor Repos Using This Dataset

| Repo | Integration | Records Used |
|---|---|---|
| [mock-servicenow_v2_api](https://github.com/repswalp-cmd/mock-servicenow_v2_api) | ServiceNow CMDB | ~1,154 |
| [mock-crowdstrike-api](https://github.com/repswalp-cmd/mock-crowdstrike-api) | CrowdStrike Falcon | 999 |
| [mock-intune-api](https://github.com/repswalp-cmd/mock-intune-api) | Microsoft Intune | 763 |
| [mock-jamfpro-api](https://github.com/repswalp-cmd/mock-jamfpro-api) | Jamf Pro | 20 |

---

## How to Update the Dataset

1. Edit `master-sheet/assets_luminary.xlsx`
2. Copy the updated file to each vendor repo: `{repo}/seed_data/source/assets_luminary.xlsx`
3. Run each vendor's generator: `python3 seed_data/generate_*.py`
4. Commit and push all repos
5. Document any data changes in `fixes/data_fixes_log.docx`
