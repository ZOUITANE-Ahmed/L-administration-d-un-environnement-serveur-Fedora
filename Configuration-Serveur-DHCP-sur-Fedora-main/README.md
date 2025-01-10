# Configuration Serveur DHCP sur Fedora

La configuration d'un serveur DHCP sur Fedora ou Red Hat nécessite l'installation du logiciel DHCP, la modification des fichiers de configuration, et le démarrage du service. Voici un guide étape par étape :

---

### **1. Installation du serveur DHCP**
- Installez le package DHCP à l'aide du gestionnaire de packages `dnf` :
  ```bash
  sudo dnf install dhcp-server
  ```

---

### **2. Configuration du fichier `dhcpd.conf`**
Le fichier principal de configuration est situé à :  
`/etc/dhcp/dhcpd.conf`

- **Éditez le fichier de configuration :**
  ```bash
  sudo nano /etc/dhcp/dhcpd.conf
  ```
  
- **Exemple de configuration :**
  ```bash
  # Définition des paramètres globaux
  default-lease-time 600;   # Durée par défaut d'une adresse IP
  max-lease-time 7200;      # Durée maximale d'une adresse IP

  # Déclaration du réseau et plage d'adresses IP
  subnet 192.168.1.0 netmask 255.255.255.0 {
      range 192.168.1.100 192.168.1.200;  # Plage d'adresses IP allouées
      option routers 192.168.1.1;         # Adresse de la passerelle
      option subnet-mask 255.255.255.0;   # Plage d'adresses IP que le serveur attribuera
      option broadcast-address 192.168.1.255;
      option domain-name-servers 8.8.8.8, 8.8.4.4;  # Serveurs DNS
      option domain-name "mondomaine.local";  # Nom du domaine
  }
  ```

---

### **3. Activer et démarrer le service DHCP**
- **Assurez-vous que la configuration réseau est prête :**
  Vérifiez que l'interface réseau correspond à la configuration dans `/etc/dhcp/dhcpd.conf`.

- **Démarrez le service DHCP :**
  ```bash
  sudo systemctl start dhcpd
  ```

- **Activez le service au démarrage :**
  ```bash
  sudo systemctl enable dhcpd
  ```

---

### **4. Configuration du pare-feu**
- Ouvrez le port pour le service DHCP dans `firewalld` :
  ```bash
  sudo firewall-cmd --add-service=dhcp --permanent
  sudo firewall-cmd --reload
  ```

---

### **5. Vérifications**
- **Vérifiez le statut du service :**
  ```bash
  sudo systemctl status dhcpd
  ```





- **Surveillez les logs pour des éventuelles erreurs :**
  ```bash
  sudo journalctl -u dhcpd
  ```

- **Testez avec un client DHCP :**
  Configurez un poste client pour obtenir une adresse IP via DHCP et vérifiez qu'il reçoit une adresse dans la plage définie.

---

### **6. Personnalisation avancée (facultatif)**
- **Réservations d'adresses IP (par adresse MAC) :**
  Ajoutez dans `dhcpd.conf` :
  ```bash
  host client1 {
      hardware ethernet 00:1A:2B:3C:4D:5E;  # Adresse MAC du client
      fixed-address 192.168.1.50;           # Adresse IP réservée
  }
  ```
- **Support de plusieurs sous-réseaux :**
  Vous pouvez définir plusieurs blocs `subnet` dans le fichier de configuration.

---

### **7. Dépannage**
- Assurez-vous que le service DHCP écoute sur la bonne interface :
  Vérifiez `/etc/sysconfig/dhcpd` et ajoutez la ligne :
  ```bash
  DHCPDARGS="eth0";
  ```

