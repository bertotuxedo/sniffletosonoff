# sniffletosonoff
Flash Sniffle Software to Sonoff Zigbee USB Dongle

Following cemaexecuter to create drone RemoteID detector. This first step is to flash sniffle to Sonoff Zigbee USB Dongle.
This was donw while Sonoff was plugged into raspberry pi 4 running DragonOS.

## Installing and Running Sniffle on a Sonoff Zigbee USB Dongle (Ubuntu 22.04)
This guide walks you through installing and running Sniffle, a BLE sniffer, on a Sonoff Zigbee USB Dongle using Ubuntu 22.04.

### Step 1: Connect to Ubuntu
On your Windows machine, open Command Prompt (cmd) and connect via SSH:

```
ssh ubuntu@<your-ip-address>
```
Replace <your-ip-address> with your Ubuntu machine’s IP address.

Enter your Ubuntu password when prompted.

### Step 2: Install Required Dependencies
Update your system and install all required packages:

```
sudo apt update && sudo apt upgrade -y
sudo apt install git python3 python3-pip python3-serial python3-intelhex python3-magic cmake make gcc-arm-none-eabi -y
```
### Step 3: Clone the Required Repositories
Download Sniffle and the flashing tool cc2538-bsl:

```
cd ~
git clone https://github.com/bkerler/DroneID.git
git clone https://github.com/JelmerT/cc2538-bsl.git
```

### Step4: Run DroneID setup

```
cd DroneID
./setup.sh
```

### Step 4: Download the Sniffle Firmware in DroneID folder 

```
sudo rm -r sniffle
git clone https://github.com/bkerler/sniffle.git
```

### Step 5: Download the Sniffle hex file for Sonoff Dongle

```
cd ~/DroneID/sniffle/fw
wget https://github.com/nccgroup/Sniffle/releases/download/v1.10.0/sniffle_cc1352p1_cc2652p1.hex
```
or from this repo
```
wget https://github.com/bertotuxedo/sniffletosonoff/sniffle_cc1352p1_cc2652p1.hex
```
### Step 6: Verify that the file was downloaded successfully:

```
ls -l sniffle_cc1352p1_cc2652p1.hex
```

### Step 7: Prepare the USB Device for Flashing
List all available serial devices:

```
ls /dev/ttyUSB*
```
If the device is listed as /dev/ttyUSB0, proceed with setting the correct permissions:

```
sudo chmod 666 /dev/ttyUSB0
sudo usermod -a -G dialout ubuntu
```
Additionally, if you have a second system conduct the same command with ttyUSB1 or whatever it is labeled.
```
sudo chmod 666 /dev/ttyUSB1
sudo usermod -a -G dialout ubuntu
```
Reboot the system
```
sudo reboot
```
After the system reboots, reconnect via SSH.

### Step 8: Flash the Sniffle Firmware
Once reconnected, navigate to the cc2538-bsl directory and flash the firmware:

```
cd ~/cc2538-bsl
sudo python3 cc2538-bsl.py -p /dev/ttyUSB0 --bootloader-sonoff-usb -ewv ~/DroneID/sniffle/fw/sniffle_cc1352p1_cc2652p1.hex
```
If the flashing is successful, you should see output similar to this:

```
Performing mass erase
Erasing all main bank flash sectors
    Erase done
Writing 360448 bytes starting at address 0x00000000
Write 104 bytes at 0x00057F980
    Write done
Verifying by comparing CRC32 calculations.
    Verified (match: 0xaf3a8c0e)
```
It is recommended to have a second Sonoff Dongle to conduct testing to spoof. Follow the same steps. If already plugged in you may have to run it as ttyUSB1.

**Unplug device and plug back in**

Verify by running the help command 
```
python3 ~/DroneID/sniffle/python_cli/sniff_receiver.py --help
```

## Step 7: Test DroneID receive and Spoof

Open two terminal windows 
In terminal 1 run the receiver

```
cd ~/DroneID
./bluetooth_receiver.sh -b 2000000 -s /dev/ttyUSB1 --zmqsetting 127.0.0.1:4222 -v
```
In terminal 2 run the spoofer
```
cd ~/DroneID
./bluetooth_spoof.py -s /dev/ttyUSB0 -b 2000000
```
The readout should contain some of the information from the drone.json file
