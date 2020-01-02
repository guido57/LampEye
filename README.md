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
[![](https://github.com/guido57/IReye/blob/master/screenshots/IReye%20logic%20diagram.jpg)](https://github.com/guido57/IReye/blob/master/screenshots/IReye%20logic%20diagram.jpg)

# Hardware Components
### 0. [Raspberry PI Zero W]
### 1. [ Camera Module for Raspberry Pi Zero W 3 Model B B+ A+ 2 1 5MP 1080p OV5647 Sensor HD Video Webcam Night](https://www.amazon.it/gp/product/B0759NWTGC/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
### 1. DAC with PCM5102[PCM5102 I2S IIS High Quality Lossless Digital Audio DAC](https://www.amazon.it/gp/product/B07QBY5Y9K/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1)
### 2. Audio Ampilfier. I Chose a PAM8403 5V 3W Class D Audio Ampifier like [this](https://www.amazon.it/Muzoct-PAM8403-Digital-Amplificatore-Classe/dp/B07791Z9WC/ref=sr_1_14?keywords=pam8403&qid=1565554203&s=gateway&sr=8-14)
### 3. Amplified microphone
### 4. USB Analog to Digital Converter (ADC) 

# Prepare your Raspberry
### 0. Start from a clean sd: I tested 8M and 32M SD Samsung cards.
### 1. Download and install [Raspbian Buster Lite](https://www.raspberrypi.org/downloads/raspbian/) following the official instructions
### 3. (Optional, if you don't have screen, keyboard and mouse) Prepare the SD you just created for headless operations following these instructions. See also [Raspbian Stretch Headless Setup Procedure](https://www.raspberrypi.org/forums/viewtopic.php?t=191252) 

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
```
the package gstreamer1.0-plugins-good includes v42lsrc 
the package gstreamer1.0-plugins-bad includes h264parse
the package gstreamer1.0-omx includes omxh264enc
the package gstreamer1.0-alsa includes alsasrc

### 4. Test gstreamer 


### 5. Enable i2s and DAC audio output

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

# Create a gst2janus systemd service
### 1. Copy gst2janus.service into /etc/systemd/system/


### 3. Assign ownership to the standard user "pi" 
```
sudo chown pi: IRsend.log
```
### 4. Start the IRsend service 
```
sudo systemctl start IRsend
```
### 5. Check if IRsend service started correctly.

You should see something like this:
```
pi@raspberrypi:/usr/share/uv4l/www $ sudo systemctl status IRsend
● IRsend.service - Flask web app to send IR commands to a TV
   Loaded: loaded (/etc/systemd/system/IRsend.service; disabled; vendor preset: enabled)
   Active: active (running) since Fri 2019-08-09 21:24:07 BST; 7ms ago
 Main PID: 6556 (python)
   CGroup: /system.slice/IRsend.service
           └─6556 /usr/bin/python IRsend.py > IRsend.log 2>&1

Aug 09 21:24:07 raspberrypi systemd[1]: Started Flask web app to send IR commands to a TV.
```
### 6. Now enable IRsend service, so that it will start automatically on boot
```
sudo systemctl stop IRsend
sudo systemctl enable IRsend
```
# Test the WebRtcIR.html website
Navigate to: https://localhost/webrtcir.html:8092


# Activate a WIFI hotspot (access point) - OPTIONAL
In this way your IReye can be reached using VNC or SSH and properly configured.

Follow this instructions [Set a Raspberry WIFI hotspot (access point) and client](https://github.com/guido57/Raspberry-WIFI-hotspot)
 
# Infrared TX & RX schematic
[![](https://github.com/guido57/IReye/blob/master/screenshots/IR_TX_RX.PNG)](https://github.com/guido57/IReye/blob/master/screenshots/IR_TX_RX.PNG)

# Screenshots

# Logic Diagram 
