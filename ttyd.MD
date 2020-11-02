# TTYD - web-accesible ssh interface

Warning! 
1. This asumes you're using the "pi" user, change the lines below if you're using something else
2. This gives ssh access with simple web authentication, it's not very secure

## Install
Find the last .arm file from https://github.com/tsl0922/ttyd/releases then createa a folder "ttyd" in your home folder and download that file 

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
./ttyd -p user:password 8080 bash
```
Open browser and go to http://raspberry_pi_ip:8080/

## Nginx integration
see https://github.com/tsl0922/ttyd/wiki/Nginx-reverse-proxy
