🌞 Dans le rendu, je veux

un cat des 3 fichiers de conf (fichier principal + deux fichiers de zone)
```
[vincent@dns ~]$ sudo cat /var/named/tp6.b1.rev
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Reverse lookup
101 IN PTR dns.tp6.b1.
11 IN PTR john.tp6.b1.
```

```
[vincent@dns ~]$ sudo cat /var/named/tp6.b1.db
$TTL 86400
@ IN SOA dns.tp6.b1. admin.tp6.b1. (
    2019061800 ;Serial
    3600 ;Refresh
    1800 ;Retry
    604800 ;Expire
    86400 ;Minimum TTL
)

; Infos sur le serveur DNS lui même (NS = NameServer)
@ IN NS dns.tp6.b1.

; Enregistrements DNS pour faire correspondre des noms à des IPs
dns       IN A 10.6.1.101
john      IN A 10.6.1.11
```

```
[vincent@dns ~]$ sudo cat /etc/named.conf
//
// named.conf
//
// Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
// server as a caching only nameserver (as a localhost DNS resolver only).
//
// See /usr/share/doc/bind*/sample/ for example named configuration files.
//

options {
        listen-on port 53 { 127.0.0.1; any; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; any; };
        allow-query-cache { localhost; any; };

        /*
         - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
         - If you are building a RECURSIVE (caching) DNS server, you need to enable
           recursion.
         - If your recursive DNS server has a public IP address, you MUST enable access
           control to limit queries to your legitimate users. Failing to do so will
           cause your server to become part of large scale DNS amplification
           attacks. Implementing BCP38 within your network would greatly
           reduce such attack surface
        */
        recursion yes;

        dnssec-validation yes;

        managed-keys-directory "/var/named/dynamic";
        geoip-directory "/usr/share/GeoIP";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
        include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

# référence vers notre fichier de zone
zone "tp6.b1" IN {
     type master;
     file "tp6.b1.db";
     allow-update { none; };
     allow-query {any; };
};
# référence vers notre fichier de zone inverse
zone "1.4.10.in-addr.arpa" IN {
     type master;
     file "tp6.b1.rev";
     allow-update { none; };
     allow-query { any; };
};
```


un systemctl status named qui prouve que le service tourne bien.

```
[vincent@dns ~]$ systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
     Loaded: loaded (/usr/lib/systemd/system/named.service; enabled; preset: disabled)
     Active: active (running) since Fri 2023-11-17 11:09:40 CET; 12min ago
   Main PID: 4297 (named)
      Tasks: 5 (limit: 4605)
     Memory: 20.4M
        CPU: 39ms
     CGroup: /system.slice/named.service
             └─4297 /usr/sbin/named -u named -c /etc/named.conf

Nov 17 11:09:40 dns.tp6.b1 named[4297]: zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.>
Nov 17 11:09:40 dns.tp6.b1 named[4297]: zone 1.0.0.127.in-addr.arpa/IN: loaded serial 0
Nov 17 11:09:40 dns.tp6.b1 named[4297]: zone tp6.b1/IN: loaded serial 2019061800
Nov 17 11:09:40 dns.tp6.b1 named[4297]: zone localhost.localdomain/IN: loaded serial 0
Nov 17 11:09:40 dns.tp6.b1 named[4297]: zone localhost/IN: loaded serial 0
Nov 17 11:09:40 dns.tp6.b1 named[4297]: all zones loaded
Nov 17 11:09:40 dns.tp6.b1 systemd[1]: Started Berkeley Internet Name Domain (DNS).
Nov 17 11:09:40 dns.tp6.b1 named[4297]: running
Nov 17 11:09:40 dns.tp6.b1 named[4297]: managed-keys-zone: Initializing automatic trust anchor manage>
Nov 17 11:09:40 dns.tp6.b1 named[4297]: resolver priming query complete
```

une commande ss qui prouve que le service écoute bien sur un port

```
[vincent@dns ~]$ sudo ss -tulpn
Netid State  Recv-Q Send-Q   Local Address:Port   Peer Address:Port Process
udp   UNCONN 0      0            127.0.0.1:323         0.0.0.0:*     users:(("chronyd",pid=676,fd=5))
udp   UNCONN 0      0           10.6.1.101:53          0.0.0.0:*     users:(("named",pid=4297,fd=19))
udp   UNCONN 0      0            127.0.0.1:53          0.0.0.0:*     users:(("named",pid=4297,fd=16))
udp   UNCONN 0      0                [::1]:323            [::]:*     users:(("chronyd",pid=676,fd=6))
udp   UNCONN 0      0                [::1]:53             [::]:*     users:(("named",pid=4297,fd=22))
tcp   LISTEN 0      10          10.6.1.101:53          0.0.0.0:*     users:(("named",pid=4297,fd=21))
tcp   LISTEN 0      10           127.0.0.1:53          0.0.0.0:*     users:(("named",pid=4297,fd=17))
tcp   LISTEN 0      128            0.0.0.0:22          0.0.0.0:*     users:(("sshd",pid=691,fd=3))
tcp   LISTEN 0      4096         127.0.0.1:953         0.0.0.0:*     users:(("named",pid=4297,fd=24))
tcp   LISTEN 0      4096             [::1]:953            [::]:*     users:(("named",pid=4297,fd=25))
tcp   LISTEN 0      128               [::]:22             [::]:*     users:(("sshd",pid=691,fd=4))
tcp   LISTEN 0      10               [::1]:53             [::]:*     users:(("named",pid=4297,fd=23))
```

🌞 Ouvrez le bon port dans le firewall

grâce à la commande ss vous devrez avoir repéré sur quel port tourne le service

```
[vincent@dns ~]$ sudo ss -tulpn | grep :53
udp   UNCONN 0      0         10.6.1.101:53        0.0.0.0:*    users:(("named",pid=4297,fd=19))
udp   UNCONN 0      0          127.0.0.1:53        0.0.0.0:*    users:(("named",pid=4297,fd=16))
udp   UNCONN 0      0              [::1]:53           [::]:*    users:(("named",pid=4297,fd=22))
tcp   LISTEN 0      10        10.6.1.101:53        0.0.0.0:*    users:(("named",pid=4297,fd=21))
tcp   LISTEN 0      10         127.0.0.1:53        0.0.0.0:*    users:(("named",pid=4297,fd=17))
tcp   LISTEN 0      10             [::1]:53           [::]:*    users:(("named",pid=4297,fd=23))
```

ouvrez ce port dans le firewall de la machine dns.tp6.b1 (voir le mémo réseau Rocky)

```
[vincent@dns ~]$ sudo firewall-cmd --add-port=53/udp --permanent
success
[vincent@dns ~]$ sudo firewall-cmd --reload
success
```

3. Test

🌞 Sur la machine john.tp6.b1

configurez la machine pour qu'elle utilise votre serveur DNS quand elle a besoin de résoudre des noms

assurez vous que vous pouvez :

résoudre des noms comme john.tp6.b1 et dns.tp6.b1

```
[vincent@john ~]$ ping john.tp6.b1
PING john.tp6.b1(john.tp6.b1 (fe80::a00:27ff:fe68:4adc%enp0s3)) 56 data bytes
64 bytes from john.tp6.b1 (fe80::a00:27ff:fe68:4adc%enp0s3): icmp_seq=1 ttl=64 time=0.024 ms
64 bytes from john.tp6.b1 (fe80::a00:27ff:fe68:4adc%enp0s3): icmp_seq=2 ttl=64 time=0.036 ms
^C64 bytes from fe80::a00:27ff:fe68:4adc%enp0s3: icmp_seq=3 ttl=64 time=0.037 ms

--- john.tp6.b1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2054ms
rtt min/avg/max/mdev = 0.024/0.032/0.037/0.005 ms
```

```
[vincent@john network-scripts]$ ping dns.tp6.b1
PING dns.tp6.b1 (10.6.1.101) 56(84) bytes of data.
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=1 ttl=64 time=0.202 ms
64 bytes from 10.6.1.101 (10.6.1.101): icmp_seq=2 ttl=64 time=0.305 ms
^C
--- dns.tp6.b1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1032ms
rtt min/avg/max/mdev = 0.202/0.253/0.305/0.051 ms
```

mais aussi des noms comme www.ynov.com

```
[vincent@john network-scripts]$ ping www.ynov.com
PING www.ynov.com (104.26.10.233) 56(84) bytes of data.
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=1 ttl=55 time=19.4 ms
64 bytes from 104.26.10.233 (104.26.10.233): icmp_seq=2 ttl=55 time=20.7 ms
^C
--- www.ynov.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 19.367/20.030/20.693/0.663 ms
```

🌞 Sur votre PC

```
PS C:\WINDOWS\system32> Set-DNSClientServerAddress "Wi-Fi" -ServerAddresses ("10.6.1.101")

PS C:\Users\vince> Get-DnsClientServerAddress
Wi-Fi                               10 IPv4    {10.6.1.101}

PS C:\Users\vince> nslookup john.tp6.b1
Serveur :   UnKnown
Address:  10.6.1.101

Nom :    john.tp6.b1
```
```
voir tp6_dns.pcap
```

III. Serveur DHCP


🖥️ Machine dhcp.tp6.b1

🌞 Installer un serveur DHCP

```
[vincent@dhcp ~]$ sudo nano cat /etc/dhcp/dhcpd.conf
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
# create new
# specify domain name
option domain-name     "dns.tp6.b1";
# specify DNS server's hostname or IP address
option domain-name-servers     10.6.1.101;
# default lease time
default-lease-time 600;
# max lease time
max-lease-time 7200;
# this DHCP server to be declared valid
authoritative;
# specify network address and subnetmask
subnet 10.6.1.0 netmask 255.255.255.0 {
    # specify the range of lease IP address
    range dynamic-bootp 10.6.1.13 10.6.1.37;
    # specify broadcast address
    option broadcast-address 10.6.1.255;
    # specify gateway
    option routers 10.6.1.254;
}
```

🌞 Test avec john.tp6.b1

```
[vincent@john ~]$ ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a3:16:88 brd ff:ff:ff:ff:ff:ff
    inet 10.6.1.13/24 brd 10.6.1.255 scope global dynamic noprefixroute enp0s3
       valid_lft 570sec preferred_lft 570sec
    inet6 fe80::a00:27ff:fea3:1688/64 scope link
       valid_lft forever preferred_lft forever
```

```
[vincent@john ~]$ ip r s
default via 10.6.1.254 dev enp0s3 proto dhcp src 10.6.1.13 metric 100
```

```
[vincent@john ~]$ cat /etc/resolv.conf
# Generated by NetworkManager
search srv.world
nameserver 10.6.1.101
```

```
[vincent@john ~]$ ping google.com
PING google.com (142.250.201.174) 56(84) bytes of data.
64 bytes from par21s23-in-f14.1e100.net (142.250.201.174): icmp_seq=1 ttl=115 time=15.9 ms
64 bytes from par21s23-in-f14.1e100.net (142.250.201.174): icmp_seq=2 ttl=115 time=18.7 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1126ms
rtt min/avg/max/mdev = 15.918/17.317/18.717/1.679 ms
```

🌞 Requête web avec john.tp6.b1

```
[vincent@john ~]$ curl www.ynov.com
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>302 Found</title>
</head><body>
<h1>Found</h1>
<p>The document has moved <a href="https://www.ynov.com/">here</a>.</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at ynov.com Port 80</address>
</body></html>
```

```
voir tp6_web.pcap
```