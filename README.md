# LampEye

An unattended Raspberry PI Zero W with webcam inside a table lamp.

## Overview
Remotely connect via web to this table lamp webcam, equipped with an audio class D amplifier, a loudspeaker and a microphone to:

- view the USB camera
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
### 1. Install "Raspbian Buster Lite", I tested:
   - Stretch "	2018-11-13-raspbian-stretch.zip" downloaded and with "installation guide" at [Download Raspbian Stretch](http://downloads.raspberrypi.org/raspbian/images/raspbian-2018-11-15/)
   - DON'T DON'T DON'T install any newer version of the Raspian Kernel than this (Linux 4.14). Otherwise IR infrared transmitter won't work! Never do "sudo apt-get upgrade"
### 3. (Optional, if you don't have screen, keyboard and mouse) Prepare the SD you just created for headless operations following these instructions. See also [Raspbian Stretch Headless Setup Procedure](https://www.raspberrypi.org/forums/viewtopic.php?t=191252) 

# Install the USB camera and microphone

# Install gstreamer package

Install the gstreamer package. For details see also [Streaming Video Using gstreamer](https://raspberry-projects.com/pi/pi-hardware/raspberry-pi-camera/streaming-video-using-gstreamer) 
You need to edit the sources.list file so enter:
```
sudo nano /etc/apt/sources.list
```

```
and add the following to the end of the file:
```
deb http://vontaene.de/raspbian-updates/ . main
```
Press CTRL+X to save and exit
Now run an update (which will make use of the line just added):
```
sudo apt-get update 
```

Now install gstreamer
```
sudo apt-get install gstreamer1.0
``` 

### 4. Test uv4l-server 
Navigate to [http://localhost:8090](http://localhost:8090)
This page should appear:
[![](https://github.com/guido57/IReye/blob/master/screenshots/UV4L%20Streaming%20Server%20Home%20Page.PNG)](https://github.com/guido57/IReye/blob/master/screenshots/UV4L%20Streaming%20Server%20Home%20Page.PNG)

### 5. Click on WEBRTC and view your camera
This page should appear:
[![](https://github.com/guido57/IReye/blob/master/screenshots/UV4L%20Streaming%20Server%20-%20Web%20RTC.PNG)](htt### ps://github.com/guido57/IReye/blob/master/screenshots/UV4L%20Streaming%20Server%20-%20Web%20RTC.PNG)
 
### 6. Clicking on Call (the green button) the image of your camera should appear on the "remote" rectangle

#  Add autosigned certificate for uv4l https web server
For Google Chrome and other recent browsers versions is mandatory to use https insead of http.

### 0. Generate a selfsigned certificate and use it in uv4l web server
``` 
sudo openssl genrsa -out selfsign.key 2048 && sudo openssl req -new -x509 -key selfsign.key -out selfsign.crt -sha256
``` 
### 1. Move the certificate to the proper folder
``` 
$ sudo mv selfsign.* /etc/uv4l/
``` 
### 2. Add the certificate to uv4l changing the following text in /etc/uv4l/uv4l-uvc.conf
```
#HTTPS options:
server-option = --use-ssl=yes
server-option = --ssl-private-key-file=/etc/uv4l/selfsign.key
server-option = --ssl-certificate-file=/etc/uv4l/selfsign.crt
```
### 3. Set OPENSSL_CONF in Raspbian Buster only!
In Raspian Buster only it looks like the environment OPENSSL_CONF='/etc/ssl/' is not set.
For this reason, uv4l works fine with http but it doesn't with https. Therefore add the line 
```
Environment="OPENSSL_CONF='/etc/ssl/'"
```
to /etc/systemd/system/uv4l_uvc@.service in this way:
```
[Unit]
Description=UV4L UVC driver

[Service]
Type=simple
Environment="OPENSSL_CONF='/etc/ssl/'" # <------------ this line added
ExecStart=/etc/init.d/uv4l_uvc add %I
ExecStop=/etc/init.d/uv4l_uvc remove %I
RemainAfterExit=yes
KillMode=none
```

### 4. After rebooting, verify that[https://localhost:8090](https://localhost:8090) is accessible by your Raspberry PI browser

# Enable your USB microphone in uv4l
### 1. Change (or create) your ALSA config file /home/pi/.asoundrc
```
pcm.!default {
  type asym
  capture.pcm "mic"
  playback.pcm "speaker"
}
pcm.mic {
  type plug
  slave {
    pcm "hw:1,0"
  }
}
pcm.speaker {
  type plug
  slave {
    pcm "hw:0,0"
  }
}
```
### 2. after rebooting your USB microphone will be the default one and your RPI speaker (the headphone jack) will be the default speaker.
### 3. list the available PCM capture devices end find the line of the proper configuration:
```
arecord --list-pcms
```

my working result is: 
```
hw:CARD=C525,DEV=0
    HD Webcam C525, USB Audio
    Direct hardware device without any conversions
```
which is at the 11th position (starting from 0) in the list.
Now uncomment and set the following line in /etc/uv4l/uv4l-uvc.conf
```
server-option = --webrtc-recdevice-index=11
```
After rebooting, even the USB microphone should work when uv4l web server is called at: [https://your_raspberry-PI_IP_address:8090/stream/rtc/](https://your_raspberry-PI_IP_address:8090/stream/rtc/)


# Install and configure lirc library (Infrared transmitter and receiver)

### 1. install lirc
```
sudo apt-get install lirc
```
### 2. Add the following lines to /etc/modules file
In my Raspberry PI 3 circuit, the infrared receiver is connected at GPIO 23 (connector pin 16) while the infrared LED transmittter at GPIO 22 (connector pin 15). Change them accordingly to your circuit.
```
lirc_dev
lirc_rpi gpio_in_pin=23 gpio_out_pin=22
```

### 3. Add the following lines to /etc/lirc/hardware.conf file
```
LIRCD_ARGS="--uinput --listen"
LOAD_MODULES=true
DRIVER="default"
DEVICE="/dev/lirc0"
MODULES="lirc_rpi"
```


### 4. Update the following line in /boot/config.txt
See point 2 for the correct pin settings.
```
dtoverlay=lirc-rpi,gpio_in_pin=23,gpio_out_pin=22
```
### 5. Update the following lines in /etc/lirc/lirc_options.conf
```
driver    = default
device    = /dev/lirc0
```
### 6. Restart the lirc daemon and check its status 
```
sudo /etc/init.d/lircd stop
sudo /etc/init.d/lircd start
sudo /etc/init.d/lircd status
```
then reboot
```
sudo reboot
```

### 7. Test lirc recorder
To test if lirc driver is working, firstly stop the lirc deamon:
```
sudo /etc/init.d/lircd stop
```
then start a continuos IR receiver:
```
mode2 -d /dev/lirc0
```
now try to press a key of any infrared remote control in front of the IR LED receiver and you should see multiple lines like below
```
pulse 560
space 1706
pulse 535
```
# Install Flask, Flask-BasicAuth, python IRsend.py module and WebRtcIR.html web page
### 1. Install Flask and Flask-BasicAuth 
```
sudo pip install Flask 
sudo pip install Flask-BasicAuth
```
### 2. Copy IRsend.py into /usr/share/uv4l/ 
### 3. Copy WebRtcIR.html into /usr/share/uv4l/templates/

# Create a IRsend systemd service
### 1. Copy IRsend.service into /etc/systemd/system/
### 2. Create an empty log file in /usr/share/uv4l/www/ and assign it user permissions 
```
sudo touch IRsend.log
```
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
