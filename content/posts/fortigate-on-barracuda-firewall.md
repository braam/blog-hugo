---
title: "Fortigate on Barracuda Firewall"
date: 2025-11-19T13:05:31+01:00
draft: false
tags:
- Fortigate
---

# Running FortiGate on a Barracuda F80 via QEMU/KVM

I had an old Barracuda Firewall F80b lying around in the office, and I decided to repurpose it by running a FortiGate VM. Barracuda appliances are essentially small Linux-based PCs, which makes them perfect candidates for lightweight virtualization. Since FortiGate cannot be installed natively on this hardware, virtualization via QEMU/KVM is the only viable approach.

&nbsp;
## Choosing the Host OS

FortiGate provides **KVM images** for FortiOS, available on the [Fortinet Support site](https://support.fortinet.com/) (account required). For the host OS, I chose **Alpine Linux Standard** — a minimal, lightweight Linux distribution ideal for this type of project.

&nbsp;
### BIOS Setup

Before installing Alpine, the BIOS on the Barracuda appliance needs to be prepared:

- BIOS password: bcndk1
- Enable **VT-d / Intel Virtualization Technology**
- Enable **UEFI boot**
- Optional: Enable legacy USB support if installing via USB
- Connect with puty to **serial console** (default baud rate: 19200)

![Barracuda_BIOS_01](/posts_images/fortigate-on-barracuda_01.png)
![Barracuda_BIOS_02](/posts_images/fortigate-on-barracuda_02.png)

&nbsp;
## Installing Alpine Linux

1. Use rufus to flash iso to USB drive.
2. Boot into the Alpine live environment and connect with putty to **serial console** (default baud rate: 115200).
3. Log in as `root` (no password required).
4. Run the guided setup:

```bash
setup-alpine
```

5. Complete disk partitioning, networking, timezone, and SSH setup.
6. Reboot into the installed Alpine system (connect to **serial console** with baud rate: 9600).

After the first boot, enable the community repository and install the required virtualization packages:

```bash
sed -i '/community/ s/^#//' /etc/apk/repositories
apk update
apk add qemu qemu-system-x86_64 libvirt virt-install bridge-utils
```

&nbsp;
## Configuring Network Bridges
Barracuda appliances have out of order ethX interfaces detected by Linux, so to align them with the physical location on the box us my script located [here](https://github.com/braam/barracuda-iface-mapper) ethX will then physically matched with pY.

To expose the VM to the network, create Linux bridges corresponding to the physical interfaces by editing /etc/network/interfaces:

```bash
# /etc/network/interfaces
auto lo
iface lo inet loopback

auto p1
iface p1 inet manual

auto br1
iface br1 inet dhcp
   	bridge-ports p1
   	bridge-stp off
	bridge-fd 0

auto p2
iface p2 inet manual

auto br2
iface br2 inet manual
    bridge-ports p2
    bridge-stp off
    bridge-fd 0
```

Add additional bridges for extra FortiGate ports if needed. After configuring, restart networking to activate the bridges (or reboot).
**Note** that you should use the interface names using the Barracuda firewall modeltype mapping file.

&nbsp;
## Creating the FortiGate VM

Download the Fortigate KVM from the support website and place it in /root/fortios.qcow2

Use `virt-install` to create the FortiGate KVM VM. **Important:** Use the **E1000 NIC driver**, otherwise FortiOS will fail to boot.

```bash
virt-install \
  --name fortios \
  --memory 2048 \
  --vcpus 1 \
  --disk path=/root/fortios.qcow2,format=qcow2,bus=ide \
  --import \
  --network bridge=br0,model=e1000 \
  --network bridge=br1,model=e1000 \
  --console pty,target_type=serial \
  --noautoconsole
```

Connect to the VM console:

```bash
virsh console fortios
```

You will see the **serial number**, which is required to request a **Permanent Trial License** (conditions: 1 vCPU, 2GB RAM).

&nbsp;
## Useful `virsh` Commands

For managing the VM, these commands are essential:

```bash
# List all VMs
virsh list --all

# Connect to the console
virsh console fortios

# Enable autostart
virsh autostart fortios

# Start / Stop the VM
virsh start fortios
virsh shutdown fortios

# Force stop VM
virsh destroy fortios

# Remove VM
virsh undefine fortios

# Check VM configuration
virsh dumpxml fortios
```

&nbsp;
## Finding the FortiGate IP Address

By default, `port1` uses DHCP. After booting, connect to the console and login with admin user:

```bash
virsh console fortios
```

Run the following command:

```bash
get system interface physical
```

Sample output:

```
== [onboard]
    ==[port1]
        mode: dhcp
        ip: 192.168.0.193 255.255.255.0
        ipv6: ::/0
        status: up
        speed: 1000Mbps (Duplex: full)
```

This IP can be used to log into the FortiGate web interface for further configuration.

