To install
```
sudo apt update
sudo apt install unattended-upgrades
```

Check that it's running
```
sudo systemctl status unattended-upgrades.service
```

If it is running you should see something similar to this
```
root@localhost:~# sudo systemctl status unattended-upgrades.service
● unattended-upgrades.service - Unattended Upgrades Shutdown
     Loaded: loaded (/lib/systemd/system/unattended-upgrades.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2023-09-09 15:21:23 UTC; 1min 2s ago
       Docs: man:unattended-upgrade(8)
   Main PID: 653 (unattended-upgr)
      Tasks: 2 (limit: 2256)
     Memory: 11.7M
     CGroup: /system.slice/unattended-upgrades.service
             └─653 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal

Sep 09 15:21:23 localhost systemd[1]: Started Unattended Upgrades Shutdown.
root@localhost:~#
```

To edit the config file
```
sudo vi /etc/apt/apt.conf.d/50unattended-upgrades
```

Restart the service for changes to take effect
```
sudo systemctl restart unattended-upgrades.service
```
