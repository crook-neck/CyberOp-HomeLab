# SOP: Perimeter & Connectivity (Phase 2 - Initial Staging)
**Date:** May 29, 2026
**Phase:** [2]
**Node(s) Affected:** Infra-01 (OPNsense), Netgear GS108PE, Attack-01 (ThinkPad T490)

---

## 1. Objective
Establish stable Layer 3 communication with the managed switch and configure the out-of-band management framework using a standalone staging environment.

## 2. Technical Inventory
| Component | Role | Configuration |
| :--- | :--- | :--- |
| OptiPlex 3050 | Firewall | OPNsense 24.x (Management Gateway) |
| Netgear GS108PE | Core Switch | Fallback IP: 192.168.0.239 |
| ThinkPad T490 | Attacker Station | Kali Linux (NIC: eth0 / enp0s31f6) |

## 3. Procedure Steps

### A. Terminal Interface Preparation
1. Flushed the physical interface on the Kali Linux node to remove stale ARP entries and IP mappings.
2. Manually assigned a staging IP (`192.168.0.100/24`) to match the factory default Netgear subnet.
3. Forced the physical link state to "Up" to initiate the hardware handshake.

### B. Hardware "Cold Swap" Recovery
1. Performed a physical power drain on the Netgear GS108PE (10-second disconnection).
2. Utilized the **45-Second Fallback Window**: Observed switch firmware behavior where the web server initializes on the fallback IP (`192.168.0.239`) only after failing to find a DHCP server on the wire.
3. Verified connectivity via ICMP ping before attempting GUI login.

### C. Management Plane Migration
1. Accessed the ProSAFE Web Interface via an Incognito browser session (bypassing cache issues).
2. Configured the Static Management IP:
   - **IP:** `10.0.10.2`
   - **Subnet:** `255.255.255.0`
   - **Gateway:** `10.0.10.1` (Target OPNsense Management Gateway).

### D. Physical Parallel Lab Cabling (Final Blueprint)
1. Wired the Out-of-Band (OOB) loop: Dell `re0` (Realtek) to Netgear Port 2.
2. Wired the Trunk line: Dell `igb2` (Intel) to Netgear Port 1.
3. Connected the Attacker Station (ThinkPad) to Netgear Port 3.

## 4. Troubleshooting & Lessons Learned

> [!TIP]
> **Issue:** NetworkManager stripping manual IP assignments.
> **Fix:** Used `ip addr flush` followed by `nmcli device connect` to force the daemon to respect the new physical topology once the firewall was introduced.

> [!TIP]
> **Issue:** Switch GUI unreachable immediately after IP change.
> **Fix:** Understood that Netgear firmware often requires a hard reboot to restart the HTTP service on a newly assigned static IP address.

## 5. Verification Checklist
- [x] **Fallback Recovery:** Successfully pinged `192.168.0.239` after cold boot.
- [x] **Subnet Migration:** Switch successfully accepted the `10.0.10.2` identity.
- [x] **OOB Handshake:** ThinkPad received a dynamic lease (`10.0.10.200`) from OPNsense across the new management loop.

## 6. Version History
* v1.0: Hardware Staging - Successfully established the Out-of-Band management framework
