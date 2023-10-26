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
"voir tp3_arp.pcap"
```







🌞Ajouter les routes statiques nécessaires pour que john et marcel puissent se ping

voir le mémo Rocky pour ça
il faut taper une commande ip route add

il faut ajouter une seule route des deux côtés
une fois les routes en place, vérifiez avec un ping que les deux machines peuvent se joindre (john et marcel)

Par exemple, la route de john pour joindre le réseau LAN2 de marcel, elle va ressembler à :

# 10.2.2.0/24 est l'adresse du réseau que john ne connaît pas
# 10.2.1.254 est l'adresse de la passerelle qu'il peut déjà joindre : votre router
10.2.2.0/24 via 10.2.1.254 dev enp0s3




2. Analyse de trames
🌞Analyse des échanges ARP

videz les tables ARP des trois noeuds
effectuez un ping de john vers marcel

essayez de déduire les échanges ARP qui ont eu lieu

en regardant les tables ARP des 3 machines
en lançant tcpdump pour capturer le trafic et l'analyser



écrivez, dans l'ordre, les échanges ARP qui ont eu lieu, puis le ping et le pong, je veux TOUTES les trames utiles pour l'échange (ARP et ping/pong)

Par exemple (copiez-collez ce tableau ce sera le plus simple) :



ordre
type trame
IP source
MAC source
IP destination
MAC destination




1
Requête ARP
x

marcel AA:BB:CC:DD:EE

x
Broadcast FF:FF:FF:FF:FF



2
Réponse ARP
x
?
x

marcel AA:BB:CC:DD:EE



...
...
...
...




?
Ping
?
?
?
?


?
Pong
?
?
?
?




3. Accès internet
🌞Donnez un accès internet à vos machines - config routeur

ajoutez une carte NAT (dans l'interface de Virtualbox) en 3ème inteface sur le router pour qu'il ait un accès internet
une fois que c'est fait, vérifiez qu'il a bien un accès internet
sous Rocky...

pour autoriser le routeur à router des paquets vers internet, et faire ça propre, ça nécessite quelques manips
pour qu'il agisse comme un routeur normal quoi
référez-vous au mémo Rocky




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