![Licence GPL-3.0](https://img.shields.io/badge/Licence-GPL_3.0-red)

# Launch-Win-AD-SRV-in-AWS



## Représentation de l'infrastructure

![INFRA](https://user-images.githubusercontent.com/46109209/183467799-d3e88133-aca2-4d43-acfc-fc2d6f58cfbf.png)


## Langages


 - :white_check_mark: YAML

 - :white_check_mark: SHELL



## Pré-requis

  * Avoir des bases solides en Linux;
  * Un serveur Linux Ubuntu 20.04 (Disposer des droits 'root'sur le serveur Linux);
  * Un compte AWS;
  * Un ordinateur Windows 10;
  * Un serveur Windows 2016 (ou 2019);
  * Une connexion internet;
  * une addresse 'IP Public'.


## Configuration du serveur Linux Ubuntu 20.04

Commencez en attribuant une adresse IP statique à votre serveur. Ubuntu Server utilise « netplan » pour la gestion du réseau.
Votre configuration réseau ressemblera à ceci :

### ~$ sudo nano /etc/netplan/00-installer-config.yaml

![g1](https://user-images.githubusercontent.com/46109209/183467332-44c3da57-6c8a-40eb-a66d-2bf2a3906bee.png)

N.B. Ne pas utiliser de tabulations dans « netplan » mais des espaces !

Appliquer la configuration avec la commande :
### ~$ sudo netplan apply


Vérifiez si l'heure de votre serveur synchronise avec un serveur Internet.
### ~$ sudo timedatectl


Procédez à la mise à jour du système, en vous plaçant dans le terminal et frappez : 
### ~$ sudo apt –y update && apt –y upgrade


Modifiez le nom d'hôte (hostname), mettez à jour le fichier « hosts » et identifiez dans « resolv.conf » le serveur DNS à utiliser pour résoudre le nom de domaine
### ~$ sudo nano /etc/hostname
![hostname](https://user-images.githubusercontent.com/46109209/183491231-0e5b19d5-e38c-492d-a3cf-22570de30874.png)


### ~$ sudo nano /etc/hosts
![hosts](https://user-images.githubusercontent.com/46109209/183491232-4195a645-b802-4f78-ba3a-1a11155ff862.png)

 
### ~$ sudo nano /etc/resolv.conf
![resolv](https://user-images.githubusercontent.com/46109209/183492830-cd0a054c-de5d-44c5-882b-3c1f393b2545.png)


À présent, il faut activer l’ « IP forwarding ». Le « forwarding » est la capacité pour un système d'exploitation d'accepter les paquets réseau entrants sur une interface, de reconnaître qu'ils ne sont pas destinés au système lui-même, mais qu'ils doivent être transmis à un autre réseau, puis de les transférer en conséquence. 
### ~$ sudo nano /etc/sysctl.conf
![image](https://user-images.githubusercontent.com/46109209/179643061-02ab35c0-8519-41c3-945d-f97eb7390367.png)
 

Téléchargez et installez ensuite le paquet « netfilter-pesistent », « iptables-persistent » et « ipables » avec les commandes suivantes :
### ~$ sudo apt-get install netfilter-persistent
### ~$ sudo apt-get install iptables-persistent
![image](https://user-images.githubusercontent.com/46109209/179643292-6f26abb2-ed40-4c93-9c43-6d77237692aa.png)

![image](https://user-images.githubusercontent.com/46109209/179643339-edfc2859-f461-4bfc-bbb9-542eecb08bbb.png)

### ~$ sudo apt-get install iptables

Activez « netfilter-persistent » avec :
### ~$ sudo service netfilter-persistent start


Vous pourrez à présent établir vos règles de routage. Voici la règle routage NAT que nous mettons en place sur les interfaces « enp0s3 » et « enp0s8 » avec la commande suivante :
### ~$ sudo iptables –t nat –A POSTROUTING –o enp0s3 –j MASQUERADE
![image](https://user-images.githubusercontent.com/46109209/179643583-77709aba-da6e-4020-8ed6-fbd749e78e7c.png)

### ~$ sudo iptables –t nat –A POSTROUTING –o enp0s3 –j MASQUERADE

Enregistrez les règles avec la commande ci-dessous:
### ~$ sudo iptables-save


La règle NAT est maintenant enregistrée de manière temporaire. Rendez-la permanente avec la commande : 
### ~$ sudo iptables-save > /etc/iptables/rules.v4

On pourra constater la présence de la règle de NAT avec la commande :
### ~$ sudo iptables –t nat –L –n -v
![iptables](https://user-images.githubusercontent.com/46109209/183494672-9a20a8b2-c19d-4d4f-9b1d-0fb872c54637.png)

Effectuez un redémarrage du système pour prendre en compte les changements effectués.
### ~$ sudo reboot

Après le redémarrage, on procède à l'installation de 'strongSwan' sur notre serveur VPN local avec la commande suivante :
### ~$ sudo apt -y install strongswan

Mise en place de la liaison VPN à travers l'édition de deux fichiers : 'ipsec.conf' et 'ipsec.secrets'

Editez le fichier /etc/ipsec.conf, comme c-dessous et ajoutez “votre_ip_publique”. Le reste de la configuration doit correspondre à votre réseau :
### ~$ sudo nano /etc/ipsec.conf
![5](https://user-images.githubusercontent.com/46109209/183499039-70253be5-c118-4adc-a721-7498b8d0de36.png)

Info ++ : Attention Strongswan est très susceptible avec ce fichier, toutes les lignes écrites après “conn nom_de_votre_liaison” doivent obligatoirement commencer par une tabulation sinon le démarrage d’ipsec sera en erreur.

Éditez le fichier /etc/ipsec.secrets comme suit 
![6](https://user-images.githubusercontent.com/46109209/183501050-fd192a38-45d7-40d3-80ea-7ee6598b2903.png)

Il est temps d'interconnecter notre LAN à AWS. Nous avons créé un fichier 'yaml' pour stack CloudFormation; nous pourrons ainsi monter notre Infra As a Code.
Il ne restera qu’à personnaliser les champs et configurer votre VPN côté LAN. 


Une fois que le ficher 'yaml' aura été exécuté, rendez-vous dans la console AWS, retournez sur le service VPC, et dans le menu “Connexions VPN site à site“. Sélectionnez la liaison VPN que nous avons créée et cliquer sur le bouton “Download configuration“.

![vpnnn](https://user-images.githubusercontent.com/46109209/183505733-8d10ce7c-a795-4e42-8dc2-07d7a58c1a18.png)

Sauf si vous avez du matériel bien spécifique, sélectionnez le fournisseur et la plateforme “Generic“. Cliquez sur “Download” pour obtenir le fichier contenant toute les informations dont nous aurons besoin pour monter le VPN côté réseau local.

![pnvvvvv](https://user-images.githubusercontent.com/46109209/183506897-57207c9c-d454-4df3-a606-4820caeb9aa8.png)

Le fichier se compose de 2 parties car AWS créé pour nous 2 tunnels VPN différents, chacun ayant sa propre adresse IP publique et sa propre clé privée.
Pourquoi deux tunnels? La redondance.

![b](https://user-images.githubusercontent.com/46109209/183508671-29059385-2fe7-4d97-9ee4-fe4dd2936ab6.png)

![c](https://user-images.githubusercontent.com/46109209/183508716-99bba3be-e699-48bd-a9b5-4f247a8f06cd.png)

![d](https://user-images.githubusercontent.com/46109209/183508815-287dd636-b20c-4cd0-a031-9a0e72a04699.png)

Vous l'avez sans doute remarqué, les données de ce fichier vont permettre d'éditer le fichiers 'ipsec.conf' et 'ipsec.secrets'.
Une fois les ajustements de configuration effectués, il est temps de démarrer le VPN local (serveur Linux Ubuntu).
Placez-vous dans le terminal et lancez les commandes suivantes:

### ~$ sudo ipsec restart

### ~$ sudo ipsec up lan-to-aws

Vérifier l'état de la connexion :

### ~$ sudo ipsec status          

![7](https://user-images.githubusercontent.com/46109209/183511938-0a0827cb-83db-4c07-b648-4e35db5ba2e8.png)

Afin de confirmer que notre serveur VPN communique avec l'instance dans le cloud, faisons un PING!

![e](https://user-images.githubusercontent.com/46109209/183512958-103a9a8b-332f-4034-999d-30c541ecc6fe.png)





---


Joignez un ordinateur au domaine et connectez-vous à l'utilisateur du domaine

Configurez comme suit l'interface réseau de votre PC Windows
![image](https://user-images.githubusercontent.com/46109209/179756143-b2b5c865-5c75-4bf9-a35e-482f2912cc5c.png)
![image](https://user-images.githubusercontent.com/46109209/179645537-c5790667-a72b-424d-abea-e6db51772f0e.png)
![image](https://user-images.githubusercontent.com/46109209/179756451-6351f2a3-c03d-42f9-86ab-86fa3d9314e2.png)

![image](https://user-images.githubusercontent.com/46109209/179756574-3df64aff-9fa1-404f-ac78-c4aafa5e0c61.png)
![image](https://user-images.githubusercontent.com/46109209/179756601-656464a3-49d3-44c0-a307-44b7d51d506c.png)
