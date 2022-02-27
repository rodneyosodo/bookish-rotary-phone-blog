# It's Time To Pack Up And How To Use Systemd

Systemd is a system and service manager for Linux operating systems. When run as first process on boot (as PID 1), it acts as init system that brings up and maintains userspace services.


It has mostly replaced the previous standard for Linux distributions. Because of its widespread use, being acquainted with systemd is definitely worth the effort, as it will make server administration much easier. Learning about and utilising the systemd tools and daemons can help you realise the power, flexibility, and possibilities it provides, or at the at least make your work easier.

For our server we will  be using a raspberry pi 4. This will enable us to tests the prowess of systemd.

To setup Raspberry pi for headless support please refer to this [Toms Hardware tutorial](https://www.tomshardware.com/reviews/raspberry-pi-headless-setup-how-to,6028.html)


## Why systemd

The most common criticism levelled about systemd is that it suffers from mission creep and bloat. Subsequent criticism impacts other software that adds systemd dependencies, affecting compatibility with other Unix-like operating systems and making it difficult to shift away from systemd. Concerns have also been expressed regarding Red Hat and its parent firm, IBM, dominating the scene of Linux init systems. Some even question systemd's integrity against attackers, stating that its complexity leads in a vastly expanded attack surface, lowering the platform's overall security.

Systemd, on the other hand, has been lauded by developers and users of distributions who have embraced it for delivering a stable, easier boot up time, quick out-of-the-box solution to issues that have plagued the Linux community for years. Systemd was the only software suite that enabled dependable parallelism at boot as well as centralised control of processes, daemons, services, and mount points when it was first adopted by most Linux distributions.

Why systemd is a classical question where developers are split between and picking up an argument about it will not last just like the distro's war. But as system developers you have to ensure that you learn how to work with systemd as it is the majority init system in most linux servers. You'll definetly have more consistent adminisration across linux distro.

Reading systemd configuration files is easier as reading plain text. This is one of the key benefits of systemd. We will look at a sample configuration file later on.

Go an check out the audit service that helps system administrators check on their system and better understand the system security as it logs the activity of users in this case a hacker. 

## Why not
- Desktop users
- Server you'll not have a choice

## Theory

## Demo

Almost all versions of Linux come with systemd out of the box, but if your’s didn’t come with it then you can simply run the following command:
```bash=
sudo apt-get install -y systemd
```

To check which version of systemd you have simply run the command:

```bash=
systemd --version
```

### Service file

The target unit for service management activities will be service units, which have unit files with the suffix.service. However, for most service management tasks, you may omit the.service suffix since systemd is sophisticated enough to recognise that you are most likely attempting to operate on a service when using service management commands.



```

```

[Unit] — contains generic options that are not dependent on the type of the unit.

**Available unit types**

1. Service unit `.service` - A system service.
2. Target unit `.target` - A group of systemd units.
3. Device unit `.device` - A device file recognized by the kernel.
4. Socket unit `.socket` - An inter-process communication socket.

- [unit type] — if a unit has type-specific directives, these are grouped under a section named after the unit type. For example, service unit files contain the [Service] section

- - [Description] - A meaningful description of the unit. This text is displayed for example in the output of the systemctl status command.


- - [After] - Defines the order in which units are started. The unit starts only after the units specified in After are active. Unlike Requires, After does not explicitly activate the specified units. The Before option has the opposite functionality to After.

- - [Requires] - Configures dependencies on other units. The units listed in Requires are activated together with the unit. If any of the required units fail to start, the unit is not activated.


- [Type] - Configures the unit process startup type that affects the functionality of ExecStart and related options. e.g Simple – The default value. 



- - [ExecStart] - Specifies commands or scripts to be executed when the unit is started.

- - [ExecStop] - Specifies commands or scripts to be executed when the unit is stopped.

- - [ExecReload] - Specifies commands or scripts to be executed when the unit is reloaded.


- [Install] — contains information about unit installation used by systemctl enable and disable commands

- - [WantedBy] A list of units that weakly depend on the unit. When this unit is enabled, the units listed in WantedBy gain a Want dependency on the unit.

For more information check out the reference materials especially [RedHat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd)


Restart flag is set to always because I want to restart my service if the server gets restarted. For more information on this, you can go to this link. Now we need to reload the daemon.

```bash=
sudo systemctl daemon-reload
```

To start a systemd service, executing instructions in the service’s unit file, use the start command. 

```bash=
sudo systemctl start test.service
```

As we mentioned above, systemd knows to look for *.service files for service management commands, so the command could be typed like this:


```bash=
sudo systemctl start application
```
Now our service is up and running.


To stop a currently running service, you can use the stop command instead:

```bash=
sudo systemctl stop application.service
```

To restart a running service, you can use the restart command:

```bash=
sudo systemctl restart application.service
```

To tell systemd to start services automatically at boot, you must enable them.

To start a service at boot, use the enable command:

```bash=
sudo systemctl enable application.service
```

This will create a symbolic link from the system’s copy of the service file (usually in /lib/systemd/system or /etc/systemd/system) into the location on disk where systemd looks for autostart files (usually /etc/systemd/system/some_target.target.wants).

To disable the service from starting automatically

```bash=
sudo systemctl disable application.service
```

This will remove the symbolic link that indicated that the service should be started automatically.

Keep in mind that enabling a service does not start it in the current session. If you wish to start the service and also enable it at boot, you will have to issue both the start and enable commands.

To check the status of a service on your system, you can use the status command:

```bash=
systemctl status application.service
```

This will provide you with the service state, the cgroup hierarchy, and the first few log lines.

For instance, when checking the status of an Nginx server, you may see output like this:

## References
1. http://0pointer.de/public/systemd-man/systemd.html
2. https://www.linuxvoice.com/linux-101-get-the-most-out-of-systemd/
3. https://www.cvedetails.com/vulnerability-list/vendor_id-7971/product_id-38088/Freedesktop-Systemd.html
4. https://wiki.debian.org/Debate/initsystem/systemd
5. https://fedoraproject.org/wiki/F15_one_page_release_notes
6. https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd
7. https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files
8. https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units