# DSLR Timelapse gphoto RaspberryPi
Controlling a DSLR with the gphoto library (RaspberryPi) for timelapse shots

### Prerequisites
+ DSLR Camera ([Is my DSLR supported by the gphoto library?](http://gphoto.org/proj/libgphoto2/support.php))
+ RaspberryPi
+ USB Cable (Connection between Raspberry and DSLR)

### Installation gphoto library

#####1. Install gphoto prerequisites
Connect to your raspberryPi via SSH and install the following librarys, required to install gphoto.
```bash
sudo apt-get install libpopt-dev -y
sudo apt-get install libltdl-dev -y
sudo apt-get install libusb-dev -y
sudo apt-get install libusb-1.0-0-dev -y
```

#####2. Download gphoto
Download the gphoto source files from sourceforge.
```bash
wget http://sourceforge.net/projects/gphoto/files/libgphoto/2.5.3.1/libgphoto2-2.5.3.1.tar.gz
wget http://sourceforge.net/projects/gphoto/files/gphoto/2.5.3/gphoto2-2.5.3.tar.gz
```

#####3. Install gphoto
Now you need to compile gphoto from the downloaded source files. Make sure you are **superuser**. This may take some time.

```bash
cd ~
tar zxvf libgphoto2-2.5.3.1.tar.gz
cd libgphoto2-2.5.3.1/
./configure --prefix=/usr
make
sudo make install

cd ~
tar zvxf gphoto2-2.5.3.tar.gz
cd gphoto2-2.5.3/
./configure --prefix=/usr
make
sudo make install
```

### Test the DSLR/RPI Setup
**turn ON your DSLR.** in the command-line test your setup

```bash
#auto-detect the device (useful if you need to troubleshoot)
gphoto2 --auto-detect

#take a picture
gphoto2 --capture-image

#take a picture and download to RaspberryPi (current working path)
gphoto2 --capture-image-and-download 

#take a picture and download to RaspberryPi (desired path)
#the gphoto2 filename option allows timestamps like %Y %m %d %H %M %S
gphoto2 --capture-image-and-download  --filename /home/pi/name.jpg
```
### useful tips for timelapse capture
I changed my DSLR Mode to Manual (M) and did all the setup on the DSLR, although its possible to controll ISO, shutter speed etc. with the gphoto library. It's even possible to access the battery level from the DSLR.

+ DSLR manual mode
+ turn OFF Auto White Balance (reduce flicker)
+ turn OFF Capture Preview in DSLR Settings (saves battery power)
+ reduce Capture Quality (e.g. JPEG NORMAL, small to save battery power)
+ avoid downloading the captured images (saves battery power)

### Example python script
This is my example setup which is executed every minute from a crontab. Check out the call function of gphoto.

```python
import time
import sys
import pysftp
import datetime
import RPi.GPIO as GPIO

#call required to call gphoto script
from subprocess import call

def upload(ts):
    with pysftp.Connection('server', username='usrname', password='illuminati') as sftp:
    	with sftp.cd('html/d_fhp/io-slime-mold/data'):
    		sftp.put('/home/pi/io/slime_'+ts+'.jpg')

## START

## RPI GPIO setup
GPIO.setmode(GPIO.BOARD)

## LEDS in physical setup: 
## L=> 37,35,33 M=> 31,32 R=>36,38,40 
leds = [37,35,33,31,32,36,38,40]

## LED ON
for ID in range(len(leds)):
	GPIO.setup(leds[ID], GPIO.OUT)
	GPIO.output(leds[ID], GPIO.HIGH)

## TIMESTAMP
ts = time.time()
TIMESTAMP = datetime.datetime.fromtimestamp(ts).strftime("%Y-%m-%d_%H-%M-%S")
	
## DSLR PHOTO
call (["gphoto2","--capture-image-and-download","--filename","/home/pi/io/slime_"+TIMESTAMP+".jpg"])

## SLEEP
time.sleep(5)

## LED OFF
for ID in range(len(leds)):
	GPIO.output(leds[ID], GPIO.LOW)

## UPLOAD / CLEANUP
upload(TIMESTAMP)
GPIO.cleanup()
```

### Further Reading
+[short test footage physarum (15MB, 3*12s)](topada.hercules.uberspace.de/d_fhp/slime-lapse01-720p.mov)
+[gphoto2 Documentation](http://www.gphoto.org/doc/manual/ref-gphoto2-cli.html)


### Troubleshoot
#####*** Error (-60: 'Could not lock the device') ***
My RaspberryPi detects the SD Card of my DSLR as a mass storage device. In order to unmount the SD Card from raspberryPi filesystem detection, i found the following workaround.

```bash
#Gnome Virtual File System (GVFS)
cd /usr/lib/gvfs/

#Deactivate the GVFS by renaming the monitor. 
sudo mv gvfs-gphoto2-volume-monitor  gvfs-gphoto2-volume-monitor-bak
```
 
