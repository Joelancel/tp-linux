# TP2 : Appréhender l'environnement Linux

# I. Service SSH

Le service SSH est déjà installé sur la machine, et il est aussi déjà démarré par défaut, c'est Rocky qui fait ça nativement.

## 1. Analyse du service

On va, dans cette première partie, analyser le service SSH qui est en cours d'exécution.

🌞 **S'assurer que le service `sshd` est démarré**

```
[joel@localhost ~]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor pres>
     Active: active (running) since Tue 2022-11-22 16:30:59 CET; 2min 1s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 736 (sshd)
      Tasks: 1 (limit: 5899)
     Memory: 5.7M
        CPU: 44ms
     CGroup: /system.slice/sshd.service
             └─736 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```


🌞 **Analyser les processus liés au service SSH**

```
[joel@localhost ~]$ ps -ef | grep sshd
root         736       1  0 16:31 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root         889     736  0 16:31 ?        00:00:00 sshd: joel [priv]
joel         903     889  0 16:31 ?        00:00:00 sshd: joel@pts/0
joel         944     904  0 16:39 pts/0    00:00:00 grep --color=auto sshd
```


🌞 **Déterminer le port sur lequel écoute le service SSH**

```
[joel@localhost ~]$ ss -t | grep ssh
ESTAB 0      0      192.168.56.102:ssh  192.168.56.1:52460  
``` 

🌞 **Consulter les logs du service SSH**

```
[joel@localhost ~]$ sudo !!
sudo cat /var/log/secure | tail
[sudo] password for joel: 
Nov 22 16:30:59 localhost sshd[736]: Server listening on :: port 22.
Nov 22 16:31:05 localhost systemd[860]: pam_unix(systemd-user:session): session opened for user root(uid=0) by (uid=0)
Nov 22 16:31:05 localhost login[740]: pam_unix(login:session): session opened for user root(uid=0) by (uid=0)
Nov 22 16:31:05 localhost login[740]: ROOT LOGIN ON tty1
Nov 22 16:31:13 localhost sshd[889]: Accepted publickey for joel from 192.168.56.1 port 52460 ssh2: RSA SHA256:gBDPvE6ttsVQKbO1iZPl0yXTYKQK+1JZITdeVb97c0E
Nov 22 16:31:13 localhost systemd[894]: pam_unix(systemd-user:session): session opened for user joel(uid=1000) by (uid=0)
Nov 22 16:31:13 localhost sshd[889]: pam_unix(sshd:session): session opened for user joel(uid=1000) by (uid=0)
Nov 22 16:35:54 localhost sudo[929]:    joel : TTY=pts/0 ; PWD=/home/joel ; USER=root ; COMMAND=/sbin/service ssh status
Nov 22 16:35:54 localhost sudo[929]: pam_unix(sudo:session): session opened for user root(uid=0) by joel(uid=1000)
Nov 22 16:35:54 localhost sudo[929]: pam_unix(sudo:session): session closed for user root
```

## 2. Modification du service

Dans cette section, on va aller visiter et modifier le fichier de configuration du serveur SSH.

Comme tout fichier de configuration, celui de SSH se trouve dans le dossier `/etc/`.

Plus précisément, il existe un sous-dossier `/etc/ssh/` qui contient toute la configuration relative au protocole SSH

🌞 **Identifier le fichier de configuration du serveur SSH**
`/etc/ssh/sshd_config`
🌞 **Modifier le fichier de conf**

```
[joel@localhost ~]$ echo $RANDOM
28096
```
```
[joel@localhost ~]$ sudo cat /etc/ssh/sshd_config | grep Port
#Port 28096
#GatewayPorts no
```
```
[joel@tp2linux ~]$ sudo firewall-cmd --list-all | grep ports
  ports: 28096/tcp
  forward-ports: 
  source-ports:
```

🌞 **Redémarrer le service**

`[joel@tp2linux ~]$ sudo systemctl restart NetworkManager`


🌞 **Effectuer une connexion SSH sur le nouveau port**

`ssh joel@192.168.56.102 -p 28096`