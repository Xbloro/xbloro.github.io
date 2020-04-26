---
title: "Windows DFIR methodo "

categories:
  - DFIR
tags:
  - DFIR
  - Windows
---

# Ce document est en cours de révisions 


# Méthodologie d'analyse Inforensique Windows

Ce document détaille quelques procédures et éléments intéressants lors d'une investigation forensique Windows.
N'hésitez pas à me donner votre avis ou me dire si j'ai fait des erreurs. 
Si vous avez des questions n'hésitez pas sur twitter : https://twitter.com/HHUG0

# Disclamer

Le but de ce document est de présenter UNE méthodologie ainsi que certains aspects de Windows qui pourront être utiles lors d'une investigation.
Ce document n'est pas exhaustif, en fonction de votre entreprise, expérience, cas traité et évolution des technologies, les procédures peuvent être complètements différentes.

Ce document n'est pas orienté CTF mais plutôt cas réel.  



# Introduction


Cet article est destiné aux débutants. Cependant une connaissance de base du fonctionnement de Windows est recommandée.

Quand un attaquant pénètre dans un système, il laisse toujours des traces de son passage. Dans cet article nous allons étudier les mécanismes de Windows afin de trouver des "preuves". Par la suite nous allons essayer de construire une timeline du déroulement de l'attaque.
Nous verrons en première partie quelques principes d'acquisition des médias puis nous passerons à l'investigation.

# 1 Acquisition des médias


Au cours d'une investigation acquisition des données est cruciale ! En effet l'intégrité des données doit être respectée afin que les "preuves" trouvées soient crédibles et recevables.



  1.1 Dump de la R.A.M et récupération d'informations machine
  -

Vous êtes arrivés devant la machine cible mais que faire?
Nous allons essayer de collecter le plus d'informations possible, R.A.M, profile machine, profil réseau, table arp, etc.

C'est partie pour le  dump de la R.A.M.
La R.A.M permet de récupérer beaucoup d'informations sur la machine ainsi que les processus tournant en direct.

Pour cela nous allons utiliser l'outils Redline de FireEye : https://www.fireeye.fr/services/freeware/redline.html  
Redline permet de récupérer le profil de la machine ainsi que des informations intéressantes en plus de la R.A.M.
Il propose une interface graphique pour l'analyse des données.
Si vous êtes un adepte de Volatility (et vous avez raison !), sachez que le dump est compatible.

Notez qu'il existe beaucoup d'autres outils de dump mais RedLine est un des meilleurs à mon goût.
Certains RSSI peuvent également dumper la mémoire de la machine à distance avec leur propre solution, il est donc nécessaire de se rapprocher d'eux.

Le logiciel fonctionne de cette façon, vous choisissez les informations que vous souhaitez récupérer et vous créez un payload que vous déposerez sur un média amovible.
Pourquoi? car il ne faut pas compromettre les données de la machine et donc ne rien transférer ou exécuter dessus.
C'est pour cela que notre payload sera exécuté SUR le média amovible.
Notez que la RAM sera déposée sur votre média amovible il faut donc que la capacité mémoire de ce dernier soit plus élevé que la RAM de la machine.

Allez c'est parti, on lance RedLine !

Dans le menu on nous propose 3 choix :  
-Le premier permet de récupérer des informations sur la machines (utilisateurs, profile machine, etc.) et de dumper la RAM;    
-Le second permet la même chose mais en plus de rechercher des IOCs (c'est plutôt cool);    
-Le dernier permet de rechercher ses propres IOC. (Si vous êtes dans une entreprise et que vous voulez chercher des traces d'intrusions dans toute la foret info c'est assez sympa, suffit de déployer le payload via PowerShell ;) ).  

![alt text](/assets/images/menuCollecteur.png?raw=true "ChoixMenuRedline")

Dans chaque cas Redline vous propose d'éditer le script du payload, pour ajouter des fonctions (récupération de string par ex) mais ici on va le laisser par défaut.

On va choisir l'option numéro 2 : "Comprehensive collector" 


![alt text](/assets/images/collecteur2.png?raw=true "ChoixN2")  



Une deuxième fenêtre nous demande si on veut récupérer la mémoire de la machine, on coche la case (n1)  
Puis on choisit l'emplacement de sauvegarde du Payload(n2) >> votre média Amovible.  

<img src="/assets/images/Collecteur3.png" width="600">




Et voila ! votre payload est prêt.  

![alt text](/assets/images/payload1.png?raw=true "Payload")

A présent, on branche le média amovible sur le Windows cible et on exécute le script "RunRedlineAudit.bat".

Une fois l'opération terminée, un fichier "AnalysisSessionX.mans " est créé dans le répertoire de votre payload.  
Il suffit juste de l'ouvrir avec RedLine (sur votre machine d'analyse ^^') pour pouvoir commencer à travailler.  
Mais doucement ! L'acquisition n'est pas encore terminée, il faut maintenant faire une copie forensique de tous les disques de stockage de la machine.  

1.2 Acquisition des disques  
-

Prévoyez une bonne quantité de stockage car on va récupérer tout ce qu'il y a dans la machine cible.  

Pour cela, le mieux (et dans le cadre d'une enquête juridique c'est obligatoire) est utiliser un tableau bloqueur-copieur.  
C'est un équipement qui ressemble à ca :  

<img src="/assets/images/td2u.jpg" width="400">

Il permet de bloquer le flux en écriture vers la machine cible et d'en faire une image disque vers votre matériel. Avec ça, pas d'atteinte à l'intégrité des données.  
Avec l'accord (et oui toujours) de la SSI on éteint la machine et on prend les disques (juste le temps de les copier hein après il faut les rendre !). Nous avons éteint la machine pour copier les disques afin d'éviter une compromission des "preuves"  
(quoi encore ? et oui il faut être prudent).

Sur le tableau il faut choisir un format accepté comme l'EWF (E0), notez que le format ISO n'est pas recevable.  
On branche notre média amovible, on branche le/les disque(s) sources et on s'arme de patience, car c'est long...  trèèèès long.  
N'oubliez pas de faire la vérification des données copiée, le tableau propose l'option en général.

Si jamais vous n'avez pas de tableau ou même un simple bloqueur. Vous pouvez utiliser une distribution qui va bloquer en écriture ses ports USB etc. (coucou [tsurugi linux](https://tsurugi-linux.org/ "tsurugi linux") ).  
Des outils comme [FTK imager](https://accessdata.com/product-download/ftk-imager-version-4-2-0 "ftk imager") ou [EWF Toolkit](https://github.com/libyal/libewf "ewf toolkit") permettent de faire une copie EWF.  
Attention cependant sans tableau ou bloqueur, la recevabilité des preuves peut être compromise.  
Ce n'est donc pas une méthode à employer mais cela pourra surement vous être utile.  


Quelques informations à récupérer qui vous seront très utiles par la suite : 
- Un schéma réseau et de l'architecture de l'entreprise (avec les IP etc.);
- La politique de sécurité SSI de l'entreprise;
- Les autorisations de la session du poste incriminé;
- Si possible les horaires de travail de l'utilisateur;
- Le domaine d'activité de l'utilisateur (il fait de la modélisation? de la comptabilité?);
- La liste des accès réseau et aux applications de l'utilisateur.


# 2 Investigation 

On a enfin fini avec acquisition ! On va pouvoir mettre les mains dans la cyber.
Allez on est rentré dans notre laboratoire de travail et on est pressé d'investiguer, mais pas si vite !  
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

2.1 Préparation d'un environnement de travail
-

On arrive dans une partie moins ludique ou on va voir des outils et des pistes de recherches.
Chaque investigation étant unique il faudra adapter vos recherches en fonction des éléments que vous allez découvrir.

Qui de mieux que Windows peut s'occuper de votre Windows ?  
Linux ?  
Perdu ! enfin ça dépend de vous mais au cours de cet article nous utiliserons Windows car il comprend des outils utiles et inexistants sous Linux.

Attention cependant vous allez être amené à travailler sur un environnement possiblement infecté et il faut prendre les précautions adaptées.

Nous allons donc partir sur une machine virtuelle Windows 10 un peu pimpée grâce à [FlareVM](https://www.fireeye.com/blog/threat-research/2017/07/flare-vm-the-windows-malware.html "FlareVM")  
On oublie pas de bien cloisonner la machine pour éviter toute fuite par le réseau et de faire des snapshots.

Dans le cadre de l'investigation nous allons utiliser [Autopsy](https://www.sleuthkit.org/autopsy/ "Autopsy") car c'est un outils très complet, reconnu et distribué gratuitement.  
Si vous avez beaucoup d'argent vous pouvez en choisir un autre comme Encase ou Magnet.

On oublie pas de prendre [Regripper](https://github.com/keydet89/RegRipper2.8 "regripper") et puis bien évidement RedLine.

On est paré et on peut commencer.  



2.2 Primo Investigation
-

La primo investigation consiste à faire l'état des lieux de la machine.

Voici les étapes que nous allons effectuer :   
-récupération des fichiers supprimés;  
-On analyse un peu les fichiers présent (Fichier Word suspect ? un fichier mdp sur le bureau ? on y reviendra plus tard);  
-récupération du "trousseau de clé" (les clés de registre);  
-récupération des logs;
-récupération des shadows copie.

2.2.1 Autopsy
-

Allez c'est partit, on lance Autopsy ! 

Dans un premier temps il va vous demander les informations sur l'affaire, essayez de les remplir correctement, surtout si vous travaillez à plusieurs.

On arrive ensuite au Menu ou on vous demande quel est le type de support que vous voulez importer, choisissez celui qui correspond.

![alt text](/assets/images/Autmenu.png?raw=true "Menu")    


Si vous avez fait une copie EWF choisissez "Disk image" et sélectionnez le fichier n1 (E01), les autres seront importés automatiquement par Autopsy.  

Il va vous demander ensuite ce que vous voulez récupérer sur l'image disque. On sélectionne tout.  

![alt text](/assets/images/AutCheck.png?raw=true "Check")  

Allez maintenant on peut allé se chercher un café car c'est long, très long...   
Plus c'est gros, plus c'est l.. heuu ah oui le café  

![alt text](/assets/images/AutLoad.png?raw=true "load")  

En attendant que le traitement soit fini, on va se familiariser avec l'interface de notre nouveau meilleur amie.

Sur la partie gauche vous avez une arborescence. En haut on a la première partie qui nous permet de voir le contenu du disque de la même façon que quand il est  monté sur la machine hôte. C'est à partir de cet emplacement que vous allez pouvoir naviguer dans l'arborescence de la machine pour récupérer les logs ainsi que le trousseau de clés !


![alt text](/assets/images/AutSource.png?raw=true "Source")  

Pour exporter un fichier rien de plus simple, il suffit de cliquer droit et exporter, choisissez ensuite la destination sur votre machine.

![alt text](/assets/images/AutExport.png?raw=true "Export")

Attention cependant, si vous exportez un fichier malveillant, dropez le dans un fichier inaccessible par votre antivirus car il risquerait de le compromettre. je vous conseille de le zippe(avec mdp) pour éviter de cliquer dessus sans faire exprès !


Ensuite arrive la partie View qui trie les fichiers par type etc. C'est aussi la que la partie CARVING est accessible. Souvenez vous dans la méthodologie la recherche dans les fichiers supprimés, ça se passe ici !


![alt text](/assets/images/AutView.png?raw=true "view")

Enfin on a la partie Results qui permet de voir des éléments intéressants, je vous laisse regarder.
La partie Exif permet accéder rapidement aux photos et à leur emplacement.
Par exemple, si vous trouvez une image pornographique (oui oui ca arrive), et que vous voyez qu'elle était à l'origine dans un PowerPoint... Vous savez quoi faire ! (pour ceux qui savent pas, on va rechercher des macros malveillantes dans le ppt)


![alt text](/assets/images/AutRes.png?raw=true "res")

2.3 Le trousseau de clés
-
Alors avant toute chose, on va essayer de comprendre ce qu'est une clé de registre et pourquoi elles sont si importantes.

Le dictionnaire de l'informatique Microsoft (Microsoft Computer Dictionary), cinquième édition, définit le Registre comme :
une base de données hiérarchique centrale, permettant de stocker les informations qui sont nécessaires pour configurer le système pour un ou plusieurs utilisateurs, programmes et périphériques matériels.

Lors de son exécution, Windows consulte en permanence les informations contenues dans le Registre, telles que les profils des utilisateurs, les applications installées sur l'ordinateur et les types de documents qu'elles peuvent créer, les paramètres de la feuille de propriétés pour les dossiers et les icônes des applications, le matériel du système et les ports utilisés.

Une ruche du Registre est un groupe de clés, de sous-clés et de valeurs du Registre associé à un ensemble de fichiers de prise en charge qui contiennent des sauvegardes de ses données.

Vous l'avez compris, les clés de registre permettent de connaitre toute la configuration de Windows et des programmes qui y sont installés.

Les clés de registres sont situées ici : C:\Windows\system32\config\

Voici toutes les clés, on va essayer d'expliquer un petit peu à quoi elles servent.  

| Registre | localisation|
|------------------------------- | ----------------------------|
| HKEY_USERS |  \Documents and Setting\User Profile\NTUSER.DAT |
| HKEY_USERS\DEFAULT | C:\Windows\system32\config\default  |
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

Pour tout ce qui est fichier : 

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
| Les autorun | HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run |

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


2.4 L'investigations des journaux (LOG)
-

Ici malheureusement je n'ai pas de méthodologie à vous apporter.
Cela va vraiment dépendre de votre cas et des éléments que vous avez trouver précédemment et le rapport fournie par l'entreprise devrait vous aiguiller.

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

2.5 L'analyse de la R.A.M 
-
Ah vous l'attendiez celle la !
Sachez que vous pouvez faire presque toute l'investigation depuis la R.A.M, en effet  quasiment tous  les éléments présentés au dessus sont disponibles.
Vous avez accès aux clés de registre ! 
De plus vous pouvez accéder aux processus tournant en direct, leur infos de lancement (parents, pid etc.).
Lors de l'investigation, l'analyse de la R.A.M est fait en parallèle que celle du disque.

Ici notre investigation va se baser principalement sur les processus et les connexions réseaux.
Redline nous permet d'accéder à tout ca très facilement. Si vous préférez Volatility, je vous laisse regarder des tutoriels.


3 Analyse et Rapport
-

Vous n'êtes pas dans un CTF et il n'y a pas de Flag à trouver, il faut donc faire la synthèse de tous les éléments trouvés et essayer d'en tirer des conclusions. C'est la partie la plus difficile  mais elle est nécessaire.  
Il n'y a pas de technique ici, pensez que c'est comme une analyse de texte au bac. Vous avez produit le texte avec l'investigation menée juste avant, maintenant il faut analyser ce texte. C'est une partie à ne pas négliger car elle permet de poser les choses et de réfléchir autrement. 

N'oubliez pas, l'analyse permet de tirer des conclusions  grâce aux éléments trouvés lors de l'investigation.
Elle doit être présente et justifiée dans le rapport que vous allez rendre à votre client.



4 Conclusion
-

Nous voila arrivé à la fin, j'espère que cela aura pu vous être utile. Notez que cette méthodologie ne couvre pas tout car destinée aux débutants ! Si vous voyez des erreurs ou des choses à rajouter n'hésitez pas pour qu'on puisse améliorer au maximum cet article.
Un petit cas pratique étape par étape est en cours d'écriture ;) 



Sources
-
https://support.microsoft.com/fr-fr/help/256986/windows-registry-information-for-advanced-users
https://devblogs.microsoft.com/oldnewthing/20070302-00/?p=27783
https://en.wikipedia.org/wiki/Windows_Registry#HKEY_LOCAL_MACHINE_(HKLM)
https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc976337(v=technet.10)?redirectedfrom=MSDN
