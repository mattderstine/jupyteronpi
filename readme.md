#Configure Raspberry Pi with Blinka (circuitpython) library and external jupyter notebook

Set up a headless, perhaps only wireless, RaspberryPi SD Card so that Jupyter can be used remotely to develop code, including GPIO control, in a notebook.

##Setup SD Card
1. Download latest raspberry pi image
2. Load SD card with os image (I use balenaEtcher)
3. Mount sd card on your computer, navigate to “boot”
4. Edit config.txt and add (I put it just below dtparam=…) :
```
#Enable UART 
enable_uart=1
```

5. create wpa_supplicant.conf file if WIFI needed
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
6. Add empty ssh file
 * this permits 1 login using ssh so that ssh can be enabled during setup
7. Eject boot and move SD card to Pi

##Configure raspberry pi
1. If using serial: Connect serial connections – 3.3V, gnd, tx on board to rx on pi & vice versa
2. Boot, login (user = pi, password = raspberry)
 * If using ssh: ssh pi@raspberrypi.local 
 * If using serial, baud rate= 115200
3. Enter
```
sudo raspi-config
```
4.	Set up wireless networking, enable ssh, resize file partition, change password, change wifi country,setup wifi connections, change system name, change to “boot to CLI with password”
5. Leave and reboot
6. Confirm correct system name on network (I use LanScan)
7. SSH & Login
```
sudo apt-get update
sudo apt-get upgrade
sudo pip3 install --upgrade setuptools
sudo pip3 install --upgrade adafruit-python-shell
wget https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/raspi-blinka.py
sudo python3 raspi-blinka.py
pip3 install notebook #this installs it in user file space, not system
```
8. Edit .bashrc to include .local/bin in path, add to end
```
export PATH=$PATH:~/.local/bin
```
```
source .bashrc
```
9. set jupyter password so you don’t need to copy tokens
```
jupyter notebook password
```
##Run Notebook 
( if I knew grep and bash I would write a simple script so I didn’t have to look up the ip address)
```
ip addr #to find raspberry pi’s ip4 address
jupyter notebook --no-browser --ip=ip4 --port=8888
```

10. on main computer open browser, _newraspberrypiname.local:8888_

