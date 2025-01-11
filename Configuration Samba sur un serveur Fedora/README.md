**Samba** est une implémentation open-source du protocole **SMB (Server Message Block)**, permettant de partager des fichiers et des imprimantes entre différents systèmes d'exploitation, principalement entre Linux/Unix et Windows. Samba permet à un système Linux de fonctionner comme un serveur de fichiers ou d'imprimantes, offrant ainsi une compatibilité avec les clients Windows et d'autres systèmes qui utilisent SMB/CIFS (Common Internet File System).

### Fonctionnalités principales de Samba :
1. **Partage de fichiers** : Samba permet à un serveur Linux de partager des répertoires et des fichiers avec des clients Windows, tout en gérant les autorisations d'accès.
2. **Partage d'imprimantes** : Il permet également de partager des imprimantes entre des systèmes Linux et Windows.
3. **Contrôleur de domaine** : Samba peut être configuré pour agir comme un contrôleur de domaine Windows (Active Directory) sur un serveur Linux.
4. **Accès aux fichiers Windows** : Samba permet à un client Linux d'accéder à des fichiers et répertoires partagés sur un serveur Windows.
5. **Interopérabilité** : Grâce à Samba, les systèmes Linux peuvent communiquer et partager des ressources avec les systèmes Windows de manière transparente.


---

Voici une procédure de base pour configurer un serveur Samba sur **Fedora** et accéder à celui-ci depuis des clients **Windows** et **Linux**.

### Étape 1 : Installer Samba sur le serveur Fedora

1. **Installer Samba**
   Sur le serveur Red Hat, vous devez d'abord installer Samba en utilisant la commande suivante :

   ```bash
   sudo dnf install samba samba-client samba-common
   ```

2. **Vérifier l'installation**
   Pour vérifier si Samba est installé correctement, utilisez cette commande :

   ```bash
   samba -V
   ```

3. **Activer et démarrer le service Samba**

   Une fois Samba installé, activez et démarrez les services nécessaires avec les commandes suivantes :

   ```bash
   sudo systemctl enable smb
   sudo systemctl start smb
   sudo systemctl enable nmb
   sudo systemctl start nmb
   ```

### Étape 2 : Configurer le fichier `smb.conf`

1. **Éditer le fichier de configuration**
   Le fichier principal de configuration de Samba est `/etc/samba/smb.conf`. Ouvrez-le avec un éditeur de texte, par exemple `vi` ou `nano` :

   ```bash
   sudo nano /etc/samba/smb.conf
   ```

2. **Exemple de configuration**
   Ajoutez ou modifiez les sections suivantes dans le fichier `smb.conf` :

   ```ini
   [global]
      workgroup = WORKGROUP
      server string = Serveur Samba
      netbios name = SAMBASERVER
      security = user
      map to guest = bad user
      dns proxy = no

   [partage]
      path = /srv/samba/partage
      browsable = yes
      writable = yes
      guest ok = yes
      read only = no
   ```

   - **workgroup** : Nom du groupe de travail (assurez-vous qu'il correspond à celui des clients Windows).
   - **netbios name** : Le nom du serveur Samba.
   - **path** : Le chemin vers le dossier que vous voulez partager.
   - **guest ok** : Permet l'accès en tant qu'invité, ce qui permet aux utilisateurs de se connecter sans fournir de mot de passe.

3. **Créer le répertoire à partager**

   Créez le répertoire que vous souhaitez partager, et donnez-lui les permissions appropriées :

   ```bash
   sudo mkdir -p /srv/samba/partage
   sudo chmod -R 0777 /srv/samba/partage
   ```

### Étape 3 : Ajouter un utilisateur Samba

Pour autoriser un utilisateur à accéder à un partage Samba, vous devez d'abord créer un utilisateur Linux, puis ajouter cet utilisateur à Samba.

1. **Créer un utilisateur Linux**
   Si l'utilisateur n'existe pas déjà, créez un utilisateur avec la commande suivante :

   ```bash
   sudo useradd utilisateur
   sudo passwd utilisateur
   ```

2. **Ajouter l'utilisateur à Samba**

   Ajoutez l'utilisateur à la base de données Samba avec la commande suivante :

   ```bash
   sudo smbpasswd -a utilisateur
   ```

   Activez ensuite l'utilisateur Samba :

   ```bash
   sudo smbpasswd -e utilisateur
   ```

### Étape 4 : Vérifier et ouvrir les ports nécessaires

1. **Vérifier le statut du pare-feu**

   Vérifiez si le pare-feu bloque les ports utilisés par Samba (généralement les ports TCP 137-139 et 445).

   ```bash
   sudo firewall-cmd --zone=public --add-service=samba --permanent
   sudo firewall-cmd --reload
   ```

2. **Vérifier les services Samba**

   Assurez-vous que les services Samba sont actifs avec :

   ```bash
   sudo systemctl status smb
   sudo systemctl status nmb
   ```

### Étape 5 : Accéder au partage Samba depuis un client Windows

1. **Accéder via l'Explorateur Windows**
   - Ouvrez l'explorateur de fichiers sur Windows.
   - Tapez `\\SAMBASERVER\partage` dans la barre d'adresse, en remplaçant `SAMBASERVER` par le nom de votre serveur Samba.
   - Entrez les informations d'identification lorsque vous y êtes invité (nom d'utilisateur et mot de passe Samba).

### Étape 6 : Accéder au partage Samba depuis un client Linux

1. **Monter le partage Samba**
   Pour accéder au partage Samba depuis un client Linux, vous pouvez utiliser la commande `mount` ou un gestionnaire de fichiers graphique.

   Exemple pour monter manuellement avec `mount` :

   ```bash
   sudo mount -t cifs //SAMBASERVER/partage /mnt/samba -o username=utilisateur,password=motdepasse
   ```

   Remplacez `SAMBASERVER`, `partage`, `utilisateur` et `motdepasse` par les valeurs appropriées.

   Vous pouvez également ajouter l'entrée au fichier `/etc/fstab` pour monter automatiquement le partage au démarrage.

### Conclusion

Cela couvre les étapes de base pour configurer un serveur Samba sur Red Hat et accéder aux partages depuis des clients Windows et Linux. N'oubliez pas de toujours tester votre configuration avec la commande `testparm` pour vérifier les erreurs dans le fichier de configuration Samba.