## Projet Format Dialogue
Fait par *Avenir84 (Alexandre Caldato, Badre Mokhlis, Oksana Leroy)* pour *Avignoun Conseil*.;

### Description du projet
Création d'une application web pour les cours en ligne pour une entreprise de formation pour des adultes à base d'une API de visioconférence existante  avec des interfaces de connexion pour les différents niveaux des intervenants (admin, manager, formateur).

Pour API de visioconférence, le choix de l'équipe s'est porté sur **JITSI Meet** - une API de visioconférence libre de droit (open source) - [home page JITSI Meet](https://meet.jit.si/) installé sur un serveur Ubuntu 18.04.
Il est également possible de l'installer sur un serveur virtuel ([VirtualBox](https://www.virtualbox.org/wiki/Downloads) ou équivalent).
>**Aspect important** : Le serveur doit être connecté en Ethernet et non en wifi local, branché directement sur la box internet : fibre FTTH plus que conseillée.

#### Configuration minimale requise du serveur :
 4 cœurs, 4 Go de RAM et un disque dur de 10 Go (prévoir une capacité suffisante en cas d'enregistrement de sessions).


## Installation de JITSI Meet


### Étape 1 — Définition du nom d'hôte du système

Au cours de cette étape, nous changeons le nom d'hôte du système pour qu'il corresponde au nom de domaine que nous avons l'intention d'utiliser et on résout ce nom d'hôte en le remplaçant par l'IP localhost, `127.0.0.1`. Jitsi Meet utilise ces deux paramètres lorsqu'il installe et génère ses fichiers de configuration.
Si vous ne disposez pas d'un nom de domaine, vous pouvez mettre l'IP du serveur (si elle ne change pas).
La commande suivante permet de définir le nom d'hôte actuel et de modifier le fichier `/etc/hostname` qui conserve le nom d'hôte du système entre deux redémarrages : 

```bash
sudo hostnamectl set-hostname jitsi.your-domain
```


Vérifions que cela a réussi en effectuant les opérations suivantes :

```bash
hostname
```

Le nom d'hôte défini avec la commande `hostnamectl` sera alors renvoyé :

```
Output
jitsi.your-domain
```

Ensuite, on établie une correspondance locale entre le nom d'hôte du serveur et l'adresse IP de rebouclage, `127.0.0.1` en ouvrant le fichier `/etc/hosts` avec un éditeur de texte : 

```bash
sudo nano /etc/hosts
```

Ajoutons la ligne suivante :

```bash
127.0.0.1 jitsi.your-domain
```
Enregistrer et fermer le fichier.

Si l'ordinateur / serveur ou routeur a une adresse IP dynamique (l'adresse IP change constamment), il est possible d'utiliser un service DNS dynamique à la place.

### Étape 2 — Configuration du pare-feu

Normalement, lors de la configuration initiale du serveur avec Ubuntu 18.04] le pare-feu UFW est activé et  le port SSH est ouvert. 

Nous allons ouvrir les ports suivants :

-   `80/tcp` utilisé dans la demande de certificat TLS.
-   `443/tcp` utilisé pour la page web de création de la salle de conférence.
-   `4443/tcp,10000/udp` utilisé pour transmettre et recevoir le trafic d'appels cryptés. 

Exécuter les commandes `ufw` suivantes pour ouvrir ces ports : 

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 4443/tcp
sudo ufw allow 10000/udp

```

Vérifier qu'ils ont tous été ajoutés avec la commande `ufw status` :
```
sudo ufw status
```
On verra le résultat suivant si ces ports sont ouverts :
```bash
Output
Status: active

To                              Action      From
--                              ------      ----
OpenSSH                         ALLOW       Anywhere
80/tcp                          ALLOW       Anywhere
443/tcp                         ALLOW       Anywhere
4443/tcp                        ALLOW       Anywhere
10000/udp                       ALLOW       Anywhere
OpenSSH (v6)                    ALLOW       Anywhere
80/tcp (v6)                     ALLOW       Anywhere
443/tcp (v6)                    ALLOW       Anywhere
4443/tcp (v6)                   ALLOW       Anywhere
10000/udp (v6)                  ALLOW       Anywhere

```

### Étape 3 — Installation de Jitsi Meet

Dans cette étape, nous allons ajouter le dépôt stable Jitsi à votre serveur et ensuite installer le paquet Jitsi Meet à partir de ce dépôt. Nous serons ainsi assuré de toujours utiliser le dernier paquet stable de Jitsi Meet.

Télécharger la clé Jitsi GPG avec l'utilitaire de téléchargement `wget` :

```bash
wget https://download.jitsi.org/jitsi-key.gpg.key
```

ajoutez la clé GPG téléchargée au porte-clés d’`apt` à l'aide de l'utilitaire `apt-key` :

```bash
rm jitsi-key.gpg.key
```

Supprimer le fichier de la clé GPG car il n'est plus nécessaire :

```bash
rm jitsi-key.gpg.key
```

Maintenant, nous allons ajouter le dépôt Jitsi à notre serveur en créant un nouveau fichier source qui contient le dépôt Jitsi.

```bash
sudo nano /etc/apt/sources.list.d/jitsi-stable.list
```

Ajouter cette ligne au fichier pour le dépôt Jitsi :

```bash
deb https://download.jitsi.org stable/
```

Sauvegarder et quitter l'éditeur.

Effectuer une mise à jour du système pour recueillir la liste des paquets à partir du dépôt Jitsi et installer le paquet `jitsi-meet` : 

```bash
sudo apt update
sudo apt install jitsi-meet
```
Saisir le nom de domaine (par exemple, `jitsi.your-domain`) 

![Image montrant le dialogue de nom d'hôte de l'installation de jitsi-meet ](https://assets.digitalocean.com/articles/jitsimeet1804/step3a.png)

**Note :** Déplacer le curseur à partir du champ du nom d'hôte pour mettre en évidence le **<OK>** avec la touche `TAB`. Appuyer sur `ENTER` lorsque **<OK>** est mis en évidence pour soumettre le nom d'hôte.  

Si vous n'avez pas de certificat TLS pour votre domaine Jitsi, sélectionnez la première option, **Générer un nouveau certificat auto-signé**.

![Image montrant la boîte de dialogue du certificat d'installation de jitsi-meet ](https://assets.digitalocean.com/articles/jitsimeet1804/step3b.png)
### Étape 4 — Obtention d'un certificat TLS signé

Jitsi Meet fournit un programme permettant de télécharger automatiquement un certificat TLS pour le nom de domaine qui utilise l'utilitaire [Certbot](https://certbot.eff.org/). On va installer ce programme avant d'exécuter le script d'installation du certificat.

Ajoutons le dépôt Certbot à notre système pour nous assurer que nous avons la dernière version de Certbot. Exécuter la commande suivante pour ajouter le nouveau référentiel et mettre à jour le système :

```bash
sudo add-apt-repository ppa:certbot/certbot
```
Installer le paquet `certbot` :

```bash
sudo apt install certbot

```
Exécuter le programme d'installation du certificat TLS fourni par Jitsi Meet :

```bash
sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```
Fermer le port 80 dans le pare-feu avec la commande `ufw` suivante :

```bash
sudo ufw delete allow 80/tcp
```

Le serveur Jitsi Meet est maintenant opérationnel et disponible pour des tests. Ouvrir un navigateur et pointer vers le nom de domaine choisi. Il est possible maintenant de créer une nouvelle salle de conférence et inviter d'autres personnes à la rejoindre.

## Quelques réglages

### Configuration du serveur
Afin d’éviter que n'importe qui puisse créer les nouvelles salles de conférence, on va configurer notre serveur de façon suivante :
```
$ sudo nano /etc/prosody/conf.avail/your.domain.com.cfg.lua
```
Et remplacer la ligne d’authentification comme suit :

> ... authentication = "internal_plain" ...

Et on fini par ajouter les lignes suivantes tout à la fin :

> .... VirtualHost "guest._your.domain.com_" authentication = "anonymous" c2s_require_encryption = false

Enregistrer et fermer le fichier pour appliquer les modifications.
Ouvrir le fichier suivant :
```
$ sudo nano /etc/jitsi/meet/your.domain.com-config.js
```
Dans ce fichier trouver ces lignes :

> ... // anonymousdomain: 'guest.example.com', ...

**Décommenter** et changer comme ceci :

> ... anonymousdomain: 'guest._your.domain.com_', …

Enregistrer et fermer le fichier pour appliquer les modifications.

Passons au fichier *properties* :
```
$ sudo nano /etc/jitsi/jicofo/sip-communicator.properties
```
Et ajoutons cette ligne : 

`org.jitsi.jicofo.auth.URL=XMPP:your.domain.com`

### Gestion de la base de données Prosody
Vérifier les utilisateurs enregistré dans la base de données :
```
$ sudo prosodyctl mod_listusers
```
Ajouter un utilisateur :
```
$ sudo prosodyctl register <username> <hostname> <password>
```
Supprimer un utilisateurs de la base de données :
```
$ sudo prosodyctl deluser <user@hostname>
```
Changer le mot de passe d'un utilisateur :
```
$ sudo prosodyctl passwd <username>
```
Vérifier le statut de l'utilisateur :
```
sudo prosodyctl status
```
À la fin de la configuration il est nécessaire de redémarrer tous les services :
```
$ sudo systemctl restart prosody.service    
$ sudo systemctl restart jicofo.service
$ sudo systemctl restart jitsi-videobridge2.service
```
Si vous avez installer Prosody à partir d'un package :
```
$ sudo /etc/init.d/prosody restart
```
### Customisation de  JITSI

Redirection de la page d'accueil :
```
$ sudo nano /etc/jitsi/jicofo/sip-communicator.properties
```
Éditer les lignes suivantes :

>org.jitsi.jicofo.BRIDGE_MUC=JvbBrewery@internal.auth.(your_adress_domain)
org.jitsi.jicofo.auth.URL=XMPP:your.domain.com



*Sources :* [Framasoft](https://framacloud.org/fr/cultiver-son-jardin/jitsi-meet.html) , [jitsi.github.io](https://jitsi.github.io/handbook/docs/devops-guide/devops-guide-quickstart) , [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-jitsi-meet-on-ubuntu-18-04-fr), [Prosody](https://prosody.im/doc/prosodyctl) ; [https://github.com/jitsi/jitsi-meet](https://github.com/jitsi/jitsi-meet) ; [oldfag.ru](https://www.oldfag.ru/2020/05/jitsi-meet-with-active-directory-authentication-and-guest-access.html) ;









> Written with [StackEdit](https://stackedit.io/).
