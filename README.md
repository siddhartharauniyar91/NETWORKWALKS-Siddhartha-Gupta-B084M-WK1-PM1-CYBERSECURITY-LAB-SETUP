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

![Network Configuration](images/01-network-configuration.png)

---

### Step 2: Network Adapter Settings
Attached the Kali Linux virtual machine interface to the Shared Network using the `virtio-net-pci` network adapter.

![VM Adapter Settings](images/02-vm-adapter-settings.png)

---

### Step 3: Clipboard Sharing & Shared Directories
Enabled bidirectional clipboard sharing and configured a shared directory (`KALI-SHARED`) between the host system and the guest OS.

![VM Sharing Settings](images/03-vm-sharing-settings.png)

---

### Step 4: Kali Linux Static IP Assignment
Configured manual IPv4 settings inside Kali Linux:
* **IP Address:** `10.0.0.3`
* **Netmask:** `24` (`255.255.255.0`)
* **Default Gateway:** `10.0.0.2`
* **DNS Server:** `8.8.8.8`

![Kali Static IP Configuration](images/04-kali-static-ip-gui.png)

---

### Step 5: Verification & Connectivity Proof
Verified interface assignment and external internet access through HTTP/2 headers via `curl`:

![Terminal IP & Internet Proof](images/05-terminal-ip-and-ping-proof.png)

---

### Step 6: Baseline Backup Snapshot / Clone
Created a clean baseline clone (`Linux Clone`) before conducting security testing tasks to allow quick recovery.

![VM Clone / Snapshot](images/06-vm-snapshot.png)

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
