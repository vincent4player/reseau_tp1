I. Setup IP

🌞 Mettez en place une configuration réseau fonctionnelle entre les deux machines
```
PS C:\Users\vince> ipconfig

Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv6 de liaison locale. . . . .: fe80::26fb:37cb:53b0:a838%7
   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.2
   Masque de sous-réseau. . . . . . . . . : 255.255.255.252
   Passerelle par défaut. . . . . . . . . : 10.10.10.0
```
- Adresse Broadcast
```
10.10.10.3 
```
```
PS C:\Users\vince> arp -a 10.10.10.1

Interface : 10.10.10.2 --- 0x7
  Adresse Internet      Adresse physique      Type
  10.10.10.1            22-e0-4c-a1-31-36     dynamique
```


🌞 Prouvez que la connexion est fonctionnelle entre les deux machines

```
PS C:\Users\vince> ping 10.10.10.1

Envoi d’une requête 'Ping'  10.10.10.1 avec 32 octets de données :
Réponse de 10.10.10.1 : octets=32 temps=1 ms TTL=64
Réponse de 10.10.10.1 : octets=32 temps=1 ms TTL=64
Réponse de 10.10.10.1 : octets=32 temps=1 ms TTL=64
Réponse de 10.10.10.1 : octets=32 temps=1 ms TTL=64

Statistiques Ping pour 10.10.10.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms
```

🌞 Wireshark it


🦈 PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP (juste quelques-uns)

II. ARP my bro

🌞 Check the ARP table
```
PS C:\Users\vince> arp -a

Interface : 10.10.10.2 --- 0x7

  Adresse Internet      Adresse physique      Type
  10.10.10.1            08-bf-b8-c2-2a-57     dynamique
  10.10.10.3            ff-ff-ff-ff-ff-ff     statique
  10.10.10.12           08-bf-b8-c2-2a-57     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

🌞 Manipuler la table ARP
```
PS C:\WINDOWS\system32> arp -d
La suppression de l'entrée ARP a échoué : Paramètre incorrect.

PS C:\WINDOWS\system32> netsh interface ip delete arpcache
Ok.

PS C:\WINDOWS\system32> arp -a

Interface : 192.168.20.1 --- 0x4
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.10.145.156 --- 0x6
  Adresse Internet      Adresse physique      Type
  10.10.128.1           28-de-65-73-6f-e6     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.56.1 --- 0xb
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.100.1 --- 0x13
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

```


🌞 Wireshark it

vous savez maintenant comment forcer un échange ARP : il sufit de vider la table ARP et tenter de contacter quelqu'un, l'échange ARP se fait automatiquement
mettez en évidence les deux trames ARP échangées lorsque vous essayez de contacter quelqu'un pour la "première" fois

déterminez, pour les deux trames, les adresses source et destination
déterminez à quoi correspond chacune de ces adresses



🦈 PCAP qui contient les DEUX trames ARP

L'échange ARP est constitué de deux trames : un ARP broadcast et un ARP reply.


III. DHCP

DHCP pour Dynamic Host Configuration Protocol est notre p'tit pote qui nous file des IPs quand on arrive dans un réseau, parce que c'est chiant de le faire à la main :)
Quand on arrive dans un réseau, notre PC contacte un serveur DHCP, et récupère généralement 3 infos :


1. une IP à utiliser

2. l'adresse IP de la passerelle du réseau

3. l'adresse d'un serveur DNS joignable depuis ce réseau

L'échange DHCP  entre un client et le serveur DHCP consiste en 4 trames : DORA, que je vous laisse chercher sur le web vous-mêmes : D
🌞 Wireshark it

identifiez les 4 trames DHCP lors d'un échange DHCP

mettez en évidence les adresses source et destination de chaque trame


identifiez dans ces 4 trames les informations 1, 2 et 3 dont on a parlé juste au dessus

🦈 PCAP qui contient l'échange DORA

Soucis : l'échange DHCP ne se produit qu'à la première connexion. Pour forcer un échange DHCP, ça dépend de votre OS. Sur GNU/Linux, avec dhclient ça se fait bien. Sur Windows, le plus simple reste de définir une IP statique pourrie sur la carte réseau, se déconnecter du réseau, remettre en DHCP, se reconnecter au réseau (ché moa sa march). Essayez de regarder par vous-mêmes s'il existe une commande clean pour faire ça ! Sur MacOS, je connais peu mais Internet dit qu'c'est po si compliqué, appelez moi si besoin.