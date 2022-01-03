# Multi-channel Video Installation using Raspberry Pi
In 2020, Amay Kataria was invited for the [BRAHMAN](https://brahman.ai/) residency organized by [Gene Kogan](https://genekogan.com/) at Bombay Beach, California. These steps were put together that time to setup a distributed video installation using multiple raspberry pis. 

These instructions summarize how to setup multiple raspberry pis using omxplayer-sync to create a multi-channel video installation. This will require raspberry pis, display devices to show content, a router, and ethernet cables to connect each raspberry pi. 

## Setup Raspberry Pis
Follow the instructions [here](https://github.com/phillipdavidstearns/brahman-ai/blob/master/guides/rpi-setup/rpi-setup.md) to setup your Raspberry pi. Make sure each pi has a unique hostname. 

## Install omxplayer-sync
omxplayer is the default application that plays media on raspberry pi. [omxplayer-sync](https://github.com/turingmachine/omxplayer-sync) is a python-based wrapper for omxplayer that does video synchronization between master and slave. Every setup has one master and multiple slaves that sync with the master. Perform the following steps on all the raspberry pis to install omxplayer-sync. 
```
sudo su
apt-get remove omxplayer
rm -rf /usr/bin/omxplayer /usr/bin/omxplayer.bin /usr/lib/omxplayer
apt-get install libpcre3 fonts-freefont-ttf fbset libssh-4 python3-dbus
wget https://github.com/magdesign/PocketVJ-CP-v3/raw/master/sync/omxplayer_0.3.7-git20170130-62fb580_armhf.deb
dpkg -i omxplayer_0.3.7~git20170130~62fb580_armhf.deb (This will give an error about a missing dependency for libssl1.0.0. Install libssl1.0.2 instead in the next step.)
apt-get install libssl1.0.2
apt --fix-broken install (This will reinstall omxplayer properly. Test that omxplayer is installed by typing "omxplayer --help" in the command line.)
wget -O /usr/bin/omxplayer-sync https://github.com/turingmachine/omxplayer-sync/raw/master/omxplayer-sync
chmod 0755 /usr/bin/omxplayer-sync
wget https://github.com/turingmachine/omxplayer-sync/raw/master/synctest.mp4
```

## Network Setup
This installation setup performs better when the raspberry pis are connected to the same local network using ethernet cables. For this, one can use a regular router with ethernet jacks that the pis can plug into. With this, they can communicate over a local network without communicating with the internet. An external computer can connect to that router and ssh into the pis, so the pis can be remotely managed. 

## Horizontal vs Vertical Orientation
Based on how one wants to setup the installation, one needs to decide the orientation of the screens. Unfortunately, omxplayer-sync doesn't support changing the orientation of the video, unlike omxplayer does. Therefore, if the installation requires to setup the screens vertically, then one must rotate the raspberry pi display by 90 degree clockwise as well. To do this, you need to do the following: 
```
sudo nano /boot/config.txt
add the line "lcd_rotate=90" on the top to rotate the screen clockwise by 90 degrees. 
```

## Preparing Content
The content can be prepared in multiple ways by using various kinds of editing tools. If one wants to split a high-resolution video into multiple displays, calculate the total resolution of the display by multiplying the width of each display with the number of displays. This will be the new width of the video. Then use a video editing tool to create sequences that match the resolution of each screen. Make sure the content is exported with h.264 codec and mp4 format because that's what omxplayer supports. 

## Initiating master and slave
Once omxplayer-sync is installed on all the raspberry pis and the content is prepared, copy the sequence for each display to its corresponding raspberry pi. Make sure the filename of each sequece playing on each raspberry pi is the *same*, else omxplayer-sync will unexpectedly shut down. The following commands are supported by omxplayer-sync. 
```
$ ./omxplayer-sync -h
Usage: omxplayer-sync [options] filename

Options:
  -h, --help            show this help message and exit
  -m, --master          
  -l, --slave           
  -x DESTINATION, --destination=DESTINATION
  -u, --loop            
  -v, --verbose         
  -o ADEV, --adev=ADEV  
  -a ASPECT, --aspect=ASPECT  Aspect Mode - fill, letterbox, stretch
```
To start the installation, ssh into each pi using an external computer. Start off by executing this command on all the slave computers. 
```
omxplayer-sync -lu filename.mp4
```
Nothing will happen when this command is executed. The command prompt will be in a wait state with this command because it's waiting to sync with the master. On the master raspberry pi, execute this command.
```
omxplayer-sync -mu filename.mp4
```
With this command, all the ready slaves will turn on along with the master computer and start running the sequences on the respective computers. If the master is execued before the slave, it can take a long time before the slave can actually catch up to the master. Therefore, it's advisable to always ready the slave computers first and then execute the command on the master computer. 

## Troubleshooting & Tips
1. In the beginning, slave and master will not be in sync completely. You may notice some jerks or pauses in the slave video sequence. This is expected as the slave computer is trying to sync with the master by seeking constantly. This only lasts momentarily and will eventually disappear. To see the syncing actions in detail, give the v (verbose) flag like "omxplayer-sync -luv filename.mp4." This will print detailed logs and will show when the slave tries to seek to the master. 

2. Sometimes, you want to fill the entire screen with the video content but when you play the file it doesn't stretch it. For that, there is the -a (aspect) flag that you can use. To fill the screen with the content, you may use it like this "omxplayer -muv -a stretch filename.mp4." 
