## Install Debian Headless on CHIP
For installing Debian headless, you need to install latest Ubuntu version. 17.10. worked fine for me. Probably it is also possible to just start Ubuntu from USB stick. I did not try that one.

After installing Ubuntu install git via
~~~~
sudo apt-get install git
~~~~

Then clone the official CHIP-SDK with
~~~~
git clone https://github.com/NextThingCo/CHIP-SDK
~~~~

Then switch to the installed folder CHIP-SDK and run the setup script
~~~~
cd CHIP-SDK
sudo ./setup-ubuntu1404.sh
~~~~

dont worry that it says ubuntu1404. It works perfectly in 1710. 
Then you can proceed as explaind in the CHIP-SDK github.
Enter
~~~~
cd CHIP-tools
sudo ./chip-update-firmware.sh -s
~~~~
for installing headless debian. 

## Windows 10 settings for serial connection to Chip
I did that using Windows 10. First Problem, CHIP is recognized as some weird CDC Gadget. You need to change that in order to connect to the chip via serial connection.
Open the device manager in windows
Select "other Devices", then "CDC Composite Gadget", then right click
Properties > Update driver > browse > let me pick > ports > Microsoft > USB Serial device.

Now you see in the device manager with the right COM port number. Just connect via putty or anything like that using serial connection to that port (COM9 e.g.) with speed 9600.

standart credentials are
~~~~
user: chip
pw: chip
~~~~

or 
~~~~
user: root
pw: chip
~~~~

make sure you change the passwords with
~~~~
passwd
~~~~
and the root password with
~~~~
sudo passwd root
~~~~


## Enable Wifi
Run 
~~~~
nmtui 
~~~~

and select the correct wifi and enter pw. Afterwards click edit and make sure that automatic reconnection is enabled. 
Now your good to go to ssh in the chip.

## Install samba to share folders over your home network

To make the jukebox easy to administer, it is important that you can add new songs and register new RFID cards over your home network. This can be done from any machine. The way to integrate your RPi into your home network is using *Samba*, the standard [Windows interoperability suite for Linux and Unix](https://www.samba.org/).

Open a terminal and install the required packages with this line:

~~~~
sudo apt-get install samba samba-common-bin 
~~~~

First, let's edit the *Samba* configuration file and define the workgroup the RPi should be part of.

~~~~
sudo nano /etc/samba/smb.conf
~~~~
Edit the entries for workgroup and wins support:

~~~~
workgroup = WORKGROUP
wins support = yes
~~~~

If you are already running a windows home network, add the name of the network where I have added `WORKGROUP`. 

Now add the specific folder that we want to be exposed to the home network in the `smb.conf` file. 

~~~~
[musicchest]
   comment= Musicchest
   path=/home/chip/shared
   browseable=Yes
   writeable=Yes
   only guest=no
   create mask=0777
   directory mask=0777
   public=no
~~~~

**Note:** the `path` given in this example works (only) if you are installing the jukebox code in the directory `/home/chip/`.

Finally, add the user `chip` to *Samba*. For simplicity and against better knowledge regarding security, I suggest to stick to the default user and password:

~~~~
user     : chip
password : raspberry
~~~~

Type the following to add the new user:

~~~~
sudo smbpasswd -a chip
~~~~

Restart samba to see efect
~~~~
sudo /etc/init.d/samba restart
~~~~
## Adding python libraries
Install pip
~~~~
sudo apt-get install python-pip
~~~~

Install gpio lib for chip from https://github.com/xtacocorex/CHIP_IO
~~~~
sudo apt-get update
sudo apt-get install git build-essential python-dev python-pip flex bison chip-dt-overlays -y
git clone git://github.com/xtacocorex/CHIP_IO.git
cd CHIP_IO
sudo python setup.py install
cd ..
~~~~


## Running the web app

There is a second way to control the RFID jukebox: through the browser. You can open a browser on your phone or computer and type in the static IP address that we assigned to the RPi earlier. As long as your phone or PC are connected to the same WiFi network that the RPi is connected to, you will see the web app in your browser.

### Installing lighttpd and PHP

~~~~
sudo apt-get install lighttpd php5-common php5-cgi php5
~~~~

### Configuring lighttpd

Open the configuration file:

~~~~
sudo nano /etc/lighttpd/lighttpd.conf
~~~~

Change the document root, meaning the folder where the webserver will look for things to display or do when somebody types in the static IP address. To point it to the Jukebox web app, change the line in the configuration to:

~~~~
server.document-root = "/home/pi/RPi-Jukebox-RFID/htdocs"
~~~~

The webserver is usually not very powerful when it comes to access to the system it is running on. From a security point of view, this is a very good concept: you don't want a website to potentially change parts of the operating system which should be locked away from any public access.

We do need to give the webserver more access in order to run a web app that can start and stop processes on the RPi. To make this happen, we need to add the webserver to the list of users/groups allowed to run commands as superuser. To do so, open the list of sudo users in the nano editor:

~~~~
sudo nano /etc/sudoers
~~~~

And at the bottom of the file, add the following line:

~~~~
www-data ALL=(ALL) NOPASSWD: ALL
~~~~

The final step to make the RPi web app ready is to tell the webserver how to execute PHP. To enable the lighttpd server to execute php scripts, the fastcgi-php module must be enabled.

~~~~
sudo lighty-enable-mod fastcgi-php
~~~~

Now we can reload the webserver with the command:

~~~~
sudo service lighttpd force-reload
~~~~

Next on the list is the media player which will play the audio files and playlists: VLC. In the coming section you will also learn more about why we gave the webserver more power over the system by adding it to the list of `sudo` users.

## Install the media player VLC (and python vlc lib)

The VLC media player not only plays almost everything (local files, web streams, playlists, folders), it also comes with a command line interface `CLVC` which we will be using to play media on the jukebox.

Install *VLC*

~~~~
sudo apt-get install vlc
~~~~

Ok, the next step is a severe hack. Quite a radical tweak: we will change the source code of the VLC binary file. We need to do this so that we can control the jukebox also over the web app. VLC was designed not to be run with the power of a superuser. In order to trigger VLC from the webserver, this is exactly what we are doing.

Changing the binary code is only a one liner, replacing `geteuid` with `getppid`. If you are interested in the details what this does, you can [read more about the VLC hack here](https://www.blackmoreops.com/2015/11/02/fixing-vlc-is-not-supposed-to-be-run-as-root-sorry-error/).


**Note:** changing the binary of VLC to allow the program to be run by the webserver as a superuser is another little step in a long string of potential security problems. In short: the jukebox is a perfectly fine project to run for your personal pleasure. It's not fit to run on a public server.


## Install mpg123 (Probably dont needed)

While we are using *VLC* for all the media to be played on the jukebox, we are using the command line player *mpg123* for the boot sound. More about the boot sound in the file [`CONFIGURE.md`](CONFIGURE.md). To install this tiny but reliable player, type:

```
sudo apt-get install mpg123
```

## install i2c for communication with arduino
~~~~
sudo apt-get install i2c-tools
sudo apt-get install python-smbus
~~~~

add chip user to i2c group to have correct permissions.
~~~~
sudo usermod -aG i2c chip
~~~~


## Install git

[*git* is a version control system](https://git-scm.com/) which makes it easy to pull software from GitHub - which is where the jukebox software is located.

~~~~
sudo apt-get update
sudo apt-get install git
~~~~




+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
## Install the jukebox code

~~~~
$ cd /home/pi/
$ git clone https://github.com/MiczFlor/RPi-Jukebox-RFID.git
~~~~

## Using a USB soundcard

In order to use an external USB soundcard instead of the inbuilt audio out, you might need to update your system and tweak a couple of config files, depending on your card. The most comprehensive explanation on why and how, you can find at [adafruit](https://learn.adafruit.com/usb-audio-cards-with-a-raspberry-pi/instructions). 

Using the Jessie distribution, you might be lucky and there is a quick fix setting the [~/.asoundrc](https://raspberrypi.stackexchange.com/questions/39928/unable-to-set-default-input-and-output-audio-device-on-raspberry-jessie) file.

## Reboot your Raspberry Pi

Ok, after all of this, it's about time to reboot your jukebox. Make sure you have the static IP address at hand to login over SSH after the reboot.

~~~~
sudo reboot
~~~~

# Configure the jukebox

Continue with the configuration in the file [`CONFIGURE.md`](CONFIGURE.md).
