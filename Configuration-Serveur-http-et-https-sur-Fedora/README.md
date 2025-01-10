# Serveur-http-et-https-sur-Fedora
# Installation et Configuration du Serveur HTTP avec Apache et SSL

## 1. Introduction au protocole HTTP

**HTTP (HyperText Transfer Protocol)** est un protocole de communication qui permet le transfert de documents hypertextes, tels que des pages web, entre un client (souvent un navigateur web) et un serveur.
- Fonctionne principalement sur le port **80 (HTTP)** ou le port **443 (HTTPS)** pour une communication sécurisée.

---

## 2. Configuration du serveur HTTP

### 2.1. Configuration du nom du serveur
```bash
sudo hostnamectl set-hostname serveur-http
```

### 2.2. Configuration de la carte réseau
```bash
sudo nmcli connection modify eth0 ipv4.addresses 192.168.1.10/24 ipv4.gateway 192.168.1.1 ipv4.dns 8.8.8.8 ipv4.method manual
```

### 2.3. Installation d’Apache
#### Installation en ligne
```bash
sudo dnf install httpd -y
```
#### Installation hors ligne
```bash
sudo rpm -ivh httpd
```

#### Vérification de l'installation
```bash
rpm -qa httpd*
```

### 2.4. Démarrage et activation du service
```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

### 2.5. Chemin des fichiers de configuration
Le fichier principal de configuration d’Apache se trouve dans :
```bash
/etc/httpd/conf/httpd.conf
```

---

## 3. Test et vérification
### 3.1. Vérifier la configuration
```bash
sudo apachectl configtest
```
### 3.2. Redémarrer Apache
```bash
sudo systemctl restart httpd
```

---

## 4. Configuration du pare-feu
### Autoriser HTTP et HTTPS
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

## 5. Configuration des répertoires du site

1. Créer le répertoire pour le site :
   ```bash
   mkdir -p /var/www/html/cmc.local
   ```

2. Modifier les droits d’accès :
   ```bash
   chown -R apache:apache /var/www/html/cmc.local
   chmod -R 755 /var/www/html/cmc.local
   ```

3. Créer un fichier `index.html` :
   ```bash
   vim /var/www/html/cmc.local/index.html
   ```

4. Exemple de contenu pour `index.html` :
   ```html
   <html>
   <head><title>CMC Local</title></head>
   <body>
   <h1>Bienvenue sur le site CMC Local</h1>
   </body>
   </html>
   ```

---

## 6. Configuration Virtual Host

1. Éditer le fichier de configuration :
   ```bash
   nano /etc/httpd/conf.d/cmc.local.conf
   ```

2. Exemple de configuration :
   ```apache
   <VirtualHost *:80>
       ServerAdmin admin@cmc.local
       ServerName cmc.local
       ServerAlias www.cmc.local
       DocumentRoot /var/www/html/cmc.local
       ErrorLog /var/log/httpd/error_log
       CustomLog /var/log/httpd/access_log combined
   </VirtualHost>
   ```

3. Ajouter une entrée dans le fichier hosts :
   ```bash
   sudo nano /etc/hosts
   ```
   Ajouter :
   ```
   192.168.10.20 cmc.local
   ```

---

## 7. Sécuriser Apache avec SSL

### 7.1. Installation du module SSL
```bash
sudo dnf install mod_ssl -y
```

### 7.2. Créer un certificat SSL auto-signé (pour usage interne ou test)
```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/apache-selfsigned.key \
-out /etc/ssl/certs/apache-selfsigned.crt
```

### 7.3. Configurer Apache pour SSL

1. Éditer le fichier SSL :
   ```bash
   sudo nano /etc/httpd/conf.d/ssl.conf
   ```

2. Ajouter ou modifier les lignes suivantes :
   ```apache
   SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
   SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
   ```

---

## 8. Redirection HTTP vers HTTPS
1. Ajouter une redirection dans le Virtual Host HTTP :
   ```apache
   <VirtualHost *:80>
       ServerName cmc.local
       Redirect / https://cmc.local/
   </VirtualHost>
   ```

2. Créer un Virtual Host pour HTTPS :
   ```apache
   <VirtualHost *:443>
       ServerName cmc.local
       DocumentRoot /var/www/html/cmc.local
       SSLEngine on
       SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
       SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
   </VirtualHost>
   ```

---

## 9. Tester la configuration SSL
1. Redémarrer Apache :
   ```bash
   sudo systemctl restart httpd
   ```

2. Accéder au site via HTTPS :
   - URL : `https://cmc.local`

3. Utiliser un outil comme [SSL Labs](https://www.ssllabs.com/ssltest/) pour vérifier la configuration SSL.

---

## 10. Renouvellement automatique des certificats (Let's Encrypt)
1. Installer Certbot :
   ```bash
   sudo dnf install certbot python3-certbot-apache -y
   ```

2. Configurer un renouvellement automatique :
   ```bash
   sudo crontab -e
   ```
   Ajouter :
   ```
   0 0 * * * certbot renew --quiet && systemctl reload httpd
   ```

