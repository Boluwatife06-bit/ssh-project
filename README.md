# SOC ANalysis-project
ssh projeck and tools used basicaly a SOC analysis home lab/server
🔧 1. Creating the Ubuntu Server in VirtualBox
Download ISO

I started by downloading the latest Ubuntu Server ISO from the official Ubuntu website.

Create a New VM

In VirtualBox:

Name: Ubuntu-Server-Lab

Type: Linux

Version: Ubuntu (64-bit)

RAM: 2GB (minimum)

CPU: 2 cores

Storage: 20–40GB

Network: leave default for now

Attach the ISO

Before booting:

Settings → Storage → Empty CD

Choose the Ubuntu Server ISO

Install Ubuntu Server

Inside the installer I chose:

OpenSSH server (checked)

Standard system utilities

My username + password

No GUI (server only)

🌐 2. Configuring the Network (Host‑Only Adapter)

To make the server accessible only inside my lab:

Network Setup in VirtualBox

I enabled two adapters:

Adapter 1

NAT (for updates and package installs)

Adapter 2

Host‑only Adapter

Interface: vboxnet0

This lets me reach the server from my Kali VM, but isolates it from the internet.

Check interfaces inside Ubuntu
ip a


I use the enp0s8 interface (host‑only) for SSH access.

🔐 3. Setting a Static IP for the Server

I edited Netplan:

sudo nano /etc/netplan/00-installer-config.yaml


Example configuration:

network:
  version: 2
  ethernets:
    enp0s8:
      dhcp4: no
      addresses: [192.168.56.10/24]
      gateway4: 192.168.56.1
      nameservers:
        addresses: [8.8.8.8]


Apply settings:

sudo netplan apply


Now my server always has:

192.168.56.10

🖥️ 4. SSH Login From Kali Linux

On my Kali machine (also running in VirtualBox host‑only mode), I connect using:

ssh myuser@192.168.56.10


If it’s the first time:

yes
<enter password>


If SSH isn't installed, I install it inside Ubuntu:

sudo apt install openssh-server
sudo systemctl enable --now ssh

🔥 5. Securing the Server With UFW (Firewall)

I always install & configure UFW immediately:

Install
sudo apt install ufw

Allow SSH
sudo ufw allow 22/tcp

Enable firewall
sudo ufw enable

Check status
sudo ufw status verbose


At this point, only SSH is open — everything else is blocked and isolated.

🕵️‍♂️ 6. Deploying a Honeypot for Brute‑Force Detection

I use a lightweight honeypot to monitor SSH brute-force attempts.

Install Cowrie (SSH Honeypot)
sudo apt install git python3 python3-venv python3-dev libssl-dev libffi-dev build-essential
git clone https://github.com/cowrie/cowrie.git
cd cowrie
bin/cowrie start


Cowrie automatically creates:

Fake SSH environment

Logs of attackers

Session recordings

Credential capture

Log location
~/cowrie/var/log/cowrie

Optional: Forward logs to my SIEM

Sometimes I export to Splunk or ELK using filebeat.

🧪 7. Testing the Honeypot From Kali

From my Kali machine:

ssh root@192.168.56.10


The honeypot captures:

Username attempts

Password attempts

Connection behavior

I use these logs to track:

Brute-force patterns

Common password attempts

Potential malware payloads

Attack sources inside the lab

🛡️ 8. Why I Built This Lab

I built this VirtualBox setup to practice:

Server administration

Firewall hardening

SSH security

Brute-force detection

Threat analysis

Honeypot deployment

It gives me a safe and isolated environment where I can analyze attacks without risking any real systems.
