Voici une version corrigée et améliorée des étapes pour configurer un serveur LDAP, ajouter des utilisateurs et des groupes, et l'intégrer avec Apache pour une authentification sécurisée.

---

## **1. Configurer un serveur LDAP Dockerisé**

### **a. Installer Docker**
Si Docker n'est pas installé :
- **Sur Debian/Ubuntu :**
  ```bash
  sudo apt update
  sudo apt install docker.io
  sudo systemctl start docker
  sudo systemctl enable docker
  ```

- **Sur CentOS/RHEL :**
  ```bash
  sudo yum install docker
  sudo systemctl start docker
  sudo systemctl enable docker
  ```

### **b. Démarrer un conteneur LDAP**
Exécutez l'image Docker **`osixia/openldap`** :
```bash
docker run -d \
  --name ldap-server \
  --hostname ldap.example.com \
  -p 389:389 -p 636:636 \
  -e LDAP_ORGANISATION="Utopios" \
  -e LDAP_DOMAIN="example.com" \
  -e LDAP_ADMIN_PASSWORD="admin" \
  osixia/openldap:1.5.0
```

---

## **2. Ajouter des unités organisationnelles, utilisateurs et groupes**

### **a. Créer des unités organisationnelles**
1. Créez un fichier `add-ou.ldif` :
   ```ldif
   dn: ou=users,dc=example,dc=com
   objectClass: organizationalUnit
   ou: users

   dn: ou=groups,dc=example,dc=com
   objectClass: organizationalUnit
   ou: groups
   ```

2. Ajoutez ces OUs au serveur LDAP :
   ```bash
   ldapadd -x -H ldap://localhost:389 -D "cn=admin,dc=example,dc=com" -w admin -f add-ou.ldif
   ```

### **b. Ajouter un utilisateur**
1. Créez un fichier `add-user.ldif` :
   ```ldif
   dn: uid=john,ou=users,dc=example,dc=com
   objectClass: inetOrgPerson
   objectClass: posixAccount
   objectClass: top
   cn: John Doe
   sn: Doe
   uid: john
   uidNumber: 1000
   gidNumber: 1000
   homeDirectory: /home/john
   userPassword: {SSHA}hashed_password_here
   ```

2. Générez un mot de passe haché :
   ```bash
   slappasswd
   ```
   Copiez le résultat (par exemple : `{SSHA}VcJN...`) et remplacez **`hashed_password_here`** dans le fichier LDIF.

3. Ajoutez l'utilisateur au serveur LDAP :
   ```bash
   ldapadd -x -H ldap://localhost:389 -D "cn=admin,dc=example,dc=com" -w admin -f add-user.ldif
   ```

### **c. Ajouter un groupe**
1. Créez un fichier `add-group.ldif` :
   ```ldif
   dn: cn=admins,ou=groups,dc=example,dc=com
   objectClass: top
   objectClass: posixGroup
   cn: admins
   gidNumber: 1000
   memberUid: john
   ```

2. Ajoutez le groupe au serveur LDAP :
   ```bash
   ldapadd -x -H ldap://localhost:389 -D "cn=admin,dc=example,dc=com" -w admin -f add-group.ldif
   ```

### **d. Vérifier les ajouts**
- Vérifiez l'utilisateur :
  ```bash
  ldapsearch -x -H ldap://localhost:389 -D "cn=admin,dc=example,dc=com" -w admin -b "ou=users,dc=example,dc=com" "(uid=john)"
  ```
- Vérifiez le groupe :
  ```bash
  ldapsearch -x -H ldap://localhost:389 -D "cn=admin,dc=example,dc=com" -w admin -b "ou=groups,dc=example,dc=com" "(cn=admins)"
  ```

---

## **3. Configurer Apache pour l'authentification LDAP**

### **a. Ajouter un Virtual Host avec l'authentification LDAP**
Créez un fichier `/etc/apache2/sites-available/protected-app.conf` :
```apache
<VirtualHost *:443>
    ServerName protected.example.com
    DocumentRoot "/var/www/protected"

    SSLEngine On
    SSLCertificateFile "/etc/ssl/example.crt"
    SSLCertificateKeyFile "/etc/ssl/example.key"

    <Directory "/var/www/protected">
        AuthType Basic
        AuthName "LDAP Authentication"
        AuthBasicProvider ldap
        AuthLDAPURL "ldap://localhost:389/ou=users,dc=example,dc=com?uid?sub?(objectClass=inetOrgPerson)"
        AuthLDAPBindDN "cn=admin,dc=example,dc=com"
        AuthLDAPBindPassword "admin"

        Require ldap-group cn=admins,ou=groups,dc=example,dc=com
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/secure-error.log
    CustomLog ${APACHE_LOG_DIR}/secure-access.log combined
</VirtualHost>
```

### **b. Activer le site et SSL**
1. Activez le site et le module SSL :
   ```bash
   sudo a2ensite protected-app.conf
   sudo a2enmod ssl
   sudo systemctl reload apache2
   ```

2. Assurez-vous que les certificats SSL sont valides. Vous pouvez générer des certificats auto-signés pour les tests :
   ```bash
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
       -keyout /etc/ssl/example.key -out /etc/ssl/example.crt \
       -subj "/CN=protected.example.com"
   ```

---

## **4. Tester l'accès**

### **a. Ajouter une entrée dans le fichier `hosts`**
Ajoutez cette ligne à votre fichier `/etc/hosts` pour pointer `protected.example.com` vers localhost :
```plaintext
127.0.0.1 protected.example.com
```

### **b. Accéder au site**
1. Ouvrez un navigateur et accédez à `https://protected.example.com`.
2. Une boîte de dialogue s'affichera pour demander un nom d'utilisateur et un mot de passe.
3. Connectez-vous avec :
   - **Nom d'utilisateur** : `john`
   - **Mot de passe** : (le mot de passe défini pour `john`)

---

## **5. Résultats attendus**

- **Accès autorisé** : Si l'utilisateur appartient au groupe LDAP `admins`, l'accès est autorisé.
- **Accès refusé** : Si l'utilisateur ne fait pas partie du groupe, l'accès est refusé.

---

## **6. Nettoyage**

Pour supprimer le conteneur après la démonstration :
```bash
docker rm -f ldap-server
```

---

### **Résumé**

- Vous avez configuré un serveur LDAP Dockerisé.
- Vous avez ajouté des utilisateurs et des groupes dans LDAP.
- Vous avez intégré LDAP avec Apache pour sécuriser un Virtual Host.
- Vous pouvez tester les permissions en fonction des utilisateurs et des groupes LDAP.

Si vous rencontrez des erreurs, partagez les journaux spécifiques pour des conseils supplémentaires ! 😊