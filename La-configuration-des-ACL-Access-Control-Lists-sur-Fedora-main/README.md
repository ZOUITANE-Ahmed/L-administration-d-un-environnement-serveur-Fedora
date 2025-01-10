# La-configuration-des-ACL-Access-Control-Lists-sur-Fedora

Voici une synthèse détaillée des concepts, commandes et bonnes pratiques pour la gestion des ACL sur Fedora, en se basant sur les informations fournies :  

---

### **Permissions de Base**
- **Lecture (`r`)** : Lire le contenu d'un fichier ou lister un répertoire.
- **Écriture (`w`)** : Modifier un fichier ou le contenu d'un répertoire.
- **Exécution (`x`)** : Exécuter un fichier ou accéder à un répertoire.

### **Catégories d'Utilisateurs**
- **Propriétaire (`u`)** : Le créateur ou propriétaire du fichier.
- **Groupe (`g`)** : Groupe associé au fichier.
- **Autres (`o`)** : Tous les utilisateurs restants.

---

### **Gestion des ACL sur Fedora**

#### **Installation des outils ACL**
1. Installez le package ACL si nécessaire :
   ```bash
   sudo dnf install acl
   ```

#### **Visualisation des ACL**
- Afficher les ACL d'un fichier ou répertoire :
  ```bash
  getfacl fichier_ou_répertoire
  ```

#### **Configuration des ACL**
1. **Accorder des permissions spécifiques :**
   - À un utilisateur :
     ```bash
     setfacl -m u:<utilisateur>:<permissions> fichier
     ```
     Exemple :
     ```bash
     setfacl -m u:youssef:rw rapport.txt
     ```
   - À un groupe :
     ```bash
     setfacl -m g:<groupe>:<permissions> fichier
     ```
     Exemple :
     ```bash
     setfacl -m g:finance:x calcul.sh
     ```

2. **Appliquer les ACL de manière récursive :**
   ```bash
   setfacl -R -m u:<utilisateur>:<permissions> répertoire
   ```
   Exemple :
   ```bash
   setfacl -R -m u:fatima:rwx données
   ```

3. **Configurer les ACL par défaut :**
   - Ces ACL s'appliquent aux nouveaux fichiers ou répertoires créés dans un répertoire.
     ```bash
     setfacl -m d:u:<utilisateur>:<permissions> répertoire
     ```
     Exemple :
     ```bash
     setfacl -m d:u:karim:rw projet
     ```

#### **Comprendre et gérer le masque**
- **Visualiser le masque :**
  ```bash
  getfacl fichier.txt
  ```
- **Modifier le masque :**
  ```bash
  setfacl -m m::rw fichier.txt
  ```

---

### **Sauvegarde et Restauration des ACL**
1. **Sauvegarder les ACL :**
   - D'un fichier :
     ```bash
     getfacl fichier > savacl.txt
     ```
   - D'un répertoire (de manière récursive) :
     ```bash
     getfacl -R répertoire > savacl.txt
     ```

2. **Restaurer les ACL :**
   ```bash
   setfacl --restore=savacl.txt
   ```

---

### **Duplication des ACL**
1. **Sauvegarder les ACL d’un répertoire source :**
   ```bash
   getfacl -R source_repertoire > savacl.txt
   ```
2. **Modifier le chemin dans le fichier de sauvegarde :**
   ```bash
   sed -i -e "s/source_repertoire/destination_repertoire/g" savacl.txt
   ```
3. **Appliquer les ACL au répertoire cible :**
   ```bash
   setfacl --restore=savacl.txt
   ```

4. **Appliquer directement sans sauvegarde intermédiaire :**
   ```bash
   getfacl -R source_repertoire | setfacl --set-file=- destination_repertoire
   ```

---

### **Archivage avec support des ACL**
- **Avec `tar` :**
  ```bash
  tar --acls -cvf archive.tar répertoire
  ```
- **Avec `star` :**
  ```bash
  star -cvf archive.tar répertoire
  ```

---

### **Avantages des ACL**
- **Granularité** : Définit des permissions spécifiques.
- **Flexibilité** : Simplifie la gestion dans des environnements complexes.
- **Sécurité** : Renforce le contrôle d'accès en limitant les permissions aux seuls utilisateurs ou groupes nécessaires.

Si vous avez besoin d'exemples supplémentaires ou de conseils pour une configuration spécifique, je suis là pour vous aider !
