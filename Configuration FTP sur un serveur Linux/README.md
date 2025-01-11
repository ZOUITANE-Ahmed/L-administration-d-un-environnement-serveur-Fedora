### Configuration d'un serveur FTP sous Linux

#### 1. **Introduction au FTP**
Le **FTP** (File Transfer Protocol) est un protocole réseau standard permettant le transfert de fichiers entre un client et un serveur. Il est largement utilisé pour télécharger (upload) ou télécharger (download) des fichiers via un réseau. Plusieurs serveurs FTP existent, comme **vsftpd** (Very Secure FTP Daemon), **ProFTPd**, et **Pure-FTPd**.

Cet article se concentre sur la configuration du serveur **vsftpd**, l'un des plus sécurisés et populaires.

---

#### 2. **Installation de vsftpd**

1. **Installation du paquet**  
   Pour installer **vsftpd** sur une machine Linux, exécutez la commande suivante selon votre distribution :

   - Sur une distribution **Debian/Ubuntu** :
     ```bash
     sudo apt update
     sudo apt install vsftpd
     ```
   - Sur une distribution **CentOS/Red Hat** :
     ```bash
     sudo dnf install vsftpd
     ```

2. **Démarrer et activer le service vsftpd**  
   Une fois l'installation terminée, démarrez le service **vsftpd** et configurez-le pour qu'il se lance au démarrage :
   ```bash
   sudo systemctl start vsftpd
   sudo systemctl enable vsftpd
   ```

3. **Vérification du service**  
   Vérifiez que **vsftpd** fonctionne correctement en utilisant la commande :
   ```bash
   sudo systemctl status vsftpd
   ```

---

#### 3. **Configuration du fichier `vsftpd.conf`**

Le fichier de configuration principal se trouve dans `/etc/vsftpd.conf`.

1. **Modifier le fichier de configuration**  
   Utilisez un éditeur de texte pour ouvrir et modifier le fichier :
   ```bash
   sudo nano /etc/vsftpd.conf
   ```

2. **Paramètres essentiels à configurer** :
   
   - **Permettre l'accès aux utilisateurs locaux** :
     ```conf
     local_enable=YES
     ```

   - **Autoriser l'upload de fichiers** :
     ```conf
     write_enable=YES
     ```

   - **Mode chroot pour limiter les utilisateurs à leur répertoire personnel** :
     ```conf
     chroot_local_user=YES
     ```

   - **Désactiver les connexions anonymes** :
     ```conf
     anonymous_enable=NO
     ```

   - **Définir un répertoire FTP spécifique pour un utilisateur** :
     ```conf
     local_root=/var/ftp/pub
     ```

   - **Activer les logs pour la sécurité** :
     ```conf
     xferlog_enable=YES
     ```

---

#### 4. **Création d'un utilisateur FTP**

1. **Créer un utilisateur FTP**  
   Créez un utilisateur système pour l'accès FTP :
   ```bash
   sudo useradd -m ftpuser
   sudo passwd ftpuser
   ```

2. **Définir un répertoire FTP pour l'utilisateur**  
   Par exemple, pour l'utilisateur **ftpuser**, créez un répertoire dédié et attribuez-lui les permissions :
   ```bash
   sudo mkdir -p /var/ftp/ftpuser
   sudo chown ftpuser:ftpuser /var/ftp/ftpuser
   ```

---

#### 5. **Redémarrer le serveur vsftpd**

Une fois les modifications effectuées, redémarrez le service pour appliquer les changements :
```bash
sudo systemctl restart vsftpd
```

---

#### 6. **Accès FTP depuis un client**

1. Utilisez un client FTP comme **FileZilla**, **WinSCP** ou même un client FTP en ligne de commande pour vous connecter au serveur FTP.
2. Entrez l'adresse IP du serveur, le nom d'utilisateur et le mot de passe pour accéder au serveur FTP.

---

#### 7. **Configuration du pare-feu**

Si vous utilisez un pare-feu, vous devrez autoriser les connexions FTP. Si vous utilisez **firewalld** :
```bash
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --reload
```

---

#### 8. **Sécuriser le serveur FTP avec SSL/TLS**

Pour sécuriser la connexion FTP, vous pouvez configurer SSL/TLS sur **vsftpd** afin de chiffrer les communications.

1. **Générer un certificat SSL** (si vous n'en avez pas déjà un) :
   ```bash
   sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt -days 365
   ```

2. **Activer SSL dans la configuration de vsftpd**  
   Modifiez `/etc/vsftpd.conf` pour activer SSL et définir les chemins vers les fichiers de certificat et de clé :
   ```conf
   ssl_enable=YES
   rsa_cert_file=/etc/ssl/certs/vsftpd.crt
   rsa_private_key_file=/etc/ssl/private/vsftpd.key
   ```

3. **Redémarrer vsftpd** :
   ```bash
   sudo systemctl restart vsftpd
   ```

---

#### 9. **Tester la connexion sécurisée**

Utilisez un client FTP compatible SSL/TLS (comme **FileZilla**) pour tester la connexion sécurisée via **FTPS**.

---

#### 10. **Accéder au serveur FTP**

Testez la connexion au serveur FTP via un client FTP ou directement en ligne de commande avec :

```bash
ftp localhost
```

---

### Conclusion

Vous avez maintenant configuré un serveur FTP sécurisé avec **vsftpd** sous Linux. Grâce à la configuration SSL/TLS, vos connexions FTP sont désormais chiffrées, ce qui améliore la sécurité des transferts de fichiers.