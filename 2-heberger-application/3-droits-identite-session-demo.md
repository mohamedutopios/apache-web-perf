### **Démonstration : Droits dédiés et gestion des sessions PHP pour deux applications avec Apache et PHP-FPM**

#### **Contexte :**
Vous souhaitez :
1. Héberger deux applications PHP distinctes.
2. Isoler les deux applications avec des **identités dédiées** (utilisateurs distincts).
3. Gérer les **sessions PHP** de manière séparée pour éviter tout mélange ou interférence entre les applications.

---

### **1. Préparation de l’environnement**

#### **Installer les composants nécessaires**
1. Installez Apache et PHP avec PHP-FPM (deux versions différentes si nécessaire) :
   ```bash
   sudo apt update
   sudo apt install apache2 php7.4 php7.4-fpm php8.1 php8.1-fpm
   ```

2. Activez les modules Apache nécessaires :
   ```bash
   sudo a2enmod proxy_fcgi setenvif actions
   sudo systemctl reload apache2
   ```

---

### **2. Créer des identités dédiées pour les applications**

#### **Créer des utilisateurs système**
1. Créez un utilisateur pour chaque application :
   ```bash
   sudo adduser --system --no-create-home --group app1
   sudo adduser --system --no-create-home --group app2
   ```

#### **Configurer les répertoires des applications**
1. Créez les répertoires pour chaque application :
   ```bash
   sudo mkdir -p /var/www/app1
   sudo mkdir -p /var/www/app2
   ```

2. Changez les propriétaires pour correspondre aux utilisateurs créés :
   ```bash
   sudo chown -R app1:app1 /var/www/app1
   sudo chown -R app2:app2 /var/www/app2
   ```

---

### **3. Configurer PHP-FPM avec des pools dédiés**

1. **Créer des fichiers de configuration pour chaque pool PHP-FPM**.

- **Application 1** : `/etc/php/7.4/fpm/pool.d/app1.conf`
  ```ini
  [app1]
  user = app1
  group = app1
  listen = /run/php/php7.4-app1.sock
  listen.owner = www-data
  listen.group = www-data
  listen.mode = 0660
  php_value[session.save_path] = /var/lib/php/sessions/app1
  ```

- **Application 2** : `/etc/php/8.1/fpm/pool.d/app2.conf`
  ```ini
  [app2]
  user = app2
  group = app2
  listen = /run/php/php8.1-app2.sock
  listen.owner = www-data
  listen.group = www-data
  listen.mode = 0660
  php_value[session.save_path] = /var/lib/php/sessions/app2
  ```

2. **Créer les répertoires pour les sessions PHP** :
   ```bash
   sudo mkdir -p /var/lib/php/sessions/app1
   sudo mkdir -p /var/lib/php/sessions/app2
   sudo chown -R app1:app1 /var/lib/php/sessions/app1
   sudo chown -R app2:app2 /var/lib/php/sessions/app2
   sudo chmod 700 /var/lib/php/sessions/app1 /var/lib/php/sessions/app2
   ```

3. **Redémarrer PHP-FPM** :
   ```bash
   sudo systemctl restart php7.4-fpm
   sudo systemctl restart php8.1-fpm
   ```

---

### **4. Configurer Apache avec des Virtual Hosts**

1. **Configurer le Virtual Host pour l'application 1**
   Fichier : `/etc/apache2/sites-available/app1.conf`
   ```apache
   <VirtualHost *:80>
       ServerName app1.example.com
       DocumentRoot /var/www/app1

       <Directory /var/www/app1>
           AllowOverride All
           Require all granted
       </Directory>

       <FilesMatch \.php$>
           SetHandler "proxy:unix:/run/php/php7.4-app1.sock|fcgi://localhost"
       </FilesMatch>

       ErrorLog ${APACHE_LOG_DIR}/app1-error.log
       CustomLog ${APACHE_LOG_DIR}/app1-access.log combined
   </VirtualHost>
   ```

2. **Configurer le Virtual Host pour l'application 2**
   Fichier : `/etc/apache2/sites-available/app2.conf`
   ```apache
   <VirtualHost *:80>
       ServerName app2.example.com
       DocumentRoot /var/www/app2

       <Directory /var/www/app2>
           AllowOverride All
           Require all granted
       </Directory>

       <FilesMatch \.php$>
           SetHandler "proxy:unix:/run/php/php8.1-app2.sock|fcgi://localhost"
       </FilesMatch>

       ErrorLog ${APACHE_LOG_DIR}/app2-error.log
       CustomLog ${APACHE_LOG_DIR}/app2-access.log combined
   </VirtualHost>
   ```

3. **Activer les Virtual Hosts et redémarrer Apache** :
   ```bash
   sudo a2ensite app1.conf app2.conf
   sudo systemctl reload apache2
   ```

---

### **5. Tester les applications**

1. **Créer des fichiers de test pour chaque application** :
   - **Application 1** :
     ```bash
     echo "<?php session_start(); echo 'App1 Session ID: ' . session_id(); ?>" | sudo tee /var/www/app1/index.php
     ```

   - **Application 2** :
     ```bash
     echo "<?php session_start(); echo 'App2 Session ID: ' . session_id(); ?>" | sudo tee /var/www/app2/index.php
     ```

2. **Ajouter les noms de domaine au fichier `hosts`** :
   ```bash
   sudo nano /etc/hosts
   ```
   Ajoutez les lignes :
   ```
   127.0.0.1 app1.example.com
   127.0.0.1 app2.example.com
   ```

3. **Accéder aux applications** :
   - Visitez `http://app1.example.com` :
     - Vous verrez `App1 Session ID: [session ID unique]`.
   - Visitez `http://app2.example.com` :
     - Vous verrez `App2 Session ID: [session ID unique]`.

4. **Vérifier les fichiers de session** :
   - Les fichiers de session de l'application 1 sont stockés dans `/var/lib/php/sessions/app1`.
   - Les fichiers de session de l'application 2 sont stockés dans `/var/lib/php/sessions/app2`.

---

### **Résultat**

1. **Isolation des identités** :
   - Les applications fonctionnent sous des utilisateurs dédiés (`app1`, `app2`).
   - Les répertoires et ressources sont isolés.

2. **Gestion séparée des sessions** :
   - Chaque application stocke ses sessions dans son propre répertoire.
   - Les sessions sont totalement indépendantes.

3. **Virtual Hosts distincts** :
   - Les applications utilisent des configurations spécifiques avec différentes versions de PHP (7.4 et 8.1).

---

### **Sécurisation supplémentaire**
- **HTTPS** : Configurez HTTPS pour les Virtual Hosts en activant `mod_ssl` et en utilisant des certificats SSL.
- **Renforcement des sessions PHP** :
  Modifiez les paramètres dans `php.ini` :
  ```ini
  session.cookie_httponly = 1
  session.cookie_secure = 1
  session.use_strict_mode = 1
  ```

Avec cette configuration, vos applications sont isolées, sécurisées et optimisées pour un hébergement robuste.