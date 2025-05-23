Web GUI for RuiDeng USB testers (UM34C, UM24C, UM25C, TC66C)
==

Simple web GUI written in Python 3. Measurements are stored in sqlite database. Tables and graphs are supported.
Live preview and graphing is also available.

Tested on UM34C/UM24C/UM25C/TC66C.

Based on https://github.com/sebastianha/um34c


Requirements
--

- UM34C/UM24C/UM25C - meter connected via regular Bluetooth
- TC66C - meter is using BLE instead of regular bluetooth so pairing will not work nor RFCOMM will
    - BLE has limited support on desktop devices, see used library for supported platforms and versions:
      https://github.com/hbldh/bleak#features
- TC66C USB - meter connected with USB
    - Meter connected with USB exposes itself as serial port
- UM34C/UM24C/UM25C Serial/COM port method - meter can be connected as serial port
    - Pairing with Windows Settings works fine. Pin is 1234. After successful pairing some serial ports are
      installed. In my case two. One of them works.
    - On Linux `rfcomm` and `hcitool` can be used (both provided by bluez package)
        - Retrieve bluetooth address with `hcitool scan`
        - Bind retrieved address to serial port with `rfcomm bind 0 aa:bb:cc:dd:ee:ff`.
          This step is not persistent. Serial port will disappear after reboot. Also, rd-usb needs to
          have permissions to use /dev/rfcommX.

Installation
--

### Binaries (Win x64 only)

- Download from [releases](https://github.com/kolinger/rd-usb/releases)
    - **rd-usb-x.exe** is CLI web server application. GUI is provided by web browser.
      Run executable and web server will be shortly spawned on address http://127.0.0.1:5000.
    - **rd-usb-install-x.exe** is installer of standalone GUI application. Works without web browser.
      Embedded browser is used instead. External web browser still can be used with address (see above).
- Application will be probably blocked by Microsoft SmartScreen. For unblock click `More info`
  and `Run anyway`. I don't have certificate for signing and application does not have any
  reputation so Microsoft will block by default.

### Source code

1. Python 3.11 or newer is required
2. Download `rd-usb-source-x.zip` from [releases](https://github.com/kolinger/rd-usb/releases)
   or `git clone https://github.com/kolinger/rd-usb.git`
3. Install requirements
    - For GUI: `pip install -r requirements.txt`
    - For headless `pip install -r requirements_headless.txt` this version doesn't
      contain dependencies for embedded browser GUI. This is useful if you plan to use CLI/webserver only.
      It's also useful on ARM SBCs.
    - If you encounter failing installing of `pendulum` library then you may need different set of requirements.
      [See bellow for details](#pendulum-issue).\
      In this case use `pip install -r requirements_headless_new.txt` as workaround.
    - If you encounter failing installation of `pybluez` then make sure you have `libbluetooth` dependency installed.\
      For Debian based distribution this means `apt-get install libbluetooth-dev`.\
      For Windows you need Microsoft Visual C++ 14.0 or greater distributed via 
      [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/).
    - There is also currently issue with `pybluez` installation where it fails due to pip version. 
      Make sure you are not using pip 24.x or newer, pip 23.x works fine, if you have newer pip version you can
      downgrade to latest 23.x for example `C:\Python\python.exe -m pip install --upgrade pip==23.3.2`
4. Run with `python web.py` - this will spawn web server on http://127.0.0.1:5000, port can be changed
   with first argument: `python web.py 5555`

For additional arguments/options use --help: `python web.py --help`.

On Windows `python` can be found in Python's installation folder as `python.exe`.
For example replace `python web.py` with `C:\Python\python.exe web.py`
and `pip install -r requirements.txt` with `C:\Python\python.exe -m pip install -r requirements.txt`.

On Linux use `python3` and `pip` inside [virtual environment](https://docs.python.org/3/library/venv.html) (venv).

Usage
--

**UM34C/UM24C/UM25C Bluetooth**

1. Select your device version.
2. Name your session. For example 'testing some power bank'. This is
   used to separate multiple measurements from each other.
3. Select sample rate. Faster sample rate will result in more accurate data but also
   will create a lot more data. For short measurements use faster sample rate. For longer
   use slower rate. Choose carefully.
4. Select UM34C/UM24C/UM25C from devices and follow with Setup link.
    1. If previously connected you can initiate Setup again by clicking on current Bluetooth address
5. Scan for devices and select your device from list by clicking on it
6. After this you can connect simply by using Connect button. Setup is required only for new/different device.
7. Connection will be hopefully successful, and you will see live measurements in graph.
   Otherwise, read log for error messages.

The 100ms and 250ms polling period (‘sample rate’) is misleading since only some meters support polling this quick,
and it seems that all meters have estimated internal sample rate of 2 Hz (500ms) thus even if the meter supports
polling at 10 Hz / 100ms (like the TC66C over USB, while the UM24/25/34 has estimated polling rate up to 3 Hz) this
isn't useful since the extra samples are just duplicates of previous sample. In the original implementation the
polling period wasn't polling period but the gap between samples, that's why in past the 100ms/250ms value did have
meaningful purpose, but currently these options are essentially without purpose and left as legacy.

**TC66C Bluetooth**

1. Make sure your OS is supported and has bluetooth with BLE support (Bluetooth Low Energy)
2. Rest is same as other devices. See above.

**Bluetooth issues**

If you have Bluetooth issues then try to close rd-usb, unplug/plug USB meter and start rd-usb again. Bluetooth can be
sometimes funny and just refuses to work without any reason. If this doesn't help check proximity of your PC and USB
meter, sometimes moving USB meter real close can resolve Bluetooth issues.

Also make sure no other connection to the meter exists, like no other application is connected to the meter,
the RFCOMM isn't bound, if any other connection exists this may break the ability to use or even see the meter
from rd-usb.

If you find trouble connecting to the UM34C/UM24C/UM25C via the direct Bluetooth method then try
to use the UMxxC Serial method, where you need to first pair (in Windows) or use rfcomm bind (Linux),
see [Requirements](#Requirements) above for details.

**TC66C USB or UM34C/UM24C/UM25C Serial/COM port method**

1. Select UM34C/UM24C/UM25C with Serial suffix.
2. Follow Setup link to find your serial port or click Connect if you already have port selected.
3. Rest is same as other devices. See above.

![setup](screenshots/setup.png)

Graphs
--

![tables](screenshots/graphs-v2.png)


Tables
--

![tables](screenshots/tables.png)

Pendulum issue
--------------

There is known issue with library named `pendulum` (used for date and time) when used together with 
newer Python versions. This issue is triggered when installing requirements via pip and looks
something like this:

```
...
Building wheel for pendulum (pyproject.toml) did not run successfully.
...
ModuleNotFoundError: No module named 'distutils'
...
```

If your installation via pip fails due to this then the solution is to install different version of dependencies.
Specifically `pip install -r requirements_headless_new.txt` should fix this issue.

This issue is solved with version `>=3.0.0` of this library. Unfortunately as of now
version `3.0.0` has other compatibility issues (specifically failing Rust compilation
on systems that don't have prebuilt wheels) thus that's why `<3.0.0` is used by default.

Multiple instances
------------------

If you want to use multiple instances for multiple USB meters then you need to do some adjustments to separate these
instance from each other. As default all instances share the same port thus you can't use multiple USB meters.

This can be easily fixed by changing the port, the port is first argument, without any argument `5000` is used.
We can use default `5000` for first instance and then increment port for each additional instance.

Examples:
- `"C:\Program Files\rd-usb\rd-usb.exe" 5001` for Windows GUI
- `"C:\somewhere\rd-usb.exe" 5001` for Windows standalone binary
- `"C:\Python\python.exe" web.py 5001` for Windows Python
- `/some/python web.py 5001` for Linux

This way you can use multiple instances with multiple USB meters but all data and settings are shared together.

It may be beneficial to separate instances completely including all data and settings. For this extra parameter
`--data-dir` is required, for details see **Custom data directory** section bellow. This parameter expects directory
where all data and settings will be placed. If you combined `--data-dir` and port then you can have independent
instances.

Examples:
- `"C:\Program Files\rd-usb\rd-usb.exe" 5001 --data-dir C:\some\place\for\data` for Windows GUI
- `"C:\somewhere\rd-usb.exe" 5001 --data-dir C:\some\place\for\data` for Windows standalone binary
- `"C:\Python\python.exe web.py" 5001 --data-dir C:\some\place\for\data` for Windows Python
- `/some/python web.py 5001 --data-dir /some/place/for/data` for Linux

On Windows it may be handy to make shortcut for this - just copy existing shortcut and add the parameters at the end of
Target field separated by space like the first examples. Then you have easy access to your second instance.

Docker
--

Experimental Docker container exists. Bluetooth isn't officially supported in Docker so there is need to pass special
arguments to Docker to make it work - there are also caveats that different system may need extra system changes to
make Bluetooth in Docker work.

**Ubuntu/Debian/Raspbian caveats** - you need to load custom Docker AppArmor policy and add extra argument for Docker 
to use this policy instead of the default one. 

In terminal:
```
sudo wget https://raw.githubusercontent.com/kolinger/rd-usb/master/docker/docker-ble -O /etc/apparmor.d/docker-ble
sudo apparmor_parser -r -W /etc/apparmor.d/docker-ble
```

This may be sometimes unnecessary, but it won't hurt.
Thanks to @thijsputman for his work on [docker-ble](https://github.com/thijsputman/tc66c-mqtt/blob/main/docker/docker-ble).

**Other Linuxes caveats** - unknown - if you know, let me know. The principe is the same but details may differ. 
If you use AppArmor then you may need custom policy otherwise Bluetooth won't work.

**Don't forget** to mount `/opt/rd-usb/data` via volume otherwise you will lose all your data when container gets 
recreated/updated.

**docker-compose.yml**

```
version: '3.7'
services:
  rd-usb:
    container_name: rd-usb
    image: kolinger/rd-usb:latest
    restart: unless-stopped
    security_opt:
      - apparmor=docker-ble
    environment:
      - TZ=Etc/UTC # change your timeozne (TZ identifier here), for example Europe/London
      - ARGS=
    volumes:
      - /var/run/dbus/system_bus_socket:/var/run/dbus/system_bus_socket
      - /some/place/to/store/data:/opt/rd-usb/data
    # you need something like these examples if you want to use serial port based communication
    #devices:
    #  - /dev/ttyUSB0
    #  - /dev/rfcomm0
    network_mode: host
```

Change `/some/place/to/store/data` to your liking.

If you want to use TC66C USB or UMxxC Serial then you also need to pass specific serial device to docker. For TC66C USB
it would be something like /dev/ttyUSB0 and for UMxxC Serial something like /dev/rfcomm0.

Optional arguments for `web.py` can be passed to `ARGS` environment variables - for example `ARGS=1888` will 
change listening port to 1888, `ARGS="1888 --prefix /rd-usb"` will set path prefix for proxy purposes.

Custom data directory
--
Custom data directory can be specified with `--data-dir` option.

For example `rd-usb.exe --data-dir C:/rd-usb` will place all configuration and data files in `C:/rd-usb` directory.
This can be used together with shortcut, just make sure you escape path with quotes properly, for example:
`"C:\your\path\rd-usb.exe" --data-dir "C:\your\path\data"`.

Custom export
--

Application has basic CSV export built-in. For more advanced use-cases external script can be used.

External program/script can be specified with `--on-receive` option.
This script will be executed when new data is received.
New measurements are provided as JSON file. Path of this file is provided as first argument.

For managing overhead another option `--on-receive-interval` is available (default value is 60).
This number specified how often is external script called (in seconds, default value is every minute).
Value 0 means script will be called for every new measurement as they happen.
Script is not called when no new measurements are available.

CLI example: `python web.py --on-receive on-receive.sh`

See `on-receive.sh` or `on-receive.cmd` files for more information how implement this program/script.
Also `on-receive-python-example.cmd` and `on-receive-python-example.py` show more extended example how to call python.

Example structure of JSON file:

```
[
    {
        "timestamp": 1599556295,
        "voltage": 5.12,
        "current": 0.0,
        "power": 0.0,
        "temperature": 25,
        "data_plus": 0.0,
        "data_minus": 0.0,
        "mode_id": 7,
        "mode_name": "DCP1.5A",
        "accumulated_current": 0,
        "accumulated_power": 0,
        "accumulated_time": 0,
        "resistance": 9999.9,
        "name": "measurement name"
    },
    ...
]
```

Reverse proxy
--

If you like to have HTTPS (or use reverse proxy for other reasons) then simple reverse proxy can be used.
All common webservers can do this. Here are examples for nginx.

You can also modify on what address/interface is rd-usb listening by providing `--listen` CLI option
like `--listen 192.168.1.100` where `192.168.1.100` is address of interface you want to listen on.

At root:

````
server {
	listen 443 ssl;
	listen [::]:443 ssl;

	ssl_certificate /your/certificate.crt;
	ssl_certificate_key /your/certificate.key;

	server_name rd-usb;

	location / {
		proxy_pass http://127.0.0.1:5000;
	}
}
````

In path:

````
server {
	listen 443 ssl;
	listen [::]:443 ssl;

	ssl_certificate /your/certificate.crt;
	ssl_certificate_key /your/certificate.key;

	server_name domain.tld;

	# your other things

	location /rd-usb {
		proxy_pass http://127.0.0.1:5000;
	}
}
````

When some prefix/path is used then it needs to be specified as argument `--prefix` when launching rd-usb.
In this example `--prefix /rd-usb` is required resulting in something like `python3 web.py --prefix /rd-usb`.

Note: `rd-usb` should not be exposed on untrusted network or to untrusted users.
Use [HTTP basic auth](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/)
for example as authentication mechanism when running rd-usb at untrusted network.

Systemd service
--

If you would like to have rd-usb start automatically on unix systems, you can grab the example service files,
drop them in `/etc/systemd/system/` respectively:

/etc/systemd/system/rfcomm0.service

```
[Unit]
Description=rfcomm0
After=bluetooth.target

[Service]
Type=oneshot
User=root
ExecStart=/usr/bin/rfcomm bind 0 <METER_BT_MAC_ADDRESS>
ExecStart=/usr/bin/chown root:dialout /dev/rfcomm0
ExecStop=/usr/bin/rfcomm release 0
RemainAfterExit=yes
Restart=no

[Install]
WantedBy=multi-user.target
```

```
systemctl enable rfcomm0.service
systemctl start rfcomm0.service
```

/etc/systemd/system/rd-usb.service

```
[Unit]
Description=rd-usb
After=network.target
After=rfcomm0.service

[Service]
Type=simple
WorkingDirectory=</path/to/rd-usb>
ExecStart=/usr/bin/python3 web.py --daemon
User=www-data
Group=www-data
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

```
systemctl enable rd-usb.service
systemctl start rd-usb.service
```

Don't forget to add user of `rd-usb.service` to `dialout` group to enable access to `/dev/rfcomm0` for communication.
In this example you need to add `www-data` to `dialout` like this `usermod -aG dialout www-data`.
Webserver as reverse proxy in front of `rd-usb.service` is suggested (like nginx, apache2, ...).

Development
--

### Building binaries

1. Install pyinstaller: `pip install pyinstaller` (4.x) and [NSIS](https://nsis.sourceforge.io/Download) installer
2. Generate binaries:
    - `pyinstaller pyinstaller-cli.spec`
    - `pyinstaller pyinstaller.spec`
    - `makensis.exe installer.nsi`
    - or use `build.cmd`
3. Binaries will be saved in `dist` directory
