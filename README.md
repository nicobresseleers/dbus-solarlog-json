# dbus-solarlog-json
Integrate Solarlog Logger into Venus OS via Json Interface

## Purpose
With the scripts in this repo it should be easy possible to install, uninstall, restart a service that connects the Solarlog Logger to the VenusOS and GX devices from Victron.
It is adapted from https://github.com/vikt0rm/dbus-shelly-1pm-pvinverter
I first used the modbus tcp data communication, but with that I only see total PAC of all PV Inverters together. With JSON API, it is possible to get PAC Data of all connected PV Inverters. So it is possible to split the power to different phases.


## Inspiration
I took some ideas and approaches from the following projects - many thanks for sharing the knowledge:
- https://github.com/vikt0rm/dbus-shelly-1pm-pvinverter
- https://github.com/fabian-lauer/dbus-shelly-3em-smartmeter
- https://shelly-api-docs.shelly.cloud/gen1/#shelly1-shelly1pm
- https://github.com/victronenergy/venus/wiki/dbus#pv-inverters

## How it works
### My setup
- 3-Phase installation
- Solarlog500 with latest Firmware
  - 1 Delta PV Inverter, connected to 3 phases
  - 2 Mastervolt PV Inverters, connected to 1 phase  
- Venus OS on Multiplus-II 5000 GX with Firmware 2.91
  - No other devices from Victron connected
  - EM24 Grid Meter

### Details / Process
As mentioned above the script is inspired by @fabian-lauer dbus-shelly-3em-smartmeter implementation.
So what is the script doing:
- Running as a service
- connecting to DBus of the Venus OS `com.victronenergy.pvinverter.http_{DeviceInstanceID_from_config}`
- After successful DBus connection Solarlog is accessed via REST-API - simply the /getjp is called with a Post and a JSON is returned with all details
- Paths are added to the DBus with default value 0 - including some settings like name, etc
- After that a "loop" is started which pulls Solarlog data every 5000ms from the REST-API and updates the values in the DBus

Thats it 😄


## Install & Configuration
### Get the code
Just grap a copy of the main branche and copy them to a folder under `/data/` e.g. `/data/dbus-solarlog-json`.
After that call the install.sh script.

The following script should do everything for you:
```
wget https://github.com/notaus123/dbus-solarlog-json/archive/refs/heads/main.zip
unzip main.zip "dbus-solarlog-json/*" -d /data
mv /data/dbus-solarlog-json-main /data/dbus-solarlog-json
chmod a+x /data/dbus-solarlog-json/install.sh
/data/dbus-solarlog-json/install.sh
rm main.zip
```
⚠️ Check configuration after that - because service is already installed an running and with wrong connection data (host, username, pwd) you will spam the log-file

### Change counter.txt
The script calculates the total energy production based on the power readings and saves it to the counter.txt file. If you don't want to start with zero values, edit the counter.txt and enter the numbers for each phase in kwh. L1;L2;L3

### Change config.ini
Within the project there is a file `/data/dbus-solarlog-json/config.ini` - just change the values - most important is the deviceinstance, custom name and phase under "DEFAULT" and host, username and password in section "ONPREMISE". More details below:

| Section  | Config vlaue | Explanation |
| ------------- | ------------- | ------------- |
| DEFAULT  | AccessType | Fixed value 'OnPremise' |
| DEFAULT  | SignOfLifeLog  | Time in minutes how often a status is added to the log-file `current.log` with log-level INFO |
| DEFAULT  | Deviceinstance | Unique ID identifying the Solarlog in Venus OS |
| DEFAULT  | CustomName | Name shown in Remote Console (e.g. name of pv inverter) |
| ONPREMISE  | Host | IP or hostname of on-premise Solarlog web-interface |
| ONPREMISE  | Username | Username for htaccess login - leave blank if no username/password required |
| ONPREMISE  | Password | Password for htaccess login - leave blank if no username/password required |



## Used documentation
- https://www.matusz.ch/blog/2020/08/09/solar-log-base-per-rest-ansteuern/
- https://github.com/victronenergy/venus/wiki/dbus#pv-inverters   DBus paths for Victron namespace
- https://github.com/victronenergy/venus/wiki/dbus-api   DBus API from Victron
- https://www.victronenergy.com/live/ccgx:root_access   How to get root access on GX device/Venus OS

