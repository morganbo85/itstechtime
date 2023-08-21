#vpn

## Install Required Packages
`sudo apt install network-manager-openvpn network-manager-openvpn-gnome openvpn openvpn-systemd-resolved -y`

## Running OpenVPN Client as Service on Ubuntu 20.0

When you install openvpn package, it creates a `/etc/openvpn/client/` directory into which you can place the OpenVPN client configuration file.

Therefore, copy your OpenVPN configuration file, either .conf or .ovpn, into the OpenVPN client configurations directory.

Note that the configuration files under the `/etc/openvpn/client/` directory should have the .conf suffix. Hence, if the original file is .ovpn, rename it in the destination directory to .conf as shown below.
`sudo cp ~/gentoo.ovpn /etc/openvpn/client/gentoo.conf`

Once the client configuration file is in place, you then start OpenVPN client service. Note that, it is possible to have multiple OpenVPN client configuration files in this directory.

As such, you can use the service, `openvpn-client@{Client-config}.service` to start your OpenVPN client service using a specific configuration file placed on the `/etc/openvpn/client/` directory.

Replace the {Client-config} with the name of your OpenVPN client configuration file, without the suffix, .conf or .ovpn.

For example, to start OpenVPN client service using the gentoo.ovpn, run the service as follows:

```
sudo systemctl start openvpn-client@gentoo.service
````

### To Check Status

```
systemctl status openvpn-client@gentoo.service

● openvpn-client@gentoo.service - OpenVPN tunnel for gentoo

Loaded: loaded (/lib/systemd/system/openvpn-client@.service; indirect; vendor preset: enabled)

Active: active (running) since Sun 2020-06-14 12:30:56 EAT; 5s ago

Docs: man:openvpn(8)

https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage

https://community.openvpn.net/openvpn/wiki/HOWTO

Main PID: 5556 (openvpn)

Status: "Initialization Sequence Completed"

Tasks: 1 (limit: 2315)

CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@gentoo.service

└─5556 /usr/sbin/openvpn --suppress-timestamps --nobind --config gentoo.conf
```

Enable the service to run on system boot to ensure that the VPN connection is initiated automatically on system boot.

```
sudo systemctl enable openvpn-client@gentoo.service
```

Reboot your system and check the status again to confirm.
