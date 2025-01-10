# -configurer-un-serveur-DNS-sur-Fedora
 Pour configurer un serveur DNS sur Fedora avec le domaine **ofppt.local**, voici les étapes détaillées :  

---

### **1. Installer le serveur DNS (BIND)**

```bash
sudo dnf install bind bind-utils -y
```

---

### **2. Configurer le fichier principal de BIND**

1. Ouvrir le fichier `/etc/named.conf` :
   ```bash
   sudo nano /etc/named.conf
   ```

2. Modifier ou ajouter les sections suivantes :
   ```conf
   options {
       directory "/var/named";
       allow-query { any; };         # Permet les requêtes de toutes les IP
       recursion yes;                # Active la récursion DNS
       forwarders { 8.8.8.8; 8.8.4.4; };  # DNS publics en secours
   };

   zone "ofppt.local" IN {
       type master;
       file "/var/named/ofppt.local.db";  # Fichier de zone directe
   };

   zone "1.168.192.in-addr.arpa" IN {
       type master;
       file "/var/named/1.168.192.rev";  # Fichier de zone inverse
   };
   ```

---

### **3. Créer les fichiers de zone DNS**

1. **Fichier de zone directe :**
   - Créer le fichier `/var/named/ofppt.local.db` :
     ```bash
     sudo nano /var/named/ofppt.local.db
     ```

   - Ajouter le contenu suivant :
     ```conf
     $TTL 86400
     @   IN  SOA ns1.ofppt.local. root.ofppt.local. (
             2023122501 ; Serial
             3600       ; Refresh
             1800       ; Retry
             604800     ; Expire
             86400 )    ; Minimum TTL
     @   IN  NS  ns1.ofppt.local.
     ns1 IN  A   192.168.1.90
     @   IN  A   192.168.1.90
     client1 IN  A   192.168.1.100
     ```

2. **Fichier de zone inverse :**
   - Créer le fichier `/var/named/1.168.192.rev` :
     ```bash
     sudo nano /var/named/1.168.192.rev
     ```

   - Ajouter le contenu suivant :
     ```conf
     $TTL 86400
     @   IN  SOA ns1.ofppt.local. root.ofppt.local. (
             2023122501 ; Serial
             3600       ; Refresh
             1800       ; Retry
             604800     ; Expire
             86400 )    ; Minimum TTL
     @   IN  NS  ns1.ofppt.local.
     90  IN  PTR ns1.ofppt.local.
     100 IN  PTR client1.ofppt.local.
     ```

3. Changer les permissions des fichiers :
   ```bash
   sudo chown named:named /var/named/ofppt.local.db /var/named/1.168.192.rev
   ```

---

### **4. Démarrer et activer le service DNS**

1. Démarrer le service BIND :
   ```bash
   sudo systemctl start named
   sudo systemctl enable named
   ```

2. Vérifier l’état du service :
   ```bash
   sudo systemctl status named
   ```

---

### **5. Configurer le pare-feu**

1. Ouvrir le port DNS :
   ```bash
   sudo firewall-cmd --permanent --add-service=dns
   sudo firewall-cmd --reload
   ```

---

### **6. Vérifications**

1. **Tester le DNS localement** :
   ```bash
   nslookup ofppt.local 192.168.1.90
   nslookup client1.ofppt.local 192.168.1.90
   ```

2. **Tester la résolution inverse** :
   ```bash
   nslookup 192.168.1.90 192.168.1.90
   ```

---

Avec cette configuration, le serveur DNS répond aux requêtes pour le domaine **ofppt.local** et résout les noms et adresses correspondants.
