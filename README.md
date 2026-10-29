# Cybersecurity Lab Environment Setup (Week 1 - PM1)

Building an isolated virtual lab environment for penetration testing and ethical hacking practice.

---

## Project Overview

This project focuses on configuring an isolated virtual cybersecurity laboratory using UTM (QEMU) on macOS with Kali Linux as the attacking machine. The environment is configured on a custom private virtual network (`10.0.0.0/24`) with static addressing, host clipboard integration, shared folders, and verified external internet connectivity for future penetration testing and security assessments.

---

## Specifications & Environment

* **Host OS:** macOS (Apple Silicon ARM64)
* **Hypervisor:** UTM Virtual Machine (QEMU 10.0 ARM)
* **Guest Machine:** Kali Linux ARM64
* **Network Mode:** Shared Network (virtio-net-pci)
* **Subnet:** `10.0.0.0/24`
* **Assigned Static IP:** `10.0.0.3/24` (or `10.0.0.99/24`)
* **Gateway:** `10.0.0.2` (UTM Virtual Router)
* **DNS:** `8.8.8.8` / `10.0.0.1`

---

## Lab Architecture & Steps

### Step 1: Custom Virtual Network Configuration
Configured the custom virtual network with the `10.0.0.0/24` prefix and designated DHCP pool range.

![Network Configuration]
<img width="704" height="513" alt="02-vm-adapter-settings" src="https://github.com/user-attachments/assets/0c4c9720-58bb-4bd5-a35c-655407f9dd39" />
<img width="905" height="659" alt="01-network-configuration" src="https://github.com/user-attachments/assets/8f3bc4c9-b19d-45d6-8182-b654f3cc989b" />

---

### Step 2: Network Adapter Settings
Attached the Kali Linux virtual machine interface to the Shared Network using the `virtio-net-pci` network adapter.

![VM Adapter Settings](images/02-vm-adapter-settings.png)
![Uploading 02-vm-adapter-settings.png…]()


---

### Step 3: Clipboard Sharing & Shared Directories
Enabled bidirectional clipboard sharing and configured a shared directory (`KALI-SHARED`) between the host system and the guest OS.

![VM Sharing Settings](images/03-vm-sharing-settings.png)
<img width="905" height="659" alt="03-vm-sharing-settings" src="https://github.com/user-attachments/assets/00c1bb51-08f7-49d1-9dfc-993a78d7d068" />


---

### Step 4: Kali Linux Static IP Assignment
Configured manual IPv4 settings inside Kali Linux:
* **IP Address:** `10.0.0.3`
* **Netmask:** `24` (`255.255.255.0`)
* **Default Gateway:** `10.0.0.2`
* **DNS Server:** `8.8.8.8`

![Kali Static IP Configuration](images/04-kali-static-ip-gui.png)
<img width="1470" height="956" alt="04-kali-static-ip-gui" src="https://github.com/user-attachments/assets/ae964525-e2e8-4065-b5ea-f16304c557d5" />


---

### Step 5: Verification & Connectivity Proof
Verified interface assignment and external internet access through HTTP/2 headers via `curl`:

![Terminal IP & Internet Proof](images/05-terminal-ip-and-ping-proof.png)
<img width="1470" height="956" alt="05-terminal-ip-and-ping-proof" src="https://github.com/user-attachments/assets/573e7dd4-a5db-4602-b2e4-8c1d40556ea4" />


---

### Step 6: Baseline Backup Snapshot / Clone
Created a clean baseline clone (`Linux Clone`) before conducting security testing tasks to allow quick recovery.

![VM Clone / Snapshot](images/06-vm-snapshot.png)
<img width="896" height="657" alt="06-vm-snapshot" src="https://github.com/user-attachments/assets/1aed36db-f004-4383-a21f-89cc1a5db2e9" />


---

## Troubleshooting & Engineering Insights

### Issue 1: Gateway Routing Conflict (`Destination Host Unreachable`)
* **Symptom:** Setting the gateway to `10.0.0.1` resulted in `Destination Host Unreachable` errors during outbound connection attempts.
* **Root Cause:** In UTM/QEMU user-mode (SLIRP) networking on macOS, the hypervisor's internal virtual router gateway defaults to `10.0.0.2`. Configuring Kali to `10.0.0.2` created an immediate address collision with the gateway.
* **Solution:** Identified UTM's active routing table gateway (`10.0.0.2`) and reassigned Kali's static IP to a dedicated host address (`10.0.0.3`), setting `10.0.0.2` as the default gateway.

### Issue 2: Dual IP / Conflicting DHCP Leases
* **Symptom:** `ip route show` displayed conflicting default routes and duplicate source IPs (`10.0.0.3` and `10.0.0.16`) on interface `eth0`.
* **Solution:** Flushed stale DHCP leases and forced NetworkManager to enforce static configuration:
  ```bash
  sudo nmcli connection modify "Wired connection 1" ipv4.method manual
  sudo nmcli connection modify "Wired connection 1" ipv4.addresses 10.0.0.3/24
  sudo nmcli connection modify "Wired connection 1" ipv4.gateway 10.0.0.2
  sudo nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8"
  sudo ip addr flush dev eth0
  sudo nmcli connection up "Wired connection 1"
