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
a partir de maintenant j'utiliserai comme binome une VM qui comme ip: 10.3.1.11
```
```
PS C:\WINDOWS\system32> ping 10.3.1.11

Envoi d’une requête 'Ping'  10.3.1.11 avec 32 octets de données :
Réponse de 10.3.1.11 : octets=32 temps<1ms TTL=128
Réponse de 10.3.1.11 : octets=32 temps<1ms TTL=128
Réponse de 10.3.1.11 : octets=32 temps<1ms TTL=128
Réponse de 10.3.1.11 : octets=32 temps<1ms TTL=128

Statistiques Ping pour 10.3.1.11:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 0ms, Maximum = 0ms, Moyenne = 0ms
```

🌞 Wireshark it
```
voir "wireshark 1 ping"
j'ai ping mon pc depuis ma vm
```

II. ARP my bro

🌞 Check the ARP table

```
PS C:\Users\vince> arp -a

Interface : 10.3.1.11 --- 0x44
  Adresse Internet      Adresse physique      Type
  10.3.1.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

🌞 Manipuler la table ARP 
```
je n'ai plus mon binome je l'ai fais avec une vm
```
```
PS C:\WINDOWS\system32> arp -d
PS C:\WINDOWS\system32> arp -a

Interface : 10.3.1.11 --- 0x44
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

```


🌞 Wireshark it

"voir wireshark 2 ping"


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