### Administration des Réseaux Informatiques - Protocole DHCP et Dynamic DNS (DDNS)

#### 1. **Qu'est-ce que le protocole DHCP ?**
Le protocole **DHCP** (Dynamic Host Configuration Protocol) permet à un client (comme un ordinateur ou un périphérique réseau) d'obtenir automatiquement une adresse IP et d'autres informations de configuration réseau auprès d'un serveur DHCP. Cela simplifie la gestion des adresses IP dans un réseau en évitant l'attribution manuelle et en permettant aux appareils de se connecter facilement au réseau sans intervention humaine.

#### 2. **Avantages du protocole DHCP**
- **Automatisation** de l'attribution des configurations réseau IP.
- **Centralisation** de la gestion des adresses IP.
- **Économie de temps** pour l'administrateur réseau en évitant la gestion manuelle des configurations.
- **Prévention de la pénurie d'adresses IP** avec des mécanismes comme la réservation et la réutilisation des adresses.
- **Facilitation de la mobilité** : avec des ordinateurs portables ou des appareils mobiles, les utilisateurs peuvent se connecter au réseau sans devoir configurer manuellement les paramètres réseau à chaque connexion.

#### 3. **Principe de fonctionnement du DHCP**
Le processus de communication entre le client et le serveur DHCP suit ces étapes :
1. **DHCP Discovery** : Le client envoie un paquet de découverte pour trouver un serveur DHCP sur le réseau. Ce paquet est diffusé dans le réseau.
2. **DHCP Offer** : Le serveur DHCP répond au client avec une offre d'adresse IP et d'autres paramètres réseau (passerelle, DNS, etc.).
3. **DHCP Request** : Le client choisit une offre et envoie une demande pour confirmer l'utilisation de l'adresse IP proposée.
4. **DHCP Ack** : Le serveur confirme la demande et attribue l'adresse IP au client pour une période déterminée (bail ou lease).

#### 4. **Gestion des adresses IP avec DHCP**
- Le serveur DHCP attribue des adresses IP d'une plage définie, avec une durée de **bail** (en général, 48 heures) qui peut être ajustée.
- **Réservation d'adresses IP** : certaines adresses IP peuvent être réservées pour des périphériques spécifiques en fonction de leur adresse MAC.
- Le serveur DHCP fournit également des informations comme l'adresse de la passerelle et les serveurs DNS.

#### 5. **Mise en œuvre du serveur DHCP sous Linux**
Le service DHCP sous Linux est géré par le démon **dhcpd** (DHCP Daemon), et son fichier de configuration principal se trouve dans **/etc/dhcp/dhcpd.conf**.

Exemple de configuration de **dhcpd.conf** :

```bash
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name "ofppt.local";
    option domain-name-servers 192.168.2.200, 192.168.2.254;
    default-lease-time 600;
    max-lease-time 7200;
}
```

- **`range`** : Plage d'adresses IP que le serveur peut attribuer.
- **`option routers`** : L'adresse de la passerelle (routeur).
- **`option domain-name-servers`** : Serveurs DNS à utiliser pour la résolution des noms.

Exemple pour l'attribution d'IP statiques pour un client spécifique :

```bash
host station1 {
    hardware ethernet 00:A0:ad:41:5c:b1;  # Adresse MAC
    fixed-address 192.168.1.14;            # IP statique attribuée
}
```

#### 6. **Mise en œuvre de DDNS (Dynamic DNS)**
Le **Dynamic DNS (DDNS)** permet au serveur DHCP d'ajouter automatiquement des enregistrements DNS pour les clients qui reçoivent une adresse IP dynamique. Pour cela, il faut configurer à la fois le serveur DHCP et le serveur DNS.

- **Configuration côté serveur DHCP** dans **/etc/dhcp/dhcpd.conf** :

```bash
ddns-update-style interim;
ddns-updates on;
ddns-domainname "ofppt.local";
ddns-rev-domainname "in-addr.arpa";
zone ofppt.local. {
    primary 127.0.0.1;
}
zone 1.168.192.in-addr.arpa. {
    primary 127.0.0.1;
}
```

- **Configuration côté serveur DNS (BIND)** dans **/etc/named.conf** :

```bash
options {
    directory "/var/named";
    allow-query { any; };         # Permet les requêtes de toutes les IP
    recursion yes;                # Active la récursion DNS
    forwarders { 8.8.8.8; 8.8.4.4; };  # DNS publics en secours
};

zone "ofppt.local" IN {
    type master;
    file "/var/named/ofppt.local.db";  # Fichier de zone directe
    allow-update { 127.0.0.1; };
};

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "/var/named/1.168.192.rev";  # Fichier de zone inverse
    allow-update { 127.0.0.1; };
};
```

---

### **7. Créer les fichiers de zone DNS**

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

#### 8. **Configuration et lancement du serveur DHCP**
1. Installez le paquet **dhcp-server** :

   ```bash
   sudo dnf install dhcp-server
   ```

2. Modifiez les fichiers de configuration comme mentionné ci-dessus.

3. Démarrez le service DHCP et activez-le au démarrage :

   ```bash
   sudo systemctl start dhcpd
   sudo systemctl enable dhcpd
   ```

#### 9. **Prochaines étapes**
- **DNS** : Configuration du serveur DNS sous Linux pour résoudre les noms et gérer les mises à jour dynamiques.