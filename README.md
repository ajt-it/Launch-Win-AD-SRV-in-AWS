![Licence GPL-3.0](https://img.shields.io/badge/Licence-GPL_3.0-red)

# Launch-Win-AD-SRV-in-AWS

Pouquoi déployer Active Directoy sur AWS?

La gestion des identités, et des droits associés, est une partie sensible et essentielle de l’infrastructure IT des entreprises, 
et Microsoft Active Directory est devenu une brique incontournable au sein des entreprises ayant un environnement Microsoft.
Il devient donc nécessaire de proposer aux entreprises des solutions innovantes et flexibles pour leur développement.


## Représentation de l'infrastructure

![INFRA](https://user-images.githubusercontent.com/46109209/184464704-bf011056-c87b-4601-9345-e8908783a704.png)

N.B. Le choix des ISP (Internet Service Provider) ou FAI (Fournisseur d'Accès Internet) est personnel et n'est l'objet d'aucune entente. 


## Langages


 - :white_check_mark: YAML

 - :white_check_mark: PowerShell


## Pré-requis

  
  * Un compte AWS;
  * Une connexion internet;
  * une addresse 'IP Public';  
  * Avoir des bases en Linux, YAML & PowerShell.


## Objectifs

À l'aide d'un stack (.yaml) CloudFormation, mettre en place d'un serveur Windows ADDS sur AWS en utilisant :

  - EC2 (t2.micro) pour le serveur AD;
  - CloudFormation pour automatiser la création de l’infrastructure

Mettre en place des liaisons VPN entre des serveurs VPN locaux et le sous-réseau privé AWS pour se connecter au serveur Windows de manière sécurisée depuis divers sites.


## Technologies

Liste des technologies AWS & autres utilisées :

- aws EC2
- aws VPC
- aws CloudFormation
- Windows 10
- Windows Server 2019
- Linux Ubuntu 20.04


## Ressources

- https://www.google.com/
- https://www.neptunet.fr/
- https://www.it-connect.fr/
- https://www.rdr-it.com/
- https://docs.microsoft.com/
- https://docs.aws.amazon.com/

## Limites & Piste de solution

Avoir, un service ADDS multisites présente de nombreux avantages. Cependant, il y'a des limites par exemple:

 - la connexion internet;
 - les coûts associés à l'infrastructure AWS.

Ici, nous ferons état du fait de disposer d'une connexion internet fiable à un coût non prohibitif.

Un moyen de palier à l'indisponibilité du service internet et par conséquent à la coupure de la liaison entre deux sites serait de mettre en place sur les divers sites loxaux un serveur RODC (Read Only Domain Controller).

Le serveur RODC ne servira qu’à authentifier les postes et les utilisateurs et appliquera également les stratégies de groupe.
Comme la base de données Active Directory ne sera pas modifiable, même s’il est compromis, il ne sera pas possible d’en prendre le contrôle en créant un nouvel utilisateur administrateur par exemple et ceci n’impactera pas le serveur AD principal de l’entreprise.


## Configuration du serveur Linux Ubuntu 20.04

Cette configuration variera très peu selon les sites.

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
### ~$ sudo iptables –t nat –A POSTROUTING –o enp0s8 –j MASQUERADE

![image](https://user-images.githubusercontent.com/46109209/179643583-77709aba-da6e-4020-8ed6-fbd749e78e7c.png)

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

Editez le fichier /etc/ipsec.conf, comme ci-dessous et ajoutez “votre_ip_publique”. Le reste de la configuration doit correspondre à votre réseau :
### ~$ sudo nano /etc/ipsec.conf
![1-copy](https://user-images.githubusercontent.com/46109209/184720935-8437dc41-d567-4eeb-9e65-ee348fb9c154.png)

N.B. Attention Strongswan est très susceptible avec ce fichier, toutes les lignes écrites après “conn nom_de_votre_liaison” doivent obligatoirement commencer        par une tabulation sinon le démarrage d’ipsec sera en erreur.

Éditez le fichier /etc/ipsec.secrets comme suit 

![2-copy](https://user-images.githubusercontent.com/46109209/184721012-8570fd92-c9cc-4c02-a102-a453a9315638.png)


## Mise en place de l'infrastructure dans AWS

Il est temps d'interconnecter notre LAN à AWS. Nous avons créé un fichier 'yaml' pour stack CloudFormation; nous pourrons ainsi monter notre Infrastructure As a Code.
Il ne restera qu’à personnaliser les champs et configurer votre VPN côté LAN. 

Une fois que le ficher 'yaml' aura été exécuté, rendez-vous dans la console AWS, retournez sur le service VPC, et dans le menu “Connexions VPN site à site“. Sélectionnez la liaison VPN que nous avons créée et cliquer sur le bouton “Download configuration“.

![4-copy](https://user-images.githubusercontent.com/46109209/184723903-61124897-cfbb-4789-b1ef-3810b0bc9f5e.png)

Sauf si vous avez du matériel bien spécifique, sélectionnez le fournisseur et la plateforme “Generic“. Cliquez sur “Download” pour obtenir le fichier contenant toute les informations dont nous aurons besoin pour monter le VPN côté réseau local.

![pnvvvvv](https://user-images.githubusercontent.com/46109209/183506897-57207c9c-d454-4df3-a606-4820caeb9aa8.png)

Le fichier se compose de 2 parties car AWS créé pour nous 2 tunnels VPN différents, chacun ayant sa propre adresse IP publique et sa propre clé privée.
Pourquoi deux tunnels? La redondance.

![b](https://user-images.githubusercontent.com/46109209/183508671-29059385-2fe7-4d97-9ee4-fe4dd2936ab6.png)

![c](https://user-images.githubusercontent.com/46109209/183508716-99bba3be-e699-48bd-a9b5-4f247a8f06cd.png)

![183508815-287dd636-b20c-4cd0-a031-9a0e72a04699](https://user-images.githubusercontent.com/46109209/184721663-1c323219-8912-443c-8b11-2c873cdc3779.png)


### Finalisation de la configuration de VPN local:

Vous l'avez sans doute remarqué, les données de ce fichier vont permettre d'éditer le fichiers 'ipsec.conf' et 'ipsec.secrets'.
Une fois les ajustements de configuration effectués, il est temps de démarrer le VPN local (serveur Linux Ubuntu).
Placez-vous dans le terminal et lancez les commandes suivantes:

Redémarrage d'IPsec :
### ~$ sudo ipsec restart
![183511938-0a0827cb-83db-4c07-b648-4e35db5ba2e8](https://user-images.githubusercontent.com/46109209/184722722-15774ccd-27c3-4f37-b1dc-5fa212d75130.png)

Démarrage des liaisons VPN :
### ~$ sudo ipsec up lan-to-aws-1
### ~$ sudo ipsec up lan-to-aws-2

![3](https://user-images.githubusercontent.com/46109209/184725907-964d9d5f-daf8-4876-bb06-479815857d07.png)


Afin de confirmer que notre serveur VPN communique avec l'instance dans le cloud, faisons un PING!

![e](https://user-images.githubusercontent.com/46109209/183512958-103a9a8b-332f-4034-999d-30c541ecc6fe.png)


## Configuration réseau des machines du site local

Effectuez la configuration de l'interface réseau des postes comme suit:

![f](https://user-images.githubusercontent.com/46109209/183515803-0258a76c-cc61-408e-97c9-1cb1cf3fa4be.png)

Un test de PING permettra de s'assurer que la machine du réseau local de l'entreprise communique avec le serveur 
Windows dans le cloud :

![183516673-4b345d65-6d2e-4f3a-860f-c8fe705d2b0d](https://user-images.githubusercontent.com/46109209/183642222-3c202361-2722-4425-9e1f-0f1fe1f62235.png)

Nous allons joindre un ordinateur du réseau local au domaine (it.pro) et se connecter grâce à un utilisateur 
autorisé du domaine (voir stack CloudFormation)

Avant de procédeer, nous recommendons de désactiver le pare-feu de Microsoft Defender

https://support.microsoft.com/fr-fr/windows/activer-ou-d%C3%A9sactiver-le-pare-feu-de-microsoft-defender-ec0844f7-aebd-0583-67fe-601ecf5d774f

![ee](https://user-images.githubusercontent.com/46109209/183519139-b523a966-3f85-420b-bc54-a935a82d68df.png)

![ff](https://user-images.githubusercontent.com/46109209/183519140-0798c48a-11f3-44e6-ac33-9f246d3761f8.png)

![gg](https://user-images.githubusercontent.com/46109209/183519184-968537ac-bc69-4f5c-b722-49878ada8478.png)


Un redémarrage sera effectué pour prendre en compte les modification. Vous pouvez maintenant vous connecter
au doamine:

![dddddddd](https://user-images.githubusercontent.com/46109209/183524748-a5c7c1eb-286c-46e2-9ad4-248fbdedb952.png)

![k](https://user-images.githubusercontent.com/46109209/183519679-efa8d48e-1f9b-40b6-bac0-2aecda15e63f.png)

![vfrtg](https://user-images.githubusercontent.com/46109209/183519994-ef26a253-4a32-4915-bc6f-f566a4c64650.png)

Test de connexion au lecteur réseau du serveur 'Active Directory' dans AWS depuis le réseau local

![test](https://user-images.githubusercontent.com/46109209/183522123-97ad13fd-0bb2-4b8e-81c3-e93bea22ee16.png)


## Connexion à l'instance (Windows Server 2019) dans AWS

Le mode RDP (Remote Desktop Protocol) semble être le moyen le plus accommodant pour se connecter à son instanse Windows dans AWS.

![1a](https://user-images.githubusercontent.com/46109209/183527126-cd16e429-f7db-482a-b969-fbad8f0f1fe1.png)

![1b](https://user-images.githubusercontent.com/46109209/183527169-f0a3a918-a652-40f1-9835-104199e0a99f.png)

![aa](https://user-images.githubusercontent.com/46109209/183527327-9add712e-01c2-4892-9eb7-7c578c3e6c08.png)

![1d](https://user-images.githubusercontent.com/46109209/183527385-31c7c6fc-d2f3-439a-9139-922977f5888e.png)

Vérifions à présent si ADDSForest est bien installé :

![bb](https://user-images.githubusercontent.com/46109209/183529818-32ec6238-891a-4edf-972b-0af107610e1f.png)

![cc](https://user-images.githubusercontent.com/46109209/183529822-1b242a39-eb16-499f-8087-cc7d07f7c996.png)

![dd](https://user-images.githubusercontent.com/46109209/183529853-dc3fa493-3552-4d99-9307-3584f87cd443.png)

N.B. Une bonne pratique est de renommer le serveur ADDS. En utilisant, un compte "administrateur" du domaine; lancez la commande : 
### C:\Users\Administrator> Rename-Computer -NewName "New_Computer_Name"
