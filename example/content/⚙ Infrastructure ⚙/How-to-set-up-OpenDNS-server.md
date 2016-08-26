/*
Title: Set up VPN on Digital Ocean
Sort: 2
*/

## EPEL Package

Extra Packages for Enterprise Linux (EPEL) Repository will provide the OpenVPN package.

1. Get the latest version of EPEL on https://fedoraproject.org/wiki/EPEL

2. Install EPEL (change URL according to latest version)
```sh
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```
```sh
rpm -Uvh epel-release-latest-7.noarch.rpm
```

3. Install the OpenVPN package from EPEL:

```sh
yum install openvpn -y
```

You should get a "Complete!" message

## Configure OpenVPN

4. OpenVPN ships with only a sample configuration, so we will copy the configuration file to its destination:

```sh
cp /usr/share/doc/openvpn-*/sample/sample-config-files/server.conf /etc/openvpn
```

5. Now that we have the file in the proper location, open it for editing:

```sh
vi /etc/openvpn/server.conf
```

5.1 Our first (tape 'i' for INSERT) change will be to uncomment the "push" parameter which causes traffic on our client systems to be routed through OpenVPN:

```
  #If enabled, this directive will configure
  # all clients to redirect their default
  # network gateway through the VPN, causing
  # all IP traffic such as web browsing and
  # and DNS lookups to go through the VPN
  push "redirect-gateway def1 bypass-dhcp"
```

5.2 We'll also want to change the section that immediately follows route DNS queries to Google's Public DNS servers:

```
  # Certain Windows-specific network settings
  # can be pushed to clients, such as DNS
  # or WINS server addresses
  push "dhcp-option DNS 8.8.8.8"
  push "dhcp-option DNS 8.8.4.4"
```

5.3 In addition, to enhance security, make sure OpenVPN drops privileges after startup. Uncomment the relevant "user" and "group" lines:

```
  # It's a good idea to reduce the OpenVPN
  # daemon's privileges after initialization.
  #
  # You can uncomment this out on
  # non-Windows systems.
  user nobody
  group nobody
```

5.4 Save and exit (Tape 'ESC :wq RETURN')

## Generating Keys and Certificates

For PKI management, we will use easy-rsa

6. We need to install separetely easy-rsa to generate the keys as easy-rsa is not include anymore in OpenDNS since 2.3

```sh
yum install easy-rsa
```

7. Create the required folder and copy the files we just install over.

```sh
mkdir -p /etc/openvpn/easy-rsa/keys
```
```sh
cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa
```

8. With the files in the desired location, we'll edit the "vars" file which provides the easy-rsa scripts with required information:

```sh
vi /etc/openvpn/easy-rsa/vars
```

9. We're looking to modify the "KEY_" variables, located at the bottom of the file.
  The variable names are fairly descriptive and should be filled out with the applicable information:
  (this step is optional as the console will ask for it during the generation of the CA)

```
  export KEY_COUNTRY="SG"
  export KEY_PROVINCE="SG"
  export KEY_CITY="Singapore"
  export KEY_ORG="Popety"
  export KEY_EMAIL="tclement@popety.com"
  export KEY_CN=vpn.popety.com
  export KEY_NAME=server
  export KEY_OU=server
```

10. OpenVPN might fail to properly detect the OpenSSL version on CentOS 6. As a precaution, manually copy the required OpenSSL configuration file.

```sh
cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/easy-rsa/openssl.cnf
```

11. We'll now change into our working directory and build our Certificate Authority, or CA, based on the information provided above.

```sh
cd /etc/openvpn/easy-rsa
source ./vars
./clean-all
./build-ca
```
(Set vpn.popety.com for Common Name)

12. Now that we have our CA, we'll create our certificate for the OpenVPN server. When asked by build-key-server, answer yes to commit.

```sh
./build-key-server server
```
(Set vpn.popety.com for Common Name and keep it blank for extra attributes)
(say yes for all)

13. We're also going to need to generate our Diffie Hellman key exchange files using the build-dh script and copy all of our files into /etc/openvpn as follows:

```sh
./build-dh
cd /etc/openvpn/easy-rsa/keys
cp dh2048.pem ca.crt server.crt server.key /etc/openvpn
```

## Generate client key

14. In order to allow clients to authenticate, we'll need to create client certificates.
  You can repeat this as necessary to generate a unique certificate and key for each client or device.
  If you plan to have more than a couple certificate pairs be sure to use descriptive filenames.

```sh
cd /etc/openvpn/easy-rsa
. ./vars
./build-key tclement
./build-key achawathe
./build-key ndolli@popety.com
```

  You should get something like:
```
  The Subject's Distinguished Name is as follows
  countryName           :PRINTABLE:'SG'
  stateOrProvinceName   :PRINTABLE:'SG'
  localityName          :PRINTABLE:'Singapore'
  organizationName      :PRINTABLE:'Popety'
  organizationalUnitName:PRINTABLE:'server'
  commonName            :PRINTABLE:'tclement'
  name                  :PRINTABLE:'server'
  emailAddress          :IA5STRING:'tclement@popety.com'
```

## Configuration to Saving IPTABLES Rules on CentOS 7

  Most administrators are using to using the “service iptables save” command to save firewall rules on RHEL5 and RHEL6 servers.
  With CentOS 7 and Red Hat Enterprise Linux 7 (as well as more-recent versions of Fedora), this command is no longer enabled by default.
  Instead, Red Hat has enabled `firewalld` by default. To enable the old IPTABLES save mechanisms, just perform the following steps:

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

## Routing Configuration and Starting OpenVPN Server

15. Create an iptables rule to allow proper routing of our VPN subnet (it's related to the 'push "redirect-gateway local def1"' in conf file).

```sh
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```
```sh
service iptables save
```

> Don't forget to allow proper routing on the eth1 if exist (private networking)

16. Then, enable IP Forwarding in sysctl:
```sh
vi /etc/sysctl.conf
```

  Add the following line:
  # Controls IP packet forwarding
  net.ipv4.ip_forward = 1

17. Finally, apply our new sysctl settings. Start OpenVPN:

```sh
sysctl -p
```
```sh
openvpn /etc/openvpn/server.conf
```

18. Start the vpn as service at start:
```sh
systemctl enable openvpn@server.service
```

For information to stop it:
```sh
systemctl stop openvpn@server.service
```

## Configuring OpenVPN Client

19. Copy all certificate to your computer using sftp

```sh
get /etc/openvpn/easy-rsa/keys/ca.crt /Users/tclement/sftp
get /etc/openvpn/easy-rsa/keys/tclement.crt /Users/tclement/sftp
get /etc/openvpn/easy-rsa/keys/tclement.key /Users/tclement/sftp
get /etc/openvpn/easy-rsa/keys/achawathe.crt /Users/tclement/sftp
get /etc/openvpn/easy-rsa/keys/achawathe.key /Users/tclement/sftp
```

20. With our certificates now on our client system, we'll create another new file called client.conf (for mac),
  the contents should be as follows:
  substituting "x.x.x.x" with your cloud servers IP address, and with the appropriate files pasted into the designated areas.
  Include only the contents starting from the "BEGIN" header line, to the "END" line, as demonstrated below.
  Be sure to keep these files as confidential as you would any authentication token.

```
  client
  dev tun
  proto udp
  remote 128.123.176.57 1194
  resolv-retry infinite
  nobind
  persist-key
  persist-tun
  comp-lzo
  verb 3
  <ca>
  Contents of ca.crt
  </ca>
  <cert>
  Contents of client.crt
  </cert>
  <key>
  Contents of client.key
  </key>
```

## Revoke a client certificate

1. Got to the easy-rsa directory

```
cd /etc/openvpn/easy-rsa
```

2. change the name below to the name certificate you want to revoke

```
. ./vars
./revoke-full tclement
```

3. Copy the generated crl.pem file to the OpenVPN config directory
```
cp keys/crl.pem /etc/openvpn/
```

4. Add the folowing line to the OpenVPN server config:
> Contrary to the third first steps that need to be done at each revokation, this one and the following need to be done only one time
```
vim /etc/openvpn/server.conf
```
```
# line to add to server.conf
crl-verify /etc/openvpn/crl.pem
```

5. Restart the OpenVPN server to activate the revoke setting
```
systemctl stop openvpn@server.service
systemctl start openvpn@server.service
```
