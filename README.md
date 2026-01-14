![Pi-hole Logo](./assets/pihole-logo.png)

# Pi-hole Setup Guide

The purpose of this guide is to serve as a reference for configuring network-wide ad blocking using Pi-hole on a Raspberry Pi 3 Model B+. While this guide is written with that specific hardware in mind, the concepts and procedures apply broadly to any Unix-like system with network capabilities.

## Table of Contents

- [Introduction](#introduction)
- [Parts List](#parts-list)
- [Installation and Configuration](#installation-and-configuration)
   - [SSH Key Pair Generation](#ssh-key-pair-generation)
      - [ssh-keygen Command](#ssh-keygen-command)
   - [Flashing the MicroSD Card with Raspberry Pi Imager](#flashing-the-microsd-card-with-raspberry-pi-imager)
      - [Raspberry Pi Imager Setup Steps](#raspberry-pi-imager-setup-steps)
   - [Setting a Static IP Address](#setting-a-static-ip-address)
- [Maintenance](#maintenance)
- [Troubleshooting](#troubleshooting)

## Introduction

In the official Pi-hole documentation, it is described as:

> ...a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

DNS (Domain Name System) translates human-readable domain names (such as `google.com`) into IP addresses that computers use to communicate. In simple terms, Pi-hole provides network-wide ad blocking by blacklisting common advertising and tracking services at the DNS level. It has grown in popularity due to its ease of installation, minimal system requirements, and suitability for low-power, always-on devices such as the Raspberry Pi.

This guide walks through installing and configuring Pi-hole on a Raspberry Pi, including initial setup, network configuration, and basic maintenance. Basic familiarity with the Linux command line and home networking concepts is assumed.

## Parts List

- Raspberry Pi 3 Model B+ (other Raspberry Pi models will also work)
- MicroSD Card (8 GB minimum recommended)
- MicroSD Card Reader (required if your computer does not have a built-in reader)
- 5V 2.5A Micro-USB Power Supply
- Ethernet Cable (optional, but strongly recommended)
- A Computer (Windows or Linux)

## Installation and Configuration

### SSH Key Pair Generation

To manage the Pi-hole server remotely, we need the ability to connect via SSH. SSH authentication can be done using a traditional username and password, or by using an SSH key pair for passwordless authentication. In this guide, SSH keys are used, as they are generally considered more secure and more convenient than password-based logins.

This section demonstrates how to generate an SSH key pair from the command line. In addition to improving security, this approach helps familiarize you with basic Linux command-line tools that are commonly used when administering servers.

#### ssh-keygen Command

```bash
ssh-keygen -t ed25519 -C "user@pihole"
```

This command generates an SSH key pair. Paste it into a terminal on a Linux system (or Windows Subsystem for Linux if you are on Windows). Replace the `user` portion of the comment with your desired user account name.

After running the command, you will be prompted to choose a file location for the key pair. Press `Enter` to accept the default location, which is typically `/home/user/.ssh/id_ed25519`. You will then be prompted to enter a passphrase. This step is optional; however, if this server will be used beyond this tutorial, setting a strong passphrase is recommended, as it encrypts the private key and adds an additional layer of security.

Once these steps are complete, the SSH key pair is generated. The public key can be viewed by opening the file in a text editor or by printing it to the terminal.

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

To ensure consistent network access, the Raspberry Pi should be assigned a static IP address before installing Pi-hole.

## Maintenance

This section covers routine updates and basic upkeep for a Pi-hole installation.

## Troubleshooting

This section outlines common issues and steps to diagnose and resolve them.
