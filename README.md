# WiiStream Installation Guide

## Introduction
This guide details the steps for installing WiiStream on your Nintendo Wii. Follow each step closely for a successful setup.

## Prerequisites
- A Nintendo Wii matching one of the model numbers [here] (https://mariokartwii.com/showthread.php?tid=17)
- An SD card
- A PC with internet access
- A USB keyboard

## Installation Steps

### 1. Homebrew Your Wii
- Follow the steps outlined in these guides to homebrew your Wii:
  - [Letterbomb Guide](https://wii.guide/letterbomb)
  - [Homebrew Channel Installation](https://wii.guide/hbc) (Install BootMii as boot2. Note: If 'install as boot2' is not available, your Wii is incompatible)
  - [BootMii Guide](https://wii.guide/bootmii)

### 2. Prepare the SD Card
- Download Wii-Linux-ngx from [this link](https://github.com/neagix/wii-linux-ngx/releases/download/0.3.6/wii-jessie-sd.img.xz) and extract the file.
- Insert the SD card into your PC.

#### On Windows:
- Open Rufus from [here](https://rufus.ie/en/). 
  - Select your SD card under 'Device'.
  - Under 'Boot selection', click 'SELECT' and choose the extracted "wii-jessie-sd.img" file.
  - Click 'START' and accept the warning prompt.
  - Click 'CLOSE' when it shows 'READY'.
- Open Paragon Partition Manager.
  - Select the partition named "WII-LINUX-NGX" (usually the middle partition).
  - Click 'Expand', follow the prompts, and wait for completion.
  - Ignore the prompt about 'reinit your bootloader'.

### 3. Wii Setup
- Connect a USB keyboard to the back of the Wii, using the port at the bottom when the Wii is horizontal.
- Insert the SD card into the Wii and turn it on.
- Log in with 'root' as both the username and password.

### 4. Configure Network on Wii
- At the prompt, enter `./whiite-ez-wifi-config` and follow the instructions for WiFi setup.
- Edit network interfaces with `nano '/etc/network/interfaces'`.
  - Change 'wlan0' to 'wlan1'.
  - Save and exit (Ctrl-X, Y, Enter).
- Activate the new interface with `ifup wlan1`, then type `reboot`.

### 5. Remote Access Setup
- Modify SSH settings with `nano /etc/ssh/sshd_config`.
  - Change 'PermitRootLogin without-password' to 'PermitRootLogin yes'.
- Reboot the Wii, then log back in.
- Use `ifconfig` to find the Wii's IP address (noted under 'wlan1 inet addr').

### 6. Access Wii from Another Computer
- On another computer, access the Wii via terminal: `ssh root@<Wii's IP address>`.
  - The password is 'root' (input is invisible).

### 7. Create Swap File
- Execute the following commands to create a swap file: