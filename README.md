 # NordVPN Wireguard Docker Client (Nordlynx Alternative) HOW-TO

## Table of Contents

- [Description](#description)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Set-Up](#setup)
  - [Create config file](#create-config)
- [Usage](#usage)
- [Acknowledgements](#acknowledgements)

## Description

Detailed walkthrough of how to create your own NordVPN wireguard configuration file for use with Docker Wireguard client or others

## Getting Started

So you are a NordVPN customer who, for whatever reason, does not want to use their NordVPN local client to access their Wireguard servers.

Maybe you want to set up a Docker client for your other Docker containers?  Maybe you just dont want to run nordvpn's binaries at all times?

Currently, NordVPN refuses to provide a Wireguard configuration file that you can use to access their wireguard servers with your own Wireguard client application.

Here I'll attempt to walk you through the steps to create your own NordVPN wireguard configuration file, which you can drop in to various instances of Wireguard (in this example, we'll use Docker).

### Prerequisites

Linux OS (ubuntu in this example)

NordVPN Account

### Setup

1. Install following packages on your machine:

    wireguard: ```sudo apt install wireguard```

    curl: ```sudo apt install curl```

    jq: ```sudo apt install jq```

    nettools: ```sudo apt install net-tools```

    nordvpn: ```sh <(curl -sSf https://downloads.nordcdn.com/apps/linux/install.sh)```

2. Log in to your nordvpn app via this command:
```sudo nordvpn login```

3. Change connection protocol to nordlynx:
```sudo nordvpn set technology nordlynx```

4. Connect to your preferred server
```sudo nordvpn c nl #to connect Nederland as an example```

5. Now run command below and write down the ip somewhere, you'll need it later. (it will be your wireguard interface ip)
```ifconfig nordlynx```

<img src="https://forum.openwrt.org/uploads/default/original/3X/d/2/d203de32a20b063ce11f03407538c96abb4bc991.png">

6. Use the following command to get your private key:

```sudo wg show nordlynx private-key```

output should be something like this:

<img src="https://forum.openwrt.org/uploads/default/original/3X/3/8/38407e183d31e9b52617f31d70424047156d3a1e.png">

**Never share your private key with anyone!**

7. Now get your fastest Nordvpn server:

```curl -s "https://api.nordvpn.com/v1/servers/recommendations?&filters\[servers_technologies\]\[identifier\]=wireguard_udp&limit=1"|jq -r '.[]|.hostname, .station, (.locations|.[]|.country|.city.name), (.locations|.[]|.country|.name), (.technologies|.[].metadata|.[].value), .load'```

<img src="https://forum.openwrt.org/uploads/default/original/3X/1/c/1c3402ae4864e6499a67b9e1cc4fb6f4a97b79f4.png">

output should be like above picture which includes
nl826.nordvpn.com 56 #your endpoint host
178.239.173.207 #endpoint host ip
Amsterdam #city
Netherlands #country
5PXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXF30= #public key which is different for each server
9 #server load at the time

**Congrats, you now have all the information needed to create your own wireguard config file!**

## Create-config

1. Create a file named wg0.conf with your favourite text editor (I like nano):
```nano wg0.conf```
2. Add the following text, replacing "xxx" where indicated:
```
[Interface]
Address = 10.5.0.2/32 #interface found in step 5, replace if different for you
PrivateKey = xxx #private key found in step 6
DNS = 9.9.9.9 #dns provider of your choice, here i am using quad9

[Peer]
PublicKey = xxx #public key of chosen server, found in step 7
#PresharedKey = [Pre-shared key, same for server and client] # pre-shared key doesnt seem to be needed for nordvpn servers
Endpoint = xxx:51820 #IP address of fastest server found in step 7, using port 51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 21 ```
