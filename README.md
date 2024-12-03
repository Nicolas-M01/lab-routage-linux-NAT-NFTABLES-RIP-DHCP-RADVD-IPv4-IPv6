## Lab d'une infrastructure réseau implémentant routage sous Linux, NAT avec nftables, RIP, DHCP, radvd, IPv4 et IPv6 

## Sommaire
- [Licence](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#licence-)
- [Description du projet](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#description-du-projet-)
- [Etat d'avancement du projet](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#etat-davancement-du-projet-)
- [Schéma synoptique du lab](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#sch%C3%A9ma-synoptique-du-lab-)
- [Outils du lab](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#outils-du-lab-)
- [Spécifications techniques du lab](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#sp%C3%A9cifications-techniques-du-lab-)
- [Étape par étape](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#%C3%A9tape-par-%C3%A9tape-)
- [FAQ](https://github.com/rikiya-gabimaru/lab-routage-linux-NAT-NFTABLES-RIP-DHCP-RADVD-IPv4-IPv6/blob/main/README.md#faq-)

## Licence :
- Cette documentation est sous licence [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.fr)

## Description du projet :
Mise en place d'une infrastructure système et réseau composée des éléments suivants :
- Un routeur de bordure de zone sous Linux permettant aux réseaux locaux d'atteindre internet.
- Un routeur inter-LANs sous Linux permettant aux différents LANs de communiquer entre eux selon la politique de contrôle de flux.
- Les protocoles suivant seront implémentés (NAT, RIP, DHCP, IPv4, IPv6).
- Différents clients seront répartis dans les LANs.

## Etat d'avancement du projet :
- [x] Information sur les spécifications techniques du lab
- [x] Activation du routage sur R-EDGE et R-INTER
- [x] Configuration des régles NAT avec nftables sur R-EDGE
- [x] Configuration des interfaces réseaux sur R-EDGE
- [x] Configuration des interfaces réseaux sur R-INTER
- [x] Configuration des routes sur R-EDGE
- [ ] Installation et configuration du protocole RIP
- [ ] Installation et configuration d'un serveur DHCP (R-INTER)
- [ ] Installation et configuration de radvd (R-INTER)
- [ ] Configuration IPv6

## Schéma synoptique du lab :

<p align="center">
  <img src="https://github.com/user-attachments/assets/5bbe9964-2f54-4747-979a-a7e1e963e271">
</p>

## Outils du lab :

**[VirtualBox (7.0.22)](https://www.virtualbox.org/)**

**[drawio-desktop (v24.7.17)](https://github.com/jgraph/drawio-desktop)**

**[debian (12.8.0)](https://www.debian.org/)**

**[Windows 10](https://www.microsoft.com/en-us/software-download)**

## Spécifications techniques du lab :

  - **Router R-EDGE (Linux Debian 12.8.0) :**
	- 1 CPU (par défaut)
  	- RAM (entre 512 Mo et 1024 Mo suffisant)
	- ROM (par défaut)
	- 2 cartes réseaux (l'une en Bridge ou en NAT, l'autre en réseau interne)
  - **Router R-INTER (Linux Debian 12.8.0) :**
	- CPU (par défaut)
  	- RAM (entre 512 Mo et 1024 Mo suffisant)
	- ROM (par défaut)
	- 1 carte réseau (en réseau interne)
  - **Clients (Windows 10 Pro)**
	- 1 CPU (par défaut)
  	- RAM (2048 Mo)
	- ROM (50 Go)
	- 1 carte réseau (en réseau interne)

## Étape par étape : 

**1 - Installation des machines dans l'environnement virtualisé :**

   - (Non documenté, voir les spécifications techniques du lab)

**2 - Activer la fonction routage sur R-EDGE et R-INTER de façon persistante :**

   - Dans `/etc/sysctl.conf`, décommenter les lignes suivantes :
	`net.ipv4.ip_forward=1` et `net.ipv6.conf.all.forwarding=1`

<p align="center">
  <img src="https://github.com/user-attachments/assets/8ffb96b9-5d8e-44f1-b549-e3364b54dbf9">
</p>

- Quitter le fichier en sauvegardant : `Ctrl+x` puis `y`
- Charger la nouvelle configurration avec la commande : `sysctl -p`

**3 - Configuration des régles NAT avec nftables sur R-EDGE :**

   - Création de la table pour le NAT (qu'on va appeler "table_NAT") :  
	`nft add table ip table_NAT`
   - Création d'une chaine pour la table du NAT (qu'on va appeler "chain_postrouting") :  
	`nft add chain ip table_NAT chain_postrouting { type nat hook postrouting priority 0 \; }`
   - Création d'une règle pour la chaine de la table du NAT (on va ajouter en 2 règles 4 LANs pour les nater) :  
	`nft add rule table_NAT chain_postrouting ip saddr 10.0.99.0/30 oif enp0s3 snat <mettez entre ces balises l'IP de sortie>`  
	`nft add rule table_NAT chain_postrouting ip saddr 10.0.0.0/22 oif enp0s3 snat <mettez entre ces balises l'IP de sortie>`
   - Sauvegarder cette table pour le NAT :  
     	`nft list table ip table_NAT > table_NAT.nft`
<p align="center">
 		<img src="https://github.com/user-attachments/assets/2578fa0f-6907-4fd5-babc-bd441abaf380">
</p>

**4 - Configuration des interfaces réseaux sur R-EDGE et activation de la règle NAT au démarrage :**

- Dans le fichier `nano /etc/network/interfaces` de R-EDGE, chargement de la règle NAT et configuration de la 2ème carte réseau :
<p align="center">
 		<img src="https://github.com/user-attachments/assets/162d7f62-f729-4000-a9eb-fa454f2ceeca">
</p>

**5 - Configuration des interfaces réseaux sur R-INTER :**

- Dans le fichier `nano /etc/network/interfaces` de R-INTER, configuration des cartes réseaux :
	<p align="center">
 		<img src="https://github.com/user-attachments/assets/fc529b95-50fa-43e2-a0d8-56f601799a45">
	</p>

 `To be continued...`

## FAQ :
	- T'es de la police ? Rien à déclarer !
