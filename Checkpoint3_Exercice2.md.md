# Checkpoint 3 - Exercice 2

## Généralités
Connexion à la VM SRVLX01 :
- Utilisateur : wilder
- wcs4ever
- Utilisateur : root
- MXtTvqGXmZDf

## Partie 1 : Gestion des utilisateurs

**Q.2.1.1** Sur le serveur, créer un compte pour ton usage personnel.  
Pour créer un compte, il faut dans un premier temps que je me log en `root`. J'utiliserais la commande `adduser`.  
```bash
root@SRVLX01:~# adduser sybill
Ajout de l'utilisateur « sybill » ...
Ajout du nouveau groupe « sybill » (1001) ...
Ajout du nouvel utilisateur « sybill » (1001) avec le groupe « sybill » ...
Création du répertoire personnel « /home/sybill »...
Copie des fichiers depuis « /etc/skel »...
Nouveau mot de passe :
Retapez le nouveau mot de passe :
passwd: password updated successfully
Changing the user information for sybill
Enter the new value, or press ENTER for the default
        Full Name []: Sybill
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Cette information est-elle correcte ? [O/n]O
root@SRVLX01:~#
```

**Q.2.1.2** Quelles préconisations proposes-tu concernant ce compte ?  
Je préconise que ce compte sera un compte administrateur.

### Partie 2 : Configuration de SSH

Un serveur SSH est lancé sur le port par défaut.   
Il est possible de s'y connecter avec n'importe quel compte, y compris le compte root.  

**Q.2.2.1** Désactiver complètement l'accès à distance de l'utilisateur root.  
Dans un premier temps, nous allons chercher quel fichier donne l'autorisation à l'accès à distance de l'utilisateur root avec la commande `grep -r "PermitRootLogin" /etc/ssh/`, ce qui nous donne :  
![1](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/1-SSHRootLoginYes.png)

En suite nous modifions le fichier `/etc/ssh/sshd_config.d/local.conf` et vous changez le **yes** en **no**, puis sauvegarder le fichier. Maintenant il faut redémarrer le système avec la commande `systemctl restart ssh`. Et si vous tentez de vous connectez depuis la machine hôte en ssh sur l'utilisateur `root`, vous aurez :
```bash
10:58:54 root@MaeshviraTower ~ → ssh root@192.168.1.51
root@192.168.1.51's password:
Permission denied, please try again.
root@192.168.1.51's password:
```

**Q.2.2.2** Autoriser l'accès à distance à ton compte personnel uniquement.  
Pour autoriser l'accès à distance à mon compte seulement, modifiez le fichier `/etc/ssh/sshd_config.d/local.conf` pour rajouter la ligne `AllowUsers sybill`. Voici les différentes réponses lorsque j'essaye de me connecter aux comptes **wilder** & **sybill** :
```bash
11:01:46 root@MaeshviraTower ~ → ssh wilder@192.168.1.51
wilder@192.168.1.51's password:
Permission denied, please try again.
wilder@192.168.1.51's password:

11:11:27 root@MaeshviraTower ~ → ssh sybill@192.168.1.51
sybill@192.168.1.51's password:
Linux SRVLX01 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
sybill@SRVLX01:~$
```

**Q.2.2.3** Mettre en place une authentification par clé valide et désactiver l'authentification par mot de passe  
Cette méthode permettra de ne plus avoir besoin de rentrer le mot de passe. Pour cela, il faut dans un premier temps générer une clé avec la commande `ssh-keygen` :
- Par défaut, la clé sera enregistré dans `/root/.ssh`
- Il sera alors demandé de saisir une *passphrase* : `checkpoint3`, il est recommandé d'en mettre une.
- La clé `id_rsa.pub` est créé !  
![2](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/2-keygenSSH.png)

Maintenant, nous devons envoyer la clef sur notre machine hôte avec la commande `ssh-copy-id sybill@192.168.1.51`, ce qui donne :
```bash
11:13:23 root@MaeshviraTower ~ → ssh-copy-id sybill@192.168.1.51
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
sybill@192.168.1.51's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'sybill@192.168.1.51'"
and check to make sure that only the key(s) you wanted were added.

11:21:56 root@MaeshviraTower ~ → ssh sybill@192.168.1.51
Linux SRVLX01 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Fri Jan 17 11:11:43 2025 from 192.168.1.20
sybill@SRVLX01:~$
```

J'ai pu me connecter sans mettre de mot de passe après la copie de la clé !

### Partie 3 : Analyse du stockage

**Q.2.3.1** Quels sont les systèmes de fichiers actuellement montés ?  
Voici ce que j'ai obtenu avec les commandes `lsblk` & `cat /etc/fstab` :
```bash
sybill@SRVLX01:~$ lsblk
NAME                   MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                      8:0    0     8G  0 disk
└─sda1                   8:1    0     8G  0 part
  └─md0                  9:0    0     8G  0 raid1
    ├─md0p1            259:0    0 488,3M  0 part  /boot
    ├─md0p2            259:1    0     1K  0 part
    └─md0p5            259:2    0   7,5G  0 part
      ├─cp3--vg-root   253:0    0   2,8G  0 lvm   /
      └─cp3--vg-swap_1 253:1    0   976M  0 lvm   [SWAP]
sr0                     11:0    1  1024M  0 rom
sybill@SRVLX01:~$ cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# systemd generates mount units based on this file, see systemd.mount(5).
# Please run 'systemctl daemon-reload' after making changes here.
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mapper/cp3--vg-root /               ext4    errors=remount-ro 0       1
# /boot was on /dev/md0p1 during installation
UUID=9bba6d48-3e4b-42a6-bccc-12836de215ec /boot           ext2    defaults        0       2
/dev/mapper/cp3--vg-swap_1 none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
sybill@SRVLX01:~$
```
Les systèmes de fichier actuellement monté sont :
- LVM
- BOOT
- SWAP

**Q.2.3.2** Quel type de système de stockage ils utilisent ?
Les types de systèmes de stockage qu'ils utilisent sont :
- ext4
- ext2
- swap

**Q.2.3.3** Ajouter un nouveau disque de 8,00 Gio au serveur et réparer le volume RAID  
Avec la commande `mdadm --detail /dev/md0`, nous pouvons voir que le RAID est dégradé :  
![3](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/3-%20RAIDfail.png)

Après ajout du nouveau sur VirtualBox, nous le voyons apparaitre sous `sdb`.  
![4](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/4-sdB.png)

Nous allons le formater pour pouvoir après réparer le RAID. Il faut faire la commande `fdisk /dev/sdb`, puis faire les choix :
- *n* pour créer une nouvelle partition
- *p* pour choisir une partition primaire
- faires *entrée* 3 fois
- *t* pour définir le type de partition
- *fd* pour définir le type de partition à **Linux RAID autodetect**
- *w* pour écrire les modifications et quitter l'utilitaire
  
![5](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/5-sdb1.png)

Maintenant nous allons l'ajouter au RAID avec la commande  `mdadm --manage /dev/md0 --add /dev/sdb1`, ce qui donne :  
![6](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/6-RepairRAID.png)

**Q.2.3.4** Ajouter un nouveau volume logique LVM de 2 Gio qui servira à héberger des sauvegardes. Ce volume doit être monté automatiquement à chaque démarrage dans l'emplacement par défaut : `/var/lib/bareos/storage`.  
Dans un premier temps, il faut créer ce nouveau volume logique avec la commande `lvcreate -L 2G -n backup cp3-vg`.  
![7](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/7-lvBackup.png)

Après il faut le formater en `ext4` avec la commande `mkfs.ext4 /dev/cp3-vg/backup`. Puis créer son point de montage avec la commande `mount /dev/cp3-vg/backup /var/lib/bareos/storage`.  
![8](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/8-mountBackup.png)

Pour le monter automatiquement, il faut son UUID avec la commande `blkid /dev/cp3-vg/backup`, ce qui donne un UUID="f236f150-5b20-4a0c-A625-1902e01c0011". Puis il faut éditez le fichier `/etc/fstab`.  
![9](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/9-fstab.png)

Maintenant, il faut vérifier que le volume logique a bien été ajouter avec les commandes `vgdisplay` & `lsblk`.  
![10](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/10-vgdisplay.png)  
![11](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/11-lsblk.png)

**Q.2.3.5** Combien d'espace disponible reste-t-il dans le groupe de volume ?  
Il restera 1,79 Gio d'espace disponible.

### Partie 4 : Sauvegardes

Le logiciel bareos est installé sur le serveur.   
Les composants `bareos-dir`, `bareos-sd` et `bareos-fd` sont installés avec une configuration par défaut.  

**Q.2.4.1** Expliquer succinctement les rôles respectifs des 3 composants bareos installés sur la VM.  
- `bareos-dir` pour Director, donc le cerveau de l'opération
- `bareos-sd`  pour Storage, donc la gestion du stockage
- `bareos-fd` pour File, donc la gestion d'accès aux fichiers de sauvegarde

### Partie 5 : Filtrage et analyse réseau

**Q.2.5.1** Quelles sont actuellement les règles appliquées sur Netfilter ?  
Avec la commande `nft list ruleset`, nous avons les règles suivantes :
![12](https://github.com/Mirhazka/Checkpoint3/blob/main/Ressources/Exercice%202/12-nftrules.png)
Voici celle que je peux expliquer :
- `tcp dport 22 accept` : toute connexion à destination du port tcp 22 sont acceptés
- `ip protocol icmp accept` : toutes les connexions utilisant le protocole ip/icmp sont acceptés
- `ip6 nexthdr ipv6-icmp accept`: comme ci-dessus mais pour de l'IPv6 alors ci-dessus c'est pour de l'IPv4

**Q.2.5.2** Quels types de communications sont autorisées ?  
Les types de connexion qui sont autorisées sont :
- Par protocole TCP via le port de destination 22
- Les requêtes ICMP IPv4 et IPv6
- La connexion `ct state established, related accept`
- La connexion `iifname "lo" accept`

**Q.2.5.3** Quels types sont interdit ?  
Les types de connexion interdites sont :
- `ct state invalid drop`

**Q.2.5.4** Sur nftables, ajouter les règles nécessaires pour autoriser bareos à communiquer avec les clients bareos potentiellement présents sur l'ensemble des machines du réseau local sur lequel se trouve le serveur.  
> Rappel : Bareos utilise les ports TCP 9101 à 9103 pour la communication entre ses différents composants.

Je ne sais pas.  

### Partie 6 : Analyse de logs

**Q.2.6.1** Lister les 10 derniers échecs de connexion ayant eu lieu sur le serveur en indiquant pour chacun :
- La date et l'heure de la tentative
- L'adresse IP de la machine ayant fait la tentative

Pour lister les tentatives de connexion infructueuses d'un serveur, il faut utiliser la commande `lastb` (à savoir, pour les tentatives de connexion réussi c'est la commande `last`).
```bash
root@SRVLX01:/home/sybill# lastb
wilder   ssh:notty    192.168.1.20     Fri Jan 17 11:09 - 11:09  (00:00)
root     ssh:notty    192.168.1.20     Fri Jan 17 11:01 - 11:01  (00:00)

btmp commence Fri Jan 17 11:01:43 2025
root@SRVLX01:/home/sybill#
```
L'adresse `192.168.1.20` correspond à ma machine locale.
