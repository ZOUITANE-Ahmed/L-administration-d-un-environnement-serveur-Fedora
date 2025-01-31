### **SELinux (Security-Enhanced Linux) : Introduction et Fonctionnement**  

**SELinux** est un mécanisme de contrôle d'accès de type **Mandatory Access Control (MAC)** intégré dans le noyau Linux. Il a été initialement développé par la **NSA** et vise à renforcer la sécurité du système en limitant les actions des utilisateurs et des processus.  

## 🔹 **Principes de SELinux**
Contrairement au modèle traditionnel **Discretionary Access Control (DAC)** de Linux (où les utilisateurs et les groupes définissent les permissions), SELinux impose des **politiques de sécurité strictes** indépendamment des permissions standard des fichiers.  

**Trois concepts clés :**
1. **Politiques** : Définissent les règles d'accès entre les objets (fichiers, processus, ports, etc.).
2. **Contexte de sécurité** : Chaque fichier, processus et port possède un contexte de sécurité avec des labels spécifiques.
3. **Modes d'exécution** :  
   - **Enforcing** : Applique les règles de sécurité et bloque les actions interdites.  
   - **Permissive** : Journalise les violations sans les bloquer (utile pour le debug).  
   - **Disabled** : Désactive complètement SELinux.  

## 🔹 **Vérifier et Gérer l'État de SELinux**
- **Vérifier le statut de SELinux :**  
  ```bash
  sestatus
  ```
- **Afficher le mode actuel :**  
  ```bash
  getenforce
  ```
- **Changer temporairement le mode (sans redémarrer) :**  
  ```bash
  setenforce 0   # Mode permissive
  setenforce 1   # Mode enforcing
  ```
- **Modifier la configuration permanente :**  
  Fichier : `/etc/selinux/config`  
  ```bash
  SELINUX=enforcing  # enforcing | permissive | disabled
  ```
  Puis redémarrer le système pour appliquer les changements.

## 🔹 **Gestion des Contextes de Sécurité**
Chaque fichier sous SELinux possède un **contexte de sécurité** visible avec la commande :
```bash
ls -Z /var/www/html/
```
Exemple de sortie :
```
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 index.html
```
- `unconfined_u` → Utilisateur SELinux  
- `object_r` → Type d'objet  
- `httpd_sys_content_t` → Type du fichier  
- `s0` → Niveau de sensibilité  

### **Changer un contexte SELinux**
Si un service comme Apache ne peut pas accéder à un fichier à cause de SELinux, on peut changer son type :
```bash
chcon -t httpd_sys_content_t /var/www/html/index.html
```
Ou appliquer de manière persistante :
```bash
semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
restorecon -Rv /var/www/html
```

## 🔹 **Audit et Débogage**
Si un programme est bloqué par SELinux, on peut analyser les logs :
```bash
cat /var/log/audit/audit.log | grep AVC
```
Ou utiliser `audit2why` et `audit2allow` :
```bash
cat /var/log/audit/audit.log | audit2why
cat /var/log/audit/audit.log | audit2allow -M mypolicy
semodule -i mypolicy.pp
```

---

SELinux est une **couche de sécurité essentielle** pour protéger un système Linux, mais il peut être complexe à configurer. En cas de problème, utiliser le **mode permissive** pour identifier les règles à adapter avant de repasser en mode enforcing. 🚀