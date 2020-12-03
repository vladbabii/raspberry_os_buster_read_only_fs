## Edit boot config txt

Add to /boot/config.txt
```
# Disable Bluetooth
dtoverlay=disable-bt
```


## Disable Services

```
systemctl disable hciuart.service
systemctl disable bluealsa.service
systemctl disable bluetooth.service
```


## Stop Services

```
systemctl stop hciuart.service
systemctl stop bluealsa.service
systemctl stop bluetooth.service
```
