# UAI Discovery Scope — Per Vendor Reference

Infoblox Universal Asset Insights syncs only the asset types defined in each vendor's
**Discovery Scope** configuration. This doc records the scope for each vendor, what our
mock generates, and any known gaps or issues.

Last updated: 2026-07-05

---

## Vendors with a Defined Discovery Scope

### CrowdStrike Falcon

| UAI Scope Group | Asset Type | Mock generates? |
|---|---|---|
| Global | Alerts | ✅ 103 alert records |
| Global | Devices | ✅ 999 devices (win_laptop, mac_laptop, vdi, win_vm, linux_vm, linux_ws, win_server, linux_server) |

**Status: Clean.** Scope is simple — two types, both served.

---

### ServiceNow

| UAI Scope Group | Asset Type | SN Class | Mock generates? |
|---|---|---|---|
| Devices | Computer | `cmdb_ci_computer` | ✅ 728 records (win_laptop, mac_laptop, linux_ws) |
| Devices | Server | `cmdb_ci_server` | ✅ 0 (subclasses used instead) |
| Devices | Linux Servers | `cmdb_ci_linux_server` | ✅ 8 records |
| Devices | Windows Servers | `cmdb_ci_win_server` | ✅ 11 records |
| Devices | ESX Servers | `cmdb_ci_esx_server` | ✅ 6 records |
| Devices | Virtual Machine Instance | `cmdb_ci_vm_instance` | ✅ 261 records (vdi, win_vm, linux_vm) |
| Devices | Printers | `cmdb_ci_printer` | ✅ 40 records |
| Devices | Security Devices | `cmdb_ci_security` | ✅ 6 records |
| Network | Network Gear | `cmdb_ci_netgear` | ⚠️ 6 records — but `sys_class_name` overridden to `cmdb_ci_ip_switch` (see notes) |
| Network | IP Routers | `cmdb_ci_ip_router` | ✅ 6 records |
| Network | IP Switches | `cmdb_ci_ip_switch` | ✅ 12 records |
| Network | IP Firewalls | `cmdb_ci_ip_firewall` | ✅ 6 records |
| Network | WAP Networks | `cmdb_ci_wap_network` | ✅ 18 records |
| Network | Load Balancer Services | `cmdb_ci_lb_service` | ⚠️ 6 records — but `sys_class_name` overridden to `cmdb_ci_ip_switch` (see notes) |
| Services | Service (Business/IT) | `cmdb_ci_service` | ✅ 6 records |
| Services | Storage Devices | `cmdb_ci_storage_device` | ✅ 6 records |
| Services | Databases | `cmdb_ci_database` | ✅ 12 records |

**Known issues:**

1. ~~**`sdwan` category silently dropped**~~ — **Fixed 2026-07-04.** `sdwan` now maps to
   `cmdb_ci_ip_router` in `CAT_TO_TABLE`. The 18 Aruba EdgeConnect appliances appear in
   `08_ip_routers.json` with `sys_class_name = cmdb_ci_ip_router`, Aruba manufacturer, and
   ArubaOS 10 firmware. Total SN assets increased from 1,138 → 1,156.

2. ~~**Network Gear and Load Balancer `sys_class_name` override**~~ — **Fixed 2026-07-04.**
   The `cmdb_ci_ip_switch` override block has been removed. Both `cmdb_ci_netgear` (Network Gear)
   and `cmdb_ci_lb_service` (Load Balancer Services) now carry their correct `sys_class_name`
   and appear as distinct asset types in UAI. IP Switch count corrected from 24 → 12.

---

### Juniper Mist

| UAI Scope Group | Asset Type | Mock generates? |
|---|---|---|
| Organization | Networks | ✅ `networks.json` + `derived_networks.json` |
| Organization | Sites | ✅ `sites.json` (3 sites: BLR, AMS, SGP) |
| Sites | Devices | ✅ `devices.json` — 64 APs (AP34/AP45) + 27 switches (EX4400-48) |
| Sites | Wireless Clients | ✅ `wireless_clients.json` — 583 clients (win, mac, ios, android, iot) |
| Sites | Wired Clients | ✅ `wired_clients.json` — 41 clients (linux_ws, printer, clinic, router) |
| Sites | Maps | ✅ 15 floor maps generated — 5 floors × 3 sites, named "City - Floor" (e.g. "Bangalore - 4th Floor"); served from `/api/v1/sites/{site_id}/maps` |
| Sites | Port Stats | ❌ Not generated — mock has no per-port statistics |
| Sites | Networks | ✅ Same as `derived_networks.json` (per-site networks) |

**Status:** Port Stats are in UAI scope but not generated — UAI gets empty results,
not harmful. Maps are now implemented: 15 floor plans (1st–5th Floor per site),
named with city prefix so UAI shows "Bangalore - 4th Floor", "Amsterdam - 2nd Floor",
etc. Devices carry a `map_id` derived from their floor number in the hostname pattern
`-f{N}-`.

---

### Cisco Meraki

| UAI Scope Group | Asset Type | Mock generates? |
|---|---|---|
| Organization | Devices | ✅ `devices.json` — 155 devices (MR APs, MS switches, MX routers) |
| Organization | Devices Availability | ✅ `device_availabilities.json` — 155 records |
| Organization | Networks | ✅ `networks.json` — 3 networks |
| Networks | Appliance VLANs | ✅ `vlans.json` — 21 VLANs |
| Networks | Site-to-Site VPNs | ❌ Not generated |
| Networks | Cellular Gateway Subnets | ❌ Not generated |
| Networks | Clients | ✅ `clients.json` — 969 clients |
| Networks | VLAN Profiles | ❌ Not generated |
| Devices | Subnets | ❌ Not generated |
| Devices | Clients | ✅ `device_clients.json` — 969 records |

**Status:** Site-to-Site VPNs, Cellular Gateway Subnets, VLAN Profiles, and per-device
Subnets are in UAI scope but not generated. UAI gets empty results for those.
Also generated (not a named scope item): `device_statuses.json` (health/uptime),
`org.json` (organization profile — required by the connector before polling other endpoints).

---

## Vendors WITHOUT a Defined Discovery Scope

For these vendors, UAI syncs everything the connector returns — no scope filter applied.

### Microsoft Intune

No discovery scope configured. UAI syncs all managed devices.

**Mock generates:** 763 managed devices
- `win_laptop` (506) → Windows 11 Pro/Enterprise — corp-enrolled laptops
- `mobile_droid` (217) → Android 15 — BYOD enrolled
- `vdi` (40) → Windows 11 Enterprise — Azure Virtual Desktop / Windows 365

All three types are valid Intune MDM enrollments. VDI via Windows 365 is realistic.

---

### JAMF Pro

No discovery scope configured. UAI syncs all managed Apple devices.

**Mock generates:** 562 managed devices
- `mac_laptop` (301) → macOS 13–15 — corp-managed Macs
- `mobile_ios` (261) → iOS 18 — corp-enrolled iPhones + iPads

JAMF manages Apple-only. No Windows or Android — correct.

---

### Ordr

No discovery scope configured. UAI syncs all profiled devices.

**Mock generates:** 829 devices (passive network profiling — sees all network-connected devices)
- `iot` (227) — IoT sensors, cameras, embedded devices
- `win_laptop` (222) — Windows endpoints (profiled from network traffic)
- `mac_laptop` (129) — macOS endpoints
- `mobile_ios` (91) — iPhones / iPads
- `mobile_droid` (85) — Android phones
- `printer` (40) — network printers
- `linux_ws` (19) — Linux workstations
- `clinic` (16) — medical/clinic equipment (Ordr's primary IoT/OT use case)

Ordr is a passive network monitor — it fingerprints every device it sees on the network.
All types above are realistic. Ordr's core value is the IoT/clinic tier; Windows/Mac
appear because it sees all traffic.

---

### Aruba EdgeConnect (SD-WAN)

No discovery scope screenshot available. UAI syncs all appliances the orchestrator exposes.

**Mock generates:** 18 SD-WAN appliances
- Models: EC-S (9) and EC-SD-B (9)
- Roles: hub (12), spoke (6)
- Sites: SFO (6), AMS (6), BLR (2), NYC (2), LON (1), SGP (1)

---

## Cross-Vendor Consistency Notes

- The same 18 SD-WAN appliances are `seen_by = aruba;servicenow` in the master sheet —
  they appear in the Aruba mock but are silently dropped in the ServiceNow generator
  (see ServiceNow issue #1 above).
- Windows devices seen by both **Mist** and **CrowdStrike/Intune/ServiceNow** have been
  verified to show consistent OS: "Windows 11" in Mist (via OS override), "Windows 11 Pro"
  in CS/SN/Intune (from master sheet). The 19 remaining Windows 10 devices are in SF/NYC/LON
  only — sites not covered by Mist — so there is no cross-vendor OS inconsistency.
