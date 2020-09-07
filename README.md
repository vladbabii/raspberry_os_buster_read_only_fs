# Read only filesystem for Raspberry PI OS - Buster


# Initial setup

* write image file
* create a file named "ssh" on /boot partition
* for wifi, creata a file "wpa_supplicant.conf" on /boot

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<Insert 2 letter ISO 3166-1 country code here>

network={
 ssid="<Name of your wireless LAN>"
 psk="<Password for your wireless LAN>"
}
```


# Do over ssh

Run as root
```
apt-get remove --purge triggerhappy logrotate dphys-swapfile
apt-get autoremove --purge
```

Edit /boot/cmdline.txt
```
fastboot noswap ro
```

Log manager change (as root)
```
apt-get install busybox-syslogd
apt-get remove --purge rsyslog
```

Edit /etc/fstab and add the ,ro flags to all block devices and add tmpfs
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

Source:
* https://medium.com/swlh/make-your-raspberry-pi-file-system-read-only-raspbian-buster-c558694de79
