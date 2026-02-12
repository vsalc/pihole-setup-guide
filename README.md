#

<p align="center">
  <img src="https://raw.githubusercontent.com/pi-hole/graphics/refs/heads/master/Vortex/vortex_with_text.svg" alt="Pi-hole logo" width="168" height="270">
  <br>
  <strong>Pi-hole Setup Guide</strong>
</p>

A quickstart guide for deploying Pi-hole on a Raspberry Pi for home network-wide ad blocking.

## Table of Contents

- [Introduction](#introduction)
- [Hardware](#hardware)
- [Generating an SSH Key Pair](#generating-an-ssh-key-pair)
- [Raspberry Pi Imager](#raspberry-pi-imager)
- [Hardware Setup](#hardware-setup)
- [Router Configuration](#router-configuration)
- [Connecting to and Updating the Server](#connecting-to-and-updating-the-server)
- [Installing Pi-hole](#installing-pi-hole)
- [More Router Configuration](#more-router-configuration)
- [Server Maintenance](#server-maintenance)
- [Troubleshooting](#troubleshooting)

## Introduction

The goal of this guide is to teach you the bare minimum knowledge needed to set up Pi-hole on your home network. A basic understanding of networking and Linux is expected (since you're reading this), but anyone should be able to follow along without too much trouble.

### What is Pi-hole?

According to the official documentation, Pi-hole is:

> ...a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

This means that you only need to set up one device to get network-wide ad blocking for every device on your network. That includes phones, tablets, smart TVs, laptops, the list goes on.

### How it works

DNS (Domain Name System) is the network protocol that translates human-readable domain names like `google.com` into IP addresses like `8.8.8.8` that computers actually use to communicate.

Pi-hole intercepts all DNS requests on your network. Once it receives a request, it checks if the domain is on its blocklist. Pi-hole relies on these blocklists. The default blocklist includes some of the most common ad services. People have created their own (more aggressive) blocklists which we'll show you how to install later, if you choose. If the domain is blocked, Pi-hole drops the request. **No ads, yay!** If the request is allowed, Pi-hole forwards it to your configured upstream DNS server, and you get your content as expected.

## Hardware

- **Raspberry Pi**: Model 3B+ (any modern UNIX system with network capabilities should work)
- **MicroSD card**: 32GB (the smallest available at my local Micro Center)
- **MicroSD card reader**: For flashing the OS from your computer
- **Power supply**: Official Raspberry Pi power supply recommended (I got away with using an Apple brick and old Micro USB cable I had on hand)
- **Ethernet cable**: Not strictly necessary if your board has WiFi, but recommended from a security standpoint
- **Computer**: For running Raspberry Pi Imager and SSH access (Windows with WSL, macOS, or Linux)

## Generating an SSH Key Pair

For space reasons, but also for remote access practice, this Pi-hole is going to be set up as a headless server. A headless server is a computer without any peripherals attached to it (keyboard, mouse, monitor, etc.).

### What is SSH?

To remotely access a server we need to connect to it via SSH (Secure Shell). SSH access drops us into a shell on the remote server, allowing us to perform our administrative duties. SSH has two main authentication methods: password and passwordless. Passwordless is generally regarded as the more secure of the two, so that's what we'll use here.

### Asymmetric Key Pairs

Passwordless authentication relies on asymmetric encryption. I won't delve fully into this topic here, but essentially there are two keys needed to verify and establish the connection. We give the Pi-hole our public key (this is safe to share with anyone). We then have our own private key (**DO NOT SHARE OR LOSE**) which stays on our local machine. When we attempt to connect via SSH, the server uses our public key to verify that we possess the matching private key. This eliminates the need for a password, which is a common attack vector for malicious activity.

### ssh-keygen

The first official step in this guide is to generate this key pair. To do so, get up, head over to your computer, and pop open a terminal window.

The command we need to use is `ssh-keygen`. This command comes preinstalled with most Linux distributions and handles all the heavy lifting of creating the key pair for us. With your terminal open, paste the following command... don't worry, I will explain each and every command I tell you to run so that you know what is happening.

```bash
ssh-keygen -t ed25519 -C "user@pihole"
```

### Command Breakdown

- **ssh-keygen**: the utility used to generate SSH key pairs
- **-t ed25519**: specifies the encryption algorithm we are going to use for the keys (RSA is another common algorithm you may already know)
- **-C "user@pihole"**: specifies a comment that will be attached to the public key (put whatever you like here; this doesn't do anything functionally but may be useful for identifying keys later if you forget their purpose)

You will then be prompted to enter a file location for where you want to store the keys. The default is something like `/home/user/.ssh/id_ed25519`. Hit ENTER. Next, you'll be prompted to enter a passphrase. This is another layer of security that encrypts the private key, requiring the passphrase to unlock it. After entering a super secure passphrase and confirming it, the key pair is complete. Congrats!

## Raspberry Pi Imager

Now it's time to flash Raspberry Pi OS onto our MicroSD card. Grab your MicroSD card, plug it into your reader, and then plug that into your computer. Boot up Raspberry Pi Imager and then continue to the next section.

### Setup Steps

1. **Select your Raspberry Pi device**: Pretty self-explanatory. Select the correct device from the dropdown list. For me, this was `Raspberry Pi 3`.

2. **Choose operating system**: Since we are using a headless server, we have no use for a GUI (Graphical User Interface). So, we can install a lighter version of Raspberry Pi OS. Select `Raspberry Pi OS (other) → Raspberry Pi OS Lite (64-bit)`, then hit next.

3. **Select your storage device**: Identify the MicroSD card and select it. If there are multiple drives, try selecting `Exclude system drives`.

4. **Customization: Choose hostname**: Enter the hostname for your device. You will use this to identify the device later in your router portal. I went with something simple like `pihole`.

5. **Customization: Localization**: This section is likely autocompleted. However, if it is not, then select the proper capital city, time zone, and keyboard layout.

6. **Customization: Choose username**: Set the username and password for your user account on the device. Use something strong like `password1`.

7. **Customization: Choose Wi-Fi**: This section is not important if you plan on using an Ethernet connection, but as a fallback you can enter your credentials here.

8. **Customization: SSH authentication**: This is a really important step, so pay attention! If you skipped around and did not generate an SSH key pair, review [Generating an SSH Key Pair](#generating-an-ssh-key-pair). Go to your terminal and `cat` out the contents of your public key (this will have the `.pub` file extension). Copy it and come back to the imaging software. Select `Enable SSH → Use public key authentication`. Finally, paste the contents of your public key in the form field and click `ADD`.

9. **Customization: Raspberry Pi Connect**: Raspberry Pi Connect allows you to access your Pi from a web interface. This was not used for this setup, so it can be left disabled.

10. **Write image**: Select `WRITE` and then confirm `I UNDERSTAND, ERASE AND WRITE`. Remember that this is permanent and anything left on the drive will be lost. Back up any important files before completing this step.

That is the complete walkthrough of the Raspberry Pi Imager software. Once the write completes, you can eject your MicroSD card.

## Hardware Setup

Hardware setup is quick and easy. Take the MicroSD card and insert it in the slot on the underbelly of the Pi. Connect your power supply and your Ethernet cable (if you are using one). That's it! The Pi will boot on its own. There will be a flashing activity light. Once this light calms down, the boot process has finished. This is the last time you ever have to physically touch this device. Before we start doing anything on the server, we have a few things we need to do in our router portal first. Head back to your computer and open this up. If you don't know where this is, I will offer some help in the next section.

## Router Configuration

Remember that our Pi-hole is a DNS server. For our devices to reliably communicate with it, the Pi-hole needs a static IP address. By default in most home networks, devices receive dynamic IP addresses assigned by DHCP (Dynamic Host Configuration Protocol). DHCP automatically assigns IP addresses from a pool, which means a device might get a different IP address each time it connects to the network or when its lease expires.

### Static IP Addresses

If the IP address keeps changing, how can our devices reliably find the Pi-hole? We need to reserve a static IP address for it. This is done in your router's web portal through DHCP reservation (also called static DHCP). This binds the Pi's MAC address to a specific IP, ensuring it always has the same address.

### Accessing Your Router Portal

If you have never accessed your router web portal before, you can likely find its gateway URL and admin password on the back or underbelly of your router. Enter the gateway URL into your web browser, then log in with the admin credentials.

**Important**: Changing the default password shipped with routers is good security practice and highly recommended.

### Setting a Static IP Address

Every router portal is different, but I'll walk through my process to give you an idea of what to look for on your specific hardware. In my portal, I navigated to `Advanced → Network Settings → IPv4 Address Distribution`. There is a list of connected devices. I located my Pi by the hostname I set earlier. I selected to edit it, which brought me to `DHCP Connection Settings`. Here, select `Static Lease Type` and enter your desired IP address. The default here was fine. Make sure to note this IP address because you will need it later to SSH to the server.

## Connecting to and Updating the Server

With our static IP address set, we can now connect to our Pi for the first time. Open up a terminal and type the following command. You will have to change the file location, username, and IP address. This is only a template:

```bash
ssh -i /home/user/.ssh/id_ed25519 user@192.168.1.200
```

### Command Breakdown

- **ssh**: the utility used to establish secure shell connections
- **-i /home/user/.ssh/id_ed25519**: specifies the identity file (private key) to use for authentication
- **user@192.168.1.200**: the username and IP address of the server you want to connect to

When prompted, type `yes` to add the fingerprint to known hosts. If this does not work, go back and review the steps up to this point. If everything has gone smoothly, you should establish the connection and see a command prompt on your Pi.

### Our First Administrative Task

With the shell still open, we can perform our first administrative task: updating the server's packages. Paste the following command into the terminal and then enter the passphrase that you configured during SSH key generation.

```bash
sudo apt update && sudo apt upgrade -y
```

### Command Breakdown

- **sudo**: runs the command with superuser (root) privileges, required for system-level changes
- **apt update**: downloads the latest package lists from repositories, updating the system's available software versions
- **&&**: bash operator that only runs the second command if the first succeeds
- **apt upgrade**: installs newer versions of all currently installed packages
- **-y**: automatically answers "yes" to all prompts, allowing the upgrade to proceed automatically

The update process may take a minute. Once complete, your Pi is fully up to date and ready for Pi-hole installation. You can now proceed to the next section.

## Installing Pi-hole

- todo

## More Router Configuration

- todo

## Server Maintenance

- todo

## Troubleshooting

- todo

<!--
During my initial attempts I was using the Raspberry Pi Imager on Ubuntu Linux. I ran into an issue where the Micro SD card would be flashed with this OS, but no configuration settings had been saved. This meant I could not SSH in. I was unable to resolve this issue and had to move over to a Windows system for this step.
-->
