# README - Projet T-NSA-501

Ce dépôt contient les instructions et configurations pour un environnement comprenant 4 machines virtuelles (« VMs ») interconnectées dans le cadre du projet T-NSA-501. L'objectif est de construire une infrastructure fonctionnelle simulant un réseau composé d'une passerelle, d'un serveur web et de postes clients, avec des règles de sécurité strictes.

## Aperçu des VMs

### 1. VM 1 : Passerelle (OpenBSD 7.6)
- **Rôle** : Routeur et pare-feu.
- **Caractéristiques** :
  - 4 interfaces réseau (1 externe, 3 internes).
  - Distribution des adresses IP via DHCP.
  - Application des règles de filtrage et routage.

### 2. VM 2 : Serveur Web (FreeBSD 14)
- **Rôle** : Héberger un serveur NGINX avec PHP et MySQL.
- **Caractéristiques** :
  - Configuration en mode DHCP.
  - IP statique attribuée : 192.168.42.70.
  - Déploiement de la base de données `nsa501` et d'une application web.

### 3. VM 3 : Poste Employé
- **Rôle** : Poste client pour le réseau employés.
- **Caractéristiques** :
  - Interface graphique installée.
  - Configuration réseau automatique via DHCP.

### 4. VM 4 : Poste Administrateur
- **Rôle** : Poste client pour le réseau administrateurs.
- **Caractéristiques** :
  - Interface graphique installée.
  - Configuration réseau automatique via DHCP.

## Configuration des Réseaux

- **LAN-1 : Administration**
  - Réseau : 192.168.42.0/26
  - DHCP : 192.168.42.40 à 192.168.42.60

- **LAN-2 : Serveurs**
  - Réseau : 192.168.42.64/26
  - DHCP : 192.168.42.70 à 192.168.42.110

- **LAN-3 : Employés**
  - Réseau : 192.168.42.128/26
  - DHCP : 192.168.42.140 à 192.168.42.180

## Structure du Dépôt

- `README.md` : Ce fichier.
- `openbsd_gateway.md` : Instructions pour la configuration de la passerelle OpenBSD.
- `freebsd_server.md` : Guide pour l'installation et la configuration du serveur web.
- `client_vms.md` : Configurations des postes employé et administrateur.

## Usage

1. Lisez les fichiers d'instructions correspondants aux VMs.
2. Suivez les étapes pour créer et configurer chaque VM.
3. Testez la connectivité entre les réseaux et les VMs.

Pour tout problème ou question, veuillez vous référer aux guides d'installation ou contacter votre administrateur. Bonne configuration !

