### Installation d’une VM OpenBSD

Ce document explique les étapes de base pour installer OpenBSD sur une machine virtuelle. Il couvre la préparation de la VM, le téléchargement de l’ISO, l’installation du système et les premières configurations post-installation.

---

#### Sommaire

1. Prérequis
2. Téléchargement de l’ISO OpenBSD
3. Création de la VM
4. Paramétrage du Boot et Installation
5. Configurations Initiales Post-Installation
6. Mise à jour du Système
7. Ressources

---

#### 1. Prérequis

- Un hyperviseur installé :
  - VirtualBox, VMware Workstation/Fusion, QEMU, ou autre.
- Une connexion internet pour télécharger l’ISO d’OpenBSD.
- Au minimum 1 Go de RAM et 8 à 10 Go d’espace disque alloué pour la VM (pour un environnement minimal).

---

#### 2. Téléchargement de l’ISO OpenBSD

Rendez-vous sur le site officiel OpenBSD :
https://www.openbsd.org/ftp.html

Choisissez un miroir (mirroir proche de votre localisation) et téléchargez l’image d’installation. Par exemple, pour OpenBSD 7.6 (à adapter selon la version actuelle), vous trouverez :

- `installXX.iso` (où XX correspond à la version, par exemple `install76.iso`).

Enregistrez ce fichier sur votre machine hôte.

---

#### 3. Création de la VM

Exemple avec VirtualBox (les étapes sont similaires avec d’autres hyperviseurs) :

1. Ouvrez VirtualBox et cliquez sur **Nouvelle**.
2. Donnez un nom à la VM, par exemple « OpenBSD ».
3. Type : **BSD**, Version : **OpenBSD (64-bit)**.
4. Assignez la quantité de RAM (1 Go recommandé).
5. Créez un disque dur virtuel (VDI ou VMDK) d’au moins 8 Go.
6. Une fois la VM créée, allez dans les paramètres :
   - Dans **Système** : vérifier que PAE/NX est activé.
   - Dans **Stockage** : attachez l’ISO d’installation d’OpenBSD sur le lecteur CD virtuel.
   - Dans **Réseau** : sélectionnez le mode réseau désiré (NAT, Bridge, etc. selon vos besoins).

---

#### 4. Paramétrage du Boot et Installation

1. Démarrez la VM depuis l’ISO (`installXX.iso`).
2. Le système va booter sur l’installateur d’OpenBSD. Vous verrez un prompt du type :
   ```
   Welcome to the OpenBSD <version> installer.
   (I)nstall, (U)pgrade, (A)utoinstall or (S)hell?
   ```
3. Tapez `I` pour commencer l’installation.
4. Suivez les instructions à l’écran :
   - **Choix du clavier** : Généralement, l’installateur propose `uk` ou `us`. Pour un clavier AZERTY français, choisissez `fr` (si disponible) ou configurez plus tard.
   - **Nom de la machine (hostname)** : choisissez un nom, par exemple `openbsd-vm`.
   - **Configuration réseau** : l’installateur tentera de configurer automatiquement via DHCP. Acceptez si vous le souhaitez.
   - **Définir le mot de passe root**.
   - **Sélectionner le fuseau horaire** (Europe/Paris si vous êtes en France).
   - **Partitionnement du disque** : Laissez l’installateur faire le partitionnement automatique par défaut, en général tapez simplement `W` (Use whole disk), puis `Q` pour valider.
   - **Sélection des sets à installer** : Par défaut, l’installateur téléchargera depuis l’ISO. Acceptez les sets par défaut (`base`, `bsd`, etc.).
5. Confirmez l’installation et attendez que l’installation se termine.
6. Une fois l’installation terminée, l’installateur vous demandera de retirer le média d’installation et de redémarrer.

Éteignez la VM, détachez l’ISO du lecteur CD virtuel, puis démarrez à nouveau la VM.

---

#### 5. Configurations Initiales Post-Installation

Après le premier boot :

1. Connectez-vous en tant que `root` avec le mot de passe défini lors de l’installation.
2. Vérifiez la connectivité réseau avec une commande comme `ping openbsd.org` (si vous avez configuré le réseau).
3. Éditez `/etc/doas.conf` pour permettre des actions administratives à un utilisateur non-root (si vous souhaitez créer un compte utilisateur). Exemple :
   ```
   permit persist :wheel
   ```
4. Créez un utilisateur standard :
   ```bash
   adduser
   ```
   Répondez aux questions, puis ajoutez cet utilisateur au groupe `wheel` pour lui permettre d’utiliser `doas`.
5. Configurez le fuseau horaire si ce n’est pas déjà fait :
   ```bash
   cp /usr/share/zoneinfo/Europe/Paris /etc/localtime
   ```


### Configuration du Serveur Passerelle OpenBSD

Ce document explique les étapes de configuration de la VM passerelle, incluant l’adressage réseau, la configuration du serveur DHCP, du firewall pf, ainsi que les différents fichiers et commandes utilisés.

---

#### Sommaire

1. Présentation du Contexte
2. Adressage Réseau et Masques
3. Configuration des Interfaces Réseau
4. Configuration du Serveur DHCP
5. Configuration du Pare-feu (pf)
6. Activation du Forwarding IP
7. Tests et Vérifications
8. Fichiers Importants
9. Commandes Principales

---

#### 1. Présentation du Contexte

Cette VM (VM 1) joue le rôle de passerelle pour trois réseaux privés distincts, avec un accès Internet via une interface externe. Elle doit fournir des adresses IP via DHCP à chacune des LAN internes, et appliquer des règles de filtrage de paquets pour contrôler les flux entre les réseaux.

**Caractéristiques :**

- **OpenBSD 7.6**
- **4 interfaces réseau** (1 externe, 3 internes)
- **3 sous-réseaux privés séparés** : Administration (LAN-1), Serveurs (LAN-2) et Employés (LAN-3)
- Attribution dynamique d’adresses IP (DHCP) sur les réseaux internes
- Routage et filtrage avec pf

---

#### 2. Adressage Réseau et Masques

Les trois réseaux internes sont découpés sur une base 192.168.42.0, avec des sous-réseaux /26 (masque 255.255.255.192).

- **LAN-1 (Administration)**
  - Network : 192.168.42.0
  - Broadcast : 192.168.42.63
  - Masque : 255.255.255.192 (/26)
  - IP passerelle : 192.168.42.1
  - Plage DHCP : 192.168.42.40 – 192.168.42.60

- **LAN-2 (Serveur)**
  - Network : 192.168.42.64
  - Broadcast : 192.168.42.127
  - Masque : 255.255.255.192 (/26)
  - IP passerelle : 192.168.42.65
  - Plage DHCP : 192.168.42.70 – 192.168.42.110

- **LAN-3 (Employés)**
  - Network : 192.168.42.128
  - Broadcast : 192.168.42.191
  - Masque : 255.255.255.192 (/26)
  - IP passerelle : 192.168.42.129
  - Plage DHCP : 192.168.42.140 – 192.168.42.180

---

#### 3. Configuration des Interfaces Réseau

Les interfaces sont supposées être nommées **em0** (externe), **em1** (LAN-1), **em2** (LAN-2) et **em3** (LAN-3).

- **Fichier /etc/hostname.em1** :
  ```
  inet 192.168.42.1 255.255.255.192 NONE
  ```

- **Fichier /etc/hostname.em2** :
  ```
  inet 192.168.42.65 255.255.255.192 NONE
  ```

- **Fichier /etc/hostname.em3** :
  ```
  inet 192.168.42.129 255.255.255.192 NONE
  ```

Après modification, redémarrez ou exécutez `sh /etc/netstart` pour appliquer les changements.

---

#### 4. Configuration du Serveur DHCP

**Installation du serveur DHCP (ISC DHCP) :**
```bash
pkg_add isc-dhcp-server
```

**Fichier de configuration /etc/dhcpd.conf :**
```
# Global options
option domain-name "localdomain";
option domain-name-servers 192.168.42.1;
default-lease-time 600;
max-lease-time 7200;

# LAN-1 (Administration)
subnet 192.168.42.0 netmask 255.255.255.192 {
  range 192.168.42.40 192.168.42.60;
  option routers 192.168.42.1;
  option broadcast-address 192.168.42.63;
}

# LAN-2 (Server)
subnet 192.168.42.64 netmask 255.255.255.192 {
  range 192.168.42.70 192.168.42.110;
  option routers 192.168.42.65;
  option broadcast-address 192.168.42.127;
}

# LAN-3 (Employee)
subnet 192.168.42.128 netmask 255.255.255.192 {
  range 192.168.42.140 192.168.42.180;
  option routers 192.168.42.129;
  option broadcast-address 192.168.42.191;
}
```

**Activation du service DHCP :**
- Dans `/etc/rc.conf.local` :
  ```
dhcpd_flags="em1 em2 em3"
  ```
- Démarrer/activer le service :
  ```bash
  rcctl enable dhcpd
  rcctl start dhcpd
  ```

---

#### 5. Configuration du Pare-feu (pf)

**Fichier /etc/pf.conf :**
```bash
# Interfaces
ext_if = "em0"
lan1_if = "em1"
lan2_if = "em2"
lan3_if = "em3"

# Réseaux
lan1_net = "192.168.42.0/26"
lan2_net = "192.168.42.64/26"
lan3_net = "192.168.42.128/26"

set skip on lo
scrub in on $ext_if all fragment reassemble

# NAT vers Internet pour les réseaux internes
match out on $ext_if from { $lan1_net, $lan2_net, $lan3_net } to any nat-to ($ext_if)

# Politique par défaut : tout bloquer
block all

# Autoriser DHCP et DNS vers la passerelle
pass in on { $lan1_if, $lan2_if, $lan3_if } proto udp from any to any port {67,68,53} keep state

# Règles de segmentation et d'accès :
# - LAN Admin (lan1) vers LAN Serveurs (lan2) : accès complet
pass in on $lan1_if from $lan1_net to $lan2_net keep state

# - LAN Employés (lan3) vers LAN Serveurs (lan2) : accès HTTP/HTTPS uniquement
pass in on $lan3_if from $lan3_net to $lan2_net port {80,443} keep state

# Autoriser les 3 LAN à accéder à Internet
pass out on $ext_if from { $lan1_net, $lan2_net, $lan3_net } to any keep state

# Autoriser le ping entre sous-réseaux
pass in on { $lan1_if, $lan2_if, $lan3_if } inet proto icmp all keep state
pass out on { $lan1_if, $lan2_if, $lan3_if } inet proto icmp all keep state
```

**Activer et charger pf :**
```bash
rcctl enable pf
pfctl -f /etc/pf.conf
```

---

#### 6. Activation du Forwarding IP

Dans `/etc/sysctl.conf` :
```bash
net.inet.ip.forwarding=1
```

Appliquer immédiatement :
```bash
sysctl net.inet.ip.forwarding=1
```

---

#### 7. Tests et Vérifications

- **DHCP** : Brancher un poste sur chaque LAN et vérifier l’obtention d’une adresse IP dans la plage correcte.
- **Accès entre LAN :**
  - Depuis LAN Admin, vérifier l’accès à toutes les ressources du LAN Serveurs.
  - Depuis LAN Employés, vérifier l’accès HTTP/HTTPS uniquement vers le LAN Serveurs.
- **Internet** : Vérifier que les trois LAN peuvent accéder à Internet (pings externes, navigation web).
- **Ping entre sous-réseaux** : Tester le ping entre différents LAN pour s’assurer de la connectivité interne.

---

#### 8. Fichiers Importants

- `/etc/hostname.em1`, `/etc/hostname.em2`, `/etc/hostname.em3` : Configuration IP des interfaces internes.
- `/etc/dhcpd.conf` : Configuration du service DHCP.
- `/etc/rc.conf.local` : Paramètres de démarrage des services (dhcpd, etc.).
- `/etc/pf.conf` : Règles du pare-feu pf.
- `/etc/sysctl.conf` : Activation du routage (ip forwarding).
