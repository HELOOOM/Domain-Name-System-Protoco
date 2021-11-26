# Domain Name System Protocol

## Part 1 : DNS configuration

# 1. DNS definition, role and functionality.

Domain Name Service (DNS) is an internet service that maps IP addresses to fully qualified domain names (FQDN) and vice versa.
role 
functionality
# 2. the different DNS servers.


# 3. Explain the DNS server’s configuration steps.
   - ## Configure the name.conf.
   
   Install the bind9(or named) package using the appropriate package management utilities for your Linux distributions.
   ```
    $ sudo apt-get install bind9
   ```
   All we have to do to configure a Cache NameServer is to add your ISP (Internet Service Provider)’s DNS server or any OpenDNS server to the file /etc/bind/named.conf.options. For Example, we will use google’s public DNS servers, 8.8.8.8 and 8.8.4.4.
   Uncomment and edit the following line as shown below in /etc/bind/named.conf.options file.
   ```
   forwarders {
    8.8.8.8;
    8.8.4.4;
};
   ```
   After the above change, start the DNS server.
   
   ![Image](dns1.jpeg)
   
   - ## Create and configure zone files.

To add a DNS Forward and Reverse resolution to bind9, edit /etc/bind9/named.conf.local.

![Image](dns2.jpeg)

Now we will add the details which is necessary for forward resolution into /etc/bind/db.thetadiststuff.net.copy file /etc/bind/db.local to /etc/bind/db.thetadiststuff
```
cp db.local  db.thetadiststuff
nano db.thetadiststuff
```
![Image](dns3.jpeg)

We will add the details which are necessary for reverse resolution to the file /etc/bind/db.reverse. Copy the file /etc/bind/db.127 to /etc/bind/db.reverse
```
cp db.127  db.reverse
nano db.reverse
```
![Image](dns4.jpeg)
## Explication
![Image](explication.png)


   - ## Verify the configuration.
     Finally, restart the bind9(or named) service:
     
       ![Image](dns5.jpeg)
       
       ![Image](dns7.jpeg)
       
# 4. Configure the DNS Client.

Local name resolution is done via /etc/hosts file. If you have small network, use /etc/hosts file. DNS (domain name service is accountable for associating domain names with ip address, for example domain yahoo.com is easy to remember than IP address 202.66.66.12) provides better name resolution. To configure Linux as DNS client you need to edit or modify /etc/resolv.conf file. This file defines which name servers to use. You want to setup Linux to browse net or run network services like www or smtp; then you need to point out to correct ISP DNS servers:

## /etc/resolv.conf file

In Linux and Unix like computer operating systems, the /etc/resolv.conf configuration file contains information that allows a computer connected to the Internet to convert alpha-numeric names into the numeric IP addresses that are required for access to external network resources on the Internet. The process of converting domain names to IP addresses is called “resolving.”

The resolv.conf file typically contains the IP addresses of nameservers (DNS name resolvers) that attempt to translate names into addresses for any node available on the network.

## Setup DNS Name resolution
Steps to configure Linux as DNS client, first login as a root user (use su command):

**Step # 1**: Open /etc/resolv.conf file:

`# vi /etc/resolv.conf`

**Step #2**: Add your ISP nameserver as follows:
```
search isp.com
nameserver 202.54.1.110
nameserver 202.54.1.112
nameserver 202.54.1.115
```
Note Max. three nameserver can be used/defined at a time.

**Step #3**:Test setup nslookup or dig command:
```
$ dig www.nixcraft.com
$ nslookup www.nixcraft.com
```   
the previous configuration is explained by this exemple:

![Image](dns6.jpeg)

# 5. Configure primary and secondary DNS servers.
   -  ## Configure the primary DNS server as a master.
   Here are the steps to configure DNS master-slave server in Linux. Here is our setup.
   ```
   Master DNS Server IP: 54.43.32.21 ( ns1.example.com )
Slave  DNS Server IP: 54.43.32.22 ( ns2.example.com )
Domain Name : example.com   ( For Testing Purpose )
Domain IP   : 54.43.32.20  ( For Testing Purpose )
   ```
   Open terminal and run the following command to open named.conf file.
   `$ vi /var/named/chroot/etc/named.conf`
   Update listen-on and allow-query variables’ values in the options block as shown with the address CIDR for your server, shown in bold.
   ```
   options {
        listen-on port 53 { 127.0.0.1; 54.43.32.0/24; };
 ...
        allow-query     { localhost; 54.43.32.0/24; };
        recursion yes;

 ...
};
   ```
   We need to create separate zone for each of our domain. Since we have only 1 domain example.com, we create zone file for it.\
   
   
   `$ vi /var/named/chroot/var/named/example.com.db`
   
   Add the following lines to it. Replace example.com with your domain name, ns1.example.com with subdomain of your master, ns2.example.com with subdomain of your slave.
   ```
   ; Zone file for example.com
$TTL 14400
@      86400    IN      SOA     ns1.example.com. webmaster.example.com. (
                3215040200      ; serial, todays date+todays
                86400           ; refresh, seconds
                7200            ; retry, seconds
                3600000         ; expire, seconds
                86400 )         ; minimum, seconds

example.com. 86400 IN NS ns1.example.com.
example.com. 86400 IN NS ns2.example.com.
example.com. IN A 192.168.1.100
example.com. IN MX 0 example.com.
mail IN CNAME example.com.
www IN CNAME example.com.
```
Now we need to add details of this zone file in our named.conf file updated in previous step. Open it again in text editor and add the following lines. Replace example.com with your domain name.

```
zone "example.com" IN {
        type master;
        file "/var/named/example.com.db";
	allow-update { none; };
};
```
Save and close the file. Run the following command to start named service.
```
$ /etc/init.d/named restart
$ chkconfig named on
```

   
   -  ## Configure the secondary DNS server as a slave.

Now we need to configure Slave DNS server. In this case, we need to update only named.conf file. All changes made on Master will be automatically synced with its slaves at regular intervals of time. Open named.conf file in text editor.

`$ vi /var/named/chroot/etc/named.conf`

In this case also, update listen-on and allow-query variables’ values in the options block as shown with the address CIDR for your server, shown in **bold**.
```
options {
        listen-on port 53 { 127.0.0.1; 54.43.32.0/24; };
 ...
        allow-query     { localhost; 54.43.32.0/24; };
        recursion yes;

 ...
};
```
Start named service with the following command.

```
$ /etc/init.d/named restart
$ chkconfig named on
```
After restarting named service, you will be able to see zone files in slave DNS at /var/named/chroot/var/named/slaves/.
   -  ## Test the configuration by stopping the master DNS.
   Now you can query your DNS master & slaves with the following command.
   
   `nslookup <domainname.com> <dns server name/ip>`
   
   You should get the same response from both of them. Here’s the query to Master DNS server.
   ```
   $ nslookup example.com 54.43.32.21

Server:         54.43.32.21
Address:        54.43.32.21#53

Name:   example.com
Address: 54.43.32.20
```
The above output shows that both DNS master and slave have correctly resolved domain example.com. In this article, we have learnt how to setup DNS Master-Slave server. You can customize it according to your requirements. Although the above steps are for RHEL/Fedora/CentOS, you can also use it for Ubuntu/Debian Linux.
