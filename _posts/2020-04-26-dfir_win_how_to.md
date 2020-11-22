---

title: "Windows DFIR methodo "

categories:
  - DFIR
tags:
  - DFIR
  - Windows
---

# Ce document est en cours de révisions 

j'essaie de corriger les fautes d'orthographe au max.


# Piste de recherche en Live/DEAD forensique sous Windows

Ce document détaille quelques procédures et éléments intéressants lors d'une investigation live/ dead forensique Windows.
N'hésitez pas à me donner votre avis ou me dire si j'ai fait des erreurs. 
Si vous avez des questions n'hésitez pas sur [twitter](https://tw.itter.com/HHUG0 "twitter").

# Disclamer

Le but de ce document est de présenter une méthodologie ainsi que certains aspects de Windows qui pourront être utiles lors d'une investigation.
Ce document n'est pas exhaustif, en fonction de votre entreprise, expérience, cas traité et évolution des technologies, les procédures peuvent être complètements différentes. 

# Introduction

Quand un attaquant pénètre dans un système, il laisse toujours des traces de son passage. Dans cet article nous allons étudier les mécanismes de Windows afin de trouver des "preuves". Par la suite nous allons essayer de construire une timeline du déroulement de l'attaque.
Nous verrons en première partie quelques principes d'acquisition des médias puis nous passerons à l'investigation.



## Les types d'enquêtes

Il existe deux types d'enquêtes, l'enquête dite administrative et l'enquête judiciaire. Seul un officier de police judiciaire est habilité à faire une enquête du même nom. On se concentrera donc sur l'enquête administrative.

La principale différence entre les deux est la préservation obligatoire de l'intégrité des preuves pour qu'elle soit juridiquement recevables.

# Préparation pour l'investigation en DEAD

## 1  Acquisition des disques  
-

Prévoyez une bonne quantité de stockage car on va récupérer tout le contenu des disques présent dans la machine cible.  

Pour cela, le mieux (et dans le cadre d'une enquête judiciaire c'est obligatoire) est utiliser un tableau bloqueur-copieur.  
C'est un équipement qui ressemble à ca :  

<img src="/assets/images/td2u.jpg" width="400">

Il permet de bloquer le flux en écriture vers la machine cible et d'en faire une image disque vers votre matériel.

Avec l'accord  de la SSI on éteint la machine et on prend les disques (juste le temps de les copier hein après il faut les rendre !).

Sur le tableau il faut choisir un format  comme l'EWF (E0) ou iso.
On branche notre média amovible, on branche le/les disque(s) sources et on s'arme de patience, car c'est long...  très long.  
N'oubliez pas de faire la vérification des données copiées, le tableau propose l'option en général (md5, sha1...).

Si jamais vous n'avez pas de tableau ou même un simple bloqueur. Vous pouvez utiliser une distribution qui va bloquer en écriture ses ports USB etc. (coucou [tsurugi linux](https://tsurugi-linux.org/ "tsurugi linux") ).  
Des outils comme [FTK imager](https://accessdata.com/product-download/ftk-imager-version-4-2-0 "ftk imager") ou [EWF Toolkit](https://github.com/libyal/libewf "ewf toolkit") permettent de faire une copie EWF.  


Quelques informations à récupérer qui vous seront très utiles par la suite : 
- Un schéma réseau et de l'architecture de l'entreprise (avec les IP etc..);
- La politique de sécurité SSI de l'entreprise;
- Les autorisations de la session du poste incriminé;
- Si possible les horaires de travail de l'utilisateur;
- Le domaine d'activité de l'utilisateur (il fait de la modélisation? de la comptabilité?);
- La liste des accès réseau et aux applications de l'utilisateur.


Avant toute chose il faut faire des copies de toutes les données que nous avons récupéré précédemment.  
En effet si vous travaillez directement sur les données et que vous faites une bêtise, il faudra à nouveau faire l'acquisition dans l'entreprise. De même si le support de stockage plante et que vous perdez tout ...

On va donc faire 2 copies, on placera l'original sur un support dédié. La première copie sur un autre support dédié, c'est la copie de secours. Enfin on pourra utiliser la 2ème comme model de copie de travail.

On va donc faire une copie de travail (et oui encore) de notre model et pouvoir commencer à investiguer dessus.
Comme ca si on fait une bêtise on a juste à refaire une copie du model pour repartir.
De même on a deux exemplaires des données stockées bien au chaud en cas de problèmes.  

Pour résumer : 
-Données originales stockées sur un disque;

- 1ere Copie des données originales stockées sur un autre disque >> copie de secours;
- 2eme Copie des données originales stockées sur un troisième disque >> model de copie de travail;
  - Copie de travail des données depuis la copie n2 >> la copie sur laquelle on va investiguer.

Ca en fait hein ? Mais bon on est jamais trop prudent et croyez moi une erreur arrive vite.



## 2 Préparation d'un environnement de travail 
-

Au cours de cet article nous utiliserons Windows mais libre à vous de choisir votre environnement préféré.

Attention cependant vous allez être amené à travailler sur un environnement possiblement infecté et il faut prendre les précautions adaptées.

Nous allons donc partir sur une machine virtuelle Windows 10 un peu pimpée grâce à [FlareVM](https://www.fireeye.com/blog/threat-research/2017/07/flare-vm-the-windows-malware.html "FlareVM")  
On oublie pas de bien cloisonner la machine pour éviter toute fuite par le réseau et de faire des snapshots.

Dans le cadre de l'investigation nous allons utiliser [Autopsy](https://www.sleuthkit.org/autopsy/ "Autopsy") car c'est un outils très complet, reconnu et distribué gratuitement.  
Si vous avez beaucoup d'argent vous pouvez en choisir un autre comme Encase ou Magnet.

On oublie pas de prendre [Regripper](https://github.com/keydet89/RegRipper2.8 "regripper").



## 3 Petit point Autopsy 
-

On lance Autopsy.

Dans un premier temps il va vous demander les informations sur l'affaire, essayez de les remplir correctement, surtout si vous travaillez à plusieurs.

On arrive ensuite au Menu ou on vous demande quel est le type de support que vous voulez importer, choisissez celui qui correspond.

![alt text](/assets/images/dfirMethodo/Autmenu.png?raw=true "Menu")    


Si vous avez fait une copie EWF choisissez "Disk image" et sélectionnez le fichier n1 (E01), les autres seront importés automatiquement par Autopsy.  

Il va vous demander ensuite ce que vous voulez récupérer sur l'image disque. On sélectionne tout.  

![alt text](/assets/images/dfirMethodo/AutCheck.png?raw=true "Check")  

Allez maintenant on peut allé se chercher un thé car c'est long.

![alt text](/assets/images/dfirMethodo/AutLoad.png?raw=true "load")  

En attendant que le traitement soit fini, on va se familiariser avec l'interface.

Sur la partie gauche vous avez une arborescence. En haut on a la première partie qui nous permet de voir le contenu du disque de la même façon que quand il est  monté sur la machine hôte. C'est à partir de cet emplacement que vous allez pouvoir naviguer dans l'arborescence de la machine pour récupérer les logs ainsi que le trousseau de clés par exemple.


![alt text](/assets/images/dfirMethodo/AutSource.png?raw=true "Source")  

Pour exporter un fichier rien de plus simple, il suffit de cliquer droit et exporter, choisissez ensuite la destination sur votre machine.

![alt text](/assets/images/dfirMethodo/AutExport.png?raw=true "Export")

Attention cependant, si vous exportez un fichier malveillant, dropez le dans un fichier inaccessible par votre antivirus car il risquerait de le supprimer. je vous conseille de le zipper(avec mdp) pour éviter de cliquer dessus sans faire exprès. En général on change l'extension du fichier en le renommant.

Ensuite arrive la partie "View" qui trie les fichiers par type etc. 

C'est aussi la que la partie "CARVING" est accessible afin de rechercher les fichiers supprimés.


![alt text](/assets/images/dfirMethodo/AutView.png?raw=true "view")

Enfin on a la partie "Results" qui permet de voir des éléments intéressants, je vous laisse regarder.
La partie "Exif" permet accéder rapidement aux photos et à leurs emplacements. 

Par exemple, si vous trouvez une image pornographique (ca arrive...), vous pouvez facilement trouver son origine (powerpoint > macro malveillante etc.).

![alt text](/assets/images/dfirMethodo/AutRes.png?raw=true "res")



Maintenant , vous savez comment rechercher et récupérer des fichiers.

# Investigation LIVE or DEAD
-

Que ce soit en live or DEAD les taches suivantes sont clés.

Voici les étapes que nous allons effectuer :  

- Auditer les "autoruns" et rechercher des persistances;

- Récuperer la jumplist;

- Récupération des fichiers supprimés; 

- Récupération du "trousseau de clé" (les clés de registre); 

- Parser l'amcache;

- Parser le Shimcache;

- Parser le Ntuser.dat;

- Regarder les "last activity view";

- Parser l'usnjournal;

- Rechercher les supports amovibles;

- Rechercher des fichiers suspect (fichier lnk); 

- récupération des logs;

- récupération des shadows copie;

- Faire une fls;

- Parser la mft;

  


## 1 Autoruns et persistances : 

En cas de compromission d'une machine, il est important de rechercher une backdoor car elle permet de récupérer des informations sur l'attaquant mais aussi de lui couper ses accès.

Les persistances sont majoritairement créé grâce à un(e) :

- entrée dans la clé run

- tâche planifiée

- service

- requête wmi

- injection DLL

- driver malveillant

  

### 1.1 la clé RUN :  

| description                           | Clé                                                          |
| ------------------------------------- | ------------------------------------------------------------ |
| Liste des entrées dans l'invité "RUN" | HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU |
| Les autoruns                          | HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run           |

Il suffit de vérifier la malveillance des binaires présent dans les entrés.



### 1.2 Les tâches planifiées : 

On peut les récupérer avec PowerShell : 
```powershell
SchTasks.exe
```

![alt text](/assets/images/dfirMethodo/schtasks.png?raw=true "tasks")



### 1.3 Les services 

On peut les récupérer avec PowerShell : 

```powershell
Get-Service 
```

![alt text](/assets/images/dfirMethodo/services.png?raw=true "tasks")



### 1.4 Les requêtes wmi

Idem avec Powershell

```powershell
Get-WmiObject -Class __FilterToConsumerBinding -Namespace root\subscription
Get-WmiObject -Class __EventFilter -Namespace root\subscription
Get-WmiObject -Class __EventConsumer -Namespace root\subscription
```



### 1.5 Drivers et autres : 

Ici on peut utiliser l'outils "autoruns" de SysInternals qui est très pratique : 

![alt text](/assets/images/dfirMethodo/autoruns.png?raw=true "tasks")



## 2 Fichiers et timeline



### 2.1 Eléments supprimés

On peut les récupérer comme vue avec Autopsy, si non [PhotoRec](https://www.cgsecurity.org/wiki/PhotoRec "PhotoRec")  marche très bien.



### 2.2 Fichiers intéressants

Vous êtes parfois  amener à chercher des fichiers (IOC etc.)

Un petit One Liner Powershell pour faire une recherche récursive

```powershell
dir -Path C:\FolderName -Filter FileName.fileExtension -Recurse | %{$_.FullName}
```

Je recommande de rechercher les fichers *.lnk car ils sont beaucoup exploités lors [d'attaques](https://support.radware.com/ci/okcsFattach/get/15458_3  "attaques").



### 2.3 Parser la MFT

Parser la MFT permet de  récupérer des informations sur les fichiers notamment les dates de modifications, ce qui s'avère utile pour établir une timeline et rechercher une éventuelle compromission.

Voici une [lib](https://sourceforge.net/projects/ntfsreader/  "ibrairie ") c# qui marche très bien.



### 2.4 Parser l'Amcache

> L’AmCache est une base de données spécifiques à Windows 7, 8 et 10 et leurs équivalents serveurs, qui consigne des métadonnées portant sur l’exécution de binaires et l’installation de programmes sur un système. Méconnue et objet de recherches insuffisantes, cette base constitue un artefact sous-exploité dans le cadre des investigations numériques.

 Un petit papier de l'anssi [ici](https://www.ssi.gouv.fr/agence/publication/analyse-de-lamcache/ "ici")

Un tool pour le parser [ici](https://github.com/EricZimmerman/AmcacheParser  "ici"), fait par Eric ZIMMERMAN un des formateurs SANS, je recommande vraiment d'aller voir son travail.



### 2.5 Faire une FLS

La FLS permet, d'établir une timeline de modification des fichiers, à l'instar du parsing de la mft vu précédment.

TSK propose un tool [ici](https://wiki.sleuthkit.org/index.php?title=Fls "ici").

J'ai fait un tool en GUI pour Windows [ici](https://github.com/Xbloro/FLSGUI "ici")
([l'article](https://xbloro.github.io/tool/GUI-FLS-Tool/ "l'article"))



### 2.6 Jumplist

La jumpList contient les fichiers récent ouvert par des applications ou l'utilisateur , elle se trouve ici :

```
Nom de l'user +"\\AppData\\Roaming\\Microsoft\\Windows\\Recent"; 
```



### 2.7 Last Activity view

Last activity view est un tool bien pratique de nirsoft accessible [ici](https://www.nirsoft.net/utils/computer_activity_view.html "ici")

Il permet, comme son nom l'indique de voir les dernières activités faites sur l'ordinateur.



## 3 Matériel et autre



### 3.1 Clés usb : 

Pour lister les clés branchées au moins une fois sur la machine : 

```powershell
Get-ItemProperty -ErrorAction SilentlyContinue -Path HKLM:\SYSTEM\CurrentControlSet\Enum\USBSTOR\*\*
```



Vous pouvez lister les éléments ci-dessous en changeant le path de la commande :

```powershell
Get-ItemProperty -ErrorAction SilentlyContinue -Path clé-de-reg-ici
```



| description                              | Clé                                                          |
| ---------------------------------------- | ------------------------------------------------------------ |
| Les disques montés                       | HKLM\SYSTEM\MountedDevices                                   |
| Un autre endroit pour les disques montés | HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\CPCVolume |
| Les disques réseau                       | HKCU\Software\Microsoft\Windows\Current\VersionExplorer\MountPoints2 |
| les clés USB montée                      | HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR                   |

# Informations utiles 

## 1 Le trousseau de clés

-
Alors avant toute chose, on va essayer de comprendre ce qu'est une clé de registre .

Le dictionnaire de l'informatique Microsoft (Microsoft Computer Dictionary), cinquième édition, définit le Registre comme :
une base de données hiérarchique centrale, permettant de stocker les informations qui sont nécessaires pour configurer le système pour un ou plusieurs utilisateurs, programmes et périphériques matériels.

Lors de son exécution, Windows consulte en permanence les informations contenues dans le Registre, telles que les profils des utilisateurs, les applications installées sur l'ordinateur et les types de documents qu'elles peuvent créer, les paramètres de la feuille de propriétés pour les dossiers et les icônes des applications, le matériel du système et les ports utilisés.

Une ruche du Registre est un groupe de clés, de sous-clés et de valeurs du Registre associé à un ensemble de fichiers de prise en charge qui contiennent des sauvegardes de ses données.

Les clés de registres sont situées ici : C:\Windows\system32\config\

Voici toutes les clés, on va essayer d'expliquer un petit peu à quoi elles servent.  

| Registre | localisation|
|------------------------------- | ----------------------------|
| HKEY_USERS |  \Documents and Setting\User Profile\NTUSER.DAT |
| HKEY_USERS\DEFAULT | C(:\Windows\system32\config\default |
| HKEY_LOCAL_MACHINE\SAM | C:\Windows\system32\config\SAM  |
| HKEY_LOCAL_MACHINE\SECURITY | C:\Windows\system32\config\SECURITY  |
| HKEY_LOCAL_MACHINE\SOFTWARE | C:\Windows\system32\config\software |
| HKEY_LOCAL_MACHINE\SYSTEM | C:\Windows\system32\config\system |

HKEY_USERS contient tous les profils utilisateurs chargés activement sur l'ordinateur. HKEY_CURRENT_USER est une sous-clé de HKEY_USERS. L'abréviation « HKU » est parfois utilisée pour faire référence à HKEY_USERS.

NTUSER.dat contient toutes les informations spécifiques à chaque profil utilisateur (son compte de session)
La clé Default contient les informations spécifiques au system local et est utilisée par les programmes et services exécutés en tant que système local.

HKEY_LOCAL_MACHINE contient des informations de configuration spécifiques à l'ordinateur (pour n'importe quel utilisateur). L'abréviation « HKLM » est parfois utilisée pour faire référence à cette clé.

SAM est le gestionnaire de comptes de sécurité (The Security Account Manager), il stocke les mots de passe des utilisateurs. Il peut être utilisé pour authentifier les utilisateurs locaux et distants.

SECURITY est utilisé par le Kernel pour lire et appliquer la politique de sécurité applicable à l'utilisateur actuel et à toutes les applications ou opérations exécutées par cet utilisateur. Il contient également une sous-clé "SAM" qui est dynamiquement liée à la base de données SAM du domaine sur lequel l'utilisateur actuel est connecté.

SOFTWARE contient les logiciels et les paramètres Windows. Il est principalement modifié par les installateurs d'applications et de systèmes.

SYSTEM contient des informations sur la configuration du système Windows, les données du générateur de nombres aléatoires sécurisés (RNG), la liste des périphériques montés contenant un système de fichiers, plusieurs numéros "HKLM \ SYSTEM \ Control Sets" contenant des configurations alternatives pour les pilotes de matériel et les services en cours d'exécution.


HKEY_CLASSES_ROOT est une sous-clé de HKEY_LOCAL_MACHINE\Software. Les informations qui sont stockées à cet emplacement font en sorte que le programme approprié s'exécute lorsque vous ouvrez un fichier à l'aide de l'Explorateur Windows.

HKEY_CURRENT_CONFIG contient des informations sur le profil matériel utilisé par l'ordinateur local au démarrage du système.

Et oui ca fait un paquet de choses à voir hein ? Allez on se décourage pas je vais vous donner deux trois pistes à explorer !

Pour avoir accès au contenue des clé vous pouvez utiliser l'utilitaire Regedit de Windows et importer les ruches.  
Vous pouvez également utiliser regRipper, il vous donnera tout le contenu en fichier texte.

Pour tout ce qui est fichiers : 

| description | Clé |
|------------------------------- | -------------------------------------------------------------------------------|
| Les fichier ouvert récemment | HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSaveMRU  |
| Les fichier ouvert via winexplorer | HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs |
| Les fichier recherchés dans le menu | HKCU\Software\Microsoft\Search Assistant\ACMru |


Pour tout ce qui est Volume et média amovibles :

|   description   | Clé  |
|------------------------------- | -------------------------------------------------------------------------------|
| Les disques montés | HKLM\SYSTEM\MountedDevices  |
| Un autre endroit pour les disques montés | HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\CPCVolume |
| Les disques réseau | HKCU\Software\Microsoft\Windows\Current\VersionExplorer\MountPoints2 |
| les clés USB montée | HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR  |

Pour tout ce qui est Autorun, injection de commande et programme :  

| description | Clé |
|------------------------------- | -------------------------------------------------------------------------------|
| Liste des entrées dans l'invité "RUN" |  HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU  |
| Les autoruns | HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run |

| Les services lancés automatiquement |
|------------------------------- |
| HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\Services |
| HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\Services ou alors HKLM\SYSTEM\CurrentControlSet\Services  |
| HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\ServicesOnce |
| HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\ServicesOnce  (oui c'est pas la même que au dessus on est dans HKCU)|

| Les commandes lancées automatiquement avec cmd.exe |
|------------------------------- |
| HKLM\SOFTWARE\Microsoft\Command Processor |
| HKCU\Software\Microsoft\Command Processor (oui c'est pas la même que au dessus on est dans HKCU) |

| Les programmes se drope ici pour rester persistant|
|------------------------------- |
|HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon  (beacoup de malwares font ca ;) )|

|Donne les infos de lancement des executables |
|------------------------------- |
|HKCR\exe\fileshell\opencommand |
|HKEY_CLASSES_ROOT\batfile\shell\open\command|
|HKEY_CLASSES_ROOT\comfile\shell\open\command |
|Si vous trouvez : " default = “%1” %* >> somefilename.exe " c'est suspect suspect ! |


| Pour tout ce qui est programmes |
|------------------------------- |
| Permet de voir les programmes lancées : Ntuser.DAT  |
| HKEY_CURRENT_USER\Classes\Local Settings\Software\Microsoft\Windows\Shell\MuiCache  |
| HKEY_CURRENT_USER\Microsoft\Windows\ShellNoRoam\MUICache |
| HKEY_CURRENT_USER\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Compatibility Assistant\Persisted  |
| HKEY_CURRENT_USER\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Compatibility Assistant\Store  |
| HKEY_CLASSES_ROOT\Local Settings\Software\Microsoft\Windows\Shell\MuiCache  |
| programme installé : HKLM\SOFTWARE\Microsoft\Windows\Current\Version\Uninstall  |

Vous êtes encore la? allez encore une !

| Pour tout ce qui est Réseau |
|------------------------------- |
| SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles |
| SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles\Nla |
| SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles\NLA (c'est pas la même que au dessus) |
| SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles\Nla\Cache\IntranetAuth |
| ControlSet001\Services\Tcpip\Parameters\Interfaces\ |



## 2 L'investigations des journaux (LOG)
-

Les logs sont ici : C:\Windows\System32\winevt\log\
Pour les ouvrir vous avez votre parseur préféré ou alors le gestionnaire d'évènements Windows qui est bien pensé.

Je vous mets quand même quelques pistes à regarder : 

Trafic réseau sortant inhabituel  
Changements de comportement des Super utilisateurs  
Irrégularités géographiques dans les connexions et les modèles d'accès depuis des emplacements inhabituels  
Rechercher les échecs de connexion pour les comptes d'utilisateur inexistants et existant d'ailleurs  
Vérifier la taille de la réponse HTML si l'attaquant utilise l'injection SQL pour extraire des données  
Mise à jour inattendue du système ou des applications  
Vérifier si les données sont stockées dans des endroits incorrects  

Deux trois LOG : 

| Nom |
| ---- |
| Logon/Logoff Events |
| Security State Change Event |
| Networking Events |
| WPD (Windows Portable Devices) logs |
| User Plug n Play Event Logs |
| Remote Access Event Logs |
| Windows and Anti-Malware Update Events |

Quelques trucs intéressants dans les logs :

| Event ID (2000/XP/2003) | Event ID (Vista/7/8/2008/2012 | Description                               | Log Name |
|-------------------------|--------------------------------|-------------------------------------------|----------|
| 528                     | 4624                           | Successful Logon                          		  | Security |
| 529                     | 4625                           | Failed Login                              		  | Security |
| 680                     | 4776                           | Successful /Failed Account Authentication 		  | Security |
| 624                     | 4720                           | A user account was created		       		  | Security |
| 636                     | 4732                           | A member was added to a security-enabled local group | Security |
| 632		          | 4728			   | A member was added to a security-enabled global group| Security |
| 2934	          	  |7030			    	   | Service Creation Errors  				  |  System  |
| 2949	          	  |7045			    	   | Service Creation   				  |  System  |
| 2944	          	  |7040			    	   | The start type of the IPSEC Services service was changed from disabled to auto start. 				  |  System  |

Des infos de connexion : 

| Logon type | 	  Logon title |	Description|
|------------|----------------|------------|
| 2 | Interactive| A user logged on to this computer. |
| 3 |Network |  A user or computer logged on to this computer from the network. |
| 4 | Batch | Batch logon type is used by batch servers, where processes may be executing on behalf of a user without their direct intervention. |
| 5 | Service | A service was started by the Service Control Manager. |
| 7 | Unlock  |	This workstation was unlocked.|
| 8 | NetworkCleartext | A user logged on to this computer from the network. The user’s password was passed to the authentication package in its unhashed form.  The built-in authentication packages all hash credentials before sending them across the network. The credentials do not traverse the network in plaintext (also called cleartext). |
| 10| Connexion en RDP|





# 3 Produire une timeline 

-

La création d'une timeline est primordiale lors d'une attaque car elle permet de retracer toutes les actions de(s) attaquant(s) par ordre chronologique.

Elle regroupe, à la manière d'un tableau de bord le timestamps, les actions, les artéfacts et autre.

Elle permet de voir si des choses sont manquantes au niveau de l'enquête, par exemple si il manque une action entre deux steps d'une attaque, c'est que vous l'avez probablement ratée.

Un exemple d'une timeline fait par la SANS : 

![Timeline](/assets/images/dfirMethodo/Timeline.png?raw=true "timeline")



# 4 Analyse et Rapport
-

Il faut donc faire la synthèse de tous les éléments trouvés et essayer d'en tirer des conclusions. C'est la partie la plus difficile  mais elle est nécessaire.  
Il n'y a pas de technique ici, pensez que c'est comme une analyse de texte au bac. Vous avez produit le texte avec l'investigation menée juste avant, maintenant il faut analyser ce texte. C'est une partie à ne pas négliger car elle permet de poser les choses et de réfléchir autrement. 

N'oubliez pas, l'analyse permet de tirer des conclusions  grâce aux éléments trouvés lors de l'investigation.
Elle doit être présente et justifiée dans le rapport que vous allez rendre à votre client.

# 5 Conclusion
-

Nous voila arrivé à la fin, j'espère que cela aura pu vous être utile. Notez que cette méthodologie ne couvre pas tout ! Si vous voyez des erreurs ou des choses à rajouter n'hésitez pas pour qu'on puisse améliorer au maximum cet article.



Sources
-
https://support.microsoft.com/fr-fr/help/256986/windows-registry-information-for-advanced-users
https://devblogs.microsoft.com/oldnewthing/20070302-00/?p=27783
https://en.wikipedia.org/wiki/Windows_Registry#HKEY_LOCAL_MACHINE_(HKLM)
https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc976337(v=technet.10)?redirectedfrom=MSDN
