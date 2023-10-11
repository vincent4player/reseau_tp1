TP 1


🌞 Affichez les infos des cartes réseau de votre PC

```
PS C:\Users\vince> ipconfig /all

Carte réseau sans fil Wi-Fi :

  
   Description. . . . . . . . . . . . . . : Intel(R) Wi-Fi 6 AX201 160MHz
   Adresse physique . . . . . . . . . . . : 98-8D-46-C4-FA-E5
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.48.9(préféré)
  
   ```
```
   Carte Ethernet Ethernet :

   Statut du média. . . . . . . . . . . . : Média déconnecté
   Suffixe DNS propre à la connexion. . . : home
   Description. . . . . . . . . . . . . . : Realtek PCIe GbE Family Controller
   Adresse physique . . . . . . . . . . . : 2C-F0-5D-6C-11-B8
   DHCP activé. . . . . . . . . . . . . . : Oui
   Configuration automatique activée. . . : Oui
  
   ```

🌞 Affichez votre gateway

```
PS C:\Users\vince> ipconfig

Carte réseau sans fil Wi-Fi :

   Passerelle par défaut. . . . . . . . . : 10.33.51.254
   ```


🌞 Déterminer la MAC de la passerelle
```
PS C:\Users\vince> arp -a 10.33.51.254

Interface : 10.33.48.9 --- 0x6
  Adresse Internet      Adresse physique      Type
  10.33.51.254          7c-5a-1c-cb-fd-a4     dynamique
  ```
 
 🌞 Trouvez comment afficher les informations sur une carte IP (change selon l'OS)
 ```
  Interface graphique:

    Adresse iPv4 : 10.33.48.9
    Adresse physique(MAC) : 98-8D-46-C4-FA-E5
    Gateway: 10.33.51.254
```
🌞 Utilisez l'interface graphique de votre OS pour changer d'adresse IP :

🌞 Il est possible que vous perdiez l'accès internet. Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.
```
    Conflit d'adresses IP : Si une autre périphérie sur le même réseau local utilise déjà l'adresse IP que vous essayez d'assigner, un conflit d'adresses IP peut survenir. Cela peut entraîner des problèmes de connectivité.

    Paramètres de passerelle incorrects : L'adresse IP de la passerelle par défaut doit être mise à jour en conséquence. Si cela n'est pas fait correctement, vous perdrez l'accès à Internet.
```

TP 1 SUITE DUO