# Bluetooth-Camera-Car
Voiture télécommandée par Bluetooth avec un petit laser et caméra amovible.
//https://www.aranacorp.com/fr/arduino-et-le-module-bluetooth-hc-06/

# I-	Assemblage de la voiture

  A)	Assemblage du kit
  

Utilisation d’un kit avec deux supports plastiques, quatre roues et moteurs, un support de pile et de la visserie. 	
Assemblage selon la figure ci-dessous.
  
![Image2](https://user-images.githubusercontent.com/92324336/139718660-0ff360cb-2888-408b-87ac-ae516c1c5b0f.png)


  B)	Assemblage supplémentaire
  
Placement du support de pile sur le devant de la voiture et d’un motorShield à l’arrière.
Ajout d’un support Lego pour déposer la caméra et d’autres éléments comme le laser.
Assemblage selon la figure ci-dessous.

![Image1](https://user-images.githubusercontent.com/92324336/139718817-14d26911-e2a1-4050-9552-0c2f32952d2a.jpg)


# II-	Initialisation des modules 

  A)	Module Bluetooth 
  
L’objectif n’est pas de connecter la voiture par wifi et de la contrôler sur Blynk, mais plutôt de privilégier une connexion par Bluetooth et de contrôler le véhicule avec une manette.
Pour cela, nous avons besoin d’un module HC-05 (maitre et esclave)et d’un module HC-06(esclave). Il est aussi possible d’utiliser deux modules HC-05.

  1) Configurer la communication
    
Pour commencer nous allons avoir besoin d’établir le lien entre les deux modules à l’aide d’un code Arduino. Ce code permet de modifier le code pin, le nom du module, sur quel module s’apparier…

Les branchements :
Pour configurer le module, les branchements ne sont pas les même qu’en communication :
Le module HC-05 présente 6 broches :
•	VCC à 5V
•	GND à la masse. 
•	RX broche de réception. Typiquement connecté à la broche de transmission (TX) de l’Arduino
•	TX broche de transmission. Typiquement connecté à la broche de réception (RX) de l’Arduino
•	Key ou EN doit être alimentée pour entrer dans le mode de configuration et ne doit pas être connecté pour être en mode communication.

Remarque : Le pin Tx du module va dans le pin Rx déclaré dans le code Arduino et inversement. Cela permet d’échanger des informations entre eux.

Pour le HC-06, il n’y a que 4 broches et le pin KEY n’y est pas. C’est normal car ses fonctionnalités s’arrete à être esclave.
Pour le module HC-06, on met le baudrate à 9600 et pour le HC-05 on le met à 38400.

On compile, le code d'exemple pour chacun en commencant par l’esclave :
On écrit dans le moniteur série.
•	AT retourne OK
•	AT+NAME=HC05-Slave
•	AT+UART=9600,0,0
•	AT+ROLE=0   // On ne peut pas modifier le rôle du HC-06
•	Entrez AT+ADDR pour obtenir l’adresse du module esclave (dans notre cas, +ADDR:98d3:32:21450e)
Nous remplirons les informations de l’adresse dans le module maitre pour entrer en communication
Pour le maitre :

•	AT retourne OK
•	AT+NAME=HC05-Master
•	AT+UART=9600,0,0
•	AT+ROLE=1
•	Vous devez enregistrer l’adresse du module esclave pour que le module maître puisse s’appairer: AT+BIND=98d3,32,21450e (remplacez les deux points « : » par des virgules « , »)
 
Lorsque les deux modules Bluetooth sont connectés, les lumières qu’ils émettent sont irrégulières.

  2) Entrer en communication

Lorsque les modules sont configurés, on peut débrancher la pin Key ou EN.
Pour le code master et esclave, nous utilisons la bibliothèque #include <SoftwareSerial.h> pour ajouter une communication série.
Pour chacun, on déclare les pins Rx et Tx que l’on créé.
Exemple : SoftwareSerial ArduinoMaster(12,14); On pensera à connecter ses pins sur l’inverse du module.

On démarre la communication dans le setup
Exemple ArduinoMaster.begin(9600);

L’esclave reçoit des informations du maitre donc dès que celui-ci reçoit une donnée, il la lit et la traite :
if (ArduinoMaster.available() >0) {
     char c = ArduinoMaster.read()
     
Le maître envoie des données selon des conditions:
   ArduinoSlave.print('A');
  

En résumé, l’Arduino envoie une information au module Bluetooth master issus de la manette grâce aux ports Rx Tx créés. Les deux modules étant configurés pour communiquer, le module slave reçoit l’information et la partage à son tour à la Nodemcu grâce au port Rx Tx créés également.


B)	Connection du MotorShied 

  1)	Les moteurs
    
Nous possédons 4 moteurs mais uniquement deux ponts en H pour contrôler la vitesse et le sens des moteurs.
Nous allons donc devoir dans le motorShield, brancher dans un emplacement deux moteurs.
Pour pouvoir faire tourner le véhicule à gauche et a droite, les moteurs doivent être assemblé deux à deux du même côté.
Pour alimenter les moteurs, on utilise un boitier de pile 6V que l’on branche selon l’image suivante :


![Image3](https://user-images.githubusercontent.com/92324336/139718881-2b7898d0-e0a1-4e5e-aad1-11aacb46a1c8.jpg)

 
  2)	Les modules ajoutés

Le module Bluetooth lié au MotorShield (l’esclave) est branché à la pin VIN pour avoir suffisamment de puissance, au ground et à un pin digital
Le laser est branché au 3.3V et au ground.
Les servomoteurs branchés chacun à un pin digital PWM, au Ground et à un VCC

  3)	La manette
  
La manette est un Shield connecté directement à un Arduino et permet d’avoir directement un joystick et des boutons utilisables.
Un module Bluetooth maitre est branché dessus et en fonction des valeurs de la manette, envoie des informations au module du motorShield 
Pour connaitre les pins : https://www.google.com/search?q=joystick+shield+pinout&rlz=1C1CHBF_frFR911FR911&sxsrf=ALeKk02XzmC-Yngf9aDbZE6Xqd2bFflECQ%3A1623847269479&ei=ZfHJYN3gHISMa8f1gagO&oq=joystick+shield+&gs_lcp=Cgdnd3Mtd2l6EAEYBzIECCMQJzIECCMQJzIECCMQJzICCAAyAggAMgUIABDLATIFCAAQywEyBQgAEMsBMgUIABDLATIFCAAQywFQripYripg0EdoAHABeACAAWmIAYwCkgEDMi4xmAEAoAEBqgEHZ3dzLXdpesABAQ&sclient=gws-wiz 


 ![Image4](https://user-images.githubusercontent.com/92324336/139719004-9611d1cf-ad13-4df8-b5bd-ea5e005d1385.jpg)


# III-	Conception informatique

  A)	Envoie des données module maitre
  
On déclare les pins liés aux boutons et au joystick ainsi que des variables pour récupérer les informations.

  1)	Déplacement par Joystick ou appuie d’un bouton

En fonction de la position du joystick, on envoie une lettre au module esclave avec la fonction ArduinoSlave.print.
De même pour l’appui d’un bouton.

  2)	Bouton Laser

Le bouton du laser est très sensible donc s’il l’on clique une fois dessus, il considéra avoir été appuyé une vingtaine de fois. 
Pour pallier ce problème, une information sera envoyée si le bouton est appuyé et qu’un certain temps s’est écoulé.
De même, une autre information sera envoyée si le bouton est de nouveau longtemps pressé.


B)	Réception des données module esclave

Tous les éléments sont contrôlés par le module esclave qui reçoit les information du master. 
On regarde tout d’abord si le port reçoit une information puis on réalise un switch pour chaque élément activable :

  1)	Déclenchement des moteurs

On déclare tous les pins utilisés pour la direction des moteurs et leur vitesse.
Exemple: 
case 'A':
    digitalWrite(RightMotorDir,HIGH); 
    digitalWrite(LeftMotorDir,HIGH);              
    digitalWrite(RightMotorSpeed,HIGH);
    digitalWrite(LeftMotorSpeed,HIGH); 
    break;

  2)	Déclenchement du laser

Le laser s’active simplement :
case 'N':
         digitalWrite(led,LOW);
         break;


  3)	Déclenchement des servomoteurs
On déclare tout d’abord sur quel pin sont connectés les servomoteurs ainsi que deux valeurs qui seront leur position,
puis à chaque fois que le bouton est détecté comme appuyé, on déplace le servomoteur et on incrémente ou décrémente leur valeur.

![Image5](https://user-images.githubusercontent.com/92324336/139719094-c4b0a5d5-fff3-431a-8349-de97b54ccbb1.gif)

# IV-	Caméra ESP32

a)	Modélisation du support

La caméra vient se rajouter devant la voiture. C’est un module qui ne se connecte pas directement à l’Arduino mais dont les images sont retransmises sur une adresse IP et que l’on peut donc visualiser sur téléphone.
L’objectif est d’obtenir un support permettant à la caméra de tourner à gauche et à droite ainsi que de se lever ou baisser.

 ![Image6](https://user-images.githubusercontent.com/92324336/139719058-3ed2ca72-dc3b-4ebb-b729-651a8f5118da.png)


Le 1er support sert à placer la caméra à l’intérieur selon les dimensions de la caméra. Un servomoteur est collé sur le côté droit pour pouvoir lever ou baisser la caméra. Le trou du 2nd support permet de maintenir en place ce servomoteur.
Le 2nd support sert également à faire tourner la caméra. Un autre servomoteur est placé en dessous.

![Image7 (1)](https://user-images.githubusercontent.com/92324336/139719375-81103dfe-122c-49fb-989f-257e91e0d35a.gif)




 















