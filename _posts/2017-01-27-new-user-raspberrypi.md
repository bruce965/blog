---
title: Creating a New User on the Raspberry Pi
date: 2017-01-27 08:51:37 +0100
categories: [Software]
tags: [Raspberry Pi, Linux]
---

Creating a new user on a Linux machine is a straightforward operation, assuming you know the command line and you remember the list of commands required.

I always find myself looking for this list, which I'm publishing here once and for all.

Short on time? Go to [TL;DR](#tldr).

This commands must be run as superuser (after running `sudo su` command).

```bash
# Create a new user
useradd bruce965
```

```bash
# Set a password for this user
passwd bruce965
```

```bash
# Make the new user a sudoer (optional)
adduser bruce965 sudo
```

```bash
# Add this user to the same groups as pi (optional)
usermod --append --groups $(groups pi | sed -E "s/^(\S+)\s:\s(.+)$/\2/" | sed "s/\s/,/g") bruce965

# List pi groups with: groups pi
# pi : pi adm dialout cdrom sudo audio video plugdev games users input netdev spi i2c gpio

# Discard user (but keep groups) with: sed -E "s/^(\S+)\s:\s(.+)$/\2/"
# pi adm dialout cdrom sudo audio video plugdev games users input netdev spi i2c gpio

# Replace spaces with commas with: sed "s/\s/,/g"
# pi,adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,input,netdev,spi,i2c,gpio

# Final result should be something like the following
# usermod --append --groups pi,adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,input,netdev,spi,i2c,gpio bruce965
```

```bash
# Change new user's shell to BASH (optional)
usermod --shell /bin/bash bruce965
```

```bash
# Clone pi user's home and assign it to the new user
cp --recursive /home/pi/ /home/bruce965/
chown --recursive bruce965: /home/bruce965/
```

---

### TL;DR

```bash
sudo su
THEUSER=newusername
useradd $THEUSER
passwd $THEUSER
adduser $THEUSER sudo
usermod -a -G $(groups pi | sed -E "s/^(\S+)\s:\s(.+)$/\2/" | sed "s/\s/,/g") $THEUSER
usermod -s /bin/bash $THEUSER
cp -r /home/pi/ /home/$THEUSER/
chown -R $THEUSER: /home/$THEUSER/
```
