# Ubuntu Server + Pi-hole HomeLab

## Project Summary

For this project, I turned an old **Lenovo Yoga laptop into a HomeLab server** using Ubuntu Server and Pi-hole. The main goal was to learn more about Linux, networking, DNS, and troubleshooting while also building something useful for my home network.

I installed Ubuntu Server on the laptop, connected it to my UniFi home network, installed Pi-hole, and configured other devices to use the server for DNS. Pi-hole can then see DNS requests from those devices and block many advertising and tracking domains.

I also had to troubleshoot several real problems during the project, including network connectivity, SSH timeouts, DNS configuration, Pi-hole service checks, and keeping the laptop running with the lid closed.

> **Privacy note:** Example IP addresses are used in the written documentation. `192.0.2.10` represents the Pi-hole server and `192.0.2.1` represents the router/gateway.

## Technologies Used

- Lenovo Yoga laptop
- Ubuntu Server LTS
- Pi-hole
- UniFi router/network
- Windows 11 desktop
- Rufus
- PowerShell / Command Prompt
- Linux terminal
- SSH
- DNS, DHCP, IPv4, and ICMP

## What I Built

```text
                    Internet
                       |
                       v
                UniFi Router
                 192.0.2.1
                       |
             ---------------------
             |                   |
             v                   v
       Ubuntu Server        Home Devices
          Pi-hole           PC / Phones
        192.0.2.10               |
             ^                   |
             +------ DNS --------+
```

The Ubuntu laptop acts as the Pi-hole DNS server. Devices on the network send DNS requests to Pi-hole. Pi-hole checks those requests against its blocklists and forwards allowed requests to an upstream DNS provider.

---

# Step-by-Step Walkthrough

## Step 1 - Download Ubuntu Server

On my Windows computer, I downloaded the current Ubuntu Server LTS ISO. An ISO is the installation image used to install the operating system on the Lenovo laptop.

I saved the ISO somewhere easy to find because it would be selected later in Rufus.

## Step 2 - Create a Bootable USB with Rufus

I connected a USB flash drive to my Windows computer and opened **Rufus**.

In Rufus I:

1. Selected the USB drive under **Device**.
2. Clicked **SELECT** and chose the Ubuntu Server ISO.
3. Left the recommended boot settings in place.
4. Clicked **START**.
5. Confirmed that the USB could be erased.

<p align="center"><img src="./screenshots/01-rufus-usb-creation.jpg" alt="Creating the Ubuntu Server USB with Rufus" width="750"></p>

When Rufus finished, the USB was ready to boot the Lenovo into the Ubuntu installer.

## Step 3 - Boot the Lenovo from the USB

I connected the USB to the Lenovo Yoga and opened its boot menu. I selected the USB drive instead of allowing Windows to start normally.

At the Ubuntu menu, I selected **Try or Install Ubuntu Server**.

<p align="center"><img src="./screenshots/02-ubuntu-install-menu.jpg" alt="Ubuntu Server boot menu" width="750"></p>

This started the Ubuntu Server installer.

## Step 4 - Choose the Basic Ubuntu Settings

I followed the installer and selected the appropriate language and keyboard layout. I used the normal Ubuntu Server installation rather than adding unnecessary software.

For this project, I wanted a lightweight server that I could mainly manage from the command line.

## Step 5 - Configure the Network Connection

During installation, Ubuntu detected the laptop's network adapter.

<p align="center"><img src="./screenshots/03-ubuntu-network-config.jpg" alt="Ubuntu network configuration" width="750"></p>

I made sure the server had network connectivity before continuing. A Pi-hole server needs a dependable IP address because other devices must know exactly where to send their DNS requests.

For the public version of this project, I use:

```text
Pi-hole Server: 192.0.2.10
Gateway/Router: 192.0.2.1
```

In the actual HomeLab, the server uses a private LAN address.

## Step 6 - Configure Storage

The Lenovo was being repurposed as a dedicated server, so I allowed Ubuntu to use the selected internal drive.

Before continuing, I carefully checked that the correct disk was selected because this part of the installation can erase the existing data on the drive.

<p align="center"><img src="./screenshots/04-ubuntu-storage-confirmation.jpg" alt="Ubuntu storage confirmation" width="750"></p>

I confirmed the storage changes and allowed Ubuntu to begin installing.

## Step 7 - Create the Ubuntu Server Account

During setup, I created the server user account, password, computer name, and server name.

These credentials are used to log directly into the Lenovo and can also be used later when connecting remotely through SSH.

## Step 8 - Install OpenSSH

I enabled **OpenSSH Server** during the Ubuntu installation.

SSH is useful because it lets me manage the Lenovo from my Windows desktop instead of having to sit in front of the server every time I need to run a command.

A normal SSH connection follows this format:

```powershell
ssh username@192.0.2.10
```

## Step 9 - Finish Ubuntu Installation

I allowed Ubuntu to finish installing all of its packages.

<p align="center"><img src="./screenshots/05-ubuntu-install-complete.jpg" alt="Ubuntu installation completed" width="750"></p>

When the installation finished, I rebooted the Lenovo and removed the USB installer when prompted.

## Step 10 - Log into Ubuntu Server

After rebooting, Ubuntu displayed a text login screen.

I logged in with the username and password I created during installation.

<p align="center"><img src="./screenshots/06-server-login-and-hostname.jpg" alt="Ubuntu Server login" width="750"></p>

I then checked the server's IP address:

```bash
hostname -I
```

Other useful commands were:

```bash
ip addr
ip route
```

`ip addr` shows the network interfaces and addresses. `ip route` shows how the server reaches the router and other networks.

## Step 11 - Test the Server's Network Connection

Before installing Pi-hole, I tested whether Ubuntu could reach the network gateway.

```bash
ping -c 4 192.0.2.1
```

<p align="center"><img src="./screenshots/07-network-connectivity-test.jpg" alt="Ubuntu connectivity test" width="750"></p>

Successful replies showed that the server could communicate over the local network.

I could also test Internet connectivity with:

```bash
ping -c 4 google.com
```

If the IP-address ping works but a domain-name ping does not, that can point to a DNS problem rather than a basic network connection problem.

## Step 12 - Update Ubuntu

Before installing Pi-hole, I updated Ubuntu's package information:

```bash
sudo apt update
```

Then I installed available updates:

```bash
sudo apt upgrade -y
```

This helped make sure the server was current before adding Pi-hole.

## Step 13 - Install Pi-hole

I installed Pi-hole on the Ubuntu server and followed the installation prompts.

During the setup I selected the active network interface, chose an upstream DNS provider, enabled the web interface, and kept the main recommended Pi-hole options.

When the installation completed, Pi-hole displayed its installation information.

<p align="center"><img src="./screenshots/08-pihole-install-complete.jpg" alt="Pi-hole installation completed" width="750"></p>

I could change the Pi-hole web password with:

```bash
pihole setpassword
```

## Step 14 - Check Pi-hole Status

Before changing DNS on other devices, I checked whether Pi-hole itself was working.

```bash
pihole status
```

<p align="center"><img src="./screenshots/10-pihole-service-status.jpg" alt="Pi-hole service status" width="750"></p>

I wanted to see that the DNS service was listening and that blocking was enabled.

This is an important troubleshooting step because it separates a Pi-hole service problem from a Windows or router configuration problem.

## Step 15 - Open the Pi-hole Web Dashboard

From a computer on the same network, I opened a web browser and entered the Pi-hole server address followed by `/admin`.

For this public example:

```text
http://192.0.2.10/admin
```

The dashboard gives a graphical way to check Pi-hole, view DNS queries, manage lists, and change settings.

## Step 16 - Test Pi-hole from One Windows Computer First

Before changing DNS for my entire home network, I tested Pi-hole from my Windows 11 desktop.

I opened the network adapter settings, edited the IPv4 DNS settings, changed DNS from automatic to manual, and entered the Pi-hole server as the preferred DNS server.

```text
Preferred DNS: 192.0.2.10
```

<p align="center"><img src="./screenshots/09-windows-dns-settings.jpg" alt="Windows DNS settings pointed to Pi-hole" width="750"></p>

Testing one computer first made troubleshooting easier because I could confirm Pi-hole worked before changing the whole network.

## Step 17 - Flush the Windows DNS Cache

Windows may still have old DNS information saved after changing DNS servers.

I opened PowerShell or Command Prompt and ran:

```powershell
ipconfig /flushdns
```

This cleared the existing DNS resolver cache so new lookups would use the newly configured Pi-hole server.

## Step 18 - Verify DNS with nslookup

Next, I ran:

```powershell
nslookup google.com
```

<p align="center"><img src="./screenshots/11-dns-resolution-test.jpg" alt="Testing DNS resolution with nslookup" width="750"></p>

I checked two things:

1. The DNS server shown by `nslookup` was my Pi-hole server.
2. `google.com` still successfully resolved to an IP address.

This confirmed that the computer could send DNS requests through Pi-hole without breaking normal name resolution.

## Step 19 - Verify Requests in the Pi-hole Query Log

I returned to the Pi-hole web interface and opened the **Query Log**.

<p align="center"><img src="./screenshots/12-pihole-query-log.jpg" alt="Pi-hole query log receiving DNS requests" width="750"></p>

Seeing requests from my client confirmed that the full path was working:

```text
Windows Client
      |
      v
Pi-hole DNS Server
      |
      v
Upstream DNS
```

Pi-hole can allow normal requests while blocking domains that match its filtering lists.

## Step 20 - Configure the UniFi Network to Use Pi-hole

After confirming that Pi-hole worked from one client, I configured my home network so devices could use Pi-hole as their DNS server.

The goal was for the UniFi DHCP/network configuration to provide the Pi-hole server's address to clients automatically.

```text
UniFi Network
     |
     +--> DNS Server: 192.0.2.10
```

After a DNS/DHCP change, devices may need to disconnect and reconnect to Wi-Fi/Ethernet or renew their DHCP lease before receiving the new information.

On Windows, useful commands include:

```powershell
ipconfig /release
ipconfig /renew
ipconfig /flushdns
```

## Step 21 - Configure the Laptop to Stay Awake with the Lid Closed

Because the Lenovo is a laptop, closing its lid could normally suspend it. That would take Pi-hole offline and could cause DNS problems for devices using it.

I edited the `systemd-logind` configuration:

```bash
sudo nano /etc/systemd/logind.conf
```

I added or changed these settings:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

<p align="center"><img src="./screenshots/13-lid-switch-logind-config.jpg" alt="Configuring Ubuntu to ignore laptop lid close events" width="750"></p>

In Nano, I saved with **Ctrl + O**, pressed **Enter**, and exited with **Ctrl + X**.

Then I restarted the service:

```bash
sudo systemctl restart systemd-logind
```

I tested this by closing the Lenovo lid and checking from another computer that the server was still reachable.

## Step 22 - Verify the Finished HomeLab

For final testing, I checked the project at multiple levels:

```bash
hostname -I
ping -c 4 192.0.2.1
pihole status
```

From Windows:

```powershell
ping 192.0.2.10
nslookup google.com
```

I also checked the Pi-hole dashboard and Query Log.

The project was considered working when the server remained online, clients could reach it, DNS requests resolved through Pi-hole, queries appeared in the dashboard, and Pi-hole blocking remained enabled.

---

# Troubleshooting

## SSH Timed Out or Was Denied

One of the problems I ran into was being unable to connect to the server through SSH.

I first checked whether the server itself could be reached:

```powershell
ping 192.0.2.10
```

Then on Ubuntu I could check SSH with:

```bash
systemctl status ssh
```

If SSH is not installed:

```bash
sudo apt update
sudo apt install openssh-server -y
```

Then:

```bash
sudo systemctl enable --now ssh
```

This taught me to test basic network connectivity before assuming SSH itself was the problem.

## `sudo` Command Confusion

At one point I was working from a root shell. A root prompt normally ends with `#`, while a regular user's prompt commonly ends with `$`.

When already logged in as root, `sudo` is unnecessary because the shell already has administrative permissions.

## DNS Was Not Working as Expected

I used several checks instead of changing random settings:

```bash
pihole status
```

and from Windows:

```powershell
ipconfig /all
nslookup google.com
```

I checked which DNS server Windows was actually using and then looked for the request in Pi-hole's Query Log.

## One Computer Could Communicate but Another Could Not

During troubleshooting, different computers did not always behave the same way. Testing from more than one client helped show whether the issue was with the Ubuntu server, the network, or one Windows computer.

Commands such as `ping`, `ipconfig`, `ip addr`, and `ip route` helped narrow down the problem.

## Laptop Went to Sleep

Because Pi-hole is providing a network service, the Ubuntu laptop needs to stay available. I changed the lid-switch behavior through `systemd-logind` and tested the server again with the lid closed.

---

# Final Testing Checklist

| Test | What It Checks | Expected Result |
|---|---|---|
| `hostname -I` | Ubuntu IP configuration | Server has its expected address |
| `ping` router | Local network connectivity | Replies received |
| `ping` server from PC | Client-to-server connectivity | Replies received |
| `pihole status` | Pi-hole service | DNS listening and blocking enabled |
| `nslookup google.com` | DNS path | Lookup uses Pi-hole and resolves |
| Pi-hole Query Log | Actual client requests | Queries appear in dashboard |
| Close Lenovo lid | Headless server configuration | Server remains reachable |

# Skills Demonstrated

- Ubuntu Server installation
- Linux command-line administration
- Pi-hole installation and configuration
- DNS configuration and testing
- IPv4 networking
- SSH remote administration
- Windows 11 network configuration
- UniFi network administration
- Linux service management
- Network troubleshooting
- Technical documentation

# Final Result

I successfully repurposed an old Lenovo Yoga into an Ubuntu Server HomeLab running Pi-hole. The server can provide DNS filtering to devices on my home network, and I can manage and troubleshoot it using Linux commands, SSH, Windows networking tools, and the Pi-hole web dashboard.

The most useful part of the project was troubleshooting the problems that came up during the build. It helped me understand how DNS, IP addresses, clients, routers, and Linux services work together instead of only following a set of instructions.

# Repository Structure

For the pictures to display **inside the README**, GitHub must contain both the README and the `screenshots` folder:

```text
Ubuntu-Pi-hole-HomeLab/
|
|-- README.md
|
`-- screenshots/
    |-- 01-rufus-usb-creation.jpg
    |-- 02-ubuntu-install-menu.jpg
    |-- 03-ubuntu-network-config.jpg
    |-- 04-ubuntu-storage-confirmation.jpg
    |-- 05-ubuntu-install-complete.jpg
    |-- 06-server-login-and-hostname.jpg
    |-- 07-network-connectivity-test.jpg
    |-- 08-pihole-install-complete.jpg
    |-- 09-windows-dns-settings.jpg
    |-- 10-pihole-service-status.jpg
    |-- 11-dns-resolution-test.jpg
    |-- 12-pihole-query-log.jpg
    `-- 13-lid-switch-logind-config.jpg
```

**Important:** Do not upload the ZIP itself as the repository content. Extract it first, then upload `README.md` **and** the entire `screenshots` folder to the same GitHub repository. The `<img>` tags in this README will then display the screenshots directly on the page.
