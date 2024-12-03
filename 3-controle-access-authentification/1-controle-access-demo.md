### **Démonstration complète et réaliste : Contrôle d'accès avec pages d'erreur personnalisées et redirection**

Dans cette démonstration, nous allons configurer un serveur Apache pour :
1. Gérer un contrôle d'accès avancé avec les modules `mod_authz*`.
2. Protéger une application web via HTTPS.
3. Utiliser des pages d'erreur personnalisées pour améliorer l'expérience utilisateur.
4. Ajouter des règles d'accès spécifiques (groupes, adresses IP).

---

## **Contexte du scénario**

Nous configurons un Virtual Host pour protéger une application dans le répertoire `/var/www/secure-app`. L'accès sera limité :
1. **Aux utilisateurs authentifiés appartenant aux groupes `admin` ou `developer`.**
2. **À des IP spécifiques (`192.168.1.0/24`) pour les utilisateurs non authentifiés.**
3. **En cas d'accès refusé, les utilisateurs seront redirigés vers des pages d'erreur personnalisées.**

---

## **Étape 1 : Activer les modules nécessaires**

1. **Vérifiez les modules activés :**
   ```bash
   apache2ctl -M
   ```

2. **Activez les modules requis (si besoin) :**
   ```bash
   sudo a2enmod authz_host authz_user authz_groupfile authz_core ssl
   sudo systemctl restart apache2
   ```

---

## **Étape 2 : Configurer les utilisateurs et groupes**

1. **Créer un fichier de mots de passe utilisateur :**
   - Utilisez `htpasswd` pour ajouter des utilisateurs :
     ```bash
     sudo htpasswd -c /etc/apache2/.htpasswd john
     sudo htpasswd /etc/apache2/.htpasswd alice
     ```

2. **Créer un fichier de groupes :**
   - Fichier : `/etc/apache2/groups`
     ```text
     admin: john
     developer: alice
     ```

3. **Configurer les permissions :**
   - Assurez-vous que les fichiers sont lisibles par Apache :
     ```bash
     sudo chown root:www-data /etc/apache2/.htpasswd /etc/apache2/groups
     sudo chmod 640 /etc/apache2/.htpasswd /etc/apache2/groups
     ```

---

## **Étape 3 : Configurer les pages d’erreur personnalisées**

1. **Créer un répertoire pour les pages d’erreur :**
   ```bash
   sudo mkdir -p /var/www/secure-app/error_pages
   ```

2. **Créer des fichiers pour chaque erreur :**
   - **403 Forbidden (`/var/www/secure-app/error_pages/403.html`)**
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>403 Forbidden</title>
     </head>
     <body>
         <h1>403 - Access Denied</h1>
         <p>You are not authorized to access this resource.</p>
     </body>
     </html>
     ```

   - **404 Not Found (`/var/www/secure-app/error_pages/404.html`)**
     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>404 Not Found</title>
     </head>
     <body>
         <h1>404 - Page Not Found</h1>
         <p>The requested resource could not be found.</p>
     </body>
     </html>
     ```

3. **Assurez les permissions :**
   ```bash
   sudo chown -R www-data:www-data /var/www/secure-app/error_pages
   sudo chmod -R 755 /var/www/secure-app/error_pages
   ```

---

## **Étape 4 : Configurer le Virtual Host**

1. **Créer un Virtual Host avec contrôle d’accès et redirection des erreurs :**
   - Fichier : `/etc/apache2/sites-available/secure-app.conf`
     ```apache
     <VirtualHost *:443>
         ServerName secure.example.com
         DocumentRoot "/var/www/secure-app"

         SSLEngine On
         SSLCertificateFile "/etc/ssl/certs/example.com.crt"
         SSLCertificateKeyFile "/etc/ssl/private/example.com.key"

         # Configuration d'accès au répertoire sécurisé
         <Directory "/var/www/secure-app">
             Options Indexes FollowSymLinks
             AllowOverride None

             # Authentification et règles d'accès
             AuthType Basic
             AuthName "Restricted Access"
             AuthUserFile /etc/apache2/.htpasswd
             AuthGroupFile /etc/apache2/groups

             # Règles combinées
             <RequireAny>
                 Require group admin
                 Require group developer
                 Require ip 192.168.1.0/24
             </RequireAny>
         </Directory>

         # Pages d'erreur personnalisées
         ErrorDocument 403 /error_pages/403.html
         ErrorDocument 404 /error_pages/404.html

         ErrorLog ${APACHE_LOG_DIR}/secure-app-error.log
         CustomLog ${APACHE_LOG_DIR}/secure-app-access.log combined
     </VirtualHost>
     ```

2. **Activer le site et redémarrer Apache :**
   ```bash
   sudo a2ensite secure-app.conf
   sudo systemctl reload apache2
   ```

---

## **Étape 5 : Tester la configuration**

### **a. Vérifier l’accès utilisateur**
1. Accédez à `https://secure.example.com`.
2. **Résultat attendu :**
   - Les utilisateurs non authentifiés reçoivent une invite de connexion.
   - Les utilisateurs `john` et `alice` peuvent se connecter.
   - Les utilisateurs des groupes `admin` et `developer` ont accès.

### **b. Tester les pages d'erreur**
1. Simulez un accès non autorisé pour voir la page **403 Forbidden** :
   - Essayez de vous connecter avec un utilisateur non défini.
2. Essayez d'accéder à une page inexistante pour voir la page **404 Not Found** :
   - Exemples : `https://secure.example.com/nonexistent`.

### **c. Vérifier les journaux**
- Vérifiez les connexions et les erreurs dans les fichiers de log :
   ```bash
   sudo tail -f /var/log/apache2/secure-app-access.log
   sudo tail -f /var/log/apache2/secure-app-error.log
   ```

---

## **Résumé des étapes**

1. **Modules activés :** `mod_authz_host`, `mod_authz_user`, `mod_authz_groupfile`, `ssl`.
2. **Contrôle d’accès :**
   - Basé sur les groupes définis dans `/etc/apache2/groups`.
   - Basé sur des plages IP spécifiques.
3. **Pages d’erreur personnalisées :**
   - Situées dans `/var/www/secure-app/error_pages`.
   - Spécifiées avec `ErrorDocument`.
4. **HTTPS activé :**
   - Assure un transfert sécurisé des données.

---

Cette configuration offre une solution robuste et sécurisée pour gérer les droits d’accès tout en fournissant une meilleure expérience utilisateur grâce aux pages d'erreur personnalisées et aux journaux détaillés pour le suivi des accès. 😊