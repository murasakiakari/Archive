# OpenVPN
## Disclaimer:
This document is only purpose on archiving the procedue of setting up OpenVPN.  
This is tested on a CentOS Stream 8 VPS through SSH console on 2021/9/16.

## Procedues:
> ## Server side:
>> ## 1. Install OpenVPN and easy-rsa
>> ```
>> yum -y install epel-release
>> yum -y update
>> yum -y install openvpn easy-rsa
>> ```
>> ## 2. Set server name and client name
>> ```
>> SERVER_NAME="{server-name}"
>> CLIENT_NAME="{client-name}"
>> ```
>> Remark:  
>> Replace the {server-name} and {client-name} to the name you want to use.
>> ## 3. Create certifacate
>> ```
>> cd /usr/share/easy-rsa/3/
>> ./easyrsa init-pki
>> ./easyrsa build-ca nopass
>> ./easyrsa build-server-full $SERVER_NAME nopass
>> ./easyrsa build-client-full $CLIENT_NAME nopass
>> ```
>> Remark:  
>> No password is used to encrypt the certifacate.
>> ## 4. Copy the certifacate to the OpenVPN directory
>> ```
>> cd /usr/share/easy-rsa/3/pki
>> cp ca.crt issued/$SERVER_NAME.crt private/$SERVER_NAME.key /etc/openvpn/server/
>> cd /etc/openvpn/server
>> openssl dhparam -out dh2048.pem 2048
>> openvpn --genkey --secret ta.key
>> ```
>> ## 5. Configute OpenVPN server
>> ```
>> cp /usr/share/doc/openvpn/sample/sample-config-files/server.conf /etc/openvpn/server/
>> mv server.conf $SERVER_NAME.conf
>> nano $SERVER_NAME.conf
>> ```
>> In that file, change the following lines
>> ```
>> # SSL/TLS root cetificate (ca), ...
>> ca ca.crt
>> cert {server-name}.crt
>> key {server-name}.key
>>
>> # If enabled, this directive will configure...
>> push "redirect-gateway def1"
>> ```
>> Uncomment the following lines
>> ```
>> # Network topology ...
>> topology subnet
>> ```
>> Add the following lines
>> ```
>> # Certain Windows-specific network settings...
>> push "dhcp-option DNS 10.8.0.1"
>> ```
>> Remark:  
>> Press CTRL+X, Y, ENTER to save and exit.
>> ## 6. Configuate network option
>> ```
>> systemctl disable firewalld
>> systemctl stop firewalld
>> echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
>> sysctl -p
>> iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
>> chmod +x /etc/rc.d/rc.local
>> nano /etc/rc.local
>> ```
>> In that file, add the following line
>> ```
>> iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
>> ```
>> Remark:  
>> The last few command is purposed on add iptable rule when the system boot.
>> ## 7. Start OpenVPN service
>> ```
>> systemctl start openvpn-server@$SERVER_NAME
>> systemctl enable openvpn-server@$SERVER_NAME
>> ```
>> Remark:  
>> The second command can automatically start OpenVPN service when the system boot.
>>
> ## Client Side:
>> ## 1. Download OpenVPN client
>> Download [here](https://openvpn.net/community-downloads/)
>> ## 2. View content of files
>> ```
>> cat /usr/share/easy-rsa/3/pki/ca.crt
>> cat /usr/share/easy-rsa/3/pki/issued/$CLIENT_NAME.crt
>> cat /usr/share/easy-rsa/3/pki/private/$CLIENT_NAME.key
>> cat /etc/openvpn/server/ta.key
>> cat /usr/share/doc/openvpn/sample/sample-config-files/client.conf
>> ```
>> ## 3. Copy the content to files
>> Save the above context to ca.crt, {client-name}.crt, {client-name}.key, ta.key and {client-name}.ovpn respectively under directory C:\Users\\{user-name}\OpenVPN\config\
>> ## 4. Configuate OpenVPN client
>> In {client-name}.ovpn file, change the following lines:
>> ```
>> # The hostname/IP and port of the server. ...
>> remote {OpenVPN-server-IP} 1194
>>
>> # SSL/TLS parms. ...
>> ca ca.crt
>> cert {client-name}.crt
>> key {client-name}.key
>> ```
