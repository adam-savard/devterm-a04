# DevTerm Tweaks

Below are some of the tweaks I made to the DevTerm a04 to make it a much better text-driven device.

## Before starting
You might see some guides that suggest updating the system immediately; unless you enjoy waiting 20+ minutes waiting for locales to generate, run the [command listed here first!](#remove-unneeded-locales)

## Enable SSH server

SSH is disabled by default and is extremely useful in the event that something goes wrong. Enable it by entering the following:

```shell
sudo rm -f /etc/ssh/sshd_not_to_be_run
sudo systemctl enable ssh
sudo systemctl start ssh
```

You likely will want to [change your hostname](#set-hostname) after this in order to more easily log in to your DevTerm:

```shell
# As an example, this is normally what you would do:
ssh user@192.168.2.138 # or some other IP
# You can instead do this:
ssh user@devterm # or whatever you set your hostname to
```

## Remove unneeded locales
There are a _lot_ of locales that aren't needed on this device; as I only need English, I decided to disable all the others. You can modify the command below to suit your needs.

```shell
sudo locale-gen --purge en_US # Add any locales here that you want to keep
```

This will _greatly_ speed up package installation.

## Enable all CPU cores

Out of the box, only 2/4 CPU cores are enabled, which is far from ideal. Actions take _far_ too long to perform, compilation is a nightmare, etc. To make things more bearable, follow the steps below:

First, install the devterm gearbox:

```shell
sudo apt update && sudo apt upgrade -y
sudo apt install devterm-a04-gearbox
```
Then, append this to your `.bashrc`:

```shell
if [ $( nproc ) == 2 ]; then
	sudo /usr/bin/devterm-a04-gearbox -s 5
fi
```

This will keep temperatures under control and enable the other 2 CPU cores. While this may be possible with a cron job or systemd, I haven't had any luck. This should still only run once, and will do so on the first shell you launch.

## Remove the GUI

One thing about the DevTerm; the built in trackball is terrible. I find myself more at home in the terminal, so I got rid of the GUI (which also saves a significant amount of space on the SD card).

The following commands will remove the GUI and boot you into text mode:

```shell
# All commands listed need to be run as root; remember to exit from this shell at the end.
sudo su
# Remove all XFCE software
apt-get purge 'xfce4*' 
# The command below will purge ALL GNOME software that doesn't get cleared by the commands above
apt-get purge $(apt-cache depends gnome | grep Depends | awk '{print $2}')
# The command will remove gdm3 if it's being stubborn
apt-get purge gdm3
# If gdm3 is being *really* stubborn, you'll need to delete this file and re-run the purge command
rm -rf /var/lib/dpkg/info/gdm3.* && apt-get purge gdm3
# Remove all X11 apps (transmission added because it is stubborn)
apt-get purge 'libx11*' x11-common transmission
# exit root shell
exit
```
To make absolutely sure that you boot into text mode:

```shell
sudoedit /etc/X11/default-display-manager
```

Empty the file completely, and save. Reboot at this point:


```shell
systemctl reboot
```

## Install `tmux`

Multitasking is important, adn the screen has more than enough horizontal space to do it. `tmux` will become your best friend.

```shell
sudo apt install tmux
```

To enter into a session, type `tmux`.

A short command list:

- Open terminal to right:
	- `ctrl+b+%` (this also requires the shift key)
- Open terminal on the bottom:
	- `ctrl+b+"` (this also requires the shift key)
- Switch between terminals:
	- `ctrl+b+arrowKey`

## Set Hostname

I prefer the hostname `devterm` so I can quickly access it on my network, but the hostname can be anything you like.

To set the hostname:

```shell
hostnamectl set-hostname devterm # replace devterm with the hostname you want
```

## Change username/home folder

_This section was adapted from [This forum post](https://askubuntu.com/questions/34074/how-do-i-change-my-username)_

To change your username, you'll need to run the following while logged in as another user. I recommend creating a user called `temp` and removing them immediately after.

Follow the directions below:

```shell
# Run this as the cpi user
sudo adduser temp
# Give the temp user sudo rights
sudo adduser temp sudo
# create a password for temp
sudo passwd temp
# Exit this shell
exit
```
Log in using `temp` and execute the following:

```shell
# Change the name of the cpi user
sudo usermod -l newUserName cpi
# Change the home folder
sudo usermod -d /home/newUserName -m newUserName
# Exit this shell
exit
```
Log in using your new username; don't forget to change your password!

```shell
sudo passwd newUserName
```

Finally, remove the temporary user:

```shell
sudo deluser temp
sudo rm -r /home/temp
```

## Don't require password for `sudo` commands (NOT RECOMMENDED)

This is purely because I am lazy and don't like typing out my password every time I want to run a command as root. This is _NOT_ recommended, especially if you bring your system out with you/travel/have sensitive information.

Again, let me be clear: if you are inexperienced or do not know what you are doing, _DO NOT_ do this. This _WILL_ make your system less secure.

Open the sudoers file with visudo:

```shell
# When prompted, I recommend nano, but any text editor will do
sudo visudo
```

Navigate to the bottom of the file, and you'll notice a line referring to the `cpi` user. Change this line to the following:

```shell
YOUR_USER_NAME ALL=(ALL) NOPASSWD: ALL
```

Save the file, and exit the editor. To test that this worked:

```shell
# exit the current shell
exit
# log in and then type the following:
sudo su
# you should not see a prompt for a password if successful
```

# Install Docker

While you won't be able to run all available docker containers, many _can_ be run, giving you a portable server. Enter the following:

```shell
# Get Docker Script using CURL
curl -fsSL https://get-docker.com -o get-docker.sh
# make script executable after you inspect it
chmod +x ./get-docker.sh
# Install docker
./get-docker.sh
```
