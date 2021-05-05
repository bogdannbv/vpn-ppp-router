# VPN PPP Router
Some simple `ip-up` and `ip-down` scripts with a shared configuration.

> :warning: **Developed and tested only on macOS**. Support for other `*nix` operating systems to come!

## Prerequisites

* `macOS` `^11.3.0` (not tested on previous versions, your mileage may vary).

## Getting started
After downloading and extracting the [latest version](https://github.com/bogdannbv/vpn-ppp-router/releases/latest), navigate inside the project directory.
```shell
cd /path/to/vpn-ppp-router
```

Rename the example `ip.config.example` file to `ip.config`.
```shell
mv ip.config.example ip.config
```

## Configuration
Inside the `ip.config` file you'll have to replace the `REMOTE_IP` value with the remote IP of your `PPP` interface.
To find the remote IP, connect to the VPN and run the following command:
```shell
ifconfig | grep -A1 ppp
```

After running it, you'll get an output similar to this one:
```text
ppp0: flags=8051<UP,POINTOPOINT,RUNNING,MULTICAST> mtu 1280
	inet 10.50.1.22 --> 10.50.1.0 netmask 0xffffff00
```

In this case, `10.50.1.0` would be the `REMOTE_IP`.

In addition to `REMOTE_IP`, you'll also need to configure the `ROUTES`.

The `ROUTES` values are defined using the [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) notation. \
There are various online tools to help you convert IPv4 ranges to `CIDR` (e.g.: https://www.ipaddressguide.com/cidr).

For a single IPv4 you can essentially just slap a `/32` at the end and call it a day :grin: (e.g.: `192.168.1.113` -> `192.168.1.113/32`).

To sum it all up, based on the examples above, the `ip.config` file will look something like this:
```shell
#!/bin/bash

REMOTE_IP=10.50.1.0

ROUTES=(
  "192.168.1.113/32"
)
```

## Installation
Copy or move `ip-up`, `ip-down` and `ip.config` to `/etc/ppp`.
```shell
sudo cp ip-up ip-down ip.config /etc/ppp
```

Set the permissions.
```shell
sudo chmod 755 /etc/ppp/ip-up /etc/ppp/ip-down
```

## Trying it out
If everything went well, after reconnecting/connecting to your VPN all the traffic to the IP ranges configured in `ip.config`
should be routed through your `PPP` interface.

To test if the routes were added correctly you can run the following command:
```shell
netstat -rn | grep ppp
```

The output should be similar to this one:
```text
default            link#13            UCSIg         ppp0
1.1.1.1            link#13            UHWIig        ppp0
192.168.1.113/32   ppp0               USc           ppp0
10.50.1.0          10.50.1.22         UH            ppp0
10.50.1.22         ppp0               USc           ppp0
224.0.0/4          link#13            UmCSI         ppp0
239.255.255.250    link#13            UHmW3I        ppp0      3
255.255.255.255/32 link#13            UCSI          ppp0
```

As you can see on the third line, our route has been added correctly.
```text
192.168.1.113/32   ppp0               USc           ppp0
```