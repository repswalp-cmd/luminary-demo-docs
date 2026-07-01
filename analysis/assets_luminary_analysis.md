# Luminary Systems — Master Assets Analysis Report

**Dataset:** `assets_luminary.xlsx` — UAI Demo Dataset Spec v6  
**Scope:** 2,291 assets across 19 categories, 6 global sites  
**Analyzed:** 2026-07-01

---

## Sites

| Site | Abbreviation |
|---|---|
| San Francisco | sfo |
| New York | nyc |
| London | lon |
| Amsterdam | ams |
| Singapore | sgp |
| Bangalore | blr |

---

## Category Inventory

| Category | Count | Notes |
|---|---|---|
| win_laptop | 545 | |
| mac_laptop | 310 | |
| mobile_ios | 275 | Not in ServiceNow/CrowdStrike |
| iot | 245 | Not in ServiceNow |
| mobile_droid | 230 | Not in ServiceNow/CrowdStrike |
| wap | 160 | Not in ServiceNow |
| win_vm | 130 | |
| linux_vm | 115 | |
| switch | 80 | Not in ServiceNow |
| vdi | 43 | |
| printer | 43 | |
| linux_ws | 32 | |
| sdwan | 18 | Not in ServiceNow |
| clinic | 17 | Not in ServiceNow/CrowdStrike |
| router | 14 | Not in ServiceNow |
| win_server | 12 | |
| linux_server | 9 | |
| security | 7 | |
| esx_server | 6 | |
| **Total** | **2,291** | |

---

## Findings

### HIGH — Duplicate IP Address

**Issue:** IP `10.11.1.254` is assigned to 40 different San Francisco `win_laptop` assets.

- **Affected assets:** `lsys-sfo-lap-0770` through `lsys-sfo-lap-0887` (non-contiguous, 40 records)
- **Root cause:** `10.11.1.254` is a reserved/gateway address — these laptops were likely assigned the subnet default gateway IP instead of their actual DHCP lease. Every other laptop in the dataset has a unique IP.
- **Impact:** Any integration that correlates assets by IP address will collapse all 40 into one record or flag them as conflicts. Affects both ServiceNow and CrowdStrike mock APIs since the same xlsx is the source for both.
- **Fix:** Assign unique IPs to the 40 affected records. The rest of the SFO laptop fleet (IDs below 770 and above 887) have valid unique IPs in the `10.x.x.x` range.

---

### MEDIUM — `linux_ws` Hostnames Use Laptop Naming Convention

**Issue:** All 32 `linux_ws` (Linux workstation) records use the `lap` hostname segment, which is the same segment used by `win_laptop` and `mac_laptop`.

| Field | Value |
|---|---|
| Expected segment | `lws` or `lwks` |
| Actual segment | `lap` (all 32 records) |
| OS | Fedora 41 (19), Ubuntu 24.04 LTS (13) |
| Manufacturer | Lenovo (20), Dell (12) |
| Example hostname | `lsys-blr-lap-0856` |

These records are also seen by Meraki/Mist (wireless network) and Ordr alongside CrowdStrike, which suggests they are mobile/WiFi-connected devices — more consistent with laptops than desktop workstations. The dataset spec has no `linux_laptop` category, so these may have been assigned to `linux_ws` as the closest available category.

**Impact:** Low functional impact (all map to `cmdb_ci_computer` in ServiceNow), but misleading for anyone parsing hostnames to determine form factor.

---

### MEDIUM — Zebra Label Printers in Office Printer Category

**Issue:** 13 of 43 printers (30%) are Zebra ZT411 industrial label printers, mixed in with standard office printers.

| Manufacturer | Model | Count | Type |
|---|---|---|---|
| HP | LaserJet M507 | 16 | Office laser printer |
| Canon | imageRUNNER C3226 | 14 | Office color copier |
| **Zebra** | **ZT411** | **13** | **Industrial label/barcode printer** |

**Zebra ZT411 distribution by site:**

| Site | Count |
|---|---|
| San Francisco | 4 |
| London | 3 |
| Amsterdam | 2 |
| Bangalore | 2 |
| New York | 1 |
| Singapore | 1 |

The Zebra ZT411 is an industrial thermal label printer used for shipping labels, barcodes, and asset tags — not a document/office printer. Its presence is valid if Luminary Systems has shipping, warehouse, or logistics operations, but unexpected if the scenario is a pure office environment.

**Recommendation:** If the demo scenario is office-only, replace Zebra records with HP or Canon. If shipping/logistics operations are part of the story, document the Zebra printers as intentional.

---

### LOW — `win_vm` / `linux_vm` Use a 4-Part Hostname Scheme

**Issue:** All 245 VM records use a 4-segment hostname format (`lsys-{site}-prd-app-{n}`), while every other category uses 3 segments (`lsys-{site}-{type}-{n}`).

| Category | Count | Hostname Pattern | Example |
|---|---|---|---|
| win_vm | 130 | `lsys-{site}-prd-app-{n}` | `lsys-sfo-prd-app-1436` |
| linux_vm | 115 | `lsys-{site}-prd-app-{n}` | `lsys-lon-prd-app-1566` |

The `prd-app` segment means "production application server" — naming more typical of servers (`win_server`/`linux_server`) than VM instances. All 245 are seen by CrowdStrike, confirming they are active compute workloads. The category choice may be intentional to represent infrastructure-tier VMs vs. business application servers.

`esx_server` records follow the same 4-part convention (`lsys-{site}-prd-esx-{n}`), suggesting this pattern was deliberate for the production infrastructure tier.

**Impact:** Low — data is internally consistent. Worth noting if building hostname-parsing integrations.

---

### LOW — `router` Uses Non-Standard `rt` Abbreviation

**Issue:** All 14 router records use the hostname segment `rt` instead of the more common `rtr`.

| Detail | Value |
|---|---|
| Affected category | router |
| Hostname pattern | `lsys-{site}-rt-{n}` |
| Example | `lsys-sfo-rt-1921` |
| Manufacturer | Cisco (all 14) |
| OS | Cisco IOS XE (all 14) |

The abbreviation is consistent across all records, so it is intentional — but `rt` is atypical. Most network naming standards use `rtr` (router) to avoid confusion with `rt` (route table) in some contexts.

---

## Informational Notes

### Mobile Devices — Identical Hostname Prefix for iOS and Android

Both `mobile_ios` (275 records, Apple) and `mobile_droid` (230 records, Samsung) use the same `mob-{n}` hostname prefix. The hostname alone does not distinguish iOS from Android devices.

| Category | Manufacturer | MDM | Hostname Prefix |
|---|---|---|---|
| mobile_ios | Apple (100%) | JAMF | `mob-` |
| mobile_droid | Samsung (100%) | Intune | `mob-` |

Neither mobile category appears in ServiceNow or CrowdStrike — iOS is JAMF-managed, Android is Intune-managed. This is realistic behavior.

---

### Security Devices — Amsterdam and Singapore Have No Firewalls

Only 7 firewall records exist across 6 sites, with Bangalore (3), New York (2), London (1), and San Francisco (1) covered. Amsterdam and Singapore have zero security/firewall records.

| Site | Firewall Count | Vendor |
|---|---|---|
| Bangalore | 3 | Fortinet (all 3) |
| New York | 2 | Fortinet (both) |
| San Francisco | 1 | Palo Alto |
| London | 1 | Palo Alto |
| Amsterdam | 0 | — |
| Singapore | 0 | — |

Amsterdam and Singapore may share a regional hub with the nearest site, or this is a gap in the dataset.

---

### SD-WAN — Aruba-Only Integration Source

All 18 SD-WAN devices (Aruba EdgeConnect) are seen exclusively by the `aruba` integration source, which appears nowhere else in the dataset. These assets will not appear in ServiceNow, CrowdStrike, Tenable, Meraki, or any other mock API.

---

### Clinic / Medical Devices — Ordr-Only, Absent from All Other APIs

17 medical devices (GE Healthcare, Philips) running `Embedded Medical OS` are seen only by Ordr (IoT/OT security). They do not appear in ServiceNow, CrowdStrike, or any endpoint agent source. This is intentional and realistic — medical devices typically cannot run endpoint agents and are discovered only by passive OT security tools.

---

### IoT — Camera-Heavy Fleet

The 245 IoT devices break down entirely into surveillance/camera manufacturers:

| Manufacturer | Count |
|---|---|
| Hikvision | 83 |
| Cisco Meraki | 82 |
| Axis | 80 |

All run `Embedded IoT OS` and are discovered by Ordr, Tenable, and Meraki. No IoT devices appear in ServiceNow or CrowdStrike.

---

## Summary Table

| Severity | Finding | Assets Affected |
|---|---|---|
| High | Duplicate IP `10.11.1.254` on 40 SFO laptops | 40 |
| Medium | `linux_ws` all use `lap` hostname segment (laptop naming) | 32 |
| Medium | 13 Zebra ZT411 label printers mixed with office printers | 13 |
| Low | `win_vm`/`linux_vm` use 4-part hostname — inconsistent with other categories | 245 |
| Low | `router` uses `rt` abbreviation (non-standard, should be `rtr`) | 14 |
| Info | Amsterdam and Singapore have no firewall records | 0 |
| Info | `mobile_ios` and `mobile_droid` share identical `mob-` hostname prefix | 505 |
| Info | SD-WAN (`aruba`) is the only category with a unique-source integration | 18 |
| Info | Clinic/medical devices seen only by Ordr — absent from all other mock APIs | 17 |
| Info | IoT fleet is 100% surveillance cameras (Hikvision, Axis, Meraki) | 245 |
