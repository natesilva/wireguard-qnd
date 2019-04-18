# Wireguard Q-n-D

Unofficial quick-n-dirty scripts for setting up a Wireguard VPN server.

## About Wireguard

Wireguard is a fast, modern, and secure VPN tunnel. The Wireguard project can be found at https://wireguard.io/.

## About these scripts

These scripts are one person’s attempt to build a simple and easy-to-manage VPN server using Wireguard.

Each Wireguard client is a peer. This gives the sytem enormous flexibility in how it can be used. For our purposes we are only interested in one scenario:

* Clients connect to a central peer that we will call the **VPN server**.
* While connected, some or all of the clients’ traffic is routed through the VPN server.

This enables these use cases:

* Clients want to connect to the Internet securely even though they are on an untrusted Wi-Fi network.
* Clients want to access a corporate network from the road. (“Road warrior”)

## Server setup

This setup is tested on Ubuntu Linux 18.04.

### Install Wireguard

Setup Wireguard as directed on the [Wireguard install page](https://www.wireguard.com/install/).

### Enable IP forwarding

Enable IP forwarding:

```
$ sudo sysctl net.ipv4.ip_forward=1
$ sudo sysctl net.ipv6.conf.all.forwarding=1
```

**Note:** These settings will not persist when the system reboots. To make them permanent, edit `/etc/sysctl.conf`.

### Install Wireguard Q-n-D scripts

* Copy these files (and their subdirectores) into `/etc/wireguard/manage`.
   * One way is: `git clone https://github.com/natesilva/wireguard-qnd.git /etc/wireguard/manage`

### Create the Wireguard service

```
$ sudo systemctl enable wg-quick@wg0.service
```

### Configure the Q-n-D scripts

* Edit `vpn.conf`.
    * Make sure the IP adresses are what you want.
    * Create a unique key for your server.
    * Add a `user` line for each user (client) you want to create.
* Run `./update-config`

### Firewall

Ensure the UDP port that you specified in `vpn.conf` is open at your firewall.

## Client setup

Each client needs its own configuration file. To get the file, start on the server. Go to `/etc/wireguard/manage` and run:

```
./print-user-config <user-name>
```

Where `<user-name>` is one of the users you defined in `vpn.conf`. Install the configuration on the client.

**Note**: If you install qrencode (`sudo apt install qrencode`) then the `./print-user-config` script will show the QR-code version of the configuration file, in addition to the plaintext version. This is a convenient way to set up mobile clients.

## Results

* Clients cannot see other clients.
* With the default `route` configuration:
  * When a client connects, all traffic is routed through the VPN.
  * Clients can access the Internet and the local LAN on the server side of the connection.

The `route` setting can be changed to meet your needs. For example, you may only want to use the VPN for traffic that is going to your own LAN (instead of all traffic).

## Manage the server

* To add a user: add a `user` line to `vpn.conf` and run `./update-config`.
* To delete a user: remove the corresponding `user` line from `vpn.conf` and run `./update-config`.

⚠️ Running `./update-config` will restart the VPN server, interrupting any currently-active VPN connections.

* To stop the server, run `sudo systemctl stop wg-quick@wg0.service`
* To start the server, run `sudo systemctl start wg-quick@wg0.service`

## Done

That’s all.
