Le **Network File System (NFS)** est un protocole de partage de fichiers qui permet à différents systèmes de partager des fichiers et des répertoires via un réseau, comme si ces fichiers étaient stockés localement. Développé initialement par Sun Microsystems, NFS est particulièrement utilisé dans les environnements Linux/Unix, mais il est aussi compatible avec d'autres systèmes d'exploitation.

---

### **Principes de fonctionnement**
1. **Architecture client-serveur** :
   - Le serveur NFS exporte un ou plusieurs répertoires pour les rendre accessibles aux clients.
   - Les clients montent ces répertoires partagés, les rendant utilisables comme des systèmes de fichiers locaux.

2. **Avantages** :
   - Partage centralisé des fichiers.
   - Facilité d'accès aux fichiers pour les utilisateurs sur différents systèmes.
   - Gestion simplifiée grâce à une infrastructure unifiée.

3. **Versions de NFS** :
   - **NFSv3** : Stable et largement utilisé.
   - **NFSv4** : Offre des fonctionnalités avancées comme l'authentification Kerberos et un meilleur support des environnements WAN.

---

### **Mise en œuvre du partage NFS**

#### **1. Configuration côté serveur**

Ce document présente les étapes pour configurer un partage de fichiers NFS (Network File System) sous Linux. Voici un résumé des principales étapes mentionnées :


### 1. **Installation des paquets nécessaires** :
   - Utiliser la commande suivante pour installer les outils NFS :
     ```bash
     sudo dnf install nfs-utils
     ```

### 2. **Activation et démarrage du service NFS** :
   - Activer et démarrer le service NFS avec :
     ```bash
     sudo systemctl enable nfs-server 
     sudo systemctl start nfs-server
     ```

### 3. **Configuration des répertoires à partager** :
   - Créer le répertoire à partager :
     ```bash
     sudo mkdir -p /srv/nfs/share
     ```
   - Appliquer les permissions :
     ```bash
     sudo chmod -R 755 /srv/nfs/share
     sudo chown -R nobody:nobody /srv/nfs/share
     ```

### 4. **Configuration du fichier `/etc/exports`** :
   - Ajouter le partage dans le fichier `/etc/exports` :
     ```bash
     /srv/nfs/share 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)
     ```
   - Cette ligne déclare que le répertoire `/srv/nfs/share` sera accessible en lecture-écriture pour les hôtes du réseau `192.168.1.0/24`.
   - Signification des options :
    - `rw` : Lecture et écriture autorisées.
    - `sync` : Les modifications sont écrites immédiatement sur le disque.
    - `no_root_squash` : Permet au root distant d’agir en tant que root local.
    - `no_subtree_check` : Désactive les vérifications de sous-arborescences.

### 5. **Appliquer la configuration** :
   - Rafraîchir la liste des partages après modification du fichier `/etc/exports` :
     ```bash
     sudo exportfs -arv
     ```

### 6. **Configuration du pare-feu** :
   - Ajouter les services NFS, `mountd`, et `rpc-bind` au pare-feu :
     ```bash
     sudo firewall-cmd --add-service=nfs --permanent 
     sudo firewall-cmd --add-service=mountd --permanent 
     sudo firewall-cmd --add-service=rpc-bind --permanent 
     sudo firewall-cmd --reload
     ```

### 7. **Commandes utiles pour le serveur NFS** :
   - Liste des partages avec :
     ```bash
     exportfs -v
     ```
   -  Rafraîchit la liste des partages après modification de /etc/exports :
     ```bash
     exportfs -r
     ```
   -  exporte (o u recharge) tous les partages de /etc/exports ou un 
partage donné :
     ```bash
     exportfs -a
     ```
     - Stoppe le partage donné et -a pour tous.  :
     ```bash
     exportfs -u
     ```

### 8. **Configuration d'un client NFS** :
   - Installer le client NFS :
     ```bash
     sudo dnf install nfs-utils
     ```
   - Monter le partage NFS sur le client :
     ```bash
     sudo mount -t nfs <adresse_du_serveur>:/srv/nfs/share /mnt/partage
     ```
   - Vérifier que le partage est monté :
     ```bash
     df -h
     ```
   - Pour un montage automatique, ajouter l'entrée suivante dans `/etc/fstab` :
     ```bash
     <adresse_du_serveur>:/srv/nfs/share /mnt/partage nfs defaults 0 0
     ```

### 9. **Commande `showmount`** :
   - Voir les partages d'un hôte donné :
     ```bash
     showmount -e <host>
     ```

### Conclusion :
Le partage de fichiers NFS permet de partager efficacement des répertoires entre machines sous Linux. Cette configuration de serveur et de client NFS peut être utilisée dans un environnement réseau pour accéder à des fichiers distants de manière transparente.
