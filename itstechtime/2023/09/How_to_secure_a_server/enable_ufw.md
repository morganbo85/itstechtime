Enable ufw
```
$ sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

Allow the non default ssh port
```
$ sudo ufw allow 30
Rule added
Rule added (v6)
```
