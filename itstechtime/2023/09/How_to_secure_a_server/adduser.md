Create a user
```
root@localhost:~# adduser bmorgan
Adding user `bmorgan' ...
Adding new group `bmorgan' (1000) ...
Adding new user `bmorgan' (1000) with group `bmorgan' ...
Creating home directory `/home/bmorgan' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for bmorgan
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
root@localhost:~#
```

add to the sudo group
```
root@localhost:~# usermod -aG sudo bmorgan
root@localhost:~# id bmorgan
uid=1000(bmorgan) gid=1000(bmorgan) groups=1000(bmorgan),27(sudo)
root@localhost:~#
```

