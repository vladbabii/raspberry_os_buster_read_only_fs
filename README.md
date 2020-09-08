# Read only filesystem for Raspberry PI OS - Buster

This works on Buster releases of raspberry pi - tested on a PI 4 with a fresh install of the lite version

# Initial setup

* write image file
* create a file named "ssh" on /boot partition
* for wifi, creata a file "wpa_supplicant.conf" on /boot and fill in country code ( US, DE, etc) and your wifi name (ssid) and password

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<Insert 2 letter ISO 3166-1 country code here>

network={
 ssid="<Name of your wireless LAN>"
 psk="<Password for your wireless LAN>"
}
```

# First boot
Run raspi-config and configure everything as you want to
* memory split
* localisation
* time zone
* ...
then reboot

# Second boot

Update packages
```
apt-get update
apt-get upgrade
```

Run as root
```
apt-get remove --purge triggerhappy logrotate dphys-swapfile
apt-get autoremove --purge
```

Edit /boot/cmdline.txt and add at the end of the file
```
fastboot noswap ro
```

Log manager change (as root)
```
apt-get install busybox-syslogd
apt-get remove --purge rsyslog
```

Edit /etc/fstab and add the ",ro" flags to all block devices that start with PARTUUID=...
```
proc                  /proc     proc    defaults             0     0
PARTUUID=fb0d460e-01  /boot     vfat    defaults,ro          0     2
PARTUUID=fb0d460e-02  /         ext4    defaults,noatime,ro  0     1
```


Edit /etc/fstab and add tmpfs
```
tmpfs        /tmp            tmpfs   nosuid,nodev         0       0
tmpfs        /var/log        tmpfs   nosuid,nodev         0       0
tmpfs        /var/tmp        tmpfs   nosuid,nodev         0       0
```

Move files
```
rm -rf /var/lib/dhcp /var/lib/dhcpcd5 /var/spool /etc/resolv.conf
ln -s /tmp /var/lib/dhcp
ln -s /tmp /var/lib/dhcpcd5
ln -s /tmp /var/spool
touch /tmp/dhcpcd.resolv.conf
ln -s /tmp/dhcpcd.resolv.conf /etc/resolv.conf
```

System random seed
```
rm /var/lib/systemd/random-seed
ln -s /tmp/random-seed /var/lib/systemd/random-seed
```

Edit /lib/systemd/system/systemd-random-seed.service , add line
```
ExecStartPre=/bin/echo "" >/tmp/random-seed
```
under [Service] section like this
```
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStartPre=/bin/echo "" >/tmp/random-seed
ExecStart=/lib/systemd/systemd-random-seed load
ExecStop=/lib/systemd/systemd-random-seed save
TimeoutSec=30s
```

Edit /etc/bash.bashrc and add the lines
```
set_bash_prompt() {
    fs_mode=$(mount | sed -n -e "s/^\/dev\/.* on \/ .*(\(r[w|o]\).*/\1/p")
    PS1='\[\033[01;32m\]\u@\h${fs_mode:+($fs_mode)}\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
}
alias ro='sudo mount -o remount,ro / ; sudo mount -o remount,ro /boot'
alias rw='sudo mount -o remount,rw / ; sudo mount -o remount,rw /boot'
PROMPT_COMMAND=set_bash_prompt
```

Create /etc/bash.bash_logout
```
mount -o remount,ro /
mount -o remount,ro /boot
```

Reboot and enjoy!


# How to use

Connect via ssh and type "rw" to make the filesystem writable again and install anything you want.

Type "ro" to make the filesystem readonly again - it can take some time untill all writes are finished so be a little patient.



# Sources
* https://media.ccc.de/v/30C3_-_5294_-_en_-_saal_1_-_201312291400_-_the_exploration_and_exploitation_of_an_sd_memory_card_-_bunnie_-_xobs#t=2
* https://learn.adafruit.com/read-only-raspberry-pi
* https://medium.com/swlh/make-your-raspberry-pi-file-system-read-only-raspbian-buster-c558694de79

