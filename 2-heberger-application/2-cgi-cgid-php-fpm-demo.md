### **Démonstration pratique : Hébergement d'applications PHP avec PHP-FPM et Apache**

Cette démonstration explique comment configurer Apache pour utiliser **PHP-FPM** via **FastCGI**, permettant d'exécuter des scripts PHP de manière performante.

---

### **Contexte**
- Vous hébergez deux applications :
  1. Une application utilisant **PHP 7.4**.
  2. Une autre application utilisant **PHP 8.1**.
- Vous configurez Apache pour utiliser PHP-FPM, avec des Virtual Hosts distincts pour chaque application.

---

### **Étape 1 : Installer les prérequis**

1. **Installer Apache**
   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. **Installer PHP et PHP-FPM**
   - Installez PHP 7.4 et PHP 8.1 avec PHP-FPM :
     ```bash
     sudo apt install php7.4 php7.4-fpm php8.1 php8.1-fpm
     ```

3. **Activer les modules Apache nécessaires**
   ```bash
   sudo a2enmod proxy_fcgi setenvif
   sudo a2enmod actions
   sudo systemctl restart apache2
   ```

---

### **Étape 2 : Configurer PHP-FPM**

1. **Configurer les pools pour PHP 7.4 et PHP 8.1**

   - Par défaut, PHP-FPM pour chaque version est configuré dans `/etc/php/7.4/fpm/pool.d/www.conf` et `/etc/php/8.1/fpm/pool.d/www.conf`.
   - Assurez-vous que chaque pool écoute sur un socket unique.

   Exemple pour **PHP 7.4** (`/etc/php/7.4/fpm/pool.d/www.conf`) :
   ```ini
   [www]
   listen = /run/php/php7.4-fpm.sock
   ```

   Exemple pour **PHP 8.1** (`/etc/php/8.1/fpm/pool.d/www.conf`) :
   ```ini
   [www]
   listen = /run/php/php8.1-fpm.sock
   ```

2. **Redémarrer les services PHP-FPM**
   ```bash
   sudo systemctl restart php7.4-fpm
   sudo systemctl restart php8.1-fpm
   ```

---

### **Étape 3 : Configurer Apache avec des Virtual Hosts**

1. **Créer un Virtual Host pour l’application PHP 7.4**
   Créez un fichier de configuration dans `/etc/apache2/sites-available/php74.example.com.conf` :
   ```apache
   <VirtualHost *:80>
       ServerName php74.example.com
       DocumentRoot /var/www/php74

       <Directory /var/www/php74>
           AllowOverride All
           Require all granted
       </Directory>

       <FilesMatch \.php$>
           SetHandler "proxy:unix:/run/php/php7.4-fpm.sock|fcgi://localhost"
       </FilesMatch>

       ErrorLog ${APACHE_LOG_DIR}/php74-error.log
       CustomLog ${APACHE_LOG_DIR}/php74-access.log combined
   </VirtualHost>
   ```

2. **Créer un Virtual Host pour l’application PHP 8.1**
   Créez un fichier de configuration dans `/etc/apache2/sites-available/php81.example.com.conf` :
   ```apache
   <VirtualHost *:80>
       ServerName php81.example.com
       DocumentRoot /var/www/php81

       <Directory /var/www/php81>
           AllowOverride All
           Require all granted
       </Directory>

       <FilesMatch \.php$>
           SetHandler "proxy:unix:/run/php/php8.1-fpm.sock|fcgi://localhost"
       </FilesMatch>

       ErrorLog ${APACHE_LOG_DIR}/php81-error.log
       CustomLog ${APACHE_LOG_DIR}/php81-access.log combined
   </VirtualHost>
   ```

3. **Activer les Virtual Hosts**
   ```bash
   sudo a2ensite php74.example.com.conf
   sudo a2ensite php81.example.com.conf
   sudo systemctl reload apache2
   ```

---

### **Étape 4 : Créer les répertoires des applications**

1. **Créer les dossiers pour les applications**
   ```bash
   sudo mkdir -p /var/www/php74
   sudo mkdir -p /var/www/php81
   ```

2. **Ajouter un fichier `index.php` de test**
   - Pour PHP 7.4 :
     ```bash
     echo "<?php phpinfo(); ?>" | sudo tee /var/www/php74/index.php
     ```
   - Pour PHP 8.1 :
     ```bash
     echo "<?php phpinfo(); ?>" | sudo tee /var/www/php81/index.php
     ```

3. **Ajuster les permissions**
   ```bash
   sudo chown -R www-data:www-data /var/www/php74 /var/www/php81
   sudo chmod -R 755 /var/www/php74 /var/www/php81
   ```

---

### **Étape 5 : Tester la configuration**

1. **Modifier votre fichier `hosts`**
   Ajoutez les entrées suivantes dans `/etc/hosts` pour simuler les noms de domaine :
   ```
   127.0.0.1 php74.example.com
   127.0.0.1 php81.example.com
   ```

2. **Accéder aux Virtual Hosts**
   - Naviguez vers [http://php74.example.com](http://php74.example.com).
     - Vous devriez voir la page PHP Info avec la version **PHP 7.4**.
   - Naviguez vers [http://php81.example.com](http://php81.example.com).
     - Vous devriez voir la page PHP Info avec la version **PHP 8.1**.

---

### **Résumé des configurations**

1. **PHP-FPM** :
   - Deux versions de PHP (7.4 et 8.1) avec des pools distincts.
   - Chaque pool écoute sur un socket unique.

2. **Apache** :
   - Virtual Hosts distincts configurés pour chaque version PHP.
   - Utilisation de `proxy_fcgi` pour déléguer l’exécution des scripts PHP à PHP-FPM.

3. **Résultats** :
   - Applications PHP peuvent cohabiter sur le même serveur avec des versions PHP différentes.

---

### **Avantages de cette approche**

- **Isolation** : Chaque application utilise sa propre version de PHP.
- **Performance** : PHP-FPM permet une gestion performante des requêtes grâce à des processus persistants.
- **Flexibilité** : Facile à étendre si de nouvelles applications nécessitent d'autres versions de PHP.

Avec cette configuration, vous êtes prêt à héberger plusieurs applications PHP sur le même serveur tout en utilisant différentes versions de PHP pour chaque application.