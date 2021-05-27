# How to install JITSI MEET

# Server configuration

Connect to the server using SSH.
Before installing JITSI, the server configuration is mandatory.
## Server requirements

Server: should work on 1 core, 1GB RAM - suggested (at least) 2 core, 2GB RAM (scale at your needs)

-   Ubuntu 18.04 LTS
    
-   Host name: `your.domain.com` (change it to the actual domain you are using)
    
-   create DNS A records for `your.domain.com`

## Hostname
Check your current hostname
```bash
$ hostname
>>> currentHostname
```
Change the host name
```bash
$ sudo hostnamectl set-hostname your.domain.com
```
Update hosts file
Read the nano documentation to use 'nano'.

https://www.nano-editor.org/dist/v2.1/nano.html
```bash
$ sudo nano /etc/hosts
```
Add the following line

`127.0.0.1 your.domain.com`

## Firewall

Several ports have to be opened.
```bash
$ sudo ufw allow 80/tcp
$ sudo ufw allow 443/tcp
$ sudo ufw allow 4443/tcp
$ sudo ufw allow 10000/udp
```
Check the status with this command:
```bash
$ sudo ufw status
```

# Installation of JITSI

## Gnu Privacy Guard (GPG)

```bash
$ wget https://download.jitsi.org/jitsi-key.gpg.key
$ sudo apt-key add jitsi-key.gpg.key
```
Then remove the private key.
```bash
$ rm jitsi-key.gpg.key
```
## Add JITSI repository
```bash
$ sudo nano /etc/apt/sources.list.d/jitsi-stable.list
```
Add this line.
`deb https://download.jitsi.org stable/`

Update the repo file
```bash
$ sudo apt update
```
**Installation of JITSI**
```bash
$ sudo apt install jitsi-meet
```
> Type your domain name

> Choose auto-signed key
> 
## Certbot – Transport Layer Security (TSL)

  ```bash
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt install certbot
$ sudo /usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```
Once the TSL is set you can close the HTTP port since it’s no longer useful
```bash
$ sudo ufw delete allow 80/tcp
```
# Server configuration
Change the default config to prevent guests from creating new rooms
```bash
$ sudo nano /etc/prosody/conf.avail/your.domain.com.cfg.lua
```
Replace the authentication line with
>...
>authentication = "internal_plain"
>...
>
And add the following lines at the end
> ....
>VirtualHost "guest._your.domain.com_"
>authentication = "anonymous"
>c2s_require_encryption = false

Save and close this file to modify another config file
```bash
$ sudo nano /etc/jitsi/meet/your.domain.com-config.js
```
  Find these lines:
>...
>// anonymousdomain: 'guest.example.com',
>...

**Uncomment** and change like

>...
>anonymousdomain: 'guest._your.domain.com_',
>…

Save and close this file to modify the last config file
```bash
$ sudo nano /etc/jitsi/jicofo/sip-communicator.properties
```
And add this line
`org.jitsi.jicofo.auth.URL=XMPP:your.domain.com`

Check BDD Users
```bash
$ sudo prosodyctl mod_listusers
```
ADD User
```bash
$ sudo prosodyctl register <username> <hostname> <password>
```
DELET User
```bash
$ sudo prosodyctl deluser <user@hostname>
```
Change password
```bash
$ sudo prosodyctl passwd <username>
```
Check the status of user accounts :
```bash
sudo prosodyctl status
```
Everything is set up, you can restart all services
```bash
$ sudo systemctl restart prosody.service    
$ sudo systemctl restart jicofo.service
$ sudo systemctl restart jitsi-videobridge2.service
```
If you installed Prosody from a package :
```bash
$ sudo /etc/init.d/prosody restart
```

# Custom JITSI

Redirect home page
```bash
$ sudo nano /etc/jitsi/jicofo/sip-communicator.properties
```
edit this lines
```bash
org.jitsi.jicofo.BRIDGE_MUC=JvbBrewery@internal.auth.(your_adress_field)
org.jitsi.jicofo.auth.URL=XMPP:visio.triplea.fr
```

Sources : https://prosody.im/doc/prosodyctl ;
          https://github.com/jitsi/jitsi-meet ;
          https://www.oldfag.ru/2020/05/jitsi-meet-with-active-directory-authentication-and-guest-access.html ;
