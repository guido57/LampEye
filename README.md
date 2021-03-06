# LampEye

An unattended Raspberry PI Zero W with webcam inside a table lamp.

## Overview
Remotely connect via web to this table lamp webcam, equipped with a loudspeaker and a microphone to:

- view the camera
- listen from the USB microphone 
- speak to the people standing around it

# Libraries
- gstreamer to stream to a janus server via RTP
- See the logic diagram below also.

# Logic diagram
[![](https://github.com/guido57/LampEye/blob/master/screenshots/LampEye.png)](https://github.com/guido57/LampEye/blob/master/screenshots/LampEye.png)

# Hardware Components
0. [Raspberry PI Zero W]
1. [ Camera Module for Raspberry Pi Zero W 3 Model B B+ A+ 2 1 5MP 1080p OV5647 Sensor HD Video Webcam Night](https://www.amazon.it/gp/product/B0759NWTGC/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
2. DAC with PCM5102[PCM5102 I2S IIS High Quality Lossless Digital Audio DAC](https://www.amazon.it/gp/product/B07QBY5Y9K/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)
3. Audio Ampilfier. I Chose a PAM8403 5V 3W Class D Audio Ampifier like [this](https://www.amazon.it/Muzoct-PAM8403-Digital-Amplificatore-Classe/dp/B07791Z9WC/ref=sr_1_14?keywords=pam8403&qid=1565554203&s=gateway&sr=8-14)
4. Amplified microphone
5. USB Analog to Digital Converter (ADC) 

# Prepare your Raspberry
0. Start from a clean sd: I tested 8M and 32M SD Samsung cards.
1. Download and install [Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/) following the official instructions
2. (Optional, if you don't have screen, keyboard and mouse) Prepare the SD you just created for headless operations following these instructions. See also [Raspbian Stretch Headless Setup Procedure](https://www.raspberrypi.org/forums/viewtopic.php?t=191252) 

# Install the PI camera
After connecting the PI camera to the Raspberry PI Zero W using the flex cable, enable the camera by raspi-config
```
sudo raspi-config
```
go to "Interfacing Options" / "Camera" / "Yes"
then raspi-config automatically reboot


# Install the USB microphone
Plug your USB microphone to the Raspberry PI Zero W
check if it is visible to the operating system entering
```
arecord -l
```
you should see something like this:
```
**** List of CAPTURE Hardware Devices ****
card 1: AK5371 [AK5371], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```
in this case,  my AK5371 USB microphone is on card 1 

therefore in gst2janus command you'll set 
```
... alsasrc device=hw:1 ! ...
```
# Install the gstreamer packages

Install the gstreamer package. For details see also [Streaming Video Using gstreamer](https://raspberry-projects.com/pi/pi-hardware/raspberry-pi-camera/streaming-video-using-gstreamer) 
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install gstreamer1.0-tools
```

```
sudo apt-get install gstreamer1.0-plugins-good
sudo apt-get install gstreamer1.0-plugins-bad
sudo apt-get install gstreamer1.0-omx
sudo apt-get install gstreamer1.0-alsa
sudo apt-get install libgstreamer1.0-dev
```
the package gstreamer1.0-plugins-good includes v42lsrc

the package gstreamer1.0-plugins-bad includes h264parse

the package gstreamer1.0-omx includes omxh264enc

the package gstreamer1.0-alsa includes alsasrc

the package libgstreamer1.0-dev contains .h .so and all the useful things to compile gstreamer C code 

### 4. Test gstreamer 


# Enable i2s and DAC audio output

Add these lines at the end of /boot/config.txt
```
# enable audio via I2S to PCM5102 PCM5122
dtoverlay=hifiberry-dac
dtoverlay=i2s-mmap
```
and comment the line dtaparam=audio=on

```
# dtaparam=audio=on
```
# Setup a Janus Gateway instance
You have two main options:
1. install Janus in your private LAN, in one of your PC's inside your LAN
2. install Janus on a public server, on Internet. See here my Aruba installation [https://github.com/guido57/Aruba-Janus](https://github.com/guido57/Aruba-Janus)

Choose 1. if all your clients (PC's accessing LampEye) are inside the LAN itself. A client on Internet could access it, but you have to open ports in your router and setup a dynamic DNS
Choose 2. otherwise. 

# Create a gst2janus systemd service
This step is necessary to have the streaming automatically start on Raspberry reboot

1. Copy gst2janus.service into /etc/systemd/system/
2. Start the gst2janus service 
```
sudo systemctl start gst2janus
```
3. Check if gst2janus service started correctly.

You should see something like this:
```
● gst2janus.service - streaming audio and video to janus gateway server via RTP
   Loaded: loaded (/etc/systemd/system/gst2janus.service; disabled; vendor preset: enabled)
   Active: active (running) since Thu 2020-01-02 10:19:57 GMT; 415ms ago
 Main PID: 1599 (gst2janus)
   Memory: 708.0K
   CGroup: /system.slice/gst2janus.service
           ├─1599 /bin/sh /home/pi/gst2janus/gst2janus
           └─1601 gst-launch-1.0 -v v4l2src device=/dev/video0 ! video/x-raw,width=320,height=240,framerate=4/1 !

Jan 02 10:19:57 raspberrypi systemd[1]: Started streaming audio and video to janus gateway server via RTP.
```
4. Now enable gstjanus service, so that it will start automatically on boot
```
sudo systemctl enable gst2janus
```
5. Check if your janus gateway is receiving the audio and video RTP streams


# Activate a WIFI hotspot (access point) - OPTIONAL
In this way your LampEye can be reached using VNC or SSH and properly configured.

Follow this instructions [Set a Raspberry WIFI hotspot (access point) and client](https://github.com/guido57/Raspberry-WIFI-hotspot)
 
# Activate a monitoring system - OPTIONAL
Using a few colored LED's you can have a useful diagnostic system

### LED's table

| LED color | Description | 
| --- | --- |
| green | microphone audio level |
| blue  | WIFI signal strength   |
| yellow | CPU temperature |
| white | janus streaming on line|

### Installing wiringPI (python and C libraries)
```
sudo apt-get install wiringpi
```
### Activate the monitoring.py python program



# Screenshots

