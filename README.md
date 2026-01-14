![Pi-hole Logo](./assets/pihole-logo.png)

# Pi-hole Setup Guide

The purpose of this guide is to serve as a reference for configuring network-wide ad blocking using Pi-hole on a Raspberry Pi 3 Model B+. While this guide is written with that specific hardware in mind, the concepts and procedures apply broadly to any Unix-like system with network capabilities.

## Table of Contents

- [Introduction](#introduction)
- [Parts List](#parts-list)
- [Installation and Configuration](#installation-and-configuration)
- [Maintenance](#maintenance)
- [Troubleshooting](#troubleshooting)

## Introduction

In the official Pi-hole documentation, it is described as:

> ...a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

DNS (Domain Name System) translates human-readable domain names (such as `google.com`) into IP addresses that computers use to communicate. In simple terms, Pi-hole provides network-wide ad blocking by blacklisting common advertising and tracking services at the DNS level. It has grown in popularity due to its ease of installation, minimal system requirements, and suitability for low-power, always-on devices such as the Raspberry Pi.

This guide walks through installing and configuring Pi-hole on a Raspberry Pi, including initial setup, network configuration, and basic maintenance. Basic familiarity with the Linux command line and home networking concepts is assumed.

## Parts List

- Raspberry Pi 3 Model B+ (other Raspberry Pi models will also work)
- Micro SD Card (8 GB minimum recommended)
- Micro SD Card Reader (required if your computer does not have a built-in reader)
- 5V 2.5A Micro-USB Power Supply
- Ethernet Cable (optional, but strongly recommended)
- A Computer (Windows, macOS, or Linux)

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

## Maintenance

This section covers routine updates and basic upkeep for a Pi-hole installation.

## Troubleshooting

This section outlines common issues and steps to diagnose and resolve them.
