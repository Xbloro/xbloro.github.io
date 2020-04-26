---
title: "Tool : DFIR"

categories:
  - toool
  - DFIR
tags:
  - cuckoo
  - tool
---

# Mise en place d'un serveur Cuckoo 
---
---

## Sources 

  * https://cuckoo.readthedocs.io
  * https://infosecspeakeasy.org/t/howto-build-a-cuckoo-sandbox/27
  * https://gist.github.com/braimee/bf570a62f53f71bad1906c6e072ce993
  * https://hatching.io/blog/cuckoo-sandbox-setup



## Disclamer

Cuckoo est encore en cours de développement et comporte donc des bugs.
De plus il est écrit en python2 ce qui provoque des conflits avec les dépendences, non suportées etc.

Il arrive que sur deux installations identiques des problèmes différents apparaisent.
Il est donc possible que vous ayez à résoudre des problèmes même en suivant ce guide.

Correction des fautes d'orthographe : **En COURS**

---
---

## Préparation de l'hôte

On va partir sur un [serveur ubuntu lts](https://ubuntu.com/download/server).

On met à jour : `sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y`


## Installation des dépendences 

### Les paquets

On aura besoin des dépendences suivantes : 

```console
sudo apt  install -y git mongodb python python-dev python-pip libmagic1 swig libvirt-dev upx-ucl libffi-dev libssl-dev liblzma-dev libjpeg-dev zlib1g-dev liblcms2-dev libfreetype6-dev wget unzip p7zip-full geoip-database libgeoip-dev mono-utils ssdeep libfuzzy-dev exiftool clamav clamav-daemon clamav-freshclam wkhtmltopdf xvfb xfonts-100dpi  curl libguac-client-rdp0 libguac-client-vnc0 libguac-client-ssh0 guacd libcap2-bin tcpdump apparmor-utils
qemu-kvm libvirt-clients libvirt-daemon virt-manager vsftpd nginx apache2-utils uwsgi uwsgi-plugin-python 
```
On installera les dépendences python au fur et à mesure.

### YARA

Yara permet d'indentifier les malwares selon des règles prédéfinies.

On intalle : 

```console
sudo apt-get install git automake libtool make gcc flex bison libssl-dev libjansson-dev libmagic-dev checkinstall python-pip python3-pip 
git clone https://github.com/VirusTotal/yara
cd yara
./bootstrap.sh
./configure --enable-cuckoo --enable-magic --enable-dotnet
make
sudo checkinstall -y --deldoc=yes --pkgversion=3.8.1+git1
cd ..
rm -rf yara
sudo -H pip install -U git+https://github.com/VirusTotal/yara-python
sudo -H pip3 install -U git+https://github.com/VirusTotal/yara-python
```

### PostgresSQL
PostgresSQL est plus performant que SQLiteDB, on va donc l'utiliser.

On installe : 

```console
sudo apt-get install postgresql postgresql-contrib libpq-dev
```

On configure : 

```console
sudo su postgres
psql
CREATE USER cuckoo WITH PASSWORD 'somePassword';
CREATE DATABASE cuckoo;
GRANT ALL PRIVILEGES ON DATABASE cuckoo to cuckoo;
```


### TCPDUMP 

Par défaut tcpdump nécessite les droits root pour tourner. Cependant Cuckoo ne tournera pas en root, il faut donc le configurer :

`sudo aa-disable /usr/sbin/tcpdump`
`sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump`


### ClamaV
Clamav est un "anti-virus"

On installe : 

```console
git clone https://github.com/rieck/malheur.git
cd malheur
./bootstrap
./configure --prefix=/usr
make
sudo checkinstal
```


### Suricata

Suricata est un IDS.

On installe : 

```console
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata
sudo apt install python-pip python-yaml
sudo pip install --pre --upgrade suricata-update
```

On va créer notre configuration de suricata pour cuckoo

`sudo cp /etc/suricata/suricata.yaml /etc/suricata/suricata-cuckoo.yaml`

On édite la configuration :
 
* désactivez `fast` and `unified2` log types
* activez `file-store`
* activez `force-md5`, `force-filestore` et `file-log`
* dans `reassembly`: mettez `depth`, `request-body-limit` et `response-body-limit` à 0.
* Mettez `EXTERNAL_NET` à any.

### Création d'un utilisateur

On créé un utilisateur nommé cuckoo:

`sudo usermodd -a -G kvm cuckoo`
`sudo usermodd -a -G libvirt-cuckoo`

On le met dans le groupe pcap : 

```console 
sudo groupadd pcap
sudo usermod -a -G pcap cuckoo
sudo chgrp pcap /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

---
---

## Partie Virtualisation 

On va utiliser kvm avec libvirt.

Notre server ne comporte pas d'interface graphique et kvm est compliqué à utiliser en cmd.

On va donc utiliser l'interface de notre ordinateur via ssh pour configurer kvm.

`ssh -X -L 5900:127.0.0.1:5900 user@hostname`

On lance ensuite libvirt
`virt-manager`

Sur l'interface cliquer droit > détails.
Dans Réseau Virtuel, créez un nouveau réseau virtuel isolé. La range IP sur `192.168.100.0/24`.


### VSFTPD

Afin que la VM et l'hôte puissent transférer des fichiers de manière anonyme et sécurisé, sans utiliser de dossier partager, on va monter un FTP.

On créé un dossier public et on sécurise tout ça : 

```console
sudo mkdir -p /home/cuckoo/vmshared/pub
sudo chown -R cuckoo:cuckoo /home/cuckoo
sudo chmod -R ug=rwX,o=rX /home/cuckoo/vmshared/
sudo chmod -R ugo=rwX /home/cuckoo/vmshared/pub
```

on édite la conf (`/etc/vsftp.conf`) :

* mettez `listen` à `yes`
* `listen_ipv6` à `no`
* `anonymous_enable` à `yes`

Ensuite décommentez les lignes:
 
```console
write_enable=YES
anon_upload_enable=YES
anon_mkdir_write_enable=YES
```

Ajoutez à la fin du fichier :

```console
listen_address=192.168.100.1
listen_port=2121
anon_root=/home/cuckoo/vmshared
anon_umask=000
chown_upload_mode=0666
pasv_enable=Yes
pasv_min_port=10090
pasv_max_port=10100
```

Redémarrez le service (`sudo service vsftpd restart`)

Les VM devraient pouvoir:

* lire dans le dossier `/home/cuckoo/vmshared/`
* écrire dans le dossier `/home/cuckoo/vmshared/pub/`

Le ftp est joignable sur `ftp://192.168.100.1:2121`

### Inetsim

Certains malwares communiquent avec internet, il est donc intéressant de simuler un faux internet afin de récupérer des trames réseaux sans divulguer des informations à "l'attaquant".

On installe :
 
```console
echo "deb http://www.inetsim.org/debian/ binary/" | sudo tee /etc/apt/sources.list.d/inetsim.list
wget -O - http://www.inetsim.org/inetsim-archive-signing-key.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install inetsim
```

on édite la conf (`/etc/inetsim/inetsim.conf`) :
* `enable` à `1`
* mettez `service_bind_adress` et `dns_default_ip` à `192.168.100.1`


on edite ce truc la (`/etc/default/inetsim`) :
* mettez `enable` à `1`

On peut redémarrer le service : 
`sudo service inetsim restart`

### Mise en place des machines virtuelles : 

On va filouter un peu pendant l'installation afin de rendre la machine plus rapide.

Vous aurez besoin de logiciel et de driver : 

* [les drivers virtio](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/latest-virtio/virtio-win.iso)
* [Un package bien pratique pour le guest](https://www.dropbox.com/s/ug2l9qzwj46zk6l/guest_package.zip)
* [Pillow](https://files.pythonhosted.org/packages/1d/bd/a2dbba6829429f456ae2d0451aa92c45ee058e92a71a0ba4a9cacdaf74ee/Pillow-5.3.0.win32-py2.7.exe)
* [Python2.7](https://www.python.org/ftp/python/2.7.15/python-2.7.15.msi)

Le package comprend tous les logiciels exploitables par les malwares et quelques modifications de registres afin de réduire les trames inutiles sur le réseau.

Ouvrez `virt-manager` et créez la machine virtuelle.
Nommez la avec un nom explicite, ex : `win7_sandbox`.

Associez le minimun de ressource à la machine (2cpu et 2gb de ram).

Avant de passer à l'installation cochez bien l'option "configurer la machine virtuelle".

Dans les périphériques : 

* changez l'adaptateur réseau NIC sur `virtio` et placez le sur le réseau virtuel créé précédement.
* ajoutez un disque, mettez le bus type sur : `virtio` et le mode de cache sur `writeback`

On peut passer à l'installation de la VM.
Pendant l'installation une fenêtre avec label `Help protect your computer and improve windows automaticaly`, cliquez sur `Ask me later`.

Créez un utilisateur réaliste comme `Bernadette`.

Inserez dans la vm le disque de driver virtio téléchargé plus haut.
Rendez vous dans le gestionnaire de périphériques.
Pour chaque `Other device`, choisissez d'installer les drivers manuellement et donnez le chemin du CD.

Eteingez la VM et retirez le lecteur CD.
Supprimez également le Disk virtio.

Modifiez ensuite le disque principal:

* mettez le bus type sur : `virtio` et le mode de cache sur `writeback`

Redémarrez la VM.

Configurez le réseau : 
IPV4 :
 
* `Adresse IP` sur `192.168.100.101`
* `masque de sous réseau` sur `255.255.255.0`
* `Default gateway` sur `192.168.100.1`
* `Primary` et `secondary DNS` sur `1.1.1.1` et `1.0.0.1`

Connectez-vous au ftp et téléchargez le package ainsi que les exécutables python.

* Installez les outils correspondant à votre version de Windows.
* Exécutez les fichiers reg.
* Installez python et pillow.
* Désactivez windows defender, le firewall, les mises à jour automatique ainsi que les ACL.

Eteignez la VM.

---
---

## Installation de Cuckoo

### Préparation de l'environnement Virtuel


On va lancer cuckoo dans un environement virtuel car c'est plus pratique.

```console
sudo su cuckoo
cd
virtualenv venv
. venv/bin/activate
pip install -U pip==9.0.2 setuptools
pip install upgrade pip
pip install pyvmomi
pip install libvirt-python
pip install weasyprint=0.36
pip install m2crypto==0.24.0
pip install psycopg2 #peut poser une erreur en fonction de la version
pip install -U backports.lzma
pip install -U git+https://github.com/VirusTotal/yara-python
pip install -U git+https://github.com/kbandla/pydeep.git
pip install -U pillow distorm3 pycrypto openpyxl ujson
pip install -U git+https://github.com/volatilityfoundation/volatility.git
pip install -U cuckoo
```

On va lancer cuckoo, il devrait créer le `CWD` : `cuckoo`

On copie notre agent python vers le ftp : `cp /home/cuckoo/.cuckoo/agent/agent.py /home/cuckoo/vmshared/agent.pyw`


### Configuration de Cuckoo

On édite `$CWD/conf/cuckoo.conf` :

* `machinery` sur `kvm`
* `IP` sur `192.168.100.1`
* `Postgresql` sur `connection = postgresql://cuckoo:password@localhost:5432/cuckoo`

Bien sûr `password` correspond au mot de passe que vous avez mis lors de la configuration de postgresql.


On édite `$CWD/conf/processing.conf` : 
  
* `Suricata` sur `Yes`
* changez le path de la configuration de suricata sur `/etc/suricata/conf/suricata-cuckoo.conf`


On édite `$CWD/conf/reporting.conf` :

Activez :
* `html`
* `pdf`
* `mongodb`
* `malhur`


On édite `$CWD/conf/kvm.conf` :

Dans la machine `cuckoo1` :

* `label` sur le nom de votre VM dans KVM (win7_Sandbox)
* `resultserver_ip` sur `192.168.100.1`
* `Tag` sur des tags qui vous permettent d'identifier la machine virtuel et ses services ex : win7, java6...
* `Snapshot` sur le nom du snapchot de la vm que vous allez prendre (je l'ai appelé "ready")
* `interface` sur l'interface de la machine virtuel sur le résea privé (virbr1 pour moi)
* `ip` sur l'ip de la vm `192.168.100.101`
* `osprofile` sur le profile volatily de l'os de la VM (RTFM)



#### Fin de configuration de la VM

On fini de configurer la VM : 

* Redémarrez la VM.
* Ouvrez msconfig et désactivez toutes les applications qui se lancent au démarrage.
* Dans l'onglet service cachez les services Microsofts et désactivez les mises à jour des logiciels tiers.
* Téléchargez l'agent.pyw, sur le FTP renommez le avec un nom plus légitime et placez le dans le dossier de démarrage automatique.
* Redémarrez la VM et regadez si l'agent.py tourne (processus pythonw.exe lancé)


On va obfusquer la VM aux malwares : 

`sudo virsh edit win7_sandbox`

Changez `<domain type='kvm'>` par `<domain type='kvm' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>`

Dans le cpu mettez la valeur de `check` sur `partial`

Dans la partie domaine ajoutez : 

```
<<qemu:commandline>
<qemu:arg value='-cpu'/>
<qemu:arg value='host,hv_time,kvm=off,hv_vendor_id=null,-hypervisor'/>
</qemu:commandline>
```
Modifiez ou ajoutez : 

```
 <kvm>
    <hidden state='on'/>
  </kvm>
```

Ajoutez juste au dessus de `</feature>`

```  
<cpu mode='host-model' check='partial'>
    <model fallback='allow'/>
    <feature policy='disable' name='hypervisor'/>
 </cpu>
```

Prennez un snapchot de la VM et nommez le comme vous l'avez configurer dans `kvm.conf` (j'avais mis "ready")

Vous pouvez faire des backups de la vm en cas de problèmes.

---
---

## Installation des services WEB

On va utiliser un Nginx comme reverse proxy afin de sécuriser l'accés à l'interface WEB.

On s'ajoute au groupe cuckoo : `sudo usermod -a -G cuckoo $USER`

#### SSL

On créé un dossier ssl et on lance la génération des certificats:

```console
mkdir ~/ssl
cd ~/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:4096 -keyout cuckoo.key -out cuckoo.crt
rm cuckoo.csr
openssl dhparam -out dhparam.pem 4096
```
On sécurise les clées : 

```console
cd
sudo mv ssl /etc/nginx
sudo chown -R root:www-data /etc/nginx/ssl
sudo chmod -R u=rX,g=rX,o= /etc/nginx/ssl
```

On met en place une authentification par mot de passe : 

```console
sudo htpasswd -c /etc/nginx/htpasswd exampleuser
sudo chown root:www-data /etc/nginx/htpasswd
sudo chmod u=rw,g=r,o= /etc/nginx/htpasswd
```

#### UWSGI et NGINX

On supprime la configuration par défault et on créé la notre : 

`sudo rm /etc/nginx/sites-enabled/default`

Cuckoo permet de lancer le serveur web comme une application, c'est ce que nous allons faire.

```console
sudo su cuckoo
cd
. venv/bin/activate
pip install uwsgi
cuckoo web --uwsgi > cuckoo-web.ini
sudo cp cuckoo-web.ini /etc/uwsgi/apps-available/cuckoo-web.ini
sudo ln -s /etc/uwsgi/apps-available/cuckoo-web.ini /etc/uwsgi/apps-enabled/cuckoo-web.ini*
sudo adduser www-data cuckoo
sudo systemctl restart uwsgi
```


On configure ensuite Nginx

```console
cuckoo web --nginx > cuckoo-web.conf
sudo cp cuckoo-web.conf /etc/nginx/sites-available/cuckoo-web.conf
sudo ln -s /etc/nginx/sites-available/cuckoo-web.conf /etc/nginx/sites-enabled/cuckoo-web.conf
sudo systemctl restart nginx
```

On va ensuite modifier la configuration afin d'appliquer les précaution sde sécurité et rendre l'interface web disponnible sur le réseau.


```sh
upstream _uwsgi_cuckoo_web { 
server unix:/run/uwsgi/app/cuckoo-web/socket;
} 

server { 
    listen IP_Local:443 ssl http2;
    sl_certificate /etc/nginx/ssl/cuckoo.crt;
    ssl_certificate_key /etc/nginx/ssl/cuckoo.key; 
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;
    ssl_protocols TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';                                                                        
    ssl_prefer_server_ciphers on;                                                                                                                             
    add_header X-Frame-Options SAMEORIGIN;                                                                                  
    add_header X-Content-Type-Options nosniff;                                                                              
    root /usr/share/nginx/html;                                                                                             
    index index.html index.htm;                                                                                             
    client_max_body_size 101M;                                                                                              
    auth_basic "Login required";                                                                                            
    auth_basic_user_file /etc/nginx/htpasswd;   

    location / {                                                                                                                
        proxy_pass http://127.0.0.1:8000; 
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        client_max_body_size 1G; 
        proxy_redirect off;                                                                                                     
        proxy_set_header X-Forwarded-Proto $scheme;                                                                             
        uwsgi_pass  _uwsgi_cuckoo_web;
        include uwsgi_params;    
    }                                                                                                          
}                
```

On relance Nginx `sudo service nginx restart`

Ensuite on peut lancer Cuckoo dans le virtual environement :

```console
cuckoo community
cuckoo
```

### FIN
---
---

Félicitation vous pouvez utiliser Cuckoo !
