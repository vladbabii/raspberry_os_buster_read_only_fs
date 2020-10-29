# Nginx with logs in tmpfs

Nginx wants to write in /var/log/nginx/ which won't exist with tmps so 2 files need to be edited

(1) /etc/init.d/nginx - add */bin/mkdir -p /var/log/nginx* near the top of the file
```
#!/bin/sh

### BEGIN INIT INFO
# Provides:       nginx
# Required-Start:    $local_fs $remote_fs $network $syslog $named
# Required-Stop:     $local_fs $remote_fs $network $syslog $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the nginx web server
# Description:       starts nginx using start-stop-daemon
### END INIT INFO

/bin/mkdir -p /var/log/nginx

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/sbin/nginx
NAME=nginx
DESC=nginx

```

(2) /etc/systemd/system/multi-user.target.wants/nginx.service add a new ExecStartPre line like below with the mkdir command
```
[Service]
Type=forking
PIDFile=/run/nginx.pid
ExecStartPre=- /bin/mkdir -p /var/log/nginx
ExecStartPre=/usr/sbin/nginx -t -q -g 'daemon on; master_process on;'
ExecStart=/usr/sbin/nginx -g 'daemon on; master_process on;'
ExecReload=/usr/sbin/nginx -g 'daemon on; master_process on;' -s reload
ExecStop=-/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid
TimeoutStopSec=5
KillMode=mixed
```
