Edit sshd_config
```
$ sudo vi /etc/ssh/sshd_config
```

change default ssh port
```
Port 30
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

Change permit root login to no
```
#LoginGraceTime 2m
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```

Change password authentiation to no
```
# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
PermitEmptyPasswords no
```

Auto logout inactive sessions
```
ClientAliveInterval 200
ClientAliveCountMax 3
```

Restart sshd
```
sudo systemctl restart sshd
```

specifing port with ssh
```
$ ssh bmorgan@172.234.16.219 -p 30
```
