# VirtualBox Ubuntu VMs - Host-Only Network Setup

Complete step-by-step guide to set up two Ubuntu VMs with static IPs on `vboxnet0`.

---

## Network Overview

- **Host-only Adapter**: `vboxnet0` (192.168.56.1)
- **VM1 IP**: 192.168.56.10
- **VM2 IP**: 192.168.56.20

---

## Step 1: Configure VirtualBox Network

1. Shut down both VMs.
2. For **each VM**, do the following:
   - Right click VM → **Settings** → **Network**
   - **Adapter 2** tab:
     - Enable Network Adapter → ✅
     - Attached to → **Host-only Adapter**
     - Name → **vboxnet0**
3. Click **OK**.
4. Start both VMs.

---

## Step 2: Check Interfaces Inside VMs

Run this command on both VMs:

```bash
ip a
You should see:

enp0s3 → NAT (internet)

enp0s8 → Host-only

If enp0s8 is down, bring it up:

bash
sudo ip link set enp0s8 up
ip a
Step 3: Assign Static IPs (Temporary)
On VM1:

bash
sudo ip addr add 192.168.56.10/24 dev enp0s8
ip a show enp0s8
On VM2:

bash
sudo ip link set enp0s8 up
sudo ip addr add 192.168.56.20/24 dev enp0s8
ip a show enp0s8
Test connectivity from each machine:

bash
# From VM1
ping 192.168.56.20
ping 192.168.56.1

# From VM2
ping 192.168.56.10
ping 192.168.56.1

# From Host
ping 192.168.56.10
ping 192.168.56.20
Step 4: Make Configuration Permanent (Netplan)
On VM1:

bash
sudo nano /etc/netplan/50-cloud-init.yaml
Replace the content with:

yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.10/24
Save & Exit (Ctrl+O → Enter → Ctrl+X)

Apply the config:

bash
sudo netplan apply
On VM2:

bash
sudo nano /etc/netplan/50-cloud-init.yaml
Replace the content with:

yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: false
      addresses:
        - 192.168.56.20/24
Apply:

bash
sudo netplan apply
Verify:

bash
ip a show enp0s8
Step 5: Enable SSH on Both VMs
bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo systemctl status ssh
Step 6: SSH Configuration on Host Machine
bash
nano ~/.ssh/config
Add the following:

text
Host ubuntu-vm1
    HostName 192.168.56.10
    User rifat
    Port 22

Host ubuntu-vm2
    HostName 192.168.56.20
    User ridwanur
    Port 22
Now you can connect using:

bash
ssh ubuntu-vm1
ssh ubuntu-vm2
Quick Reference Commands
bash
ip a                          # Show all interfaces
ip a show enp0s8             # Show host-only interface
sudo netplan apply           # Apply network config
sudo systemctl status ssh    # Check SSH status
Troubleshooting
enp0s8 not visible → Restart VM after adding Adapter 2 in VirtualBox.

Ping not working → Run sudo ufw allow from 192.168.56.0/24 on VMs.

Netplan issues → sudo netplan --debug apply

Setup Complete! 🎉
You now have a stable Host ↔ VM1 ↔ VM2 network.

text

Just paste everything above into a file named `README.md` and you're good to go!