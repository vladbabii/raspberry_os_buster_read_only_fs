# TTYD - web-accesible ssh interface

Warning! 
This asumes you're using the "pi" user, change the lines below if you're using something else

## Install
Find the last .arm file from https://github.com/tsl0922/ttyd/releases then createa a folder "ttyd" in your home folder and download that file or  run the commands below

```
cd ~
mkdir ttyd
cd ~/ttyd
wget https://github.com/tsl0922/ttyd/releases/download/1.6.1/ttyd_linux.arm
mv ttyd_linux.arm ttyd
chmod +x ttyd
```

## Test run with password
Remove the "-p user:password" part to run it passwordless

```
cd ~/ttyd
./ttyd -p 8080 bash
```
Open browser and go to http://raspberry_pi_ip:8080/

## Run modes

### run without authentication of any kind
```
..../ttyd -p 8080 bash
```

### run with normal ssh-like auth
```
..../ttyd --credential "username:password" -p 8080 login
```

### run with web authentication
```
..../ttyd -p 8080 login
```

### Only allow kiauh (no shell)
```
..../ttyd -p 8080 /home/pi/kiauh/kiauh.sh
```
and in systemd file change user to "pi" and group to "pi"

## Nginx integration
see https://github.com/tsl0922/ttyd/wiki/Nginx-reverse-proxy

## Systemd
original https://github.com/tsl0922/ttyd/wiki/Systemd-service

but put this into the systemd ttyd.service file

```
[Unit]
Description=TTYD
After=syslog.target
After=network.target

[Service]
ExecStart=/home/pi/ttyd/ttyd -p 8080 login
Type=simple
Restart=always
User=root
Group=root

[Install]
WantedBy=multi-user.target

```

then do 
```
systemctl enable ttyd
systemctl start ttyd
```
