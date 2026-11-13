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
- [Conclusion](#conclusion)

## Introduction

The goal of this guide is to teach you the bare minimum knowledge needed to set up Pi-hole on your home network. A basic understanding of networking and Linux is expected (since you're reading this), but anyone should be able to follow along without too much trouble.

### What is Pi-hole?

According to the official documentation, Pi-hole is:

> ...a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software.

This means that you only need to set up one device to get network-wide ad blocking for every device on your network. That includes phones, tablets, smart TVs, laptops, the list goes on.

### How it Works

DNS (Domain Name System) is the network protocol that translates human-readable domain names like `google.com` into IP addresses like `142.250.72.14` that computers actually use to communicate. (You may recognize `8.8.8.8`. That is not google.com, it is Google's public DNS resolver, one of the servers that does this translating.)

Pi-hole intercepts all DNS requests on your network. Once it receives a request, it checks if the domain is on its blocklist. Pi-hole relies on these blocklists. The default blocklist includes some of the most common ad services. People have created their own (more aggressive) blocklists which you can add later, if you choose. If the domain is blocked, Pi-hole does not ignore the request. By default it answers with the unspecified address `0.0.0.0` (`::` for IPv6), so your device gets an instant answer that leads nowhere and gives up on the ad. **No ads, yay!** If the request is allowed, Pi-hole forwards it to your configured upstream DNS server, and you get your content as expected.

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

4. **Customization: Choose hostname**: Enter the hostname for your device. You will use this to identify the device later in your router interface. I went with something simple like `pihole`.

5. **Customization: Localization**: This section is likely autocompleted. However, if it is not, then select the proper capital city, time zone, and keyboard layout.

6. **Customization: Choose username**: Set the username and password for your user account on the device. Use a long password that you do not use anywhere else. A few random words strung together or an entry generated by a password manager both work. This is the password `sudo` may ask for later, so make sure you can actually get it back.

7. **Customization: Choose Wi-Fi**: This section is not important if you plan on using an Ethernet connection, but as a fallback you can enter your credentials here.

8. **Customization: SSH authentication**: This is a really important step, so pay attention! If you skipped around and did not generate an SSH key pair, review [Generating an SSH Key Pair](#generating-an-ssh-key-pair). Go to your terminal and `cat` out the contents of your public key (this will have the `.pub` file extension). Copy it and come back to the imaging software. Select `Enable SSH → Use public key authentication`. Finally, paste the contents of your public key in the form field and click `ADD`.

9. **Customization: Raspberry Pi Connect**: Raspberry Pi Connect allows you to access your Pi from a web interface. This was not used for this setup, so it can be left disabled.

10. **Write image**: Select `WRITE` and then confirm `I UNDERSTAND, ERASE AND WRITE`. Remember that this is permanent and anything left on the drive will be lost. Back up any important files before completing this step.

That is the complete walkthrough of the Raspberry Pi Imager software. Once the write completes, you can eject your MicroSD card.

## Hardware Setup

Hardware setup is quick and easy. Take the MicroSD card and insert it in the slot on the underbelly of the Pi. Connect your power supply and your Ethernet cable (if you are using one). That's it! The Pi will boot on its own. There will be a flashing activity light. Once this light calms down, the boot process has finished. This is the last time you ever have to physically touch this device. Before we start doing anything on the server, we have a few things we need to do in our router interface first. Head back to your computer and open this up. If you don't know where this is, I will offer some help in the next section.

## Router Configuration

Remember that our Pi-hole is a DNS server. For our devices to reliably communicate with it, the Pi-hole needs a static IP address. By default in most home networks, devices receive dynamic IP addresses assigned by DHCP (Dynamic Host Configuration Protocol). DHCP automatically assigns IP addresses from a pool, which means a device might get a different IP address each time it connects to the network or when its lease expires.

### Static IP Addresses

If the IP address keeps changing, how can our devices reliably find the Pi-hole? We need to reserve a static IP address for it. This is done in your router's web interface through DHCP reservation (also called static DHCP). This binds the Pi's MAC address to a specific IP, ensuring it always has the same address.

### Accessing Your Router's Web Interface

If you have never accessed your router web interface before, you can likely find its gateway URL and admin password on the back or underbelly of your router. Enter the gateway URL into your web browser, then log in with the admin credentials.

**Important**: Changing the default password shipped with routers is good security practice and highly recommended.

### Setting a Static IP Address

Every router interface is different, but I'll walk through my process to give you an idea of what to look for on your specific hardware. In my interface, I navigated to `Advanced → Network Settings → IPv4 Address Distribution`. There is a list of connected devices. I located my Pi by the hostname I set earlier. I selected to edit it, which brought me to `DHCP Connection Settings`. Here, select `Static Lease Type` and enter your desired IP address. The default here was fine. Make sure to note this IP address because you will need it later to SSH to the server.

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

With the shell still open, we can perform our first administrative task: updating the server's packages. Paste the following command into the terminal. If `sudo` asks for a password, it wants the account password you set in Raspberry Pi Imager, not the passphrase for your SSH key. Raspberry Pi OS often lets the first user run `sudo` without a password, so you may not be asked at all.

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

The Pi-hole team has extensive documentation that I highly encourage you to read for yourself to fully understand some of the things you can do to manage this device. Here is a [link](https://docs.pi-hole.net/) to that information.

The installation is very simple. Pi-hole has a one-step automated install command which you can copy and paste into your terminal.

```bash
curl -sSL https://install.pi-hole.net | bash
```

This is the only command that I will not give you a full breakdown for since I do not know everything that it is doing under the hood. At a high level though, you are curling (downloading) the install script and then piping the output into bash (executing the script). This will open up a blue automated installer menu. The following steps are how I configured my installation.

### Pi-hole Automated Installer

1. **Pages 1 & 2**: The first two pages are purely informational. No work is needed here, so just hit `OK` for both (as if you have any other option).

2. **Static IP Needed**: This slide is a final reminder that a static IP address is needed. If you have been following along up to this point, this is already done and you can continue. If you skipped this, exit and review [Router Configuration](#router-configuration).

3. **Choose An Interface**: Select the network interface which your Pi is going to use. If you are using Ethernet this is `eth0`. If you are using WiFi then this is `wlan0`.

4. **Select Upstream DNS Provider**: Here you configure the upstream DNS that your Pi-hole will use for all requests that are not blocked. I chose `Cloudflare (DNSSEC)`, but any should work.

5. **Blocklists**: This page asks if you would like to install the default blocklist that ships with Pi-hole. Select `Yes`. If you have a custom blocklist you want to use, you can skip this step, but you will have to install that later in the Pi-hole web interface for the Pi-hole to actually do anything.

6. **Enable Logging**: Enabling query logging is good for troubleshooting your Pi-hole later down the road. Select `Yes`.

7. **Select a privacy mode for FTL**: FTL (Faster Than Light) is Pi-hole's DNS engine. Privacy mode determines what information is logged. I chose `Show everything` for maximum visibility.

8. **Installation Complete**: With that, your Pi-hole is 99% set up. On this page you need to note a few pieces of information: IPv4 address, web interface URL, and admin password.

### Changing Default Password

Just like changing the default password of your router interface, you should also change the default password of your Pi-hole interface. Close the automated installer by selecting `OK`, then type the following command in the terminal to set a new password.

```bash
sudo pihole setpassword
```

### Command Breakdown

- **sudo**: Pi-hole v6 requires root privileges for this command
- **pihole**: the Pi-hole command-line utility for managing your Pi-hole installation
- **setpassword**: sets a new password for the web interface and API access

### Refresh Gravity

This is the final step in our terminal before fully switching over to the web interface. You may have noticed this message at the start of the Pi-hole installation:

```
[i] Script called with non-root privileges
```

This is not an error. It only means we did not start the installer with `sudo`. The installer notices, restarts itself with `sudo`, and builds the gravity database (where the blocklist lives) on its own. Nothing is broken and nothing needs fixing.

The command below is still worth knowing. It rebuilds gravity by hand, which you will want after adding a blocklist, and running it now is a quick way to confirm that your lists download correctly.

```bash
sudo pihole -g
```

### Command Breakdown

- **sudo**: runs the command with superuser privileges (you remember this one, right?)
- **pihole -g**: retrieves subscribed blocklists and consolidates them into one unique list for the built-in DNS server to use

After this finishes, let's head over to the Pi-hole web interface and confirm that we can log in. Once done, head to the next section.

## More Router Configuration

After logging into your Pi-hole web interface, you may have noticed that nothing is happening. This is totally normal. The Pi-hole is completely ready to use, but we have not configured it as our network's DNS server yet. To do so, head back to your router's web interface.

Again, everyone's hardware is different, but I will provide my process as a loose guide for completing this step.

### Setting Pi-hole as Your DNS

Back in my router's web interface, I again navigated to `Advanced → Network Settings`. This time however, I went to `Network Connections → Network Connection Broadband Settings`. Here I scrolled until I found `IPv4 DNS`. By default, this uses your ISP's DNS servers. Set this to `Use the Following IPv4 DNS Addresses`. In the first slot, enter the IP address of your Pi-hole. The second slot can be left blank or set to `0.0.0.0`. **Make sure to apply these changes.**

**A note on IPv6**: If your ISP gives you IPv6, your router may also hand out an IPv6 DNS server to your devices (usually itself or your ISP's). Devices that use it skip Pi-hole completely, and ads come back on some of them. If blocking looks patchy, look for an IPv6 DNS setting in your router and either turn off the advertised IPv6 DNS server or point it at your Pi-hole's IPv6 address.

### Ads Be Gone

Head back over to your Pi-hole web interface and you should now see some queries being requested and hopefully blocked. To test this, head over to any website that you know has a lot of ads. News sites are good, but I used [Speed Test](https://speedtest.net) to test this.

If all has gone to plan, this site should be ad-free. It is fun to toggle Pi-hole on and off as your DNS in your router interface to see the difference between sites.

At this point, you have fully installed and configured Pi-hole for network-wide ad blocking. If you are happy with this setup, you can skip the rest of the steps in this guide.

## Server Maintenance

The first extra step is keeping the server up to date. It is tempting to schedule everything (Pi-hole updates, a full system upgrade, a weekly reboot) and forget about it. I would not. This Pi is the only DNS server on your network, so if an unattended update breaks it or a reboot does not come back up, every device in the house loses internet until you fix it. The safer setup is to let the system install security patches on its own and to do everything else by hand, when you are around to see it work.

### Automatic Security Updates

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

### Command Breakdown

- **sudo**: c'mon we should know this by now
- **apt install unattended-upgrades**: installs the package that downloads and installs security updates in the background
- **dpkg-reconfigure -plow unattended-upgrades**: opens a prompt asking if updates should be installed automatically. Select `Yes`.

Out of the box this only covers Debian's own repositories. Anything that comes from the Raspberry Pi repository (the kernel and firmware, for example) still needs the manual `sudo apt update && sudo apt upgrade` from earlier, so run that every so often too. To see what the automatic run would do without changing anything:

```bash
sudo unattended-upgrade --dry-run --debug
```

### Updating Pi-hole

Pi-hole is updated by hand. Read the release notes first (the Pi-hole team asks you to, and major versions do change things), then run:

```bash
sudo pihole -up
```

Pick a time when you can check afterwards that the web interface loads and ads are still blocked. Pi-hole already refreshes its blocklists once a week on its own, so there is nothing to schedule for that.

### Rebooting

There is no need for a scheduled reboot. Reboot by hand with `sudo reboot` after a kernel update, and make sure DNS is working again before you walk away.

## Troubleshooting

Not every install goes to plan. I had a few things break for me, so I wanted to highlight them here just in case you run into some of the same issues.

### Issue #1: Raspberry Pi Imager

During my initial attempts I was using the Raspberry Pi Imager on Ubuntu Linux. I ran into an issue where the MicroSD card would be flashed with the OS, but no configuration settings had been saved. This meant I could not SSH in. I was unable to resolve this issue and had to move over to a Windows system for this step.

### Issue #2: Query Rate Limiting

My home network has a lot of devices, so I ran into some issues with the Pi-hole getting rate limited, causing network problems. This configuration setting can be changed in a few places: from the command line, in Pi-hole's configuration file, or in the web interface (`Settings → DNS`, in the `Rate-limiting` box, which only shows once the settings level is switched to Expert).

I will show you how to change it from the command line, which is what the Pi-hole team prefers since it checks your input for mistakes. The default rate limit is 1000 queries per 60 seconds, per client. Run the command with only the setting name to see the current value:

```bash
sudo pihole-FTL --config dns.rateLimit.count
```

Add a number to set a new one:

```bash
sudo pihole-FTL --config dns.rateLimit.count 5000
```

Increasing the count to 5000 solved the issues for me, but every network is different. If you want to disable rate limiting entirely, set both `dns.rateLimit.count` and `dns.rateLimit.interval` to `0`.

Pi-hole v6 stores this in `/etc/pihole/pihole.toml` (the old `/etc/pihole/pihole-FTL.conf` and its `RATE_LIMIT` line are gone). If you would rather edit the file (you need sudo privileges), find the existing `[dns.rateLimit]` section and change the values there. Do not add a second copy of the section:

```toml
[dns.rateLimit]
  count = 5000
  interval = 60
```

Either way, Pi-hole watches this file and picks up the change on its own. The old `pihole restartdns` command no longer exists in v6. If you ever do want to restart Pi-hole completely, use:

```bash
sudo systemctl restart pihole-FTL
```

### Issue #3: Private Relay

If you have Apple devices on your network, they may be using something called Private Relay. This is an Apple privacy feature that encrypts and routes DNS queries through Apple's servers, which is incompatible with Pi-hole's DNS filtering.

If you want Pi-hole to work for your Apple devices, on each device navigate to `Settings → Apple Account → iCloud → Private Relay` and disable it.

I am not making any recommendations as to which service is more secure or useful. This is just the steps one could follow if they decided for themselves that Pi-hole fits their specific needs better.

## Conclusion

Thanks for taking the time to read my guide. I hope this has helped you get your network ad-free. I am continuing to work on new projects, so stay tuned for more tutorials!
