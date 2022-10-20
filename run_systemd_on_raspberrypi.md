# It's Time To Pack Up And How To Use Systemd

###### tags: `nakuja`

Systemd is a system and service manager for Linux operating systems. When run as first process on boot (as PID 1), it acts as an init system that brings up and maintains userspace services.


It has mostly replaced the previous standard for Linux distributions.It is widely used and therefore being acquainted with systemd is definitely worth the effort.It will  also make server administration much easier. Learning about and utilising the systemd tools and daemons can help you realise the power, flexibility, and possibilities it offers, or at the very least make your work easier.

For our server we will  be using a Raspberry Pi 4. This will enable us to test the prowess of systemd.

To setup Raspberry pi for headless support please refer to this [Toms Hardware tutorial](https://www.tomshardware.com/reviews/raspberry-pi-headless-setup-how-to,6028.html)


## Why systemd

The most common criticism levelled about systemd is that it suffers from mission creep and bloat. Subsequent criticism impacts other software that adds systemd dependencies, affecting compatibility with other Unix-like operating systems and making it difficult to shift away from systemd. Concerns have also been expressed regarding Red Hat and its parent firm, IBM, dominating the scene of Linux init systems. Some even question systemd's integrity against attackers, stating that its complexity leads in a vastly expanded attack surface, lowering the platform's overall security.

Systemd, on the other hand, has been lauded by developers and users of distributions who have embraced it for delivering a stable, easier boot up time - a quick out-of-the-box solution to issues that have plagued the Linux community for years. Systemd was the only software suite that enabled dependable parallelism at boot as well as centralised control of processes, daemons, services, and mount points when it was first adopted by most Linux distributions.

"Why systemd?" is a classical question where developers are split down the middle and picking up an argument about it will not last just like the distro's war. But as system developers you have to ensure that you learn how to work with systemd as it is the majority init system in most linux servers. You'll definetly have more consistent adminisration across linux distro.

Reading systemd configuration files is easier as reading plain text. This is one of the key benefits of systemd. We will demonstrate this using a sample configuration file later in this blog.

Go and check out the audit service that helps system administrators check on their system and better understand the system security as it logs the activity of users in this case a hacker. 

## Why not
- Desktop users
- Server! You'll not have a choice

## Demo

Here at Nakuja project we are building a logger for our static test and also base station for the actual flight test.

![](https://i.imgur.com/qtKvapo.png)

Our backend system is mainly of docker services managed with systemd. At first we want to read serial data when we plug in the usb cable from which we will be streaming out data from. The read serial should be initiated after the backend docker services are up and also when the user triggers start logging from our web dashboard.

First we will be writing a python service that reads data from a serial port. 

```python=
#!/usr/bin/env python3
import os
import sys
import traceback
import time
from main import main
from lib import format_exc_for_journald, setup_logging, signalhandler, check_running_instance

# TODO there is a big on environment variables fix it

# If the DEBUG environment variable is set, uses that to set the DEBUG
# global variable
# If the environment variable isn't set, only sets DEBUG to True if we're
# running in a terminal (as opposed to systemd running our script)
if "DEBUG" in os.environ:
    # Use Environment Variable
    if os.environ["DEBUG"].lower() == "true":
        DEBUG = True
    elif os.environ["DEBUG"].lower() == "false":
        DEBUG = False
    else:
        raise ValueError("DEBUG environment variable not set to 'true' or 'false'")
else:
    # Use run mode
    if os.isatty(sys.stdin.fileno()):
        DEBUG = True
    else:
        DEBUG = False

# Script name
script_name = os.path.basename(__file__)
current_dir = os.path.dirname(__file__)

# Get logger
logger = setup_logging(name=script_name, debug=DEBUG)

signalhandler()

# Check if another instance already running #
check_running_instance(filename=script_name)

while True:
    try:
        main(logger=logger)
    except UserWarning as warn:
        logger.error(warn)
        break
    else:
        logger.error(format_exc_for_journald(traceback.format_exc(), indent_lines=False))
        time.sleep(10)


```

What does our main file do?

```python=
import os
import time
import json
import queue
import serial
import threading
from datetime import datetime
from publisher import TestPublisher

# TODO Setup as environment variable
WRITE_FILE_DURATION = 60
ESTIMATED_DURATION = 3600


class FetchData:

    def __init__(self, logger):
        """
        This initializes the fetch data class
        :param logger: Logger
        """
        self.logger = logger
        self.serial = None
        self.find_serial()
        self.publisher = TestPublisher(logger=self.logger)
        self.running = False
        # TODO compare different queues to see which one works best e.g circular queues
        self.dataqueue = queue.Queue()
        # TODO implement cython arrays to boost the efficiency
        self.datapoints = []
        self.maximum_val = 0
        self.cwd = os.path.dirname(__file__)
        self.filename = str(self.cwd) + "/" + datetime.now().strftime("%A_%d_%B_%Y_%H_%M_%S") + ".csv"
        self.readthread = threading.Thread(target=self.read_data)
        self.savethread = threading.Thread(target=self.save_data)
        self.loggingthread = threading.Thread(target=self.check_logging)
        self.stopthread = threading.Thread(target=self.stop)
        self.readthread.setDaemon(True)
        self.savethread.setDaemon(True)
        self.loggingthread.setDaemon(True)
        self.stopthread.setDaemon(True)

    def find_serial(self):
        """
        This method searches the available serial port and see which one it connected and sending data
        It will set serial to the one that is available an plugged in
        :return: None
        """
        SERIAL_PORTS = ['/dev/ttyUSB1', '/dev/ttyUSB0', '/dev/ttyACM1', '/dev/ttyACM0']
        for port in SERIAL_PORTS:
            try:
                self.serial = serial.Serial(port=port, baudrate=9600)
            except serial.serialutil.SerialException as e:
                self.logger.error(e)
            else:
                break

    def check_logging(self):
        """
        This method checks the "sensorData/internals" topic to see if a message has been published to that we can
        start logging data. It will wait for the action: Started logging with logging status set to True
        :return:
        """
        count = 0
        while True:
            try:
                message = json.loads(str(self.publisher.check_message))
                # If we find the message corresponding with start logging we break and return True
                if str(message['data']['action']) == "Started logging" and str(
                        message['data']['logging status']) == "True":
                    break
            except json.decoder.JSONDecodeError as e:
                # TODO find a better way to log the error once
                if count == 0:
                    # Logs the error once
                    self.logger.error("Check logging error: {}".format(e))
                    count = count + 1
            time.sleep(1)
        self.running = True
        return True

    def save_data(self):
        """
        This saves data to the csv file in batches. When we receive an array of data points from the queue we read that
        data while appending it to the file.
        We save the data only if we have started self.running and we stop once we set False to self.running
        """
        self.logger.info("Started saving data thread")
        while self.running:
            time.sleep(1)
            while not self.dataqueue.empty():
                datapoints = self.dataqueue.get()
                maximum_val = max(datapoints)
                if self.maximum_val < maximum_val:
                    self.maximum_val = maximum_val
                with open(self.filename, "a+") as f:
                    for i in datapoints:
                        f.write("{}\n".format(i))
                self.logger.debug("Saved batched data")
                new_filename = str(self.cwd) + "/" + str(maximum_val) + "_" + datetime.now().strftime(
                    "%A_%d_%B_%Y_%H_%M_%S") + ".csv"
                os.rename(self.filename, new_filename)
                self.filename = new_filename
                if not self.running:
                    return

    def read_data(self):
        """
        This reads data from the serial port
        We first reset the input buffer then we started reading data from the serial port
        We send the data to an array as well to our message broker
        After 60 seconds, WRITE_FILE_DURATION we write the data to a queue where we will save it to a csv file
        We read the data only if we have started self.running and we stop once we set False to self.running
        """
        self.logger.info("Started reading data thread")
        self.serial.reset_input_buffer()
        while self.running:
            start_time = time.time()
            elapsed_time = 0
            while elapsed_time <= WRITE_FILE_DURATION:
                try:
                    val = self.serial.readline().decode('utf-8').rstrip()
                except UnicodeDecodeError as e:
                    self.logger.error(e)
                    val = 0.0
                except ValueError as e:
                    self.logger.error(e)
                    val = 0.0
                if elapsed_time == 0:
                    self.logger.info("Inserting the first value")
                self.datapoints.append(val)
                self.publisher.run(val)
                elapsed_time = time.time() - start_time
            self.dataqueue.put(self.datapoints)
            self.datapoints = []
            self.logger.debug("Sent batched data to queue")
            if not self.running:
                return

    def run(self):
        self.logger.info("Started reading serial service")
        self.readthread.start()
        self.savethread.start()
        self.loggingthread.start()
        self.stopthread.start()
        self.readthread.join()

    def stop(self):
        """
        This method checks the "sensorData/internals" topic to see if a message has been published to that we can
        stop logging data. It will wait for the action: Stopped logging with logging status set to False
        """
        count = 0
        while True:
            try:
                message = json.loads(str(self.publisher.check_message))
                if str(message['data']['action']) == "Stopped logging" and str(
                        message['data']['logging status']) == "False":
                    self.logger.info("Stopped Service")
                    break
            except json.decoder.JSONDecodeError as e:
                if count == 0:
                    self.logger.error("Check logging error: {}".format(e))
                    count = count + 1
            time.sleep(1)

        self.running = False
        self.readthread.join()
        self.savethread.join()
        raise UserWarning("Stopped Service")


def main(logger):
    fetch_data = FetchData(logger=logger)
    while True:
        if fetch_data.check_logging():
            break
    fetch_data.run()
    time.sleep(ESTIMATED_DURATION)


```


Most versions of Linux have systemd out of the box, and  if yours does not, just execute the following command to get it:

```bash=
sudo apt-get install -y systemd
```

Simply execute the following command to determine the version of systemd you have:

```bash=
systemd --version
```

### Service file

Service units, which have unit files ending in.service, will be the target unit for service management operations. However, for most service management tasks, you may omit the.service suffix since systemd is sophisticated enough to recognise that you are most likely attempting to operate on a service when using service management commands.



```service
[Unit]

Description=Read the serial port for data from the load cell

[Service]

# Type of service
Type=simple

# Main task
ExecStart=/home/pi/VENVS/n2basestation/bin/python3 /home/pi/NakujaN2/n2Basestation/Services/read_serial/RS_Service.py

# Required to see Python output in logs
Environment="PYTHONUNBUFFERED=x"

# Uncomment this to see debug messages
Environment="DEBUG=true"

# If the script ever dies, restart it
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

- [Unit] - provides general options that are not reliant on the unit's kind.

**Available unit types**

1. Service unit `.service` - A system service.
2. Target unit `.target` - A group of systemd units.
3. Device unit `.device` - A device file recognized by the kernel.
4. Socket unit `.socket` - An inter-process communication socket.

- [unit type] - If a unit has type-specific directives, they are organised in a section titled after the unit type.
- [Service] section, for example, can be found in service unit files.
- [Description] - A concise description of the unit. This text appears in the output of the systemctl status command, for example.
- [After] - Determines the sequence in which units are launched. Only once the units stated in After and are active does the unit begin. In contrast to Requires, After does not explicitly activate the units given. The functionality of the Before option is the inverse of that of the After option.
- [Requires] - Sets the unit's dependencies on other units. The units specified in Requires are activated at the same time as the unit. The unit is not activated if any of the prerequisite units fail to start.


- [Type] - Configures the unit process startup type, which impacts ExecStart's functionality and associated settings. e.g Simple – The default value. 
- [ExecStart] - Specifies the instructions or scripts that will be run when the device is powered on.
- [ExecStop] - Specifies which instructions or scripts should be run when the unit is turned off.
- [ExecReload] - Specifies commands or scripts to be executed when the unit is reloaded.


- [Install] — includes information about unit installation that is used by the systemctl enable and disable commands
- [WantedBy] A list of units that are only slightly dependent on the unit.

For more information check out the reference materials especially [RedHat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd)


The restart flag is set to always because I want my service to restart if the server is rebooted. You may find out more about this by clicking on this link. We must now reload the daemon.
```bash=
sudo systemctl daemon-reload
```

Use the start command to start a systemd service by executing the instructions in the service's unit file.

```bash=
sudo systemctl start read_serial.service
```

As previously stated, systemd understands to check for `.service` files when looking for service management commands, therefore the command might be worded as follows:

```bash=
sudo systemctl start read_serial
```
Our service is now operational. To stop an already operating service, use the stop command instead:

```bash=
sudo systemctl stop read_serial.service
```

You may use the restart command to restart a running service:

```bash=
sudo systemctl restart read_serial.service
```

You must enable them to inform systemd to start services automatically at boot. Use the enable command to start a service at boot:

```bash=
sudo systemctl enable read_serial.service
```

This will build a symbolic link from the system's copy of the service file (often located in /lib/systemd/system or /etc/systemd/system) to the location on disc where systemd looks for autostart files (usually /etc/systemd/system/some_target.target.wants).

To disable the service from starting automatically

```bash=
sudo systemctl disable read_serial.service
```

This will delete the symbolic link indicating that the service should start automatically. Remember that activating a service does not initiate it in the current session. If you want to start the service as well as activate it at boot, you must use both the start and enable commands. You may use the status command to verify the status of a service on your system:
```bash=
systemctl status application.service
```
This will show you the service status, the cgroup hierarchy, and the first few lines of log data. When checking the status of a Nginx server, for example, you could see something like this:

All this operations are managed by a makefile

```make
SERVICES = read_serial docker_backend teststandAPI
START = $(addprefix start_,$(SERVICES))
LOGS = $(addprefix logs_,$(SERVICES))
STATUS = $(addprefix status_,$(SERVICES))
STOP = $(addprefix stop_,$(SERVICES))
CLEAN = $(addprefix clean_,$(SERVICES))

define start_service
	$(eval svc=$(subst start_,,$(1)))
	sudo systemctl stop $(svc).service
	sudo cp $(svc)/$(svc).service /etc/systemd/system/
	sudo systemctl start $(svc).service
endef

define status
	$(eval svc=$(subst status_,,$(1)))
	sudo systemctl status $(svc).service
endef

define clean
	$(eval svc=$(subst clean_,,$(1)))
	sudo rm $(svc)/*.csv
endef

define stop
	$(eval svc=$(subst stop_,,$(1)))
	sudo systemctl stop $(svc).service
endef

define watchlogs
	$(eval svc=$(subst logs_,,$(1)))
	journalctl -u $(svc) -f
endef

$(LOGS):
	$(call watchlogs,$(@))

$(STATUS):
	$(call status,$(@))

$(CLEAN):
	$(call clean,$(@))

$(START):
	$(call start_service,$(@))

$(STOP):
	$(call stop,$(@))
```

So we have start the application
```bash=
make start_read_serial
```

also to stop the service you 

```bash=
make stop_read_serial
```

If you want to watch the logs streaming from the service you can use

```bash=
make logs_read_serial
```

For more information check out our [Github Repository](https://github.com/nakujaproject/n2Basestation)


## References
1. http://0pointer.de/public/systemd-man/systemd.html
2. https://www.linuxvoice.com/linux-101-get-the-most-out-of-systemd/
3. https://www.cvedetails.com/vulnerability-list/vendor_id-7971/product_id-38088/Freedesktop-Systemd.html
4. https://wiki.debian.org/Debate/initsystem/systemd
5. https://fedoraproject.org/wiki/F15_one_page_release_notes
6. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd
7. https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files
8. https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units