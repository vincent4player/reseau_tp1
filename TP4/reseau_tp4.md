I. DHCP Client

🌞 Déterminer
l'adresse du serveur DHCP
l'heure exacte à laquelle vous avez obtenu votre bail DHCP
l'heure exacte à laquelle il va expirer
```
PS C:\Users\vince> ipconfig /all
Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . :
   Description. . . . . . . . . . . . . . : Intel(R) Wi-Fi 6 AX201 160MHz
   Adresse physique . . . . . . . . . . . : 98-8D-46-C4-FA-E5
   DHCP activé. . . . . . . . . . . . . . : Oui
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::3b06:b353:ddcf:f393%8(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.70.190(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.240.0
   Bail obtenu. . . . . . . . . . . . . . : ⭐mardi 6 novembre 2023 08:49:54⭐
   Bail expirant. . . . . . . . . . . . . : ⭐mercredi 7 novembre 2023 08:49:50⭐
   Passerelle par défaut. . . . . . . . . : 10.33.79.254
   Serveur DHCP . . . . . . . . . . . . . : ⭐10.33.79.254⭐
   IAID DHCPv6 . . . . . . . . . . . : 77106502
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-27-9C-7F-68-2C-F0-5D-6C-11-B8
   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       1.1.1.1
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé
```

🌞 Capturer un échange DHCP
```
PS C:\Users\vince> ipconfig /release
PS C:\Users\vince> ipconfig /renew
```
```
voir trames dhcp
```

🌞 Analyser la capture Wireshark

parmi ces 4 trames, laquelle contient les informations proposées au client ?

```
"c'est la trame intitulé DHCP Offer.
```

🦈 tp4_dhcp_client.pcapng



II. Serveur DHCP



🌞 Preuve de mise en place

Depuis dhcp.tp4.b1, envoyer un ping vers un nom de domaine public (pas une IP)

```
[vincent@dhcp ~]$ ping google.com
PING google.com (172.217.20.174) 56(84) bytes of data.
64 bytes from waw02s07-in-f174.1e100.net (172.217.20.174): icmp_seq=1 ttl=114 time=16.3 ms
64 bytes from waw02s07-in-f14.1e100.net (172.217.20.174): icmp_seq=2 ttl=114 time=17.4 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 16.315/16.853/17.392/0.538 ms
```

Depuis node2.tp4.b1, envoyer un ping vers un nom de domaine public (pas une IP)

```
[vincent@node2 ~]$ ping youtube.com
PING youtube.com (216.58.214.174) 56(84) bytes of data.
64 bytes from par10s42-in-f14.1e100.net (216.58.214.174): icmp_seq=1 ttl=114 time=16.6 ms
64 bytes from mad01s26-in-f14.1e100.net (216.58.214.174): icmp_seq=2 ttl=114 time=16.6 ms
^C
--- youtube.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 16.622/16.624/16.627/0.002 ms
[vincent@node2 ~]$
```
Depuis node2.tp4.b1, un traceroute vers une IP publique pour montrer que vos paquets à destination d'internet passent bien par le router.tp4.b1

```
[vincent@node2 ~]$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.4.1.254)  1.125 ms  1.171 ms  1.139 ms
 2  10.0.3.2 (10.0.3.2)  1.126 ms  1.093 ms  1.078 ms^C
```

4. Serveur DHCP




🌞 Rendu

```
[vincent@dhcp ~]$ sudo dnf -y install dhcp-server
[vincent@dhcp ~]$ sudo nano /etc/dhcp/dhcpd.conf

# create new
# nom de domaine
option domain-name     "srv.world";
# nom de serveur dns ou ip a mettre
option domain-name-servers     dlp.srv.world;
# tmps expiration bail par defaut
default-lease-time 600;
# tmps expiration max
max-lease-time 7200;
# activer dhcp dans le lan
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
   # definit l'intervalle d'ip que le protocole va donner
   range dynamic-bootp 10.4.1.137 10.4.1.237;
   # adresse de diffusion
   option broadcast-address 10.4.1.255;
   # specify passerelle
   option routers 10.4.1.254;
}
[vincent@dhcp ~]$ sudo systemctl enable --now dhcpd
[vincent@dhcp ~]$ sudo firewall-cmd --add-service=dhcp
[vincent@dhcp ~]$ sudo firewall-cmd --runtime-to-permanent
[vincent@dhcp ~]$ ll /var/lib/dhcpd
total 4
-rw-r--r--. 1 dhcpd dhcpd    0 Apr 20  2023 dhcpd6.leases
-rw-r--r--. 1 dhcpd dhcpd 1074 Oct 27 15:13 dhcpd.leases
-rw-r--r--. 1 dhcpd dhcpd    0 Apr 20  2023 dhcpd.leases~
```
   Un systemctl status dhcpd qui affiche l'état du serveur (je dois voir qu'il est actif)

```
[vincent@dhcp ~]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
     Active: active (running) since Wed 2023-11-08 17:14:46 CET; 5min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 11114 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 5896)
     Memory: 5.2M
        CPU: 6ms
     CGroup: /system.slice/dhcpd.service
             └─11114 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Nov 08 17:14:46 dhcp.tp4.b1 dhcpd[11114]: Sending on   LPF/enp0s3/08:00:27:a2:cc:cb/10.4.1.0/24
Nov 08 17:14:46 dhcp.tp4.b1 dhcpd[11114]: Sending on   Socket/fallback/fallback-net
Nov 08 17:14:46 dhcp.tp4.b1 dhcpd[11114]: Server starting service.
Nov 08 17:14:46 dhcp.tp4.b1 systemd[1]: Started DHCPv4 Server Daemon.
Nov 08 17:14:48 dhcp.tp4.b1 dhcpd[11114]: DHCPDISCOVER from 08:00:27:a3:bd:ca via enp0s3
Nov 08 17:14:51 dhcp.tp4.b1 dhcpd[11114]: DHCPOFFER on 10.4.1.137 to 08:00:27:a3:bd:ca (node1) via en>
Nov 08 17:14:51 dhcp.tp4.b1 dhcpd[11114]: DHCPREQUEST for 10.4.1.137 (10.4.1.253) from 08:00:27:a3:bd>
Nov 08 17:14:51 dhcp.tp4.b1 dhcpd[11114]: DHCPACK on 10.4.1.137 to 08:00:27:a3:bd:ca (node1) via enp0>
Nov 08 17:19:48 dhcp.tp4.b1 dhcpd[11114]: DHCPREQUEST for 10.4.1.137 from 08:00:27:a3:bd:ca (node1) v>
Nov 08 17:19:48 dhcp.tp4.b1 dhcpd[11114]: DHCPACK on 10.4.1.137 to 08:00:27:a3:bd:ca (node1) via enp0>
lines 1-23/23 (END)
```

je veux aussi un cat /etc/dhcp/dhcpd.conf dans le compte-rendu, pour que je vois le fichier de configuration

```
[vincent@dhcp ~]$ cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.2b1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\001,\336pv\010\000'\242\314\313";

lease 10.4.1.137 {
  starts 3 2023/11/08 16:14:51;
  ends 3 2023/11/08 16:24:51;
  cltt 3 2023/11/08 16:14:51;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:a3:bd:ca;
  uid "\001\010\000'\243\275\312";
  client-hostname "node1";
}
lease 10.4.1.137 {
  starts 3 2023/11/08 16:19:48;
  ends 3 2023/11/08 16:29:48;
  cltt 3 2023/11/08 16:19:48;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:a3:bd:ca;
  uid "\001\010\000'\243\275\312";
  client-hostname "node1";
}
```

5. Client DHCP

🌞 Test !

```
[vincent@node1 ~]$ sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3

DEVICE=enp0s3

BOOTPROTO=dhcp
ONBOOT=yes
```

🌞 Prouvez que

```
[vincent@node1 ~]$ ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:a3:bd:ca brd ff:ff:ff:ff:ff:ff
    inet 10.4.1.137/24 brd 10.4.1.255 scope global ⭐ dynamic ⭐ noprefixroute enp0s3
       valid_lft 486sec preferred_lft 486sec
    inet6 fe80::a00:27ff:fea3:bdca/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

```
Création du bail:

[vincent@node1 ~]$ sudo nmcli con show enp0s3
connection.timestamp:                   1699464618
# Mer 08/11/2023 18:30:18
```

```
Expiration du bail:

[vincent@node1 ~]$ sudo nmcli con show enp0s3
DHCP4.OPTION[6]:                        expiry = 1699465798
Mer 08/11/2023 18:49:58
```

```
Adresse ip du serveur DHCP

[vincent@node1 ~]$ sudo nmcli con show enp0s3
DHCP4.OPTION[3]:                        dhcp_server_identifier = 10.4.1.253
```

Vous pouvez ping router.tp4.b1 et node2.tp4.b1 grâce à cette nouvelle IP récupérée

   Routeur:

```
[vincent@node1 ~]$ ping 10.4.1.254
PING 10.4.1.254 (10.4.1.254) 56(84) bytes of data.
64 bytes from 10.4.1.254: icmp_seq=1 ttl=64 time=0.291 ms
64 bytes from 10.4.1.254: icmp_seq=2 ttl=64 time=0.299 ms
^C
--- 10.4.1.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1060ms
rtt min/avg/max/mdev = 0.291/0.295/0.299/0.004 ms
```
 Node2:

```
[vincent@node1 ~]$ ping 10.4.1.12
PING 10.4.1.12 (10.4.1.12) 56(84) bytes of data.
64 bytes from 10.4.1.12: icmp_seq=1 ttl=64 time=0.546 ms
64 bytes from 10.4.1.12: icmp_seq=2 ttl=64 time=0.314 ms
^C
--- 10.4.1.12 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1064ms
rtt min/avg/max/mdev = 0.314/0.430/0.546/0.116 ms
```

🌞 Bail DHCP serveur

```
[vincent@dhcp ~]$ cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.2b1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

lease 10.4.1.137 {
  starts 3 2023/11/08 17:14:58;
  ends 3 2023/11/08 17:24:58;
  cltt 3 2023/11/08 17:14:58;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:a3:bd:ca;
  uid "\001\010\000'\243\275\312";
  client-hostname "node1";
}
```

6. Options DHCP




🌞 Nouvelle conf !


```
[slayz@dhcp etc]$ sudo cat dhcp/dhcpd.conf
# create new
# nom de domaine
option domain-name     "srv.world";
# nom de serveur dns ou ip a mettre
option domain-name-servers      1.1.1.1;
# tmps expiration bail par defaut
default-lease-time 21600;
# tmps expiration max
max-lease-time 22000;
# activer dhcp dans le lan
authoritative;
# specify network address and subnetmask
subnet 10.4.1.0 netmask 255.255.255.0 {
    # definit l'intervalle d'ip que le protocole va donner
    range dynamic-bootp 10.4.1.137 10.4.1.237;
    # adresse de diffusion
    option broadcast-address 10.4.1.255;
    # specify passerelle
    option routers 10.4.1.254;
}
```

🌞 Test !

```
Redemandez une IP avec le client node1.tp4.b1:

[vincent@node1 ~]$ sudo dhclient -r enp0s3
[vincent@node1 ~]$ sudo dhclient enp0s3
```


```
Prouvez-que
vous avez enregistré l'adresse d'un serveur DNS:

[vincent@dhcp etc]$ cat resolv.conf
# Generated by NetworkManager
search tp4.b1
nameserver 1.1.1.1
```

```
Vous avez une nouvelle route par défaut qui a été récupérée dynamiquement:

[vincent@node1 ~]$ ip r s
default via 10.4.1.254 dev enp0s3
default via 10.4.1.254 dev enp0s3 proto dhcp metric 100
10.4.1.0/24 dev enp0s3 proto kernel scope link src 10.4.1.137 metric 100
[vincent@node1 ~]$
```

```
La durée de votre bail DHCP est bien de 6 heures:

[vincent@node1 dhclient]$ cd var/lib/dhclient

[vincent@node1 dhclient]$ cat dhclient.leases
lease {
  interface "enp0s3";
  fixed-address 10.4.1.138;
  option subnet-mask 255.255.255.0;
  option routers 10.4.1.254;
  option dhcp-lease-time 21600;
  option dhcp-message-type 5;
  option domain-name-servers 1.1.1.1;
  option dhcp-server-identifier 10.4.1.253;
  option broadcast-address 10.4.1.255;
  option domain-name "srv.world";
  renew 5 2023/11/08 19:27:04;
  rebind 5 2023/11/08 19:27:04;
  expire 5 2023/11/08 19:27:04;
```

```
Prouvez que vous avez un accès Internet après cet échange DHCP:

[vincent@node1 ~]$ ping google.com
PING google.com (216.58.214.174) 56(84) bytes of data.
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=1 ttl=114 time=17.0 ms
64 bytes from mad01s26-in-f174.1e100.net (216.58.214.174): icmp_seq=2 ttl=114 time=16.4 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 16.401/16.692/16.984/0.291 ms
```






🌞 Capture Wireshark

utilisez tcpdump pour capturer un échange DHCP complet entre node1.tp4.b1 et dhcp.tp4.b1


🦈 tp4_dhcp_server.pcapng
➜ Un vrai serveur DHCP qui donne tout ce qu'il faut aux clients pour qu'ils aient un accès au LAN (une adresse IP) et un accès internet en plus (l'adresse de la passerelle et l'adresse d'un serveur DNS joignable).