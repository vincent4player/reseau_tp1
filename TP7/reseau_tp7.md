B. Manips


🌞 Effectuez une connexion SSH en vérifiant le fingerprint

```
PS C:\Users\vince> ssh vincent@10.7.1.11
The authenticity of host '10.7.1.11 (10.7.1.11)' can't be established.
ED25519 key fingerprint is SHA256:LiWk49XIVPOBhCg7+FRoMGuM+WQZInba139U+eUhqtM.
This host key is known by the following other names/addresses:
    C:\Users\vince/.ssh/known_hosts:5: 10.3.1.11
    C:\Users\vince/.ssh/known_hosts:8: 10.3.1.12
    C:\Users\vince/.ssh/known_hosts:9: 10.3.2.12
    C:\Users\vince/.ssh/known_hosts:10: 10.3.2.254
    C:\Users\vince/.ssh/known_hosts:11: 10.3.1.254
    C:\Users\vince/.ssh/known_hosts:12: 10.4.1.253
    C:\Users\vince/.ssh/known_hosts:13: 10.4.1.254
    C:\Users\vince/.ssh/known_hosts:14: 10.4.1.12
    (11 additional names omitted)
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.7.1.11' (ED25519) to the list of known hosts.
vincent@10.7.1.11's password:
Last login: Thu Nov 23 11:04:07 2023 from 10.7.1.1
```

2. Conf serveur SSH

🌞 Consulter l'état actuel


vérifiez que le serveur SSH tourne actuellement sur le port 22/tcp
```
[vincent@router ~]$ sudo nano /etc/ssh/sshd_config
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

vérifiez que le serveur SSH est disponible actuellement sur TOUTES les IPs de la machine
```
[vincent@john ~]$ sudo ss -tnl | grep :22
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*
LISTEN 0      128             [::]:22           [::]:*
```

🌞 Modifier la configuration du serveur SSH

sur router.tp7.b1 uniquement
éditer le fichier de configuration du serveur SSH pour spécifier que :
le serveur doit écouter sur l'adresse IP 10.7.1.254
le serveur doit écouter sur le port de votre choix entre 20000 et 30000

```
[vincent@router ~]$ sudo nano /etc/ssh/sshd_config
Port 22000
#AddressFamily any
ListenAddress 10.7.1.254
#ListenAddress ::
```

🌞 Prouvez que le changement a pris effet

Toujours avec la même commande ss vous devriez voir que :
    le serveur SSH écoute désormais sur 10.7.1.11 uniquement
    le serveur SSH écoute désormais sur le port choisi

```
[vincent@router ~]$ sudo ss -lntp
State      Recv-Q     Send-Q          Local Address:Port            Peer Address:Port     Process
LISTEN     0          128                10.7.1.254:22000                0.0.0.0:*         users:(("sshd",pid=1036,fd=3))
```

🌞 Effectuer une connexion SSH sur le nouveau port

```
PS C:\Users\vince> ssh vincent@router -p 22000
The authenticity of host '[router]:22000 ([10.7.1.254]:22000)' can't be established.
ED25519 key fingerprint is SHA256:LiWk49XIVPOBhCg7+FRoMGuM+WQZInba139U+eUhqtM.
This host key is known by the following other names/addresses:
    C:\Users\vince/.ssh/known_hosts:5: 10.3.1.11
    C:\Users\vince/.ssh/known_hosts:8: 10.3.1.12
    C:\Users\vince/.ssh/known_hosts:9: 10.3.2.12
    C:\Users\vince/.ssh/known_hosts:10: 10.3.2.254
    C:\Users\vince/.ssh/known_hosts:11: 10.3.1.254
    C:\Users\vince/.ssh/known_hosts:12: 10.4.1.253
    C:\Users\vince/.ssh/known_hosts:13: 10.4.1.254
    C:\Users\vince/.ssh/known_hosts:14: 10.4.1.12
    (12 additional names omitted)
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[router]:22000' (ED25519) to the list of known hosts.
vincent@router's password:
Last login: Fri Nov 24 09:04:18 2023
```

3. Connexion par clé



B. Manips
Let's go :
🌞 Générer une paire de clés

```
PS C:\Users\vince> ls .\.ssh\id_rsa.pub


    Répertoire : C:\Users\vince\.ssh


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        23/11/2023     10:22            736 id_rsa.pub
```

🌞 Déposer la clé publique sur une VM

```
$ ssh-copy-id vincent@10.7.1.11
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/c/Users/vince/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed

/usr/bin/ssh-copy-id: WARNING: All keys were skipped because they already exist on the remote system.
                (if you think this is a mistake, you may want to use -f option)
```

🌞 Connectez-vous en SSH à la machine

```
PS C:\Users\vince> ssh vincent@10.7.1.11
Last login: Fri Nov 24 09:42:26 2023 from 10.7.1.1
[vincent@john ~]$
```

🌞 Supprimer les clés sur la machine router.tp7.b1

```
[vincent@router ~]$ sudo rm -r /etc/ssh/ssh_host_*
[sudo] password for vincent:
```

🌞 Regénérez les clés sur la machine router.tp7.b1

```
[vincent@router ~]$ sudo ssh-keygen -A
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
```

🌞 Tentez une nouvelle connexion au serveur

```
PS C:\Users\vince> ssh vincent@router -p 22000
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:18tEJbVnZGDv02l4eFmWBG6kyYtRQS9xbhJRFi1UVPk.
Please contact your system administrator.
Add correct host key in C:\\Users\\vince/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in C:\\Users\\vince/.ssh/known_hosts:28
Host key for [router]:22000 has changed and you have requested strict checking.
Host key verification failed.
```

```
PS C:\Users\vince> ssh vincent@router -p 22000
The authenticity of host '[router]:22000 ([10.7.1.254]:22000)' can't be established.
ED25519 key fingerprint is SHA256:18tEJbVnZGDv02l4eFmWBG6kyYtRQS9xbhJRFi1UVPk.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[router]:22000' (ED25519) to the list of known hosts.
vincent@router's password:
Last login: Fri Nov 24 09:26:49 2023 from 10.7.1.1
[vincent@router ~]$
```


🌞 Montrer sur quel port est disponible le serveur web

```
[vincent@web ~]$ sudo ss -l -t -n
State      Recv-Q     Send-Q           Local Address:Port           Peer Address:Port     Process
LISTEN     0          511                    0.0.0.0:80                  0.0.0.0:*
LISTEN     0          128                    0.0.0.0:22                  0.0.0.0:*
LISTEN     0          511                       [::]:80                     [::]:*
LISTEN     0          128                       [::]:22                     [::]:*
```


1. Setup HTTPS


🌞 Générer une clé et un certificat sur web.tp7.b1

```
[vincent@web ~]$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
.+.+........+.........+......................+...+...+..+.......+...+......+.....+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.......+.......+..+....+.....+............+...+......+...+.......+......+...+..+......................+..+....+......+...........+.......+.....+.+......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+......+.....+.+...+...........+...+.......+.........+.....+......+...+.+......+..............+.+..............+......+.+..+.............+........+....+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*..+........+...+.......+...+......+......+..+............+...+......+..........+..+...+............+....+.........+..+...+.........+.+..+....+...+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++*.+.+.........+..+.........+.+..+.......+...+...+........+....+..+.+.................+.......+......+.....+....+........+.+......+......+............+..+.........+.+...+...+...+..+.......+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:fr
State or Province Name (full name) []:Nouvelle aquitaine
Locality Name (eg, city) [Default City]:bordeaux
Organization Name (eg, company) [Default Company Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:
Email Address []:
```

🌞 Modification de la conf de NGINX

```
[vincent@web ~]$ sudo cat /etc/nginx/conf.d/site_web_nul.conf
server {
    # on change la ligne listen
    listen 10.7.1.12:443 ssl;

    # et on ajoute deux nouvelles lignes
    ssl_certificate /etc/pki/tls/certs/web.tp7.b1.crt;
    ssl_certificate_key /etc/pki/tls/private/web.tp7.b1.key;

    server_name www.site_web_nul.b1;
    root /var/www/site_web_nul;
}
```

🌞 Conf firewall

```
[vincent@web ~]$ sudo firewall-cmd --add-port=443/tcp --permanent
success
[vincent@web ~]$ sudo firewall-cmd --reload
success
```

🌞 Redémarrez NGINX

```
[vincent@web ~]$ sudo systemctl restart nginx
```

🌞 Prouvez que NGINX écoute sur le port 443/tcp

```
[vincent@web ~]$ sudo ss -tnl | grep 443
LISTEN 0      511        10.7.1.12:443       0.0.0.0:*
```

🌞 Visitez le site web en https

```
[vincent@web ~]$ curl -k https://10.7.1.12
<h1>MEOW</h1>
```