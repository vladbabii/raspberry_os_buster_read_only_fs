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
