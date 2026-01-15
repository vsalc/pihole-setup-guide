![Pi-hole Logo](./assets/pihole-logo.png)

# Pi-hole Setup Guide

The purpose of this guide is to serve as a reference for configuring network-wide ad blocking using Pi-hole on a Raspberry Pi. The specific model used in this guide is a Raspberry Pi 3 Model B+, but the concepts and procedures apply broadly to any Unix-like system with network capabilities.

## Table of Contents

- [Introduction](#introduction)
- [Parts List](#parts-list)
- [Installation and Configuration](#installation-and-configuration)
  - [SSH Key Pair Generation](#ssh-key-pair-generation)
    - [ssh-keygen Command](#ssh-keygen-command)
  - [Flashing the MicroSD Card with Raspberry Pi Imager](#flashing-the-microsd-card-with-raspberry-pi-imager)
    - [Raspberry Pi Imager Setup Steps](#raspberry-pi-imager-setup-steps)
  - [Setting a Static IP Address](#setting-a-static-ip-address)
    - [Windows Command: Find Gateway IP](#windows-command-find-gateway-ip)
    - [Linux Command: Find Gateway IP](#linux-command-find-gateway-ip)
    - [Router Configuration Steps](#router-configuration-steps)
  - [Connect to and Update the Server](#connect-to-and-update-the-server)
    - [SSH Command](#ssh-command)
    - [Updating the System Command(s)](#updating-the-system-commands)
  - [Installing Pi-hole](#installing-pi-hole) 
- [Maintenance](#maintenance)
- [Troubleshooting](#troubleshooting)

## Introduction

In the official Pi-hole documentation, it is described as:

> ...a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

A *DNS (Domain Name System)* translates human-readable domain names (such as `github.com`) into IP addresses that computers use to communicate. By acting as the DNS server for your network, Pi-hole is able to provide network-wide ad blocking by filtering requests at this level. This is accomplished by blacklisting known advertising and tracking domains, preventing them from ever reaching your browser. The result is a cleaner browsing experience and a modest improvement to overall network security.

As mentioned above, Pi-hole operates using blocklists. Pi-hole ships with an official default blocklist, but some users choose to add third-party lists that are more aggressive. While these lists can improve ad blocking effectiveness, they may also cause certain websites or services to break. Later in this guide, we will demonstrate how to add additional blocklists if you wish.

This guide walks through installing and configuring Pi-hole on a Raspberry Pi, including initial setup, network configuration, and basic maintenance. Basic familiarity with the Linux command line and home networking concepts is assumed, but all steps are presented with the goal that the guide can be followed end-to-end without much experience.

With the introduction out of the way, let’s get started!

## Parts List

- Raspberry Pi 3 Model B+ (other Raspberry Pi models will also work)
- MicroSD Card (32 GB was the smallest size available at Micro Center)
- MicroSD Card Reader (required if your computer does not have a built-in reader)
- 5V 2.5A Micro-USB Power Supply (an older Apple power brick worked for this setup; ensure the voltage and current ratings are appropriate)
- Ethernet Cable (optional, but strongly recommended)
- Computer (Windows with WSL or Linux)

## Installation and Configuration

### SSH Key Pair Generation

For this guide, the Raspberry Pi is setup as a headless server, meaning no peripherals (monitor, keyboard, mouse, etc.) are connected to the device. To interact with and administer the server, a method of remote access is required.

*Secure Shell (SSH)* is the network protocol that allows you to remotely log in to another machine and obtain shell access. SSH authentication can be performed using a traditional username and password, or by using an SSH key pair for passwordless authentication. In this guide, SSH keys are used, as they are generally considered more secure and more convenient than password-based logins.

This section demonstrates how to generate an SSH key pair from the command line. In addition to improving security, this approach helps familiarize you with basic Linux command-line tools that are commonly used when administering servers.

#### SSH Keygen Command

```bash
ssh-keygen -t ed25519 -C "user@pihole"
```

This command generates an SSH key pair. Paste it into a terminal on a Linux system (or Windows Subsystem for Linux if you are on Windows). Replace the `user` portion of the comment with your desired user account name.

After running the command, you will be prompted to choose a file location for the key pair. Press `Enter` to accept the default location, which is typically `/home/user/.ssh/id_ed25519`. You will then be prompted to enter a passphrase. This step is optional, however, if this server will be used beyond this tutorial, setting a strong passphrase is recommended, as it encrypts the private key and adds an additional layer of security.

Once these steps are complete, the SSH key pair is generated. The public and private keys can be viewed by opening the files in a text editor or by printing them to the terminal.

Here is a breakdown of each component of the command:
- `ssh-keygen`: The utility used to generate SSH key pairs.
- `-t ed25519`: Specifies the key type. Ed25519 is a modern elliptic-curve cryptographic algorithm that is secure and efficient (RSA is another commonly used option).
- `-C "user@pihole"`: Adds a comment to the key. This comment is stored with the key and is useful for identifying its purpose later.

### Flashing the MicroSD Card with Raspberry Pi Imager

To flash the microSD card, the official Raspberry Pi Imager is the quickest and easiest option. The imaging software can be downloaded from the Raspberry Pi software page, linked [here](https://www.raspberrypi.com/software/).

Before opening the application, insert the microSD card into the reader and connect it to the computer where Raspberry Pi Imager is installed.

> **Note:** During the setup process for this tutorial, I encountered an issue with the official Raspberry Pi Imager package on Ubuntu where configuration settings were not applied. If you experience a similar issue on Linux, you are not alone. In my case, the issue could not be resolved, and I completed the imaging process using a Windows system instead. The steps below are otherwise identical on Windows and Linux.

---

#### Raspberry Pi Imager Setup Steps

1. **Select your Raspberry Pi device**  
   From the device selection dropdown, choose your Raspberry Pi model.

2. **Choose an operating system**  
   This setup uses the Raspberry Pi as a headless server, meaning it will be accessed remotely via SSH and will not use a graphical desktop environment. Select `Raspberry Pi OS (other)`, then choose `Raspberry Pi OS Lite (64-bit)`. This is a lightweight version of Raspberry Pi OS without a desktop environment.

3. **Select your storage device**  
   Locate and select the microSD card.

4. **Customization – Choose hostname**  
   Enter a hostname of your choice. This name will be used to identify the device on your local network and in your router’s web interface. In this tutorial, the hostname `pihole` is used.

5. **Customization – Localization**  
   If location services are enabled, these fields may be populated automatically. Otherwise, select the appropriate time zone, keyboard layout, and locale manually.

6. **Customization – Choose username and password**  
   Set a username for the system. While not required, it is recommended to use the same username referenced in your SSH key comment. Choose a strong password for the account. This password will be used for administrative tasks.

7. **Customization – Configure Wi-Fi (optional)**  
   For this setup, an Ethernet connection is used, so Wi-Fi configuration is not required. If you do not have access to Ethernet, configure Wi-Fi by entering your network’s SSID and password. While Wi-Fi works, a wired Ethernet connection is generally preferred for servers due to improved reliability and security.

8. **Customization – Enable SSH authentication**  
   **Do not skip this step.** SSH must be enabled to access the system headlessly.  
   Enable SSH and select `Use public key authentication`. View your public key by running:
   ```bash
   cat /home/user/.ssh/id_ed25519.pub
   ```
   Copy the contents of the file, then paste the key. Confirm and save the changes.
   
9. **Customization – Raspberry Pi Connect**  
   Raspberry Pi Connect is not used in this tutorial and can be left disabled. This feature allows remote access to the device from outside the local network.

10. **Write the image**  
    At this point, all configuration steps are complete. Writing the image will erase all existing data on the microSD card. Ensure that any important files have been backed up before proceeding. Write the image, then safely eject the microSD card once the process is complete.

11. **Hardware setup**  
    Insert the microSD card into the underside of the Raspberry Pi. Connect the Ethernet cable and power supply. The device will boot and perform initial setup automatically. Once the activity lights settle, the system is ready.

### Setting a Static IP Address

For the next part of the setup, we need to access our router's web interface. Open a web browser and enter the IP address of your router. Alternatively, many routers list a gateway URL on a label on the back of the device, which can also be used.

If you are unsure of your router's IP address, you can determine it using the following commands, depending on your operating system.

#### Windows Command: Find Gateway IP
```powershell
ipconfig
```

#### Linux Command: Find Gateway IP
```bash
ip route
```

---

#### Router Configuration Steps
1. **Log in to the web interface**
   Using either the default credentials or a password you previously set, log in to the router's web interface. If you have never accessed this page before or are unsure of the credentials, they are commonly printed on the back or underside of the router near the gateway URL.

2. **Locate DHCP Settings**
   Find the DHCP configuration section within the router's settings. *DHCP (Dynamic Host Configuration Protocol)* is responsible for automatically assigning IP addresses to devices on a network. The exact location of these settings varies by manufacturer. In my case, the following navigation path was used: `Network Settings > IPv4 Address Distribution > DHCP Connection Settings`.

3. **Setting a Static IP Address**
   In the list of connected devices, locate your Raspberry Pi using the hostname you set earlier. Edit the device entry and change the lease type to `Static Lease Type`, then save or apply the changes.

> **Note**: Remember the IP address of the Raspberry Pi.

### Connect to and Update the Server

This short section covers connecting to the Raspberry Pi via SSH for the first time and updating system packages. This process is simple and can be completed with just two commands. Start by opening a terminal window and pasting the first command.

#### SSH Command 
```bash
ssh -i /home/user/.ssh/id_ed25519 user@192.168.X.X
```

Here is a breakdown of each component of the command:
- `ssh`: The utility used to connect to a remote server via SSH.
- `-i /home/user/.ssh/id_ed25519`: Specifies the identity file (private SSH key) used for authentication.
- `user@192.168.X.X`: Specifies the username and IP address of the remote machine. Replace this with the IP address of your Raspberry Pi obtained in the previous section.

You may be prompted to confirm adding the host to your list of known hosts. Accept this prompt and press `Enter` to continue. You should now be connected to the system. This is the perfect time to say *I'm in*.

Now that you are connected, the first administrative task is to update the system packages.

#### Updating the System Command(s)
```bash
sudo apt update && sudo apt upgrade -y
```

Here is a breakdown of each component of the command:
- `sudo`: Runs the command with root (administrative) privileges.
- `apt update`: Refreshes the package repository metadata.
- `&&`: Bash operator that executes the next command only if the previous command succeeds.
- `apt upgrade`: Installs available updates for installed packages.
- `-y`: Automatically accepts prompts during the upgrade process.

You will be prompted to enter your password. Type in the password you set while imaging the microSD in Raspberry Pi Imager. This may take a moment, but once it completes we are ready to move on.

### Installing Pi-hole

Finally, it is time to install the Pi-hole software. Keeping this terminal session active, open a new browser window and navigate to the Pi-hole installation guide, linked [here](https://docs.pi-hole.net/main/basic-install/). 

The Pi-hole team has made this process straightforward by providing a one-step automated installer. Copy the command:

```bash
curl -sSL https://install.pi-hole.net | bash
```

Here is a breakdown of each component of the command:
- `curl`: A command-line tool used to transfer data to or from a server using various protocols (HTTPS in this case).
- `-s`: Runs curl in silent mode, silencing progress output and error messages.
- `-S`: Displays error messages if the request fails (used in tandem with `-s`).
- `-L`: Follows HTTP redirects automatically.
- `https://install.pi-hole.net`: The URL hosting the official Pi-hole installation script.
- `|`: Bash operator that sends or *pipes* the output of the command on the left as input to the command on the right.
- `bash`: Executes the downloaded installation script using the Bash shell.

Paste the command into the terminal session connected to your Raspberry Pi. The installation process will run automatically, so you can sit back and wait. Once the screen switches to the blue installer interface, you can proceed to the next section.

### Pi-hole Configuration

Todo...

## Maintenance

This section covers routine updates and basic upkeep for a Pi-hole installation.

## Troubleshooting

This section outlines common issues and steps to diagnose and resolve them.
