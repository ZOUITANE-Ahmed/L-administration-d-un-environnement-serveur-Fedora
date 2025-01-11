### **Introduction au NTP (Network Time Protocol)**

Le **NTP** est un protocole essentiel pour garantir la synchronisation des horloges des systèmes informatiques sur un réseau, en assurant que les serveurs, ordinateurs et autres dispositifs partagent une heure précise. Il fonctionne sur la base d'une hiérarchie de serveurs de temps, dont les serveurs de niveau supérieur utilisent des horloges atomiques ou GPS pour fournir une référence de temps précise.

### **Points Clés de NTP** :
- **Fonctionnement** : Utilisation d'une hiérarchie de serveurs de temps. Les serveurs de niveau supérieur (horloges atomiques ou GPS) transmettent l'heure exacte aux serveurs inférieurs, qui la diffusent ensuite aux appareils du réseau.
- **Précision** : NTP peut obtenir une précision de l'ordre de quelques millisecondes.
- **Sécurité** : Bien que des mécanismes de sécurité soient disponibles, ils ne sont pas toujours activés par défaut pour éviter la manipulation de l'heure.

---

### **Configuration de NTP avec Chrony sur Fedora**

#### **1. Installer le service NTP**
Fedora utilise généralement **Chrony** comme client NTP. Voici les étapes pour vérifier l'installation et l'installer si nécessaire :

- **Vérifier si Chrony est installé** :
   ```bash
   rpm -q chrony
   ```
   Si ce n'est pas le cas, installez-le via :
   ```bash
   sudo dnf install chrony
   ```

#### **2. Activer et Démarrer le Service Chrony**

1. **Activer Chrony au démarrage** :
   ```bash
   sudo systemctl enable chronyd
   ```

2. **Activer et démarrer le service NFS** (si nécessaire) :
   ```bash
   sudo systemctl enable nfs-server
   sudo systemctl start nfs-server
   ```

3. **Démarrer Chrony** :
   ```bash
   sudo systemctl start chronyd
   ```

4. **Vérifier l'état du service Chrony** :
   ```bash
   sudo systemctl status chronyd
   ```

---

### **Configurer Chrony comme Serveur NTP**

1. **Autoriser les clients à se synchroniser** :
   - Modifiez le fichier `/etc/chrony.conf` pour permettre aux clients du réseau de se synchroniser :
   ```bash
   allow 192.168.1.0/24
   ```

2. **Redémarrer Chrony** :
   ```bash
   sudo systemctl restart chronyd
   ```

3. **Vérifier que le serveur fonctionne** :
   ```bash
   chronyc sources
   ```

---

### **Configurer le Pare-feu (Port UDP 123)**

1. Ouvrir le port UDP 123 pour NTP dans le pare-feu :
   ```bash
   sudo firewall-cmd --add-service=ntp --permanent
   sudo firewall-cmd --reload
   ```

---

### **Fixer l’Heure Manuellement (hors ligne)**

1. Ouvrez le fichier `/etc/chrony.conf` et commentez toutes les lignes contenant `server` :
   ```bash
   # server 0.fedora.pool.ntp.org iburst
   ```

2. Définissez manuellement l'heure et la date :
   ```bash
   sudo timedatectl set-time "2025-01-03 15:30:00"
   ```

---

### **Configurer Chrony sur le Client NTP**

1. **Installer Chrony sur le client** :
   ```bash
   sudo dnf install chrony
   ```

2. **Modifier le fichier `/etc/chrony.conf` pour ajouter l'adresse IP du serveur NTP** :
   ```bash
   server [IP_du_serveur_NTP] iburst
   ```

3. **Démarrer et activer le service Chrony sur le client** :
   ```bash
   sudo systemctl enable chronyd
   sudo systemctl start chronyd
   ```

4. **Vérifier la synchronisation sur le client** :
   ```bash
   chronyc tracking
   ```

5. **Afficher les sources de temps sur le client** :
   ```bash
   chronyc sources -v
   ```

---

### **Commandes Utiles pour la Gestion du Temps**

- Désactiver temporairement la synchronisation automatique :
   ```bash
   sudo timedatectl set-ntp false
   ```

- Réactiver la synchronisation automatique :
   ```bash
   sudo timedatectl set-ntp true
   ```

- Synchroniser immédiatement l'horloge avec un serveur NTP :
   ```bash
   sudo chronyd -q
   ```

- Forcer une synchronisation immédiate avec le serveur NTP :
   ```bash
   sudo chronyc makestep
   ```

---

### **Synthèse**

La configuration correcte du serveur et des clients NTP avec Chrony permet de maintenir une synchronisation horaire précise et cohérente sur un réseau. Il est essentiel d'ouvrir le port UDP 123 et de configurer adéquatement les services pour un bon fonctionnement du NTP.

La prochaine séance pourrait aborder des configurations plus avancées ou explorer d'autres aspects de l'administration réseau.