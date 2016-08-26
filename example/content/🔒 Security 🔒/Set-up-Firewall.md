/*
Title: VPN access
Sort: 2
*/

The aim of this document is to explain how to set up firewall to allow internal server on Digital Ocean available from our VPN and DMZ but not from internet.

* Cloud host: Digital Ocean
* OS: CentOs

### Our goals
All of the servers in the private network (MySQL, WebServer) area can only be communicated with by other servers within this private network. The load balancer will be accessible via the Internet and also be linked to the private network. The enforcement of this policy will be implemented via iptables on each server.

## Activate private networking

Activate Private Networking by choosing the checkbox called "Private Networking" in the settings section of the droplet create page.

> Now you can switch the next two section and go directly to 'Disabled connection from public network' section.

## Activate private networking on existing droplets

In this section, we will cover how to manually enable private networking for droplets with private networking didn't activate at creation.

> Pass this section if you just create a new droplet with enabling the private networking.

1. Power droplet down
```sh
sudo shutdown now
```
2. Go to DigitalOcean droplet page -> Networking and enable "Private Network"

3. Power on the droplet

4. When your droplet is booted, SSH in and go to the next section

## Configure private networking on existing droplets

In this section, we will cover how to manually configure private networking for droplets with private networking didn't activate at creation.

> Pass this section if you just create a new droplet with enabling the private networking.

1. In CentOS, the networking interfaces are configured in the /etc/sysconfig/network-scripts directory. Before we continue though, we need to find a key piece of information. Type the following command and copy the value of "ether" under the eth1 section (Looks like "04:01:30:1e:e7:02"):
```sh
sudo ifconfig -a
```

2. Go to the interface configuration directory:
```sh
cd /etc/sysconfig/network-scripts
```

3. Create a new configuration file to describe the private networking interface:
```sh
sudo vi ifcfg-eth1
```

4. Copy and paste the following information into the file. Edit the values in red with the information found in the "settings" tab of your droplet's page and the piece of information you copied from the ifconfig -a command (i then cmd + v):
```sh
DEVICE="eth1"
HWADDR=info_from_ifconfig
IPADDR=Private_IP_from_settings
BOOTPROTO=none
ONBOOT="yes"
NETMASK=Private_netmask_from_settings
NM_CONTROLLED="yes"
IPV6INIT="no"
DEFROUTE="no"
```

5. Save and close the file (esc then :wq).

6. Restart your droplet
```sh
sudo shutdown -r now
```

7. If you log back into your droplet, you should see the eth1 interface when you type:
```sh
sudo ifconfig
```

8. You should be able now to ping another droplet in the private network
```sh
ping private_ip_of_droplet
```

## Disabled connection from public network

In this section we will use a Iptables with shared private networking to simulate the network traffic isolation that a true private network can provide.
Incoming Public Network must be disabled for WebServer and MySQL database. Only the load balancer can be access from internet. Then it forward the connection through the Private Network.

> If you lose access to both your public and private interfaces, you can connect to your VPS via console access on DigitalOcean website. In the real world, this is analogous to connecting a keyboard, mouse, and monitor directly to your server. Remember that you can always access your VPS this way, if you accidentally disable both of your interfaces or SSH service.

> All step below are for information only as these steps has been automatize in a script named popety_pn_firewall. Use this script to set up the firewall

### Configuration to Saving IPTABLES Rules on CentOS 7

  Most administrators are using to using the “service iptables save” command to save firewall rules on RHEL5 and RHEL6 servers.
  With CentOS 7 and Red Hat Enterprise Linux 7 (as well as more-recent versions of Fedora), this command is no longer enabled by default.
  Instead, Red Hat has enabled `firewalld` by default. To enable the old IPTABLES save mechanisms, just perform the following steps:

  Because of the large number of network interfaces and ports that require communication within the private network, we will simplify things by whitelisting the necessary IP addresses instead of only allowing specific IP address and port combinations. Also, we will allow outgoing traffic by default, and just restrict incoming traffic.

  A. First, stop and mask the firewalld service:

```sh
systemctl stop firewalld
```
```sh
systemctl mask firewalld
```

  B. Next, install the iptables-services package and enable the service to start at boot:

```sh
yum install iptables-services
```
```sh
systemctl enable iptables
```

  C. Then, saving the firewall rules can be done using either of the following two commands:

```sh
service iptables save
```

1. A good starting point is to list the current rules that are configured for iptables. You can do that with the -L flag:
```sh
sudo iptables -L
```
by default each chain has ACCEPT as its default policy and we have no rules defined

2. If not, flush the current rules and set the default to ACCEPT:
```sh
sudo iptables -P INPUT ACCEPT
sudo iptables -P OUTPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -F
```

3. Allow VPN to SSH via its private interface to private-VPS:
```sh
sudo iptables -A INPUT -p tcp -s vpn_private_IP --dport 22 -i eth1 -j ACCEPT
```

4. Allow loopback traffic on your server. This allows your server to use 127.0.0.1 or localhost:
```sh
sudo iptables -A INPUT -i lo -j ACCEPT
```

5. Allow public and private traffic that is initiated from your server. This will allow your server to access the Internet to do things like download updates or software:
```sh
sudo iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

6. Whitelist all of the servers that only need access to the private network area (you may omit the entry for the server you are working on):
```sh
sudo iptables -A INPUT -p tcp -s mysql?_private_IP -j ACCEPT
sudo iptables -A INPUT -p tcp -s webserver?_private_IP -j ACCEPT
```

7. Only on web servers, allow LoadBalancer HTTP access (port 80), so it can retrieve pages:
```sh
sudo iptables -A INPUT -p tcp -s loadbalancer_private_IP --sport 80 -j ACCEPT
sudo iptables -A OUTPUT -p tcp -d loadbalancer_private_IP --dport 80 -j ACCEPT
```

8. Drop INPUT and FORWARD chains by default. Note that we are leaving OUTPUT's default as ACCEPT, as we trust the servers on our private network:
```sh
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
```

9. Now that you are done configuring private-VPS's firewall, you will want to make sure that everything works properly. If you are happy with your configuration, you can save it by installing the iptables-persistent package with the following apt commands:
```sh
sudo service iptables save
```

## Add whitelist ip for private network

## Configure /etc/hosts to connect to another server via hostname

Another helpful step to take when using the private networking is to set up your /etc/hosts file with a hostname that you'd like to use to connect to another server via the private network address. Doing this will allow you to connect across the private network without typing the droplet's private network address each time.

1. Open the /etc/hosts file
```sh
vi /etc/hosts
```
Within the file include the private network address of the server that you want to the connect to and the hostname by which you'd like to call it:

```
127.0.0.1 localhost machine_hostname
Private_IP_of_other_droplet hostname_of_other_droplet
# for example
# 10.134.2.18 uat2.popety.com

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
You can disregard the information on IPv6 in the file at this time. Save and exit out of that file.

## Annex
https://www.digitalocean.com/community/tutorials/how-to-enable-digitalocean-private-networking-on-existing-droplets
https://www.digitalocean.com/community/tutorials/how-to-set-up-and-use-digitalocean-private-networking
https://www.digitalocean.com/community/tutorials/how-to-isolate-servers-within-a-private-network-using-iptables
https://www.digitalocean.com/community/tutorials/how-the-iptables-firewall-works
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-iptables-on-ubuntu-14-04
Create an automatic script for that:
https://wiki.centos.org/HowTos/Network/IPTables
