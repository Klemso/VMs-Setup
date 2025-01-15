### Mise en place de la VM FreeBSD 14

#### 1. **Téléchargement de l'image ISO de FreeBSD**

- Rendez-vous sur le site officiel de FreeBSD:
  - Dernière version stable [FreeBSD 14.1](https://download.freebsd.org/releases/amd64/amd64/ISO-IMAGES/14.1/)
  - Télécharger : **FreeBSD-14.1-RELEASE-amd64-disc1.iso**

    On utilise l'image ISO car c'est le moyen le plus simple pour installer la structure et toutes les données de l'OS
    On choisit l'image **disc** car nous effectuons une installation 'standard'. Elle ne nécessite pas internet, et reste la moins volumineuse 

#### 2. **Initialisation de la VM**

- **ISO Image** : FreeBSD-14.1-RELEASE-amd64-disc1.iso  
- **Type de système** : BSD  
- **Version** : FreeBSD (64-bit)
- **Configuration** :
  - 4 Go de RAM
  - 2 cœurs CPU
  - 21,5 Go d'espace disque alloué

Avant de démarrer la VM, vérifier dans *Configuration*:
1. Dans *Stockage* :
   - La présence de l'**image ISO freeBSD** dans *Périphériques*
2. Dans *Réseau* :
   - Vérifier que le réseau est configuré en **NAT**
     Cela permettra à la VM d'avoir accès à internet

#### 3. **Installation de FreeBSD**

1. Au premier démarrage de la VM, sélectionner `1`,  pour installer FreeBSD.
2. Suivez les instructions (utiliser `Espace` pour sélectionner, `Entrée` pour valider) :
   - **Clavier** : French (accent keys)
   - **Nom de l'hôte** : `webserver` (par exemple)
   - **Options supplémentaires** : Ajouter **lib32** et **ports**.
   
     Ces options ne sont normalement pas nécessaires, mais elles garantissent une compatibilité avec les applications 32 bits et offrent plus de flexibilité pour les configurations et les compilations.
   - **Partitionnement** :
     - Choisir **Auto (ZFS)**
     
       Une configuration de partitionnement automatique est suffisante dans notre cas, et ZFS est plus récent et plus performant que UFS même si plus gourmande en mémoire
     - Conserver les options par défaut, sauf pour **Pool Type/Disks**, sélectionner **Stripe** puis `ada0` (normalement seul disque virtuel proposé).
3. Question : "**Last Chance! Are you sure you want to continue? This will erase all data on the selected disks**"
   - Confirmer avec `Yes`.

#### 4. **Configuration post-installation**

1. **Après l'extraction des archives** :
   - Définir un mot de passe pour `root` (vous pouvez utiliser `Entrée`).
   - Configuration réseau :
     - `em0` pour l'interface réseau
     - Activer IPv4
     - Refuser DHCP et IPv6 (inutile ici, l'IPv4 suffit)
     - Valider les options
   - Choisir le fuseau horaire.

2. **Configuration du système** :
   - Activer les services suivants :
     - `sshd` : active le serveur SSH
     - `ntpd` et `ntpd_sync_on_start` : synchronise l'horloge système
     - `dumpdev` : alloue un espace disque pour les logs de crash.

3. **Configuration sécurité système** :
   - Activer les options suivantes :
     - `hide_uids` et `hide_gids`: masque les noms d'utilisateurs et groupes
     - `read_msgbuf` : limite l'accès aux données système
     - `proc_debug` : empêche un utilisateur standard d'observer les processus d'autres utilisateurs
     - `clear_tmp` : vide automatiquement le dossier `/tmp` au démarrage
     - `secure_console` : empêche un accès non autorisé au système en cas de redémarrage physique forcé
     - `disable_ddtrace`: désactive les fonctionnalités de traçage du débogueur

5. **Finalisation** :
   - Il n'est pas nécessaire de créer un utilisateur, nous nous connecterons sur `root` pour les prochaines installations
   - Eteindre la VM, retourner dans *Configuration* > *Stockage* et supprimer l'image ISO utilisée pour l'installation

#### 5. **Premier démarrage**

1. Démarrer la VM et attendre le boot standard
2. Se connecter avec `root` et le mot de passe défini lors de l'installation

#### 6. **Mise à jour de la VM**

- Pour mettre à jour FreeBSD ainsi que le gestionnaire de paquets `pkg`, exécutez les commandes suivantes :
  ```bash
  freebsd-update fetch install
  pkg update && pkg upgrade
  ```

#### 7. **Installation de Nginx**
Nginx est un serveur HTTP pour héberger un site web.
- Installer `Nginx` et `curl` à l'aide de `pkg`:
  ```bash
  pkg install -y nginx curl
  ```
- Activer `Nginx`, vérifier qu'il est bien activé et activer son lancement dès le démarrage de la VM :
  ```bash
  service nginx start
  service nginx status
  sysrc nginx_enable="YES"
  ```
- Vérifier l'accessibilité du serveur :
  ```bash
  ping 10.0.2.15
  curl http://10.0.2.15
  ```

#### 8. **Installation de PHP 7.4**

1. Installer les dépendances :
   ```bash
   pkg install -y gcc bison re2c libjpeg-turbo libpng pkgconf libxml2 sqlite3 gmake nano
   ```

   Si vous rencontrez un problème vous pouvez les installer une par une :
   ```bash
   pkg install -y gcc # Nécessaire pour compiler des logiciels écrits en C/C++ (comme PHP)
   pkg install -y bison # Pour traduire les instructions en code exécutable
   pkg install -y re2c # Complémentaire à bison
   pkg install -y libjpeg-turbo # Bibliothèque pour gérer les fichiers images JPEG
   pkg install -y libpng # Bibliothèque pour gérer les fichiers images PNG
   pkg install -y pkgconf # Outil pour localiser les bibliothèques et fichiers
   pkg install -y libxml2 # Bibliothèque pour analyser des fichiers XML de PHP
   pkg install -y sqlite3 # Extension pour activer la base de donnée SQLite nativement présente dans PHP
   pkg install -y gmake # Utiliser pour compiler et construire les logiciels selon les instructions
   pkg install -y nano # Editeur de texte directement dans l'invité de commande
   ```
   
3. Télécharger et compiler PHP :
   - On crée un répertoire pour stocker les ressources
   ```bash
   mkdir /usr/local/src
   cd /usr/local/src
   ```
   - On télécharge le fichier source de PHP depuis le site officiel, puis on extrait et spécifie le fichier d'entrée
   ```bash
   fetch https://www.php.net/distributions/php-7.4.33.tar.gz
   tar -xvzf php-7.4.33.tar.gz
   ```
   - Avec `./configure`, on détecte dans notre environnement tous les paramètres nécessaires à la compilation (bibliothèques, outils, etc), afin que PHP soit construit correctement pour FreeBSD
   - On installe PHP dans le répertoire `usr/local/php` et on définit l'emplacement du fichier de configuration principale `php.ini`
   ```bash
   cd php-7.4.33
   ./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php
   ```
   - On lance la compilation et l'installation des fichiers source de PHP
   ```bash
   gmake
   gmake install
   ```

4. Finalisation de l'installation 
   - `php.ini-development` contient une configuration par defaut, pour déboguer et tester le code PHP, on le copie dans le dossier php.ini
   ```bash
   cp php.ini-development /usr/local/php/lib/php.ini
   nano /.profile
   export PATH=$PATH:/usr/local/php/bin
   ```

5. Redémarrer la VM et vérifier la version de PHP :
   ```bash
   php -v
   ```

#### 9. **Installation de MySQL**

1. Installer MySQL et `unzip` pour extraire les formats `.zip` :
   ```bash
   pkg install -y mysql80-server unzip
   ```
   - Activer `MySQL`, activer son lancement dès le démarrage de la VM :
   ```bash
   service mysql-server start
   sysrc mysql_enable="YES"
   ```
   
2. Télécharger le fichier `app-t-nsa-501.zip` depuis la machine principale :
   - Sur votre ordinateur principale, à l'aide d'un invité de commande déplacez-vous dans le dossier où se trouve `app-t-nsa-501.zip`. `http.server` est un module Python qui crée un serveur HTTP local dans le répertoire où il est exécuté.
   ```bash
   python3 -m http.server 8000
   ```
   
  - Retrouvez votre IP :
   ```bash
   ip a
   ```

  - Dans votre VM, on télécharge le fichier `app-t-nsa-501.zip` dans le répertoire `/tmp/app_t-nsa-501.zip` :
    ```bash
    fetch http://<Votre adresse IP>:8000/app_t-nsa-501.zip -o /tmp/app_t-nsa-501.zip
    ```
  - On décompresse le fichier `.zip` :
    ```bash
    cd /tmp
    unzip app_t-nsa-501.zip
    ```

3. Configuration :
   - On se connecte au client MySQL en se connectant à l'utilisateur `root` :
   ```bash
   mysql -u root -p
   ```
   - On crée la base de données `nsa501` :
   ```bash
   CREATE DATABASE nsa501;
   exit
   ```
   - On importe les instructions SQL du fichier `nsa501.sql` que l'on a extrait précédement et on lance MySQL:
   ```bash
   mysql -u root -p nsa501 < nsa501.sql
   ```

   - On crée l'utilisateur `backend`, on l'autorise à se connecter depuis n'importe quelle adresse IP, on lui donne le mot de passe `Bit8Q6a6G` et on lui donne tous les droits à la base de données `nsa501` :
   ```bash
   CREATE USER 'backend'@'%' IDENTIFIED BY 'Bit8Q6a6G';
   GRANT ALL PRIVILEGES ON nsa501.* TO 'backend'@'%';
   FLUSH PRIVILEGES; # Recharge les permissions accordées dans MySQL
   exit
   ```

  - On teste de se connecter à l'utilisateur `backend` via le `localhost` :
   ```bash
   mysql -u backend -p -h 127.0.0.1
   Password : Bit8Q6a6G
   SHOW DATABASES; # Affiche toutes les bases de données accessibles par l'utilisateur
   ```

#### 10. **Configuration service SSH & Pare-feu**
  - Dans le fichier `sshd_config`, qui contrôle les paramètres SSH de la machine, on décommente la ligne `Port 22`, afin que le SSH écoute le port 22
   ```bash
   nano etc/ssh/sshd_config
   #Dans le fichier, supprimer le '#' devant `Port 22`
   ```
  
  - On active le système de pare-feu `pf`, on assure son activation à chaque allumage de la VM, on vérifie son statut et ses règles :
   ```bash
   service pf start
   sysrc pf_enable="YES"
   service pf status
   pfctl -sr
   ```
  
  - On ouvre le fichier pf.conf, pour décommenter certaines lignes et assurer certaines règles du pare-feu:
  ```bash
   nano /etc/pf.conf
  ```

  - Permettre à l'administrateur de se connecter à la VM du serveur
  ```bash
   pass in on em0 proto tcp from 192.168.42.0/24 to any port 22
  ```

  - Bloque les connexions provenant des adresses IP de 192.168.42.140 à 192.168.42.143
  ```bash
   block in on em0 proto tcp from 192.168.42.140/30 to any port 22
  ```

  -  Permet les connexions provenant du réseau 192.168.43.0 (employés)
  ```bash
   pass in on em0 proto tcp from 192.168.43.0/24 to any port 80
  ```

  - On applique les nouvelles règles :
    ```bash
     pfctl -f /etc/pf.conf
    ```
