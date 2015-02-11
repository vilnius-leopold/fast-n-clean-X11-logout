# fast-n-clean-X11-logout
is a DIY manual for convenient, fast and graceful logout/restart/reboot of your machine.
Tested with [openbox](http://openbox.org) wm on [ArchLinux](https://www.archlinux.org/) _without_ graphical login manager (startx straight into wm). Amend it to make it work with your setup
## Setup
1. __create logout script__: install dependencies, write script(s), make executable, add action to menu
2. __amend .xinitrc__: if necessary remove `exec`, add logout status handler
3. __set sudo permissions__: allow your user to shutdown/reboot your machine without root permissions

### 1. logout script
Amend for logout/reboot/shutdown.
Requires [close-all-windows](https://github.com/vilnius-leopold/close-all-windows).

cat bin/reboot-gracefully
```bash
#!/bin/bash

# write exit status to file
echo 'REBOOT' > /home/<username>/bin/.logout-status

# close windows and logout of openbox
close-all-windows && openbox --exit
```
Make it executable `chmod +x bin/reboot-gracefully`

Run this from your openbox menu or from where ever.

cat .config/openbox/menu.xml
```xml
<item label="Reboot">
	<action name="Execute">
		<command>/home/<username>/bin/reboot-gracefully</command>
	</action>
</item>
```

### 2. .xinitrc
Add to .xinitrc
```sh
# IMPORTANT: do NOT use `exec`
# to call your window manager
# else the window manager will
# take over the process and
# the .xinitrc logout script
# that follows will not execute

openbox-session

# .xinitrc continues 
# execution here
# after logout when openbox exits
LOGOUT_STATUS=$(cat /home/<username>/bin/.logout-status)

if [ "$LOGOUT_STATUS" == "REBOOT" ]; then
	sudo reboot
fi

if [ "$LOGOUT_STATUS" == "SHUTDOWN" ]; then
	sudo shutdown -h now
fi
```

### 3. sudo permissions
Setup sudo to shutdown/reboot machine without root permissions
```
Cmnd_Alias POWER = /usr/bin/reboot, /usr/bin/shutdown -h now

<username> localhost=NOPASSWD: POWER
```
