# WiiStream Installation Guide

## Introduction
This guide details the steps for installing WiiStream on your Nintendo Wii. Follow each step closely for a successful setup.

## Prerequisites
- A Nintendo Wii matching one of the model numbers [here](https://mariokartwii.com/showthread.php?tid=17)
- An SD card
- A PC with internet access (here I'll be using Windows but macOS and Linux will work too)
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
- Open Rufus [available here](https://rufus.ie/en/). 
  - Select your SD card under 'Device'.
  - Under 'Boot selection', click 'SELECT' and choose the extracted "wii-jessie-sd.img" file.
  - Click 'START' and accept the warning prompt.
  - Click 'CLOSE' when it shows 'READY'.
- Open Paragon Partition Manager.
  - Select the partition named "WII-LINUX-NGX" (usually the middle partition).
  - Click 'Expand', follow the prompts, select 'Expand Now', and wait for completion.
  - Ignore the prompt about 'reinit your bootloader'.

### 3. Wii Setup
- Connect a USB keyboard to the back of the Wii, using the USB port at the bottom when the Wii is horizontal.
- Insert the SD card into the Wii and turn it on.
- Log in with 'root' as both the username and password.

### 4. Configure Network on Wii
- At the prompt, enter `./whiite-ez-wifi-config` and follow the instructions for WiFi setup.
- Edit network interfaces with `nano '/etc/network/interfaces'`.
  - Change 'wlan0' to 'wlan1'.
  - Save and exit (Ctrl-X, Y, Enter).
- Activate the new interface by typing `ifup wlan1`, then type `reboot`.

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
  ```
  dd if=/dev/zero of=/swapfile bs=1M count=5120
  chmod 600 /swapfile
  mkswap /swapfile
  swapon /swapfile
  nano /etc/fstab
  ```
  - Add `/swapfile none swap sw 0 0` to the file and save.

### 8. Install Mopidy and Dependencies
- Run `apt update`.
- Install required packages:
  ```
  apt-get install python curl python-dev python-pip gstreamer0.10-alsa gstreamer0.10-pulseaudio python-gst0.10 gstreamer0.10-plugins-good gstreamer0.10-plugins-ugly gstreamer0.10-tools pulseaudio python-pykka python-gi
  ```
- Install Mopidy and related components:
  ```
  curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
  python get-pip.py
  pip install Mopidy==0.19.0 Mopidy-MusicBox-Webclient==2.0.0 Mopidy-TuneIn==0.1.3 Mopidy-Podcast-iTunes==1.0.0 --no-deps
  ```

### 9. Configure Mopidy
- Create and configure the Mopidy service file with `nano /etc/init.d/c_mopidy`.
  - Paste the provided script, save and exit.
  ```
  [local]
  enabled = true
  data_dir = /var/lib/mopidy/local
  media_dir = /var/lib/mopidy/media
  playlists_dir = /var/lib/mopidy/playlists
  
  [audio]
  output = pulsesink server=127.0.0.1
  
  [stream]
  enabled = true
  protocols =
      http
      https
      mms
      rtmp
      rtmps
      rtsp
  timeout = 5000
  metadata_blacklist =
  
  [mpd]
  enabled = true
  hostname = 0.0.0.0
  port = 6600
  
  [http]
  enabled = true
  hostname = 0.0.0.0
  port = 6680
  [tunein]
  timeout = 5000
  
  [podcast-itunes]
  enabled = true
  # iTunes Store base URL
  base_url = http://itunes.apple.com/
  
  # user-friendly name for browsing the iTunes Store
  root_name = iTunes Store
  
  # format string for genre results; field names: id, name, url
  genre_format = {name}
  
  # format string for podcast results; field names: collectionId,
  # collectionName, country, kind, trackCount
  podcast_format = {collectionName}
  
  # format string for episode results; field names: collectionId,
  # collectionName, country, episodeContentType, episodeFileExtension,
  # episodeGuid, kind, releaseDate, trackId, trackName
  episode_format = {trackName} [{collectionName}]
  
  # charts to display when browsing; possible values: "Podcasts",
  # "AudioPodcasts", "VideoPodcasts"
  charts = AudioPodcasts
  
  # directory name to display for browsing charts of a genre/category
  # with subgenres; field names as for 'genre_format'
  charts_format = All {name}
  
  # ISO country code for the iTunes Store you want to use
  country = US
  
  # whether you want to include explicit content in your search
  # results; possible values: "Yes", "No", or store default (blank)
  explicit = 
  
  # HTTP request timeout in seconds
  timeout = 10
  ```
  - Make it executable with `chmod +x /etc/init.d/c_mopidy` and enable defaults with `update-rc.d c_mopidy defaults`.
- Create the 'mopidy' user and configure permissions:
  ```
  adduser mopidy
  usermod -a -G audio mopidy
  usermod -a -G audio pulse
  ```
- Configure PulseAudio:
  - Edit `nano /etc/pulse/system.pa`.
  - Add `load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1` at the end.

### 10. Final Steps
- Create Mopidy configuration directory and file:
  ```
  mkdir /etc/mopidy
  nano /etc/mopidy/mopidy.conf
  ```
  - Paste the provided configuration, save and exit.
- Modify system behavior for shutdown in `nano /etc/inittab`.
  - Change the line starting with 'ca:12345:ctrlaltdel' as instructed.
- Power off the Wii to apply changes.

## Starting Mopidy
Power on the Wii to start the Mopidy music server.
Connect to Mopidy in a browser on another device via the Wii's IP address over port 6680, e.g. http://192.168.30.10:6680
