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
```
sudo raspi-config
```
4.	Set up wireless networking, enable ssh, resize file partition, change password, change wifi country,setup wifi connections, change system name, change to “boot to CLI with password”, timezone
5. Exit program and reboot
6. Confirm correct system name on network (I use LanScan)
7. SSH to pi@newsystemname.local & Login.
8. Add blinka and jupyter notebook
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python3-pip
sudo pip3 install --upgrade setuptools
sudo pip3 install --upgrade adafruit-python-shell
wget https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/raspi-blinka.py
sudo python3 raspi-blinka.py
pip3 install notebook #this installs it in user file space, not system
```
9. Edit .bashrc to include .local/bin in path, add to end
```
export PATH=$PATH:~/.local/bin
```
```
source .bashrc
```
10. set jupyter password so you don’t need to copy tokens
```
jupyter notebook password
```
## Run Notebook 
The notebook is expected to be run in a ssh session at normal user prividges. It will terminate if the session is interupted.
( if I knew grep and bash I would write a simple script so I didn’t have to look up the ip address)
1. Start notebook on pi
```
ip addr #to find raspberry pi’s ip4 address: xx.yy.zz.aa
jupyter notebook --no-browser --ip=xx.yy.zz.aa --port=8888
```

2. on main computer open browser, _newraspberrypiname.local:8888_

