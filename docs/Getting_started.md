# Getting started with Raspberry-Pi (rpi)

## Wi-Fi Connectivity

[`host_commands`](../host/linux/host_control/host_commands) python module in `host/linux/host_control` folder implements the communication protocol between the host and ESP32. It contains following functions:
```
get_mac(mode)
get_wifi_mode()
set_wifi_mode(mode)
wifi_set_ap_config(ssid, pwd, bssid)
wifi_get_ap_config()
wifi_disconnect_ap()
wifi_set_softap_config(ssid, pwd, chnl, ecn, max_conn, ssid_hidden, bw)
wifi_get_softap_config()
wifi_ap_scan_list()
wifi_connected_stations_list()
```

These python functions can be used to control Wi-Fi functionality of the ESP32. Also see [host/linux/host_control/test.py](../host/linux/host_control/test.py) script for an example of using these functions. You can run the script as follows:
```
python test.py
```

To compile and load the host driver on a Raspberry Pi, go to `host/linux/host_control/` folder and run `./rpi_init.sh`. This script also creates `/dev/esps0` device, which is used as a WLAN control interface.

The other scripts in the same directory can be used to:

- connect to AP as a station
- disconnect from AP
- start softAP
- stop softAP
- scan available APs
- list stations connected to softAP

1. `station_connect.py` is a python script which configures ESP32 in station mode, and connects to an external AP with user-provided credentials. Also it enables the station interface and runs DHCP client. The script accepts arguments such as SSID, password, and optionally the MAC address of the AP. For example:

```
python station_connect.py 'xyz' 'xyz123456' --bssid='e5:6c:67:3c:cf:65'
```

You can check that `ethsta0` interface is up (enabled) using `ifconfig`.

2. `station_disconnect.py` is a python script to disconnect ESP32 station from AP.

```
python station_disconnect.py
```

You can check that `ethsta0` interface is down (disabled) using `ifconfig`.

3. `softap_config.py` is a python script for configuring ESP32 to work in softAP mode. The following parameters should be provided:

- SSID
- password, should be 8 ~ 64 bytes ASCII
- channel ID, 1 ~ 11
- encryption method (0: `OPEN`, 2: `WPA_PSK`, 3: `WPA2_PSK`, 4: `WPA_WPA2_PSK`)
- maximum number of stations, in range of 1 ~ 10.
- whether SSID is hidden (1 if the softAP shouldn't broadcast its SSID, else 0)
- bandwidth (1: `WIFI_BW_HT20` (20MHZ), 2: `WIFI_BW_HT40` (40MHZ))

The maximum number of connections, "SSID hidden", and bandwidth parameters are optional.

For example:
```
python softap_config.py 'xyz' 'xyz123456' 1 3 --max_conn=4 --ssid_hidden=0 --bw=1
```

You can check that `ethap0` interface is up (enabled) using `ifconfig`.

To start data connection, set up a DHCP server on the Raspberry Pi, or configure a static IP address for AP interface (`ethap0`).

4. `softap_stop.py` is a python script to disable ESP32 softAP mode. This script will change wifi mode to `null` if only softAP is running, or to `station` mode if softAP and station both are on.

```
python softap_stop.py
```

You can check that `ethap0` interface is down (disabled) using `ifconfig`.

5. `ap_scan_list.py` is a python script which gives a scanned list of available APs. The list contains SSID, channel number, RSSI, MAC address, and authentication mode of AP.

```
python ap_scan_list.py
```

6. `connected_stations_list.py` is a python script that returns a list of MAC addresses of stations connected to softAP.

```
python connected_stations_list.py
```

### Open air throughput test results for WLAN

Following are the test results conducted in open air.

```
UDP Tx: 16.4 Mbps
UDP Rx: 16.8 Mbps
TCP Tx: 14 Mbps
TCP Rx: 12 Mbps
```

## For Bluetooth/BLE functionality

- Ensure that bluez is installed on Raspberry Pi and it is downloaded in source format as well. Please refer to [Setup](Setup.md) instructions for more details.
- In following test, Android device was used as a BT/BLE test device. For BLE testing, [nRF connect for mobile APP](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp&hl=en_IN) was used.
- Go to `host/linux/host_control/` folder to run following script.

### UART based setup
1. Execute `./rpi_init.sh btuart` to prepare Raspberry Pi for Bluetooth operation
2. Execute `hciattach` command as below to add HCI interface (i.e. hciX)
```
$ sudo hciattach -s 115200 /dev/serial0 any 115200 flow
```

### SDIO based setup
Execute `./rpi_init.sh` to prepare Raspberry-Pi for SDIO+BT operation.
HCI interface (i.e hciX) will be available for use as soon as host driver detects ESP32 module over SDIO interface.
You can use standard HCI utilities over this interface to make use of BT/BLE feature.

### BT/BLE Test procedure
#### GATT server

1. run `hciconfig`. Output should show only one `SDIO` interface.
```
hci0:	Type: Primary  Bus: SDIO
	BD Address: 3C:71:BF:9A:C2:46  ACL MTU: 1021:9  SCO MTU: 255:4
	UP RUNNING PSCAN
	RX bytes:8801 acl:1000 sco:0 events:406 errors:0
	TX bytes:5097 acl:147 sco:0 commands:52 errors:0
```
2. Go to `bluez-5.xx` folder. Run `./test/example-gatt-server`. This will start GATT server on Raspberry-Pi.

3. Now start advertising. Run `sudo hciconfig hci0 leadv`.

4. Now ESP32's mac address should be listed in scan list of mobile app.

5. Connect to ESP32's mac address with mobile as GATT client.

6. Check read/write characteristics fields in `Heart Rate` service.

#### GATT Client

1. Run `./test/example-gatt-client` on Raspberry Pi. This will start GATT client on Raspberry Pi.

2. You should see a `Heart Rate Measurement` field in Raspberry Pi console.

#### BT scan

Run `hcitool scan` for BT device scanning.

#### BLE scan

Run `hcitool lescan` for BLE device scanning.
