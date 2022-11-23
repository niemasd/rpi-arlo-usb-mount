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
To enable mass storage device mode, we need to create a large file to act as the storage medium on the SD card. This file will emulate the USB flash drive that the TV sees.

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

### Step 3.3.2: Add Mount Folder to `fstab`
Next, we need to add the mount folder to the configuration file that records our available disk partitions, which will allow the USB file system to be error-checked and mounted automatically upon boot. Add the following to the bottom of `/etc/fstab` (need to edit as `sudo`):

```
/piusb.bin /mnt/usb_share vfat users,umask=000 0 2
```

### Step 3.3.3: Reload `fstab` to Mount
Instead of rebooting, we can manually reload `fstab` with the command below:

```bash
sudo mount -a
```

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
