# 🔐 Virtual Cybersecurity Lab Setup

*Building an isolated virtual lab with VirtualBox and Kali Linux for safe, repeatable security practice*

**Week:** 01 | **Program:** Cybersecurity Internship 2026 (Networkwalks)

---

## 📌 Project Overview

This project sets up a virtual cybersecurity lab using **VirtualBox** and **Kali Linux**. The lab runs on a private virtual network, so security tools can be practiced in a controlled environment without touching real systems. More machines can be added to the same network later as targets for authorized testing.

## 🎯 Objectives

- [x] Install and configure VirtualBox
- [x] Import and configure Kali Linux as a virtual machine
- [x] Create a private NAT Network for the lab
- [x] Assign a static IP address to the Kali VM
- [x] Verify network settings, connectivity, and DNS resolution
- [x] Create a clean VM snapshot for recovery
- [x] Document the full process

## 🛡️ Purpose

The lab provides an isolated environment for learning and practicing activities such as network reconnaissance, port scanning, and vulnerability assessment. It is the foundation for the projects in later weeks.

> ⚠️ **Important:** Use this lab only on systems you own or have explicit permission to test. Never use these tools against unauthorized systems.

## 🏗️ Architecture

![Lab architecture](./images/architecture.svg)

Kali Linux runs as a VM inside VirtualBox on a Windows host. It is attached to a private NAT Network (`10.0.0.0/24`), which gives it outbound access to the internet through the NAT gateway. Future target VMs will join the same network.

## ⚙️ Configuration

| Component | Configuration |
|-----------|---------------|
| Host OS | Windows 11 Home |
| Host RAM | 8 GB (5.84 GB usable) |
| Processor | AMD Ryzen 5 7530U |
| Hypervisor | VirtualBox 7.2.10 |
| Guest OS | Kali Linux 2026.3 (Rolling), amd64 |
| Guest RAM | 2048 MB |
| Guest CPUs | 2 |
| Guest disk | 80.09 GB (VDI, SATA) |
| Network adapter | Adapter 1: Intel PRO/1000 MT Desktop |
| Virtual network | NAT Network (`NatNetwork`) |
| Network range | 10.0.0.0/24 (DHCP enabled, IPv6 disabled) |
| Kali IP address | 10.0.0.2/24 (static) |
| Default gateway | 10.0.0.1 |
| DNS server | 8.8.8.8 |
| Shared folders | None |

## 🪜 Procedure

### Step 1. Install VirtualBox
I installed VirtualBox 7.2.10 on a Windows 11 host and confirmed the version under Help > About VirtualBox.

![VirtualBox version](./images/step-01-virtualbox-version.png)

### Step 2. Import and Configure Kali Linux
I downloaded the prebuilt Kali Linux VirtualBox image (2026.3) from the official Kali website and imported it into VirtualBox. The VM uses 2048 MB of RAM and 2 CPUs, and its network adapter is attached to the NAT Network created in Step 3.

![Kali VM details in VirtualBox](./images/step-02-kali-vm-details.png)

After booting, the Kali desktop loaded correctly:

![Kali Linux desktop](./images/step-02-kali-desktop.png)

### Step 3. Create the NAT Network
In VirtualBox, I created a NAT Network named `NatNetwork` with the prefix `10.0.0.0/24`, DHCP enabled, and IPv6 disabled.

I chose a NAT Network over plain NAT because VMs on the same NAT Network can communicate with each other while still reaching the internet. Plain NAT keeps each VM isolated from the others, which would not work for a lab with attacker and target machines.

![NAT Network settings](./images/step-03-nat-network.png)

### Step 4. Configure and Verify Network Settings
In Kali's network connection editor, I set the IPv4 method to **Manual** and entered a static address so the VM is easy to reference in later exercises:

| Setting | Value |
|---------|-------|
| Address | 10.0.0.2 |
| Netmask | 24 |
| Gateway | 10.0.0.1 |
| DNS server | 8.8.8.8 |

![Kali static IP settings](./images/step-04-kali-static-ip.png)

Then I ran the tests in the [Verification](#-verification) section below.

### Step 5. Create a Clean VM Snapshot
I created a snapshot to serve as a recovery point. If a future exercise breaks the VM, I can restore it to this baseline.

| Item | Value |
|------|-------|
| Snapshot name | clean setup |
| Taken | 11/7/2026, 2:54 PM |
| VM state | Powered off |

Before taking it, I confirmed with `ip a` that `eth0` had the static address `10.0.0.2/24`:

![ip a before the snapshot](./images/step-05-ip-before-snapshot.png)

![VirtualBox snapshot](./images/step-05-snapshot.png)

**Restore test.** To prove the snapshot works, I created a test file after taking it, then restored the snapshot:

1. Before restoring, I created `~/restore-test.txt` and confirmed it appeared in `ls ~`.

![Test file created](./images/step-05-restore-before.png)

2. After restoring the snapshot, the file was gone and `eth0` still had the static address `10.0.0.2/24`.

![After restoring the snapshot](./images/step-05-restore-after.png)

## 🔎 Verification

| Test | Command | Expected Result | Actual Result |
|------|---------|-----------------|---------------|
| Check IP address | `ip a` | Shows `10.0.0.2/24` | ✅ `eth0` shows `10.0.0.2/24` (static, no "dynamic" label) |
| Test gateway | `ping -c 4 10.0.0.1` | Replies received | ✅ 4/4 replies, 0% packet loss, avg 3.49 ms |
| Test internet connectivity | `ping -c 4 8.8.8.8` | Replies received | ⚠️ Intermittent loss. A 10-packet run got 8 of 10 replies (20% loss, avg 96.3 ms); an earlier 4-packet run got 2 of 4. See the Observation under Problems Encountered |
| Test DNS resolution | `nslookup kali.org` | Domain resolves to an IP | ✅ Resolved via server 8.8.8.8 (the static DNS setting) to 104.18.5.159 and 104.18.4.159, plus IPv6 addresses |
| Verify Nmap | `nmap --version` | Version information displayed | ✅ Nmap 7.99 |
| Verify snapshot | Restore the snapshot, then run `ip a` | Baseline configuration restored | ✅ A file created after the snapshot disappeared after restoring; `ip a` still showed `10.0.0.2/24` |

![Verification output](./images/step-04-verification.png)

> Pinging `8.8.8.8` tests connectivity on its own, and `nslookup` tests DNS on its own. If the first works and the second fails, the problem is DNS, not the network.

## 🐞 Problems Encountered

### Problem 1. Two network adapters, wrong address
- **Symptom:** `ip a` showed two interfaces: `eth0` with no IPv4 address and `eth1` with `10.0.0.4/24` (dynamic), instead of the planned static `10.0.0.2`.
- **Cause:** I had enabled a second network adapter on the VM, so the static profile and the DHCP lease ended up on different interfaces.
- **Solution:** I disabled Adapter 2, leaving one adapter attached to the NAT Network.

### Problem 2. Static IP saved but not active
- **Symptom:** With one adapter, `eth0` still showed a dynamic address (`10.0.0.3/24`) and a DNS server handed out by DHCP, even though "Wired connection 1" was set to manual.
- **Cause:** The manual settings were saved in the profile, but the connection had not been reactivated, so the old DHCP configuration was still in use.
- **Solution:** I confirmed the saved settings with `nmcli`, reactivated the connection, and re-ran the checks. `eth0` then showed `10.0.0.2/24` and the DNS server was `8.8.8.8`.

### Observation: intermittent packet loss to 8.8.8.8
Pings to the gateway never dropped a packet, but pings to `8.8.8.8` lost between 0% and 50% of packets across several runs (the 10-packet run below lost 2, so 20%, with an average of about 96 ms). DNS lookups worked throughout. The loss is probably happening beyond the lab, for example in how VirtualBox's NAT handles ICMP or in my host's connection, but I did not investigate it further.

![10-packet ping to 8.8.8.8](./images/step-04-ping-10-packets.png)

## 💡 What I Learned

1. **NAT vs NAT Network:** Plain NAT isolates each VM, while a NAT Network puts VMs on one shared private network. They can talk to each other and still reach the internet, which is what a multi-machine lab needs.
2. **Static addressing:** I set the IP, netmask, gateway, and DNS manually in Kali's network settings, which keeps the lab's addresses predictable and easier to document.
3. **Testing in layers:** Checking the IP, then the gateway, then the internet by IP, then DNS by name makes it clear where a failure is.
4. **Snapshots:** A snapshot gives me a known-good state to return to before running risky tools.
5. **Documentation:** Writing this up took more effort than the setup itself. A clear write-up needs screenshots, exact settings, and verified results.

## 🔐 Security & Ethical Use

This work is for **educational purposes only** and was done in an isolated environment that I control.

## 🔗 Tools & Resources

- [VirtualBox](https://virtualbox.org/wiki/Downloads): hypervisor
- [Kali Linux](https://kali.org/get-kali): security-focused operating system

## 👤 Author

**Kochiwe Anthony Tembo**
LinkedIn: [Kochiwe Tembo](https://www.linkedin.com/in/kochiwe-tembo-0a52a4400)
