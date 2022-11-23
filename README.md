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
