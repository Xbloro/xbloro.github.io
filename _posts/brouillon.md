  1.1 Dump de la R.A.M et récupération d'informations machine
  ---

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

![alt text](/assets/images/menuCollecteur.png "ChoixMenuRedline")

Dans chaque cas Redline vous propose d'éditer le script du payload, pour ajouter des fonctions (récupération de string par ex) mais ici on va le laisser par défaut.

On va choisir l'option numéro 2 : "Comprehensive collector" 


![alt text](/assets/images/collecteur2.png "ChoixN2")  



Une deuxième fenêtre nous demande si on veut récupérer la mémoire de la machine, on coche la case (n1)  
Puis on choisit l'emplacement de sauvegarde du Payload(n2) >> votre média Amovible.  

<img src="/assets/images/Collecteur3.png" width="600">




Et voila ! votre payload est prêt.  

![alt text](/assets/images/payload1.png "Payload")

A présent, on branche le média amovible sur le Windows cible et on exécute le script "RunRedlineAudit.bat".

Une fois l'opération terminée, un fichier "AnalysisSessionX.mans " est créé dans le répertoire de votre payload.  
Il suffit juste de l'ouvrir avec RedLine (sur votre machine d'analyse ^^') pour pouvoir commencer à travailler.  
Mais doucement ! L'acquisition n'est pas encore terminée, il faut maintenant faire une copie forensique de tous les disques de stockage de la machine.  