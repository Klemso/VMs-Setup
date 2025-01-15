### Mise en place des VMs Clients (Employee & Admin)

---

#### **Caractéristiques techniques des VMs**
- **Système d'exploitation** : Debian 12
- **Mémoire Vive** : 1024 Mo
- **Espace disque alloué** : 10 Go
- **Interface graphique** : GNOME

---

#### **Configuration réseau pour VirtualBox**
- **VM Employee** :
  - *Adaptateur 1* : Réseau interne
    - Nom : Employee
    - Promiscuity : Non
  - *Adaptateur 2* : NAT (à retirer pour la soumission)
- **VM Admin** :
  - *Adaptateur 1* : Réseau interne
    - Nom : Admin
    - Promiscuity : Non
  - *Adaptateur 2* : NAT (à retirer pour la soumission)

---

### Configuration des VMs

#### **VM3 - Employee**

1. Lancer la VM et exécuter les commandes suivantes :
   ```bash
   su -
   apt update
   apt install sudo
   
   sudo nano /etc/network/interfaces
   # Ethernet interface
   # auto enp0s3
   # iface enp0s3 inet dhcp

   sudo hostnamectl set-hostname vm3-client
   sudo reboot
   ip a
   ```

2. Vérification :
   - L'interface `enp0s3` doit recevoir une adresse DHCP dans la plage `192.168.42.140 - 192.168.42.180`.

---

#### **VM4 - Admin**

1. Lancer la VM et exécuter les commandes suivantes :
   ```bash
   su -
   apt update
   apt install sudo

   sudo nano /etc/network/interfaces
   # Ethernet interface
   # auto enp0s3
   # iface enp0s3 inet dhcp

   sudo hostnamectl set-hostname vm4-admin
   sudo reboot
   ip a
   ```

2. Vérification :
   - L'interface `enp0s3` doit recevoir une adresse DHCP dans la plage `192.168.42.40 - 192.168.42.60`.

---
