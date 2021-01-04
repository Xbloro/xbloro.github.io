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

<img src="/assets/images/dfirMethodo/td2u.jpg" width="400">

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

On oublie pas de prendre [Regripper](https://github.com/keydet89/RegRipper2.8 "regripper") même si c'est pas le top car il manque des infos.



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

# Petit Point Sur REDLINE :

L'outils [Redline](https://www.fireeye.fr/services/freeware/redline.html) de FireEye  permet de récupérer le profil de la machine ainsi que des informations intéressantes en plus de la R.A.M. Il propose une interface graphique pour l'analyse des données. Si vous êtes un adepte de Volatility , sachez que le dump est compatible.

Le logiciel fonctionne de cette façon, vous choisissez les informations que vous souhaitez récupérer et vous créez un payload que vous déposerez sur un média amovible. Pourquoi? car il ne faut pas compromettre les données de la machine et donc ne rien transférer ou exécuter dessus. C'est pour cela que notre payload sera exécuté SUR le média amovible. Notez que la RAM sera déposée sur votre média amovible il faut donc que la capacité mémoire de ce dernier soit plus élevé que la RAM de la machine.

Dans le menu on nous propose 3 choix :
-Le premier permet de récupérer des informations sur la machines (utilisateurs, profile machine, etc.) et de dumper la RAM;
-Le second permet la même chose mais en plus de rechercher des IOCs (vous pouvez lui fournir une liste perso) ;
-Le dernier permet de rechercher ses propres IOC. (Si vous êtes dans une entreprise et que vous voulez chercher des traces d'intrusions dans toute la foret info c'est assez sympa, suffit de déployer le payload via PowerShell  ).

A présent, on branche le média amovible sur le Windows cible et on exécute le script "RunRedlineAudit.bat".

Une fois l'opération terminée, un fichier "AnalysisSessionX.mans " est créé dans le répertoire de votre payload.
Il suffit juste de l'ouvrir avec RedLine.

![alt text](/assets/images/dfirMethodo/menuCollecteur.png?raw=true "tasks")

![alt text](/assets/images/dfirMethodo/collecteur3.png?raw=true "tasks")

# Investigation LIVE or DEAD
-

Que ce soit en live or DEAD les taches suivantes sont clés.

Voici les étapes que nous allons effectuer :  

- Auditer les "autoruns" et rechercher des persistances;

- Récupérer les informations machines;

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

- checker les connexions réseau

- Faire une fls;

- Parser la mft;

  
  
## Les informations Machine 

La timeZone : 

  ```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation
  ```

  Les Users :

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList
```

Les infos machine :

```
SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion
```



## Autoruns et persistances : 

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



### 1.6 Liste des Clés à checker

Une petite liste mais c'est long :), go checker avec autoruns !

```
Annexe : Logon
HKLM\System\CurrentControlSet\Control\Terminal Server\Wds\rdpwd\StartupPrograms
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\AppSetup
HKLM\Software\Policies\Microsoft\Windows\System\Scripts\Startup
HKCU\Software\Policies\Microsoft\Windows\System\Scripts\Logon
HKLM\Software\Policies\Microsoft\Windows\System\Scripts\Logon
HKCU\Environment\UserInitMprLogonScript
HKLM\Environment\UserInitMprLogonScript
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Userinit
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\VmApplet
HKLM\Software\Policies\Microsoft\Windows\System\Scripts\Shutdown
HKCU\Software\Policies\Microsoft\Windows\System\Scripts\Logoff
HKLM\Software\Policies\Microsoft\Windows\System\Scripts\Logoff
HKCU\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Startup
HKLM\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Startup
HKCU\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Logon
HKLM\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Logon
HKCU\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Logoff
HKLM\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Logoff
HKCU\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Shutdown
HKLM\Software\Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Shutdown
HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\System\Shell
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Shell
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Shell
HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot\AlternateShell
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Taskman
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\AlternateShells\AvailableShells
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Runonce
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunonceEx
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp\InitialProgram
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
HKCU\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnceEx
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx
HKCU\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnceEx
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\Load
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\Run
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run
HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components
HKLM\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Windows\IconServiceLib
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Runonce
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunonceEx
HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft\Windows CE Services\AutoStartOnConnect
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows CE Services\AutoStartOnConnect
HKLM\SOFTWARE\Microsoft\Windows CE Services\AutoStartOnDisconnect
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows CE Services\AutoStartOnDisconnect

Annexe 2 :  explorer
HKCU\SOFTWARE\Classes\Protocols\Filter	
HKCU\SOFTWARE\Classes\Protocols\Handler
HKCU\SOFTWARE\Microsoft\Internet Explorer\Desktop\Components				
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\SharedTaskScheduler				
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\SharedTaskScheduler					
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellServiceObjects					
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\ShellServiceObjects					
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\ShellServiceObjects					
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad					
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad					
HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\ShellServiceObjectDelayLoad					
HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\ShellExecuteHooks					
HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\ShellExecuteHooks					
HKCU\Software\Classes\*\ShellEx\ContextMenuHandlers										
HKLM\Software\Wow6432Node\Classes\*\ShellEx\ContextMenuHandlers					
HKCU\Software\Classes\Drive\ShellEx\ContextMenuHandlers				
HKLM\Software\Wow6432Node\Classes\Drive\ShellEx\ContextMenuHandlers					
HKCU\Software\Classes\*\ShellEx\PropertySheetHandlers										
HKLM\Software\Wow6432Node\Classes\*\ShellEx\PropertySheetHandlers					
HKCU\Software\Classes\AllFileSystemObjects\ShellEx\ContextMenuHandlers
HKLM\Software\Classes\AllFileSystemObjects\ShellEx\ContextMenuHandlers					
HKLM\Software\Wow6432Node\Classes\AllFileSystemObjects\ShellEx\ContextMenuHandlers					
HKCU\Software\Classes\AllFileSystemObjects\ShellEx\DragDropHandlers
HKLM\Software\Classes\AllFileSystemObjects\ShellEx\DragDropHandlers					
HKLM\Software\Wow6432Node\Classes\AllFileSystemObjects\ShellEx\DragDropHandlers					
HKCU\Software\Classes\AllFileSystemObjects\ShellEx\PropertySheetHandlers
HKLM\Software\Classes\AllFileSystemObjects\ShellEx\PropertySheetHandlers					
HKLM\Software\Wow6432Node\Classes\AllFileSystemObjects\ShellEx\PropertySheetHandlers					
HKCU\Software\Classes\Directory\ShellEx\ContextMenuHandlers	
HKLM\Software\Classes\Directory\ShellEx\ContextMenuHandlers					
HKLM\Software\Wow6432Node\Classes\Directory\ShellEx\ContextMenuHandlers					
HKCU\Software\Classes\Directory\Shellex\DragDropHandlers					
HKLM\Software\Classes\Directory\Shellex\DragDropHandlers					
HKLM\Software\Wow6432Node\Classes\Directory\Shellex\DragDropHandlers				
HKCU\Software\Classes\Directory\Shellex\PropertySheetHandlers					
HKLM\Software\Classes\Directory\Shellex\PropertySheetHandlers					
HKLM\Software\Wow6432Node\Classes\Directory\Shellex\PropertySheetHandlers					
HKCU\Software\Classes\Directory\Shellex\CopyHookHandlers					
HKLM\Software\Classes\Directory\Shellex\CopyHookHandlers					
HKLM\Software\Wow6432Node\Classes\Directory\Shellex\CopyHookHandlers					
HKCU\Software\Classes\Directory\Background\ShellEx\ContextMenuHandlers					
HKLM\Software\Classes\Directory\Background\ShellEx\ContextMenuHandlers		
HKLM\Software\Wow6432Node\Classes\Directory\Background\ShellEx\ContextMenuHandlers		
HKCU\Software\Classes\Folder\Shellex\ColumnHandlers
HKLM\Software\Classes\Folder\Shellex\ColumnHandlers					
HKLM\Software\Wow6432Node\Classes\Folder\Shellex\ColumnHandlers					
HKCU\Software\Classes\Folder\ShellEx\ContextMenuHandlers
HKLM\Software\Classes\Folder\ShellEx\ContextMenuHandlers					
HKLM\Software\Wow6432Node\Classes\Folder\ShellEx\ContextMenuHandlers					
HKCU\Software\Classes\Folder\ShellEx\DragDropHandlers
HKLM\Software\Classes\Folder\ShellEx\DragDropHandlers			
HKLM\Software\Wow6432Node\Classes\Folder\ShellEx\DragDropHandlers				
HKCU\Software\Classes\Folder\ShellEx\ExtShellFolderViews
HKLM\Software\Classes\Folder\ShellEx\ExtShellFolderViews					
HKLM\Software\Wow6432Node\Classes\Folder\ShellEx\ExtShellFolderViews					
HKCU\Software\Classes\Folder\ShellEx\PropertySheetHandlers
HKLM\Software\Classes\Folder\ShellEx\PropertySheetHandlers					
HKLM\Software\Wow6432Node\Classes\Folder\ShellEx\PropertySheetHandlers					
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers	
HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers					
HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\ShellIconOverlayIdentifiers					
HKCU\Software\Classes\Clsid\{AB8902B4-09CA-4bb6-B78D-A8F59079A8D5}\Inprocserver32	
HKCU\Software\Microsoft\Ctf\LangBarAddin					
HKLM\Software\Microsoft\Ctf\LangBarAddin		

Annexe : Objet IE
HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects		
HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects
HKCU\Software\Microsoft\Internet Explorer\UrlSearchHooks					
HKLM\Software\Microsoft\Internet Explorer\Toolbar					
HKLM\Software\Wow6432Node\Microsoft\Internet Explorer\Toolbar					
HKCU\Software\Microsoft\Internet Explorer\Explorer Bars				
HKLM\Software\Microsoft\Internet Explorer\Explorer Bars					
HKCU\Software\Wow6432Node\Microsoft\Internet Explorer\Explorer Bars
HKLM\Software\Wow6432Node\Microsoft\Internet Explorer\Explorer Bars					
HKCU\Software\Microsoft\Internet Explorer\Extensions				
HKLM\Software\Microsoft\Internet Explorer\Extensions					
HKCU\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions
HKLM\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions

Annexe : Service
HKLM\System\CurrentControlSet\Services				

Annexe : Drivers
HKLM\System\CurrentControlSet\Services
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers

Annexe : codecs
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Drivers32					
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Drivers32					
HKCU\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32
HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Drivers32					
HKCU\Software\Classes\Filter					
HKLM\Software\Classes\Filter					
HKCU\Software\Classes\CLSID\{083863F1-70DE-11d0-BD40-00A0C911CE86}\Instance
HKCU\Software\Wow6432Node\Classes\CLSID\{083863F1-70DE-11d0-BD40-00A0C911CE86}\Instance
HKCU\Software\Classes\CLSID\{AC757296-3522-4E11-9862-C17BE5A1767E}\Instance
HKCU\Software\Wow6432Node\Classes\CLSID\{AC757296-3522-4E11-9862-C17BE5A1767E}\Instance
HKCU\Software\Classes\CLSID\{7ED96837-96F0-4812-B211-F13C24117ED3}\Instance
HKCU\Software\Wow6432Node\Classes\CLSID\{7ED96837-96F0-4812-B211-F13C24117ED3}\Instance
HKCU\Software\Classes\CLSID\{ABE3B9A4-257D-4B97-BD1A-294AF496222E}\Instance
HKCU\Software\Wow6432Node\Classes\CLSID\{ABE3B9A4-257D-4B97-BD1A-294AF496222E}\Instance
HKLM\Software\Classes\CLSID\{083863F1-70DE-11d0-BD40-00A0C911CE86}\Instance					
HKLM\Software\Wow6432Node\Classes\CLSID\{083863F1-70DE-11d0-BD40-00A0C911CE86}\Instance						
HKLM\Software\Classes\CLSID\{AC757296-3522-4E11-9862-C17BE5A1767E}\Instance
HKLM\Software\Wow6432Node\Classes\CLSID\{AC757296-3522-4E11-9862-C17BE5A1767E}\Instance
HKLM\Software\Classes\CLSID\{7ED96837-96F0-4812-B211-F13C24117ED3}\Instance					
HKLM\Software\Wow6432Node\Classes\CLSID\{7ED96837-96F0-4812-B211-F13C24117ED3}\Instance					
HKLM\Software\Classes\CLSID\{ABE3B9A4-257D-4B97-BD1A-294AF496222E}\Instance
HKLM\Software\Wow6432Node\Classes\CLSID\{ABE3B9A4-257D-4B97-BD1A-294AF496222E}\Instance

Annexe : Boot execute
HKLM\System\CurrentControlSet\Control\Session Manager\BootExecute					
HKLM\System\CurrentControlSet\Control\Session Manager\SetupExecute				
HKLM\System\CurrentControlSet\Control\Session Manager\Execute					
HKLM\System\CurrentControlSet\Control\Session Manager\S0InitialCommand		

Annexe : Image Hijacks
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options					
HKLM\Software\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Image File Execution Options					
HKLM\Software\Microsoft\Command Processor\Autorun					
HKLM\Software\Wow6432Node\Microsoft\Command Processor\Autorun					
HKCU\Software\Microsoft\Command Processor\Autorun					
HKCU\SOFTWARE\Classes\Exefile\Shell\Open\Command\(Default)
HKLM\SOFTWARE\Classes\Exefile\Shell\Open\Command\(Default)			
HKLM\Software\Classes\.exe					
HKCU\Software\Classes\.exe				
HKLM\Software\Classes\.cmd					
HKCU\Software\Classes\.cmd					
HKCU\SOFTWARE\Classes\Htmlfile\Shell\Open\Command\(Default)
HKLM\SOFTWARE\Classes\Htmlfile\Shell\Open\Command\(Default)				

Annexe : Appinit
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\Appinit_Dlls					
HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Windows\Appinit_Dlls					
HKLM\System\CurrentControlSet\Control\Session Manager\AppCertDlls		

Annexe : KnownDlls
HKLM\System\CurrentControlSet\Control\Session Manager\KnownDlls			

Annexe : WinLogon
HKLM\SYSTEM\Setup\CmdLine					
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers					
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Provider Filters					
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\PLAP Providers					
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\Taskman					
HKCU\SOFTWARE\Policies\Microsoft\Windows\Control Panel\Desktop\Scrnsave.exe
HKCU\Control Panel\Desktop\Scrnsave.exe					
HKLM\System\CurrentControlSet\Control\BootVerificationProgram\ImagePath					
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\GpExtensions		

Annexe : WinsockProvider
HKLM\System\CurrentControlSet\Services\WinSock2\Parameters\Protocol_Catalog9\Catalog_Entries					
HKLM\System\CurrentControlSet\Services\WinSock2\Parameters\NameSpace_Catalog5\Catalog_Entries					
HKLM\System\CurrentControlSet\Services\WinSock2\Parameters\Protocol_Catalog9\Catalog_Entries64					
HKLM\System\CurrentControlSet\Services\WinSock2\Parameters\NameSpace_Catalog5\Catalog_Entries64			

Annexe : Print monitor
HKLM\SYSTEM\CurrentControlSet\Control\Print\Monitors					
HKLM\SYSTEM\CurrentControlSet\Control\Print\Providers	

Annexe : Lsa provider
HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\SecurityProviders					
HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Authentication Packages					
HKLM\SYSTEM\CurrentControlSet\Control\Lsa\Notification Packages	

Annexe : Network provider
HKLM\SYSTEM\CurrentControlSet\Control\NetworkProvider\Order	

Annexe : Office plugins
HKLM\Software\Microsoft\Office\Outlook\Addins						
HKCU\Software\Microsoft\Office\Outlook\Addins
HKLM\Software\Wow6432Node\Microsoft\Office\Outlook\Addins
HKCU\Software\Wow6432Node\Microsoft\Office\Outlook\Addins
HKLM\Software\Microsoft\Office\Excel\Addins
HKCU\Software\Microsoft\Office\Excel\Addins					
HKLM\Software\Wow6432Node\Microsoft\Office\Excel\Addins
HKCU\Software\Wow6432Node\Microsoft\Office\Excel\Addins
HKLM\Software\Microsoft\Office\PowerPoint\Addins
HKCU\Software\Microsoft\Office\PowerPoint\Addins	
HKLM\Software\Wow6432Node\Microsoft\Office\PowerPoint\Addins
HKCU\Software\Wow6432Node\Microsoft\Office\PowerPoint\Addins
HKLM\Software\Microsoft\Office\Word\Addins
HKCU\Software\Microsoft\Office\Word\Addins	
HKLM\Software\Wow6432Node\Microsoft\Office\Word\Addins
HKCU\Software\Wow6432Node\Microsoft\Office\Word\Addins
HKLM\Software\Microsoft\Office\Access\Addins
HKCU\Software\Microsoft\Office\Access\Addins
HKLM\Software\Wow6432Node\Microsoft\Office\Access\Addins
HKCU\Software\Wow6432Node\Microsoft\Office\Access\Addins
HKLM\Software\Microsoft\Office\Onenote\Addins
HKCU\Software\Microsoft\Office\Onenote\Addins
HKLM\Software\Wow6432Node\Microsoft\Office\Onenote\Addins
HKCU\Software\Wow6432Node\Microsoft\Office\Onenote\Addins
HKLM\SOFTWARE\Microsoft\Office test\Special\Perf\(Default)
HKCU\SOFTWARE\Microsoft\Office test\Special\Perf\(Default)
```

### 1.7 Process

Récupérer les evenement par processus :

```powershell
Get-WinEvent -FilterHashTable @{Logname = "Security" ; ID = 5059,5061}
```



## Le réseau

Sysinternal TCP view fait très bien le taff :)

![alt text](/assets/images/dfirMethodo/tcpview.png?raw=true "tasks")

Get TCP by PID

```powershell
while($true){ $processes = (Get-NetTCPConnection | ? {($_.RemoteAddress -eq "IPaddr")}).OwningProcess; foreach ($process in $processes) { Get-Process -PID $process | select ID,ProcessName } }
```



## Fichiers et timeline

###  Eléments supprimés

On peut les récupérer comme vue avec Autopsy, si non [PhotoRec](https://www.cgsecurity.org/wiki/PhotoRec "PhotoRec")  marche très bien.

### Fichiers intéressants

Vous êtes parfois  amener à chercher des fichiers (IOC etc.)

Un petit One Liner Powershell pour faire une recherche récursive

```powershell
dir -Path C:\FolderName -Filter FileName.fileExtension -Recurse | %{$_.FullName}
```

Je recommande de rechercher les fichers *.lnk car ils sont beaucoup exploités lors [d'attaques](https://support.radware.com/ci/okcsFattach/get/15458_3  "attaques").





### Parser la MFT

Parser la MFT permet de  récupérer des informations sur les fichiers notamment les dates de modifications, ce qui s'avère utile pour établir une timeline et rechercher une éventuelle compromission.

Voici une [lib](https://sourceforge.net/projects/ntfsreader/  "ibrairie ") c# qui marche très bien.

### 2.4 Parser l'Amcache

> L’AmCache est une base de données spécifiques à Windows 7, 8 et 10 et leurs équivalents serveurs, qui consigne des métadonnées portant sur l’exécution de binaires et l’installation de programmes sur un système. Méconnue et objet de recherches insuffisantes, cette base constitue un artefact sous-exploité dans le cadre des investigations numériques.

 Un petit papier de l'anssi [ici](https://www.ssi.gouv.fr/agence/publication/analyse-de-lamcache/ "ici")

Un tool pour le parser [ici](https://github.com/EricZimmerman/AmcacheParser  "ici"), fait par Eric ZIMMERMAN un des formateurs SANS, je recommande vraiment d'aller voir son travail.

### Parser le NTUSER.dat

Les ShellBags :

La pluspart des shellBags sont dans le NTUSER.DAT

>  Every time you make a change to the look and behavior of Windows and installed programs, whether that’s your desktop background, monitor resolution, or even which printer is the default, Windows needs to remember your preferences the next time it loads.

Petit  [article](https://www.sans.org/reading-room/whitepapers/forensics/windows-shellbag-forensics-in-depth-34545) SANS sur les shellBags

>  Microsoft Windows records the view preferences of folders and Desktop. Therefore, when the folder/Desktop is visited again, Windows can remember the location of the folder, view and positions of items. Microsoft Windows store the view preferences in the registry keys and values known as “ShellBags”.
>
> ShellBag information is crucial when forensicators need to know when and which folder a user accessed. For instance, when a company suspects an employee leaked a confidential document stored on the network, that employee’s computer may have the ShellBag information that demonstrates the folder containing that document was accessed shortly before the document was leaked. Furthermore, ShellBags may also show the folders or servers that employee should not access. 

###  Faire une FLS

La FLS permet, d'établir une timeline de modification des fichiers, à l'instar du parsing de la mft vu précédment.

TSK propose un tool [ici](https://wiki.sleuthkit.org/index.php?title=Fls "ici").

J'ai fait un tool en GUI pour Windows [ici](https://github.com/Xbloro/FLSGUI "ici")
([l'article](https://xbloro.github.io/tool/GUI-FLS-Tool/ "l'article"))

###  Jumplist

La jumpList contient les fichiers récent ouvert par des applications ou l'utilisateur , elle se trouve ici :

```
Nom de l'user +"\\AppData\\Roaming\\Microsoft\\Windows\\Recent"; 
```

###  Last Activity view

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
