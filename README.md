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
- [Connecting to and Updating the Server](#connecting-to-and-updating-the-server)
- [Installing Pi-hole](#installing-pi-hole)
- [Router Configuration](#router-configuration)
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
- **Micro SD card**: 32GB (the smallest available at my local Micro Center)
- **Micro SD card reader**: For flashing the OS from your computer
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
ssh-keygen -t ed25519 -C "username@pihole"
```

**Command Breakdown**

- **ssh-keygen**: the utility used to generate SSH key pairs
- **-t ed25519**: specifies the encryption algorithm we are going to use for the keys (RSA is another common algorithm you may already know)
- **-C "user@pihole"**: specifies a comment that will be attached to the public key (put whatever you like here; this doesn't do anything functionally but may be useful for identifying keys later if you forget their purpose)

You will then be prompted to enter a file location for where you want to store the keys. The default is something like `/home/username/.ssh/id_ed25519`. Hit ENTER. Next, you'll be prompted to enter a passphrase. This is another layer of security that encrypts the private key, requiring the passphrase to unlock it. After entering a super secure passphrase and confirming it, the key pair is complete. Congrats!

## Raspberry Pi Imager

- todo

## Connecting to and Updating the Server

- todo

## Installing Pi-hole

- todo

## Router Configuration

_Configuring your router to use Pi-hole as the network DNS server_

- todo

## Server Maintenance

- todo

## Troubleshooting

- todo
