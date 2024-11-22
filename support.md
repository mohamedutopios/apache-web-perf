---
marp: true
title: Kubernetes
theme: utopios
paginate: true
author: Mohamed Aijjou
header: "![h:70px](https://utopios-marp-assets.s3.eu-west-3.amazonaws.com/logo_blanc.svg)"
footer: "Utopios® Tous droits réservés"
---

<!-- _class: lead -->
<!-- _paginate: false -->

# APACHE PERFECTIONNEMENT

---

## Sommaire

<br>

1. Apache HTTPD 2.4 : rappels et nouveautés
2. Héberger des applications PHP
3. Contrôle d'accès et authentification
4. Redirection, réécriture d'adresses, filtres
5. Reverse Proxy et Cache
6. Sécuriser les échanges avec HTTPS
7. Sécurité et détection d'attaques

</div>

---

<!-- _class: lead -->
<!-- _paginate: false -->

## Apache HTTPD 2.4 : rappels et nouveautés

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Rappels :**

<br/>

<div style="font-size:28px">

- **Apache HTTP Server (HTTPD)** est l'un des serveurs web les plus populaires.
- Il repose sur une architecture modulaire.
- Supporte des fonctionnalités comme :
  - **Proxying** (mod_proxy, mod_proxy_http, mod_proxy_ajp, etc.)
  - **SSL/TLS** (via mod_ssl).
  - **Virtual hosts** (configuration de plusieurs sites sur une seule machine).
- Version 2.4 apporte des améliorations par rapport à 2.2 en termes de performance, sécurité et flexibilité.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveautés d'Apache HTTPD 2.4 (comparé à 2.2) :**


<div style="font-size:21px">

**MPM améliorés** : 
  - Nouveau MPM **Event**, optimisé pour les connexions persistantes et HTTP/2.
  - Amélioration des MPM Worker et Prefork.
- **Authentification unifiée** :
  - Nouvelle architecture avec `mod_authz_core`, permettant des directives comme `Require` pour centraliser la gestion d'accès.
  - Authentification via base de données avec `mod_authn_dbd`.
- **Configuration conditionnelle** :
  - Support des directives `If`, `ElseIf`, `Else` pour une configuration plus dynamique.
  - Exemple :
    ```conf
    <If "%{HTTP_HOST} == 'example.com'">
      DocumentRoot "/var/www/example"
    </If>
    ```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveautés d'Apache HTTPD 2.4 (comparé à 2.2) :**


<div style="font-size:23px">

- **Modules proxy améliorés** :
  - Support natif pour WebSocket via `mod_proxy_wstunnel`.
  - Meilleure gestion des backends via `mod_proxy_balancer`.
- **HTTP/2** :
  - Ajout du module `mod_http2` pour le protocole HTTP/2.
- **Amélioration des performances** :
  - Réduction de la consommation mémoire grâce à des optimisations dans le traitement des connexions.
- **Directive IncludeOptional** :
  - Permet d'inclure des fichiers de configuration optionnels, évitant les erreurs si un fichier est manquant.
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Compilation, Installation et Test Initial :**

<br>
<div style="font-size:29px">

#### **1. Pré-requis :**
- Vérifier les dépendances :
  - **APR et APR-util** (Apache Portable Runtime).
  - OpenSSL pour SSL/TLS.
  - PCRE (Perl Compatible Regular Expressions) pour `mod_rewrite` et autres modules.

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Compilation, Installation et Test Initial :**

<br>
<div style="font-size:22px">

#### **2. Télécharger les sources :**
```bash
wget https://downloads.apache.org/httpd/httpd-2.4.x.tar.gz
```

#### **3. Télécharger les dépendances APR :**
```bash
wget https://downloads.apache.org/apr/apr-1.x.tar.gz
wget https://downloads.apache.org/apr/apr-util-1.x.tar.gz
```
Décompressez APR et APR-util, puis placez-les dans le répertoire des sources d'Apache (sous `srclib/`) :
```bash
tar -xzvf apr-1.x.tar.gz
tar -xzvf apr-util-1.x.tar.gz
mv apr-1.x httpd-2.4.x/srclib/apr
mv apr-util-1.x httpd-2.4.x/srclib/apr-util
```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Compilation, Installation et Test Initial :**

<br>
<div style="font-size:35px">

#### **4. Décompression des sources Apache HTTPD :**
```bash
tar -xzvf httpd-2.4.x.tar.gz
cd httpd-2.4.x
```
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Compilation, Installation et Test Initial :**

<br>
<div style="font-size:30px">

#### **5. Configuration des options de compilation :**
```bash
./configure --enable-so --enable-ssl --enable-mods-shared=all --with-mpm=event --with-pcre --enable-rewrite
```
- **`--enable-so`** : Activer les modules dynamiques.
- **`--enable-ssl`** : Inclure le support SSL/TLS.
- **`--enable-mods-shared=all`** : Compiler tous les modules sous forme dynamique.
- **`--with-mpm=event`** : Choisir le MPM Event comme module de traitement par défaut.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Compilation, Installation et Test Initial :**

<br>
<div style="font-size:35px">

#### **6. Compilation et installation :**
```bash
make
sudo make install
```


</div>


---
## Apache HTTPD 2.4 : rappels et nouveautés

#### **Compilation, Installation et Test Initial :**


<div style="font-size:21px">

#### **7. Test initial :**
1. Vérifiez la version installée :
   ```bash
   /usr/local/apache2/bin/httpd -v
   ```

2. Démarrez le serveur Apache :
   ```bash
   /usr/local/apache2/bin/apachectl start
   ```

3. Vérifiez le processus Apache :
   ```bash
   ps aux | grep httpd
   ```

4. Testez dans un navigateur :
   - Accédez à `http://localhost`.
   - Si tout est correctement configuré, la page par défaut d'Apache apparaîtra.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Structure par défaut de l'installation :**

<br>
<div style="font-size:30px">

- **Fichiers de configuration** :
  - `/usr/local/apache2/conf/httpd.conf` : fichier principal.
  - `/usr/local/apache2/conf/extra/` : configurations supplémentaires (SSL, virtual hosts, MPM).
- **Répertoires** :
  - `/usr/local/apache2/htdocs/` : emplacement des fichiers web par défaut.
  - `/usr/local/apache2/logs/` : logs d'erreurs et d'accès.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Configuration générale du serveur:**

<br>
<div style="font-size:28px">

Les fichiers principaux sont généralement situés dans `/usr/local/apache2/conf/` :
- **httpd.conf** : fichier principal de configuration.
- **Principales directives** :
  - `ServerRoot` : répertoire racine d'Apache.
  - `DocumentRoot` : répertoire où sont stockés les fichiers web.
  - `Listen` : port d'écoute (par défaut 80).
  - `ServerName` : nom de domaine ou adresse IP du serveur.
  - `ErrorLog` et `CustomLog` : fichiers de logs.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Configuration générale du serveur:**

<br>
<div style="font-size:35px">

Exemple minimaliste :
```conf
ServerRoot "/usr/local/apache2"
Listen 80
ServerName localhost
DocumentRoot "/usr/local/apache2/htdocs"
ErrorLog "/usr/local/apache2/logs/error_log"
CustomLog "/usr/local/apache2/logs/access_log" common
```


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<br>

<div style="font-size:30px">

- **MPM Event** :
  - Une évolution de Worker.
  - Optimisé pour les connexions persistantes (ex. HTTP/2, keep-alive).
  - Idéal pour les serveurs modernes avec de nombreuses connexions simultanées.

Changer le MPM dans la configuration :
```bash
./configure --with-mpm=worker
```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Gérer la charge et les limites :**

<div style="font-size:24px">

- **Configurer les paramètres MPM** dans `httpd-mpm.conf` :
  Exemple pour MPM Worker :
  ```conf
  <IfModule mpm_worker_module>
      StartServers 2
      MinSpareThreads 25
      MaxSpareThreads 75
      ThreadsPerChild 25
      MaxRequestWorkers 150
      MaxConnectionsPerChild 1000
  </IfModule>
  ```

- **Limiter les connexions et la bande passante** :
  Avec `mod_reqtimeout` :
  ```conf
  RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500
  ```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**


<div style="font-size:20px">

- Les modules dynamiques sont gérés avec la directive `LoadModule`.
  Exemple :
  ```conf
  LoadModule rewrite_module modules/mod_rewrite.so
  LoadModule ssl_module modules/mod_ssl.so
  ```

- **Modules recommandés** :
  - **Performance** : `mod_cache`, `mod_deflate`, `mod_http2`.
  - **Sécurité** : `mod_ssl`, `mod_security`.
  - **Redirection/URL rewriting** : `mod_rewrite`.
  - **Logs avancés** : `mod_log_config`.
  - **Proxy** (si utilisé comme reverse proxy) : `mod_proxy`, `mod_proxy_http`.

Vérifier les modules chargés :
```bash
/usr/local/apache2/bin/apachectl -M
```
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**


<div style="font-size:20px">

- Les modules dynamiques sont gérés avec la directive `LoadModule`.
  Exemple :
  ```conf
  LoadModule rewrite_module modules/mod_rewrite.so
  LoadModule ssl_module modules/mod_ssl.so
  ```

- **Modules recommandés** :
  - **Performance** : `mod_cache`, `mod_deflate`, `mod_http2`.
  - **Sécurité** : `mod_ssl`, `mod_security`.
  - **Redirection/URL rewriting** : `mod_rewrite`.
  - **Logs avancés** : `mod_log_config`.
  - **Proxy** (si utilisé comme reverse proxy) : `mod_proxy`, `mod_proxy_http`.

Vérifier les modules chargés :
```bash
/usr/local/apache2/bin/apachectl -M
```
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:18px">

Apache HTTPD 2.4 introduit des fonctionnalités permettant une gestion plus fine des configurations grâce à de nouveaux **types de contextes**. Ces contextes permettent de personnaliser les comportements selon des conditions ou des environnements spécifiques.

#### **Principaux contextes et leur usage :**

1. **Directives conditionnelles avec `<If>` :**
   - Permet d'évaluer des conditions dynamiques dans les fichiers de configuration.
   - Syntaxe :
     ```conf
     <If "%{HTTP_HOST} == 'example.com'">
       DocumentRoot "/var/www/example"
     </If>
     ```
   - Supporte les expressions complexes :
     - Accès à des variables d'environnement (`%{ENV:VARIABLE}`).
     - Comparaisons entre en-têtes HTTP (`%{HTTP_*}`).
     - Expression booléenne complexe (`&&`, `||`, `!`).
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:27px">

#### **Principaux contextes et leur usage :**

2. **Directives dynamiques avec `<ElseIf>` et `<Else>` :**
   - Exemple :
     ```conf
     <If "%{HTTP_HOST} == 'example.com'">
       DocumentRoot "/var/www/example"
     <ElseIf "%{HTTP_HOST} == 'example.org'">
       DocumentRoot "/var/www/example_org"
     <Else>
       DocumentRoot "/var/www/default"
     </Else>
     ```
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:28px">

#### **Principaux contextes et leur usage :**

3. **Directives `IncludeOptional` :**
   - Permet d’inclure des fichiers optionnels, évitant les erreurs si un fichier est absent.
   - Exemple :
     ```conf
     IncludeOptional conf.d/*.conf
     ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:28px">

#### **Principaux contextes et leur usage :**

4. **Directives spécifiques au trafic avec `<IfModule>` :**
   - Active une configuration seulement si un module est chargé.
   - Exemple :
     ```conf
     <IfModule mod_ssl.c>
       Listen 443
       SSLEngine on
     </IfModule>
     ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Panorama des modules d'Apache 2.4**


<div style="font-size:30px">

Apache HTTPD 2.4 offre une large gamme de modules pour diverses fonctionnalités. Voici un aperçu des catégories principales et des modules les plus utiles.

#### **Catégorie : Performance**
- **`mod_http2`** : Support HTTP/2.
- **`mod_cache`** : Mise en cache des contenus.
- **`mod_deflate`** : Compression des contenus.
- **`mod_expires`** : Contrôle des en-têtes de cache HTTP.
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Panorama des modules d'Apache 2.4**


<div style="font-size:30px">

#### **Catégorie : Sécurité**
- **`mod_ssl`** : Support SSL/TLS.
- **`mod_security`** : Protection contre les attaques (pare-feu applicatif).
- **`mod_authz_core`** : Cadre centralisé pour l'autorisation.
- **`mod_authn_dbd`** : Authentification via base de données.
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Panorama des modules d'Apache 2.4**


<div style="font-size:28px">

#### **Catégorie : Proxy**
- **`mod_proxy`** : Support du proxy HTTP/HTTPS.
- **`mod_proxy_http`** : Proxy pour le trafic HTTP.
- **`mod_proxy_balancer`** : Équilibrage de charge.
- **`mod_proxy_wstunnel`** : Proxy pour WebSocket.

#### **Catégorie : Journalisation**
- **`mod_log_config`** : Configuration avancée des logs.
- **`mod_logio`** : Ajoute des informations sur la taille des réponses dans les logs.
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Panorama des modules d'Apache 2.4**


<div style="font-size:28px">

#### **Catégorie : Réécriture et redirection**
- **`mod_rewrite`** : Réécriture d'URL avancée.
- **`mod_alias`** : Redirection simple.

#### **Catégorie : Autres fonctionnalités**
- **`mod_cgi`** : Support des scripts CGI.
- **`mod_dav`** : Prise en charge du protocole WebDAV.
- **`mod_headers`** : Modification des en-têtes HTTP.
- **`mod_dir`** : Gestion des répertoires (pages index).

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Panorama des modules d'Apache 2.4**


<div style="font-size:35px">

#### **Gestion des modules dynamiques :**
- Les modules sont chargés dynamiquement via la directive `LoadModule` :
  ```conf
  LoadModule rewrite_module modules/mod_rewrite.so
  LoadModule ssl_module modules/mod_ssl.so
  ```
- Vérifiez les modules chargés :
  ```bash
  apachectl -M
  ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**


<div style="font-size:28px">

Apache HTTPD 2.4 introduit le support natif du protocole **HTTP/2** via le module `mod_http2`. HTTP/2 est conçu pour améliorer les performances des sites web modernes.

#### **Avantages de HTTP/2 :**
- **Multiplexage des requêtes** : Plusieurs requêtes/connexions parallèles via une seule connexion TCP.
- **Compression des en-têtes** : Réduction de la taille des en-têtes HTTP grâce à HPACK.
- **Priorisation des requêtes** : Optimisation de l'ordre d'exécution.
- **Push serveur** : Envoi proactif de ressources avant la requête client.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**


<div style="font-size:30px">


#### **Activation de HTTP/2 :**
1. **Vérifier les dépendances :**
   - OpenSSL version 1.0.2 ou supérieure.
   - Apache 2.4.x compilé avec `mod_http2`.

2. **Charger le module `mod_http2` :**
   Ajouter dans le fichier de configuration :
   ```conf
   LoadModule http2_module modules/mod_http2.so
   ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**


<div style="font-size:23px">


#### **Activation de HTTP/2 :**
3. **Activer HTTP/2 dans un VirtualHost :**
   Exemple pour un site HTTPS :
   ```conf
   <VirtualHost *:443>
       Protocols h2 http/1.1
       SSLEngine on
       SSLCertificateFile /path/to/certificate.crt
       SSLCertificateKeyFile /path/to/private.key
       DocumentRoot "/var/www/example"
   </VirtualHost>
   ```

4. **Redémarrer Apache :**
   ```bash
   apachectl restart
   ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**


<div style="font-size:30px">


#### **Activation de HTTP/2 :**

5. **Vérifier le support de HTTP/2 :**
   - Utilisez un outil comme [curl](https://curl.se/) :
     ```bash
     curl -I --http2 https://example.com
     ```
   - Vérifiez les en-têtes pour `HTTP/2`.

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

<br>
<center>
<img src="./assets/practice.png" width="400px">
</center>


---

<!-- _class: lead -->
<!-- _paginate: false -->

## Héberger des applications PHP

---

## Héberger des applications PHP

<br>
<center>
<img src="./assets/practice.png" width="400px">
</center>


---


<!-- _class: lead -->
<!-- _paginate: false -->

## Contrôle d'accès et authentification

---
## Contrôle d'accès et authentification

<br>
<center>
<img src="./assets/practice.png" width="400px">
</center>


---


<!-- _class: lead -->
<!-- _paginate: false -->

## Redirection, réécriture d'adresses, filtres

---

## Redirection, réécriture d'adresses, filtres

<br>
<center>
<img src="./assets/practice.png" width="400px">
</center>


---

<!-- _class: lead -->
<!-- _paginate: false -->

## Reverse Proxy et Cache

---

## Reverse Proxy et Cache

<br>
<center>
<img src="./assets/practice.png" width="400px">
</center>


---

<!-- _class: lead -->
<!-- _paginate: false -->

## Sécuriser les échanges avec HTTPS

---

## Sécuriser les échanges avec HTTPS

<br>
<center>
<img src="./assets/practice.png" width="400px">
</center>


---

<!-- _class: lead -->
<!-- _paginate: false -->

## Sécurité et détection d'attaques


---

## Sécurité et détection d'attaque

<br>
<center>
<img src="./assets/practice.png" width="400px">
</center>


---

<br>
<center>
<img src="./assets/end.jpg" width="800px">
</center>


