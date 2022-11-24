This was all done using a Raspberry Pi Zero. Much of this guide was pulled from the following resources:
* https://magpi.raspberrypi.com/articles/pi-zero-w-smart-usb-flash-drive

# Step 1: Enable USB Driver
All of the following tasks are performed while SSH'd into the Raspberry Pi.

## Step 1.1: Edit `/boot/config.txt`
Add the following to the bottom of `/boot/config.txt` (need to edit as `sudo`):

```
dtoverlay=dwc2
```

## Step 1.2: Edit `/etc/modules`
Add the following to the bottom of `/etc/modules` (need to edit as `sudo`):

```
dwc2
```

## Step 1.3: Halt
Run the following to halt the Raspberry Pi:

```
sudo halt
```

# Step 2: Plug Into Base Station
Now, we can power the Raspberry Pi using its USB port that's marked `USB` (rather than the one marked `PWR IN`). Here's a summary of the trade-offs:

> You could plug the TV into the Pi Zero W USB port, not the PWR IN port, using a standard micro USB cable. The cable will both supply power from the TV and make the USB data connection. The disadvantage is that the TV must be switched on to supply power to the Pi. When someone turns the TV off with the remote, the Pi will also lose power, which can corrupt your SD card.

> Alternatively, you can connect a separate, always-on power supply to the PWR IN port, and use a slightly modified micro USB cable to connect the TV to the USB port. The modification is to cut the red wire inside the micro USB cable. This protects the Pi from damage that could be caused by drawing power from two different power sources. The advantage of this method is that the Pi is powered independently from the TV. It will be available on the network even if the TV is off, and there is a reduced risk of sudden power loss and SD card corruption.

The Arlo Base Station will always be on, so powering the Raspberry Pi via the USB port it's plugged into will be fine. Once plugged in and powered on, we can SSH back into the Raspberry Pi.

# Step 3: Create a Container File
To enable mass storage device mode, we need to create a large file to act as the storage medium on the SD card. This file will emulate the USB flash drive that the TV sees. The following instructions will make the virtual USB storage read-only from the Raspberry Pi's perspective, but write-able from the device it's plugged into.

## Step 3.1: Create Empty File
The command below will create an empty 16 GB binary file (change the `count=16384` argument if you want a different size). Note that this will be limited by the available free space on your SD card (check the `Avail` column in `df -h`), and it may take a few minutes to complete the setup:

```bash
sudo dd bs=1M if=/dev/zero of=/piusb.bin count=16384
```

## Step 3.2: Format as FAT32
The command below will format the empty binary file created in [Step 3.1](#step-31-create-empty-file) into the FAT32 format:

```bash
sudo mkdosfs /piusb.bin -F 32 -I
```

## Step 3.3: Mount the Container File
### Step 3.3.1: Create Mount Folder
First, we create a folder on which we can mount the file system:

```bash
sudo mkdir /mnt/usb_share
```

### Step 3.3.2: Mount Using `kpartx`
I originally tried using `mount` to get it to mount, but it didn't work properly (probably starting offset issue?), but `kpartx` was able to do it fine:

```bash
sudo kpartx -a /piusb.bin
sudo mount /dev/mapper/loop0p1 /mnt/usb_share
```

I added these `.bashrc` so they run on boot, and it worked fine.

# Step 4: Enable/Disable USB Mass Storage Device Mode
We can now enable/disable USB mass storage device mode. You can also create an `alias` for each of the following commands for convenience.

## Step 4.1: Enable
The command below will **enable** USB mass storage device mode:

```bash
sudo modprobe g_mass_storage file=/piusb.bin stall=0 ro=0 removable=1
```

## Step 4.2: Disable
The command below will **disable** USB mass storage device mode:

```bash
sudo modprobe -r g_mass_storage
```

# Step 5: Format USB in Arlo App
The Arlo Base Station should now be able to see the Raspberry Pi's virtual disk as a USB device:
1. Open the Arlo app
2. Click "SmartHub / Base Station / Bridge"
3. Click on the Base Station the Raspberry Pi is plugged into
4. Click "Storage Settings"
5. Click "USB Device 1"
6. Click "FORMAT USB FLASH DRIVE" to format for use by your Arlo Base Station

Once this is complete, the Arlo Base Station should now be able to use the virtual disk in the Raspberry Pi.

# General Comments
## Refreshing Mounted Directory in Raspberry Pi
In Raspberry Pi, the mounted folder won't automatically refresh as files are added. Thus, you need to periodically unmount and remount the folder:

```bash
sudo umount /mnt/usb_share
sudo mount /dev/mapper/loop0p1 /mnt/usb_share
```

I created an `alias` called `usb_remount` that peforms both of these. I also added a `cron` job to do this every minute by adding the following to `root`'s `crontab` (`sudo crontab -e`):

```
* * * * * source /home/niema/.bash_alias ; umount /mnt/usb_share && mount /dev/mapper/loop0p1 /mnt/usb_share
```

## Automatic Upload to YouTube
THIS DIDN'T WORK

I installed [youtube-upload](https://github.com/tokland/youtube-upload) as follows:

```bash
sudo -H pip3 install --upgrade --no-cache-dir google-api-python-client oauth2client progressbar2
sudo -H pip3 install git+https://github.com/tokland/youtube-upload.git
```

I then had to create a `~/.client_secrets.json` file from the Google Console:
1. Go to: https://console.developers.google.com/
2. Click "New Project" or "Create Project" or similar
3. Name it something meaningful (e.g. "Arlo Video Upload"), create it, and select it
4. In the left navigation menu (the 3 lines), click "APIs & Services"
5. In the left menu, click on "Enabled APIs & servies"
6. Click "+ ENABLE APIS AND SERVICES"
7. Search for "YouTube" and enable all of them
8. In the left navigation menu (the 3 lines), click "APIs & Services"
9. In the left menu, click on "Credentials"
10. Click "+ CREATE CREDENTIALS", then "OAuth client ID"
11. Click "CONFIGURE CONSENT SCREEN" and configure it (if needed)
  1. For "App name", I put "youtube-upload"
  2. For "Scopes", I picked all the YouTube APIs
  3. Then I returned back to the "OAuth client ID" selection in "+ CREATE CREDENTIALS" to continue
13. For "Application type", I picked "Desktop app"
14. For "Name", I put "youtube-upload" and then clicked "Create"
15. I then clicked "DOWNLOAD JSON", which is the `~/.client_secrets.json` file that will be used by youtube-upload

I then wrote the following Python script that uploads all video files in a given directory to YouTube:

```python
TODO
```

# Brainstorming Next Steps
* Upload videos directly to YouTube via [youtube-upload](https://github.com/tokland/youtube-upload)?
  * Maybe automatically upon creation via [`inotifywait`](https://unix.stackexchange.com/a/323919/244551)?
