# SOP: Hardware Stabilization
**Date:** April 2026
**Phase:** 1 - Hardware
**Node(s) Affected:** Infra-01, Compute-01, Util-01, Attack-01

---

## 1. Objective
Establish a physically optimized, safe, and "Server-Ready" hardware foundation for a multi-node cybersecurity lab by salvaging failing hardware, upgrading core components, and hardening BIOS settings.

## 2. Technical Inventory
| Node | Model | Specifications | Storage Strategy |
| :--- | :--- | :--- | :--- |
| **Infra-01** | Dell OptiPlex 3050 Tower | 16GB DDR4, Intel i350-T4 Quad-NIC | 128GB SanDisk Z400s (OS) + 500GB HDD (Vault) |
| **Compute-01** | Intel NUC5i5MYHE | 16GB DDR3L, Mini-DP to HDMI | 256GB Samsung 850 EVO (DRAM Cache enabled) |
| **Util-01** | HP 15-ac143wm | 8GB DDR4, Battery Removed | 256GB Samsung PM871 SSD |
| **Attack-01** | ThinkPad T490 | 16GB DDR4 (Soldered + Slot) | 512GB NVMe (Kali Linux) |
| **Network** | Netgear GS108PEv3 | 8-Port PoE Managed Switch | Firmware v2.06.24 (2023 Release) |

## 3. Procedure Steps

### 3.1 ThinkPad T490 RAM Salvage & Stress Test
1. **Identify Failure:** Performed a full scan with Memtest86+ to locate the exact address of the bad sector on the soldered motherboard RAM.
2. **Kernel Bypass:** Appended the `mem=15G` parameter to the Linux boot line to instruct the kernel to ignore the upper faulty memory range.
3. **Persistence:** Modified `/etc/default/grub` to include `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash mem=15G"` followed by `sudo update-grub` to make the bypass permanent.
4. **Stress Testing:** Re-ran Memtest86+ and monitored system logs after the bypass to ensure zero crashes during heavy resource allocation.

### 3.2 Switch Reset & Firmware Update
1. **Physical Reset:** Performed a 15-second pinhold on the Factory Default button to clear legacy VLAN configurations.
2. **Firmware Upgrade:** Manually updated firmware from v2.00.12 to v2.06.24 to resolve known 802.1Q tagging bugs critical for Phase 2.

### 3.3 BIOS Hardening (All Server Nodes)
1. **Virtualization:** Enabled Intel VT-x and VT-d to support Proxmox hypervisors and PCIe passthrough for the quad-port NIC.
2. **Power Reliability:** Set AC Power Recovery to "Power On" to ensure the lab automatically restarts following Michigan power flickers.
3. **Efficiency:** Disabled Deep Sleep (S4/S5) to keep NICs responsive for management pings.

### 3.4 HP 15 Safety Surgery
1. **Battery Removal:** Physically extracted the dead battery to mitigate fire risks during 24/7 "always-on" operation.
2. **Flea Power Drain:** Disconnected AC, held the power button for 30 seconds to drain residual energy, enabling the motherboard to boot strictly from AC power.

## 4. Troubleshooting & Lessons Learned
> [!TIP]
> **Issue:** DRAM-less Stuttering on SanDisk Z400s.
> **Fix:** Allocated the DRAM-less SanDisk drive to the read-heavy Firewall role (OPNsense) and reserved the Samsung DRAM-cache SSDs for the Compute node to handle high-volume random writes from VMs.

> [!TIP]
> **Issue:** HP 15 not turning on after battery removal.
> **Fix:** Performed a 30-second "Flea Power" hold to clear the motherboard's safety state.

## 5. Verification (How to know it works)
- [ ] **T490 Stability:** `dmesg | grep -i mem` confirms the kernel is ignoring the mapped bad sectors.
- [ ] **NIC Validation:** Link lights appear on all 4 ports of the i350-T4 when connected to the Netgear switch.
- [ ] **Network Access:** Netgear Switch Web UI is accessible at `192.168.0.239` (default).
- [ ] **Headless Boot:** HP 15 and NUC boot automatically when the AC adapter is plugged in.

## 6. Version History
* **v1.0:** Initial Setup - Hardware physically stabilized and BIOS hardened.
