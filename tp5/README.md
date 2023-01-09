# Partie 1 : Mise en place et maîtrise du serveur Web

Dans cette partie on va installer le serveur web, et prendre un peu la maîtrise dessus, en regardant où il stocke sa conf, ses logs, etc. Et en manipulant un peu tout ça bien sûr.

On va installer un serveur Web très très trèèès utilisé autour du monde : le serveur Web Apache.

- [Partie 1 : Mise en place et maîtrise du serveur Web](#partie-1--mise-en-place-et-maîtrise-du-serveur-web)
  - [1. Installation](#1-installation)
  - [2. Avancer vers la maîtrise du service](#2-avancer-vers-la-maîtrise-du-service)

![Tipiii](../pics/linux_is_a_tipi.jpg)

## 1. Installation

🖥️ **VM web.tp5.linux**

**N'oubliez pas de dérouler la [📝**checklist**📝](../README.md#checklist).**

| Machine         | IP            | Service     |
|-----------------|---------------|-------------|
| `web.tp5.linux` | `10.105.1.11` | Serveur Web |

🌞 **Installer le serveur Apache**
```

- [joel@web ~]$ sudo dnf install httpd
```


🌞 **Démarrer le service Apache**

- le service s'appelle `httpd` (raccourci pour `httpd.service` en réalité)
```
  - `[joel@web ~]$ sudo systemctl start httpd.service`

  - [joel@web ~]$ sudo systemctl enable httpd.service
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
```
```
[joel@web ~]$ systemctl status httpd.service
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor pr>
     Active: active (running) since Tue 2022-12-13 14:44:12 CET; 16s ago
       Docs: man:httpd.service(8)
   Main PID: 41240 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes>
      Tasks: 213 (limit: 5904)
     Memory: 35.2M
        CPU: 43ms
     CGroup: /system.slice/httpd.service
             ├─41240 /usr/sbin/httpd -DFOREGROUND
             ├─41241 /usr/sbin/httpd -DFOREGROUND
             ├─41242 /usr/sbin/httpd -DFOREGROUND
             ├─41243 /usr/sbin/httpd -DFOREGROUND
             └─41244 /usr/sbin/httpd -DFOREGROUND

Dec 13 14:44:11 web.server.tp5 systemd[1]: Starting The Apache HTTP Server...
Dec 13 14:44:12 web.server.tp5 httpd[41240]: Server configured, listening on: p>
Dec 13 14:44:12 web.server.tp5 systemd[1]: Started The Apache HTTP Server.
```
  
  - ouvrez le port firewall nécessaire
```
   [joel@web ~]$ sudo ss -ltunp | grep httpd
tcp   LISTEN 0      511                *:80              *:*    users:(("httpd",pid=41244,fd=4),("httpd",pid=41243,fd=4),("httpd",pid=41242,fd=4),("httpd",pid=41240,fd=4))
```


**En cas de problème** (IN CASE OF FIIIIRE) vous pouvez check les logs d'Apache :

```bash
# Demander à systemd les logs relatifs au service httpd
$ sudo journalctl -xe -u httpd

# Consulter le fichier de logs d'erreur d'Apache
$ sudo cat /var/log/httpd/error_log

# Il existe aussi un fichier de log qui enregistre toutes les requêtes effectuées sur votre serveur
$ sudo cat /var/log/httpd/access_log
```

🌞 **TEST**

- vérifier que le service est démarré
- vérifier qu'il est configuré pour démarrer automatiquement
```
- [joel@web ~]$ curl localhost | head -n 10
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7620  100  7620    0     0  3720k      0 --:--:-- --:--:-- --:--:-- 3720k
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      
      html {

- vérifier depuis votre PC que vous accéder à la page par défaut
  - avec votre navigateur (sur votre PC)

joel@joel-HP-Pavilion-Gaming-Laptop-15-ec2xxx:~$ curl 10.10.15.14 | head -n 10
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7620  100  7620    0     <!doctype html> --:--:-- --:--:-- --:--:--     0
0<html>
   <head>
     <meta charset='utf-8'>
1    <meta name='viewport' content='width=device-width, initial-scale=1'>
6    <title>HTTP Server Test Page powered by: Rocky Linux</title>
7    <style type="text/css">
3      /*<![CDATA[*/
k      
       html {
     0 --:--:-- --:--:-- --:--:-- 1860k

```

🌞 **Le service Apache...**

- affichez le contenu du fichier `httpd.service` qui contient la définition du service Apache

🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

- [joel@web ~]$ cat /etc/httpd/conf/httpd.conf | grep User
User apache
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio

- [joel@web ~]$ ps -ef | grep httpd
root         732       1  0 15:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       753     732  0 15:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       754     732  0 15:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       755     732  0 15:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache       756     732  0 15:03 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
joel        4340    4291  0 15:31 pts/0    00:00:00 grep --color=auto httpd

- la page d'accueil d'Apache se trouve dans `/usr/share/testpage/`
  - vérifiez avec un `ls -al` que tout son contenu est **accessible en lecture** à l'utilisateur mentionné dans le fichier de conf

🌞 **Changer l'utilisateur utilisé par Apache**

- créez un nouvel utilisateur
  - pour les options de création, inspirez-vous de l'utilisateur Apache existant
    - le fichier `/etc/passwd` contient les informations relatives aux utilisateurs existants sur la machine
    - servez-vous en pour voir la config actuelle de l'utilisateur Apache par défaut (son homedir et son shell en particulier)
- modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur
  - montrez la ligne de conf dans le compte rendu, avec un `grep` pour ne montrer que la ligne importante
- redémarrez Apache
- utilisez une commande `ps` pour vérifier que le changement a pris effet
  - vous devriez voir un processus au moins qui tourne sous l'identité de votre nouvel utilisateur

🌞 **Faites en sorte que Apache tourne sur un autre port**

- modifiez la configuration d'Apache pour lui demander d'écouter sur un autre port de votre choix
  - montrez la ligne de conf dans le compte rendu, avec un `grep` pour ne montrer que la ligne importante
- ouvrez ce nouveau port dans le firewall, et fermez l'ancien
- redémarrez Apache
- prouvez avec une commande `ss` que Apache tourne bien sur le nouveau port choisi
- vérifiez avec `curl` en local que vous pouvez joindre Apache sur le nouveau port
- vérifiez avec votre navigateur que vous pouvez joindre le serveur sur le nouveau port

📁 **Fichier `/etc/httpd/conf/httpd.conf`**

➜ **Si c'est tout bon vous pouvez passer à [la partie 2.](../part2/README.md)**




# Partie 2 : Mise en place et maîtrise du serveur de base de données

Petite section de mise en place du serveur de base de données sur `db.tp5.linux`. On ira pas aussi loin qu'Apache pour lui, simplement l'installer, faire une configuration élémentaire avec une commande guidée (`mysql_secure_installation`), et l'analyser un peu.

🖥️ **VM db.tp5.linux**

**N'oubliez pas de dérouler la [📝**checklist**📝](#checklist).**

| Machines        | IP            | Service                 |
|-----------------|---------------|-------------------------|
| `web.tp5.linux` | `10.105.1.11` | Serveur Web             |
| `db.tp5.linux`  | `10.105.1.12` | Serveur Base de Données |

🌞 **Install de MariaDB sur `db.tp5.linux`**

-  `[joel@db ~]$ sudo dnf install mariadb-server`

```
[joel@db ~]$ sudo systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
```




`[joel@db ~]$ sudo systemctl start mariadb`


```
[joel@db ~]$ sudo systemctl status mariadb
● mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor p>
     Active: active (running) since Tue 2022-12-13 16:28:14 CET; 15s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 3698 ExecStartPre=/usr/libexec/mariadb-check-socket (code=exited, >
    Process: 3720 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir mariadb.serv>
    Process: 3816 ExecStartPost=/usr/libexec/mariadb-check-upgrade (code=exited>
   Main PID: 3802 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 13 (limit: 5904)
     Memory: 76.4M
        CPU: 230ms
     CGroup: /system.slice/mariadb.service
             └─3802 /usr/libexec/mariadbd --basedir=/usr

Dec 13 16:28:14 db.tp5.linux mariadb-prepare-db-dir[3759]: you need to be the s>
Dec 13 16:28:14 db.tp5.linux mariadb-prepare-db-dir[3759]: After connecting you>
Dec 13 16:28:14 db.tp5.linux mariadb-prepare-db-dir[3759]: able to connect as a>
Dec 13 16:28:14 db.tp5.linux mariadb-prepare-db-dir[3759]: See the MariaDB Know>
Dec 13 16:28:14 db.tp5.linux mariadb-prepare-db-dir[3759]: Please report any pr>
Dec 13 16:28:14 db.tp5.linux mariadb-prepare-db-dir[3759]: The latest informati>
```



`[joel@db ~]$ mysql_secure_installation`



🌞 **Port utilisé par MariaDB**

- vous repérerez le port utilisé par MariaDB avec une commande `ss` exécutée sur `db.tp5.linux`
  - filtrez les infos importantes avec un `| grep`
- il sera nécessaire de l'ouvrir dans le firewall

> La doc vous fait exécuter la commande `mysql_secure_installation` c'est un bon réflexe pour renforcer la base qui a une configuration un peu *chillax* à l'install.

🌞 **Processus liés à MariaDB**

- repérez les processus lancés lorsque vous lancez le service MariaDB
```
[joel@db ~]$ ps -ef | grep mariadb
mysql        806       1  0 16:33 ?        00:00:00 /usr/libexec/mariadbd --basedir=/usr
joel        1256     951  0 16:54 pts/0    00:00:00 grep --color=auto mariadb
```

