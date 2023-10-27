I. ARP

Machine
LAN 1 10.3.1.0/24

john
10.3.1.11

marcel
10.3.1.12

1. Echange ARP
🌞Générer des requêtes ARP

```
[vincent@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.604 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.384 ms

--- 10.3.1.12 ping statistics ---
14 packets transmitted, 14 received, 0% packet loss, time 13342ms
rtt min/avg/max/mdev = 0.187/0.353/0.604/0.101 ms
```
```
[vincent@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.316 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.198 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.376 ms

--- 10.3.1.11 ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6127ms
rtt min/avg/max/mdev = 0.198/0.319/0.393/0.079 ms
```
```
ARP:
[vincent@localhost ~]$ ip neigh show
10.3.1.12 dev enp0s3 lladdr ⭐08:00:27:55:3c:64⭐ STALE

[vincent@localhost ~]$ ip neigh show
10.3.1.11 dev enp0s3 lladdr ⭐08:00:27:c5:08:3e⭐ STALE
```
```
Commande 1:[vincent@localhost ~]$ ip neigh show
10.3.1.12 dev enp0s3 lladdr ⭐08:00:27:55:3c:64⭐ STALE

Commande 2:[vincent@localhost ~]$ ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether ⭐08:00:27:55:3c:64⭐ brd ff:ff:ff:ff:ff:ff
```

2. Analyse de trames
🌞Analyse de trames
```
sudo ip neigh flush all
```
```
sudo tcpdump -i enp0s3 -c 10 -w tp3-arp.pcap not port 22
```
```
scp vincent@10.3.1.11:/home/vince/tp3-arp.pcap ./
```


```
"voir tp3_arp.pcap"
```







II. Routage


II. Routage

1. Mise en place du routage
2. Analyse de trames
3. Accès internet



Vous aurez besoin de 3 VMs pour cette partie. Réutilisez les deux VMs précédentes.



Machine
LAN 1 10.3.1.0/24

LAN 2 10.3.2.0/24





router
10.3.1.254
10.3.2.254


john
10.3.1.11
no


marcel
no
10.3.2.12




Je les ai appelés marcel et john PASKON EN A MAR des noms nuls en réseau 🌻


   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘


➜ AVANT de continuer, configurez correctement les IP sur les 3 VMs. Normalement :


john peut ping router sur 10.3.1.254
```
[vincent@localhost ~]$ ping 10.3.1.254
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.520 ms
64 bytes from 10.3.1.254: icmp_seq=2 ttl=64 time=0.427 ms
64 bytes from 10.3.1.254: icmp_seq=3 ttl=64 time=0.320 ms
^C
--- 10.3.1.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2030ms
rtt min/avg/max/mdev = 0.320/0.422/0.520/0.081 ms
```
marcel peut ping router sur 10.3.2.254
```
[vincent@localhost ~]$ ping 10.3.2.254
PING 10.3.2.254 (10.3.2.254) 56(84) bytes of data.
64 bytes from 10.3.2.254: icmp_seq=1 ttl=64 time=0.394 ms
64 bytes from 10.3.2.254: icmp_seq=2 ttl=64 time=0.364 ms
^C
--- 10.3.2.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1054ms
rtt min/avg/max/mdev = 0.364/0.379/0.394/0.015 ms
```


1. Mise en place du routage
➜ Activer le routage sur le noeud router

Cette étape est nécessaire car Rocky Linux c'est pas un OS dédié au routage par défaut. Ce n'est bien évidemment une opération qui n'est pas nécessaire sur un équipement routeur dédié comme un routeur Cisco.


# pour autoriser une machine Linux à traiter des paquets IP qui ne lui sont pas destinés
# autrement dit "activer le routage"
$ sudo sysctl -w net.ipv4.ip_forward=1


🌞Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping

```
john

[vincent@localhost ~]$ sudo ip route add 10.3.2.0/24 via 10.3.1.254 dev enp0s3
[vincent@localhost ~]$ ip r s
10.3.1.0/24 dev enp0s3 proto kernel scope link src 10.3.1.11 metric 100
10.3.2.0/24 via 10.3.1.254 dev enp0s3
[vincent@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.545 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.795 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.593 ms
^C
--- 10.3.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2035ms
rtt min/avg/max/mdev = 0.545/0.644/0.795/0.108 ms
```
```
Marcel

[vincent@localhost ~]$ sudo ip route add 10.3.1.0/24 via 10.3.2.254 dev enp0s3
[vincent@localhost ~]$ ip r s
10.3.1.0/24 via 10.3.2.254 dev enp0s3
10.3.2.0/24 dev enp0s3 proto kernel scope link src 10.3.2.12 metric 100
[vincent@localhost ~]$ ping 10.3.2.254
PING 10.3.2.254 (10.3.2.254) 56(84) bytes of data.
64 bytes from 10.3.2.254: icmp_seq=1 ttl=64 time=0.314 ms
64 bytes from 10.3.2.254: icmp_seq=2 ttl=64 time=0.424 ms
^C
--- 10.3.2.254 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1055ms
rtt min/avg/max/mdev = 0.314/0.369/0.424/0.055 ms
```



2. Analyse de trames
🌞Analyse des échanges ARP

```
sudo ip neigh flush all
```

```
| ordre | type trame  | IP source | MAC source                | IP destination | MAC destination            |
| ----- | ----------- | --------- | ------------------------- | -------------- | -------------------------- |
| 1     | Requête ARP | x         |`marcel` `08:00:27:55:3c:64`| x             | Broadcast `FF:FF:FF:FF:FF` |
| 2     | Réponse ARP | x         |`john` ` 08:00:27:c5:08:3e` | x              |`marcel` `08:00:27:55:3c:64`|
| ...   | ...         | ...       |...                        |                |                            |
| 3     | Ping        | 10.3.2.12 |`marcel` `08:00:27:55:3c:64`| 10.3.1.11     |`john` ` 08:00:27:c5:08:3e`  |
| 4     | Pong        | 10.3.1.11 |`john` ` 08:00:27:c5:08:3e` | 10.3.2.12      |`marcel` `08:00:27:55:3c:64`|
```






3. Accès internet
🌞Donnez un accès internet à vos machines - config routeur

```
[vincent@localhost ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=16.8 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=16.5 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1003ms
rtt min/avg/max/mdev = 16.530/16.675/16.820/0.145 ms
```

```
[vincent@localhost ~]$ sudo firewall-cmd --add-masquerade --permanent
success
[vincent@localhost ~]$ sudo firewall-cmd --reload
success
```


🌞Donnez un accès internet à vos machines - config clients

ajoutez une route par défaut à john et marcel (voir le mémo Rocky)

vérifiez que vous avez accès internet avec un ping

le ping doit être vers une IP, PAS un nom de domaine


donnez leur aussi l'adresse d'un serveur DNS qu'ils peuvent utiliser (voir mémo Rocky)


ping vers un nom de domaine pour vérifier la résolution de nom



🌞Analyse de trames

effectuez un ping depuis john vers marcel

capturez le ping depuis router avec tcpdump

faites deux captures : une sur chaque interface du routeur (celle qui est dans le LAN1 et celle qui est dans le LAN2)


analysez un ping aller et le retour qui correspond et mettez dans un tableau :




ordre
type trame
IP source
MAC source
IP destination
MAC destination





1
ping

marcel 10.3.1.12


marcel AA:BB:CC:DD:EE

8.8.8.8
?



2
pong
...
...
...
...
...




Avec les deux captures vous devez observer 4 trames pour chaque ping échangé : 1) trame ping entre john et router (capture1) 2) trame ping entre router et marcel (capture2) 3) trame pong entre marcel et router (capture1) 4) trame pong entre router et john (capture2)

🦈 Capture réseau tp3_routage_lan1.pcapng
🦈 Capture réseau tp3_routage_lan2.pcapng