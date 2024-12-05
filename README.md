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
- [x] Configuration du routage dynamique
- [x] Implémentation et configuration du protocole RIP avec FRRouting
- [ ] Installation et configuration d'un serveur DHCP (R-INTER)
- [ ] Installation et configuration de radvd (R-INTER)
- [ ] Configuration IPv6

## Schéma synoptique du lab :

<p align="center">
  <img src="https://github.com/user-attachments/assets/5bbe9964-2f54-4747-979a-a7e1e963e271">
</p>

## Outils du lab :

[VirtualBox (7.0.22)](https://www.virtualbox.org/)  
[drawio-desktop (v24.7.17)](https://github.com/jgraph/drawio-desktop)  
[debian (12.8.0)](https://www.debian.org/)  
[nftables](https://nftables.org/)  
[FRRouting Project](https://frrouting.org/)  
[Windows 10](https://www.microsoft.com/en-us/software-download)

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
  <img src="https://github.com/user-attachments/assets/a0cb14f9-0cca-496f-b3d5-2e4edb0abbbc">
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
 		<img src="https://github.com/user-attachments/assets/e79556b0-d306-49f3-9c3a-28cd66f75bcc">
</p>

**4 - Configuration des interfaces réseaux sur R-EDGE et activation de la règle NAT au démarrage :**

- Dans le fichier `nano /etc/network/interfaces` de R-EDGE, chargement de la règle NAT et configuration de la 2ème carte réseau :
<p align="center">
 		<img src="https://github.com/user-attachments/assets/f2ad6496-2bf2-43a9-b848-3ae4e4ddcb97">
</p>

**5 - Configuration des interfaces réseaux sur R-INTER :**  
- Dans le fichier `nano /etc/network/interfaces` de R-INTER, configuration des cartes réseaux :
 
<p align="center">
<img src="https://github.com/user-attachments/assets/f19d8fd0-39de-407f-9277-5d4159c9cf2b">
</p>

  **6 - Implémentation et configuration du protocole RIP avec FRRouting dans R-INTER :**  
- Installation de l'outil FRRouting avec la commande : `apt install frr`
- Dans le fichier `/etc/frr/daemons` activer le protocole RIP : `ripd=yes`
- Dans le fichier `/etc/frr/frr.conf` configurer le protocole RIP puis redémarrer le service frr.service avec la commande : `systemctl restart frr.service` :
<p align="center">
<img src="https://github.com/user-attachments/assets/6d08967a-a5ac-4933-bcf5-253f8ac44367">
</p>
 
 **7 - Implémentation et configuration du protocole RIP avec FRRouting dans R-EDGE :**  
- Installation de l'outil FRRouting avec la commande : `apt install frr`
- Dans le fichier `/etc/frr/daemons` activer le protocole RIP : `ripd=yes`
- Dans le fichier `/etc/frr/frr.conf` configurer le protocole RIP puis redémarrer le service frr.service avec la commande : `systemctl restart frr.service` :
<p align="center">
<img src="https://github.com/user-attachments/assets/1d938c23-fa77-42f2-8a03-c5752f081d1c">
</p>

 **8 - Vérification du partage des routes de R-INTER à R-EDGE**

 - Vérication directe depuis R-EDGE : `ip route`
 - Vérification depuis le shell de configuration de FRRouting : `show ip rip` ou en dehors du shell `vtysh -c "show ip rip"`
 - Vérification depuis le shell de configuration de FRRouting des routers voisins : `show ip neigh` ou en dehors du shell `vtysh -c "show ip neigh"`

 `To be continued...`

## FAQ :
	- Hey toi là ! Mot de passe ?
 	- Un pantin voulait acheté une trompette à un lama mais le lama ne comprenait pas ce que disait le pantin. Il demanda au pantin de parler un peu plus fort. Le lama lui donna alors une citrouille. Mais le pantin voulait une trompette et le répéta au lama. Le lama lui donna une statue. Le pantin devint complétement fou, il insulta le lama et le frappa. Maintenant le lama est mort.
  	- C'est quoi ça ?
   	- Bah c'est le mot de passe.
