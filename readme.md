# Configure Raspberry Pi with Blinka (circuitpython) library and external jupyter notebook

Set up a headless, perhaps only wireless, RaspberryPi SD Card so that Jupyter can be used remotely to develop code, including GPIO control, in a notebook.

## Setup SD Card
1. Download latest raspberry pi image
2. Load SD card with os image (I use balenaEtcher)
3. Mount sd card on your computer, navigate to “boot”
4. Edit config.txt and add (I put it just below dtparam=…) :
```
#Enable UART 
enable_uart=1
```

5. Create [wpa_supplicant.conf](https://github.com/mattderstine/jupyteronpi/blob/master/wpa_supplicant.conf?raw=true) file if WIFI needed. Edit for the correct ssid and password.
```
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
       ssid="name of network"
       psk="password"
       key_mgmt=WPA-PSK
    }
```
6. Create ssh file
 * this permits a single login using ssh so that ssh can be enabled during setup. This file is deleted after the login. 
7. Eject "boot" and move SD card to Pi

## Configure raspberry pi
1. Configure the hardware
 * If using serial: Connect serial connections – 3.3V, gnd, tx on rs232 board to rx on pi & vice versa
 * If using wired ethernet, make connection 
 * Apply power
2. Boot, login (user = pi, password = raspberry)
 * If using ssh: ssh pi@raspberrypi.local 
 * If using serial set main computer serial port baud rate= 115200
3. Configure the pi using:
```bash
sudo raspi-config
```
 * Enable ssh
 * Resize file partition
 * Change password
 * Change wifi country (if you skipped the wpa_supplacant file)
 * Setup wifi connections (you can add additional networks)
 * Change system name
 * Change to “boot to CLI with password”
 * Timezone
 * Exit program and reboot
5. Confirm correct system name on network (I use LanScan)
6. SSH to pi@newsystemname.local & Login.
7. Add [blinka](https://learn.adafruit.com/circuitpython-on-raspberrypi-linux/installing-circuitpython-on-raspberry-pi) and jupyter notebook. 
 * some scripts will require user input
 * don't reboot in the blinka install script. This can be done after all is installed
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python3-pip
sudo pip3 install --upgrade setuptools
sudo pip3 install --upgrade adafruit-python-shell
wget https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/raspi-blinka.py
sudo python3 raspi-blinka.py # don't reboot yet
pip3 install notebook #this installs it in user file space, not system
```
8. Edit .bashrc to include .local/bin in path, add to end
```
export PATH=$PATH:~/.local/bin
```
```bash
source .bashrc
```
9. set jupyter password so you don’t need to copy tokens
```bash
jupyter notebook password
```
10. reboot now
```bash
sudo reboot
```
## Run Notebook 
The notebook is expected to be run in a ssh session at normal user prividges. It will terminate if the session is interupted.  
( if I knew grep and bash I would write a simple script so I didn’t have to look up the ip address)
1. Start notebook on pi
```bash
ip addr #to find raspberry pi’s ip4 address: xx.yy.zz.aa
jupyter notebook --no-browser --ip=xx.yy.zz.aa --port=8888
```

2. on main computer open browser, _newraspberrypiname.local:8888_

## Set Fixed IP and autorun Notebook

One way to minimize connection complexity is to connect to the RaspberryPi using a dedicated ethernet link.


### Set up fallback fixed IP and autorun Notebook on the Raspberry PI

While still connected via DHCP, edit ```/etc/dhcpcd.conf``` and add the following lines
```
# It is possible to fall back to a static IP if DHCP fails:
define static profile
profile static_eth0
static ip_address=192.168.57.23/24
static routers=192.168.57.1
static domain_name_servers=192.168.57.1

# fallback to static profile on eth0
interface eth0
fallback static_eth0

```

Edit ```crontab -e``` and add
```
@reboot /home/pi/.local/jupyter notebook --no-browser --ip=192.168.57.23 --port=8888
```

When this RP boots, this will abort if DHCP is used to assign an ethernet address. Use the instructions to "Run Notebook" above in that case.

Shut down the RP.

### Set up a fixed ethernet port on PC or Mac and connect to the RP
1. Plug in the doggle to the computer. 

1. In the networking settings, find this adapter and change it to have a fixed IP of something like ```192.168.57.2``` with a subnet mask
of ```255.255.255.0``` or /24.  You may need to specify a router port. Use ```192.168.57.1```. If you need to add a DNS address use ```8.8.8.8``` (Google) and/or ```192.168.57.1```. These are placeholders and don't matter.

1. Connect the RP to the doggle and let it boot. 

1. In a browser enter ```192.168.57.23:8888```  The login to the jupyter notebook should appear.  Shell access is with 
```bash
ssh pi@192.168.57.23
```

