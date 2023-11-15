

I. First steps

🌞 Déterminez, pour ces 5 applications, si c'est du TCP ou de l'UDP


IP et port du serveur auquel vous vous connectez
le port local que vous ouvrez pour vous connecter

netflix (service 1)
```
ip : 52.17.149.210
Port local : 58446
Port du serveur : 443
```

Whatsapp (service 2)
```
ip : 185.60.219.61
Port local : 58620
Port du serveur : 443
```

steam (service 3)
```
ip : 23.33.90.34
Port local : 58643
Port du serveur : 443
```

minecraft (service4)
```
ip : 40.126.32.72
Port local: 58707
Port du serveur : 443
```

teams (service 5)
```
ip : 20.50.73.11
Port local: 58777
Port du serveur : 443
```


🌞 Demandez l'avis à votre OS

netflix (ne 'affiche pas correctement)
```
PS C:\WINDOWS\system32> netstat -n -b | Select-String "www.netflix.com" -Context 1,0
PS C:\WINDOWS\system32> netstat -n -b | Select-String "104.16" -Context 0,1
>   TCP    10.33.70.190:59247     104.16.248.249:443     TIME_WAIT
>   TCP    10.33.70.190:59248     104.16.248.249:443     TIME_WAIT
    TCP    127.0.0.1:54040        127.0.0.1:54041        ESTABLISHED
```

Whatsapp
```
PS C:\WINDOWS\system32> netstat -n -b | Select-String "185.60.219" -Context 0,1

>   TCP    10.33.70.190:59256     185.60.219.61:80       TIME_WAIT
    TCP    10.33.70.190:59271     20.42.73.24:443        ESTABLISHED
>   TCP    10.33.70.190:59319     185.60.219.61:80       ESTABLISHED
   [WhatsApp.exe]
```

Steam
```
PS C:\WINDOWS\system32> netstat -n -b | Select-String "92.122.218" -Context 0,1

>   TCP    10.33.70.190:59340     92.122.218.96:443      ESTABLISHED
   [steamwebhelper.exe]
>   TCP    10.33.70.190:59343     92.122.218.73:443      ESTABLISHED
   [steamwebhelper.exe]
```

minecraft
```
PS C:\WINDOWS\system32> netstat -n -b | Select-String "52.237.134" -Context 0,1

>   TCP    10.33.70.190:59304     52.237.134.81:443      ESTABLISHED
   [GameBarPresenceWriter.exe]
>   TCP    10.33.70.190:59305     52.237.134.81:443      ESTABLISHED
   [GameBarPresenceWriter.exe]
```

Teams
```
PS C:\WINDOWS\system32> netstat -n -b | Select-String "52.114.75" -Context 0,1

>   TCP    10.33.70.190:58783     52.114.75.169:443      ESTABLISHED
   [Teams.exe]
```


🦈🦈🦈🦈🦈 Bah ouais, 5 captures Wireshark à l'appui évidemment. Ce sera tp5_service_1.pcapng jusqu'à tp5_service_5.pcapng.

Une capture pour chaque application, qui met bien en évidence le trafic en question. C'est à dire on ne doit voir QUE le trafic de l'application en question. Vous devez donc utiliser un filtre dans Wireshark pour isoler le trafic que vous souhaitez.


II. Setup Virtuel

1. SSH

Connectez-vous en SSH à votre VM.
🌞 Examinez le trafic dans Wireshark


Déterminez si SSH utilise TCP ou UDP
```
ssh utilise TCP
```

Repérez le 3-Way Handshake à l'établissement de la connexion

```
1	0.000000	10.5.1.1	10.5.1.11	TCP	66	59538 → 22 [SYN] Seq=0 Win=64240 Len=0 MSS=1460 WS=256 SACK_PERM

2	0.000172	10.5.1.11	10.5.1.1	TCP	66	22 → 59538 [SYN, ACK] Seq=0 Ack=1 Win=64240 Len=0 MSS=1460 SACK_PERM WS=128

3	0.000340	10.5.1.1	10.5.1.11	TCP	54	59538 → 22 [ACK] Seq=1 Ack=1 Win=262656 Len=0
```

Repérez le FIN ACK à la fin d'une connexion
```
77	11.355864	10.5.1.1	10.5.1.11	TCP	54	59538 → 22 [FIN, ACK] Seq=2542 Ack=3014 Win=2096384 Len=0
```

🌞 Demandez aux OS

Repérez, avec une commande adaptée (netstat ou ss), la connexion SSH depuis votre machine et repérez la connexion SSH depuis la VM

```
[vincent@node1 ~]$ ss -tpn
State     Recv-Q     Send-Q           Local Address:Port           Peer Address:Port      Process
ESTAB     0          0                    10.5.1.11:22                 10.5.1.1:59461
```
```
PS C:\WINDOWS\system32> netstat -n -b | Select-String "ssh" -Context 1,0

    TCP    10.5.1.1:59461         10.5.1.11:22           ESTABLISHED
>  [ssh.exe]
```
🦈 tp5_3_way.pcapng : une capture clean avec le 3-way handshake, un peu de trafic au milieu et une fin de connexion

Là encore, dans la capture, il ne doit y avoir QUE votre connexion SSH, du commencement de la session SSH, jusqu'à ce qu'elle se termine.

2. Routage


🌞 Prouvez que


node1.tp5.b1 a un accès internet

```
[vincent@node1 ~]$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=24.6 ms
^C
--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 24.576/24.576/24.576/0.000 ms
```

node1.tp5.b1 peut résoudre des noms de domaine publics (comme www.ynov.com)

```
[vincent@node1 ~]$ ping google.com
PING google.com (172.217.20.206) 56(84) bytes of data.
64 bytes from par10s50-in-f14.1e100.net (172.217.20.206): icmp_seq=1 ttl=115 time=30.2 ms
64 bytes from par10s50-in-f14.1e100.net (172.217.20.206): icmp_seq=2 ttl=115 time=23.2 ms
^C
--- google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1005ms
rtt min/avg/max/mdev = 23.248/26.725/30.202/3.477 ms
```

. Serveur Web

🌞 Installez le paquet nginx

```
[vincent@web ~]$ sudo dnf install ngix -y
```

🌞 Créer le site web

quand on héberge des sites web sur un serveur Linux, il existe un dossier qu'on utilise pour stocker les sites web, par convention

ce dossier c'est /var/www/
créer donc un dossier /var/www/site_web_nul/
créer un fichier /var/www/site_web_nul/index.html qui contient un code HTML simpliste de votre choix


```
[vincent@web ~]$ cd /var/
[vincent@web var]$ cd www/
-bash: cd: www/: No such file or directory
[vincent@web var]$ sudo mkdir www
[vincent@web var]$ cd www/
[vincent@web www]$ sudo mkdir site_web_nul
[vincent@web www]$ cd site_web_nul/
[vincent@web site_web_nul]$ sudo touch index.html
[vincent@web site_web_nul]$ sudo nano index.html
<h1>MEOW</h1>
```


🌞 Donner les bonnes permissions

```
[vincent@web site_web_nul]$ sudo chown -R nginx:nginx /var/www/site_web_nul
[vincent@web site_web_nul]$ ls -l
total 4
-rw-r--r--. 1 nginx nginx 14 Nov 14 11:31 index.html
```
🌞 Créer un fichier de configuration NGINX pour notre site web

```
[vincent@web ~]$ cd /etc/nginx/conf.d/
[vincent@web conf.d]$ sudo touch site_web_nul.conf
[vincent@web conf.d]$ sudo nano site_web_nul.conf


server {
  # le port sur lequel on veut écouter
  listen 80;

  # le nom de la page d'accueil si le client de la précise pas
  index index.html;

  # un nom pour notre serveur
  server_name www.site_web_nul.b1;

  # le dossier qui contient notre site web
  root /var/www/site_web_nul;
}
```

🌞 Démarrer le serveur web !

```
[vincent@web ~]$ sudo systemctl start nginx
[vincent@web ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-11-14 11:36:02 CET; 32s ago
    Process: 1194 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1195 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1196 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1197 (nginx)
      Tasks: 2 (limit: 5896)
     Memory: 1.9M
        CPU: 13ms
     CGroup: /system.slice/nginx.service
             ├─1197 "nginx: master process /usr/sbin/nginx"
             └─1198 "nginx: worker process"

Nov 14 11:36:02 web.tp5.b1 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Nov 14 11:36:02 web.tp5.b1 nginx[1195]: nginx: the configuration file /etc/nginx/nginx.conf syntax is>
Nov 14 11:36:02 web.tp5.b1 nginx[1195]: nginx: configuration file /etc/nginx/nginx.conf test is succe>
Nov 14 11:36:02 web.tp5.b1 systemd[1]: Started The nginx HTTP and reverse proxy server.
lines 1-18/18 (END)
```

🌞 Ouvrir le port firewall

```
[vincent@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[vincent@web ~]$ sudo firewall-cmd --reload
success
[vincent@web ~]$
```

🌞 Visitez le serveur web !

```
[vincent@node1 ~]$ curl 10.5.1.12:80
<h1>MEOW</h1>
```


🌞 Visualiser le port en écoute

```
[vincent@node1 ~]$ sudo ss -l -t -n
[sudo] password for vincent:
State      Recv-Q     Send-Q           Local Address:Port           Peer Address:Port     Process
LISTEN     0          128                    0.0.0.0:22                  0.0.0.0:*
LISTEN     0          128                       [::]:22                     [::]:*
```


🌞 Analyse trafic

```
voir tp5_web.pcapng
```