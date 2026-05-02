# SOP: Hardware Stabilization
**Date:** May 2026
**Phase:** 1 - Hardware
**Node(s) Affected:** Infra-01, Compute-01, Util-01, Attack-01

---

## 1. Objective
[cite_start]Establish a physically optimized, safe, and "Server-Ready" hardware foundation for a multi-node cybersecurity lab by salvaging failing hardware, upgrading core components, and hardening BIOS settings[cite: 819].

## 2. Technical Inventory
| Node | Model | Specifications | Storage Strategy |
| :--- | :--- | :--- | :--- |
| **Infra-01** | Dell OptiPlex 3050 Tower | [cite_start]16GB DDR4 [cite: 821][cite_start], Intel i350-T4 Quad-NIC [cite: 821] | [cite_start]128GB SanDisk Z400s (OS) + 500GB HDD (Vault) [cite: 822] |
| **Compute-01** | Intel NUC5i5MYHE | [cite_start]16GB DDR3L [cite: 823][cite_start], Mini-DP to HDMI [cite: 823] | [cite_start]256GB Samsung 850 EVO (DRAM Cache enabled) [cite: 824] |
| **Util-01** | HP 15-ac143wm | [cite_start]8GB DDR4, Battery Removed [cite: 825] | [cite_start]256GB Samsung PM871 SSD [cite: 825] |
| **Attack-01** | ThinkPad T490 | [cite_start]16GB DDR4 (Soldered + Slot) [cite: 826] | [cite_start]512GB NVMe (Kali Linux) [cite: 826] |
| **Network** | Netgear GS108PEv3 | 8-Port PoE Managed Switch | [cite_start]Firmware v2.06.24 (2023 Release) [cite: 827] |

## 3. Procedure Steps

### 3.1 ThinkPad T490 RAM Salvage & Stress Test
1. [cite_start]**Identify Failure:** Performed a full scan with Memtest86+ to locate the exact address of the bad sector on the soldered motherboard RAM[cite: 828].
2. [cite_start]**Kernel Bypass:** Appended the `mem=15G` parameter to the Linux boot line to instruct the kernel to ignore the upper faulty memory range[cite: 829].
3. [cite_start]**Persistence:** Modified `/etc/default/grub` to include `GRUB_CMDLINE_LINUX_DEFAULT="quiet splash mem=15G"` followed by `sudo update-grub` to make the bypass permanent[cite: 830].
4. [cite_start]**Stress Testing:** Re-ran Memtest86+ and monitored system logs after the bypass to ensure zero crashes during heavy resource allocation[cite: 831].

### 3.2 Switch Reset & Firmware Update
1. [cite_start]**Physical Reset:** Performed a 15-second pinhold on the Factory Default button to clear legacy VLAN configurations[cite: 832].
2. [cite_start]**Firmware Upgrade:** Manually updated firmware from v2.00.12 to v2.06.24 to resolve known 802.1Q tagging bugs critical for Phase 2[cite: 833].

### 3.3 BIOS Hardening (All Server Nodes)
1. [cite_start]**Virtualization:** Enabled Intel VT-x and VT-d to support Proxmox hypervisors and PCIe passthrough for the quad-port NIC[cite: 834].
2. [cite_start]**Power Reliability:** Set AC Power Recovery to "Power On" to ensure the lab automatically restarts following Michigan power flickers[cite: 835].
3. [cite_start]**Efficiency:** Disabled Deep Sleep (S4/S5) to keep NICs responsive for management pings[cite: 836].

### 3.4 HP 15 Safety Surgery
1. [cite_start]**Battery Removal:** Physically extracted the dead battery to mitigate fire risks during 24/7 "always-on" operation[cite: 837].
2. [cite_start]**Flea Power Drain:** Disconnected AC, held the power button for 30 seconds to drain residual energy, enabling the motherboard to boot strictly from AC power[cite: 838].

## 4. Troubleshooting & Lessons Learned
> [!TIP]
> **Issue:** DRAM-less Stuttering on SanDisk Z400s.
> [cite_start]**Fix:** Allocated the DRAM-less SanDisk drive to the read-heavy Firewall role (OPNsense) and reserved the Samsung DRAM-cache SSDs for the Compute node to handle high-volume random writes from VMs[cite: 839].

> [!TIP]
> **Issue:** HP 15 not turning on after battery removal.
> [cite_start]**Fix:** Performed a 30-second "Flea Power" hold to clear the motherboard's safety state[cite: 841].

## 5. Verification (How to know it works)
- [ ] [cite_start]**T490 Stability:** `dmesg | grep -i mem` confirms the kernel is ignoring the mapped bad sectors[cite: 843].
- [ ] [cite_start]**NIC Validation:** Link lights appear on all 4 ports of the i350-T4 when connected to the Netgear switch[cite: 844].
- [ ] [cite_start]**Network Access:** Netgear Switch Web UI is accessible at `192.168.0.239` (default)[cite: 845].
- [ ] [cite_start]**Headless Boot:** HP 15 and NUC boot automatically when the AC adapter is plugged in[cite: 846].

## 6. Version History
* [cite_start]**v1.0:** Initial Setup - Hardware physically stabilized and BIOS hardened[cite: 847].
