Make a directory for ssh keys
```
mkdir ~/.ssh && chmod 700 ~/.ssh
```

Generate your ssh keys
```
~$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/bmorgan/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/bmorgan/.ssh/id_rsa
Your public key has been saved in /home/bmorgan/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:DInVJQYzbC7IqstwtJBarwu8PwUFXtsF5g+IYdd5nUE bmorgan@
The key's randomart image is:
+---[RSA 3072]----+
|   +.+*==o+Eo    |
|  o =+OBoo o     |
| . +o=o+.        |
| .o.. .oo        |
|o.. ..  S.       |
|++ . .           |
|=.+ o            |
|++.o             |
|.++o.            |
+----[SHA256]-----+
```

To view your public ssh key
```
~$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDVbQlIHOowNYM8DParnger1FoqKqtyc7smR0xhCLJcSyhGxF913qiYGpMfThaULOj1ys6xL9XSQRdLFVWu1a/nGpYn6MG+6YSFYyx/uPDgU5rUW0S9VZWvXkcLAq2KryJrlcUtGMDghNKOUULCYMxsechzFe9S2j8HD+hahjfgRBWswNJepgCT4Oz8VGK/2EWwfC6xoWmPt5xKFFCjEdqRK6Rt/E2chYnNQG8rwo9V0xw9ym9rKGd4dv4dlbn7DDmyzbvbnRayTp3Spdlhd5LOPv5U0Wtg2nBZiWIyKM71qeRmPrtCcuWkP/2dURhA1t9CAL0PPBSwGgkwuR7r8p9TlXfBKjxSIDJcScQA1DbrHLa3mb0YTeRzIDbX9EcP9Cn5a9CsJ7XxEwxVChsBLag5hbdVD8ZCVTAvfJZqoaTesG/+Ypj+Qs9JFKfM5wUwv0dHUVgc4lTHspr0sAAp85GvoKs+buz59g60MSUj/nzzfzOaC7uAAKGDxTXYzobIJWU= bmorgan@
```
To copy your ssh key to the server
```
bmorgan@:~/sbin$ ssh-copy-id bmorgan@172.234.16.219
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/bmorgan/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
bmorgan@172.234.16.219's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'bmorgan@172.234.16.219'"
and check to make sure that only the key(s) you wanted were added.

bmorgan@:~/sbin$
```


