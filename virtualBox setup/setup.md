# VirtualBox Ubuntu VMs Host-Only Network Setup

This manual explains how to configure two Ubuntu virtual machines in VirtualBox so they can communicate over a host-only network using static IP addresses.

## Purpose

Use this setup when you want:

- A private network between your host and two Ubuntu VMs
- Stable, fixed IP addresses for SSH and testing
- Internet access through the default NAT adapter

## Network Plan

| Device | Interface | IP Address | Role |
| --- | --- | --- | --- |
| VirtualBox host-only adapter | `vboxnet0` | `192.168.56.1` | Host gateway |
| VM 1 | `enp0s8` | `192.168.56.10/24` | Ubuntu VM 1 |
| VM 2 | `enp0s8` | `192.168.56.20/24` | Ubuntu VM 2 |

## Prerequisites

Before you begin, make sure you have:

- Oracle VirtualBox installed
- Two Ubuntu virtual machines already created
- Access to the Ubuntu terminal on both VMs
- A user account on each VM

## Step 1: Configure VirtualBox

1. Shut down both virtual machines.
2. Open the settings for VM 1.
3. Go to Network and open Adapter 2.
4. Enable the network adapter.
5. Set Attached to to Host-only Adapter.
6. Set Name to `vboxnet0`.
7. Repeat the same configuration for VM 2.
8. Start both virtual machines.

## Step 2: Confirm the Network Interfaces

Open a terminal on each VM and run:

```bash
ip a
```

You should see:

- `enp0s3` for NAT internet access
- `enp0s8` for the host-only adapter

If `enp0s8` is down, bring it up with:

```bash
sudo ip link set enp0s8 up
```

## Step 3: Assign Temporary Static IP Addresses

These commands apply the IP addresses until the next reboot.

### On VM 1

```bash
sudo ip addr add 192.168.56.10/24 dev enp0s8
ip a show enp0s8
```

### On VM 2

```bash
sudo ip link set enp0s8 up
sudo ip addr add 192.168.56.20/24 dev enp0s8
ip a show enp0s8
```

## Step 4: Test Connectivity

Run the following tests to confirm the network is working.

### From VM 1

```bash
ping 192.168.56.20
ping 192.168.56.1
```

### From VM 2

```bash
ping 192.168.56.10
ping 192.168.56.1
```

### From the host machine

```bash
ping 192.168.56.10
ping 192.168.56.20
```

## Step 5: Make the Configuration Permanent

Use Netplan on each VM so the static IP survives reboots.

### VM 1 Netplan configuration

Open the Netplan file:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Replace the content with:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.10/24
```

Save the file and apply the configuration:

```bash
sudo netplan apply
```

### VM 2 Netplan configuration

Open the Netplan file:

```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Replace the content with:

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.20/24
```

Apply the configuration:

```bash
sudo netplan apply
```

Verify the interface:

```bash
ip a show enp0s8
```

## Step 6: Install and Enable SSH

Run these commands on both VMs:

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

## Step 7: Configure SSH on the Host

On the host machine, open your SSH config file:

```bash
nano ~/.ssh/config
```

Add these entries:

```text
Host ubuntu-vm1
    HostName 192.168.56.10
    User rifat
    Port 22

Host ubuntu-vm2
    HostName 192.168.56.20
    User ridwanur
    Port 22
```

You can now connect with:

```bash
ssh ubuntu-vm1
ssh ubuntu-vm2
```

## Quick Reference

| Command | Description |
| --- | --- |
| `ip a` | Show all network interfaces |
| `ip a show enp0s8` | Show the host-only interface |
| `sudo netplan apply` | Apply network configuration |
| `sudo systemctl status ssh` | Check SSH service status |

## Troubleshooting

- If `enp0s8` does not appear, shut down the VM and confirm Adapter 2 is set to Host-only Adapter in VirtualBox.
- If ping fails, allow traffic on the VMs with `sudo ufw allow from 192.168.56.0/24`.
- If Netplan fails, run `sudo netplan --debug apply` to see detailed errors.

## Result

After completing these steps, your host and both Ubuntu VMs should communicate over a stable host-only network with SSH access enabled.