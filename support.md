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

#### **Installation complète d'Apache avec apt**

<br>
<div style="font-size:28px">

#### **1. Mettre à jour les paquets**
Avant toute installation, mettez à jour les informations des paquets pour garantir que vous obtenez la dernière version disponible :

```bash
sudo apt update
sudo apt upgrade -y
```
#### **2. Installer Apache**
Installez Apache HTTP Server avec la commande suivante :

```bash
sudo apt install apache2 -y
```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Installation complète d'Apache avec apt**

<br>
<div style="font-size:35px">

#### **3. Vérifier l'installation**
Vérifiez qu’Apache est bien installé et en cours d’exécution :

```bash
sudo systemctl status apache2
```

Vous devriez voir une sortie indiquant que le service est **`active (running)`**.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Installation complète d'Apache avec apt**

<br>
<div style="font-size:35px">

#### **4. Tester Apache**
1. **Dans un navigateur** : Accédez à votre serveur à l'adresse suivante :
   - Si c'est en local : [http://localhost](http://localhost) ou [http://127.0.0.1](http://127.0.0.1).
   - Si c'est une machine distante : [http://<adresse_IP>](http://<adresse_IP>).
2. Vous devriez voir une page intitulée **"Apache2 Ubuntu Default Page"**.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Installation complète d'Apache avec apt**

<br>
<div style="font-size:35px">

#### **5. Configurer le pare-feu**
Si vous utilisez un pare-feu (comme UFW), ouvrez le port 80 (HTTP) et 443 (HTTPS) pour permettre l'accès au serveur :

```bash
sudo ufw allow 'Apache Full'
sudo ufw enable
sudo ufw status
```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Installation complète d'Apache avec apt**

<div style="font-size:23px">

#### **6. Activer les modules nécessaires**
Apache inclut plusieurs modules utiles. Voici les commandes pour activer les modules les plus courants :

1. **Module SSL** (pour HTTPS) :
   ```bash
   sudo a2enmod ssl
   ```

2. **Module Rewrite** (pour les réécritures d’URL comme dans WordPress) :
   ```bash
   sudo a2enmod rewrite
   ```

3. Redémarrez Apache pour appliquer les changements :
   ```bash
   sudo systemctl restart apache2
   ```

</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Installation complète d'Apache avec apt**

<br>

<div style="font-size:30px">

### **Outils et emplacements importants**
- **Fichiers de configuration des sites** :
  - **Disponibles** : `/etc/apache2/sites-available/`
  - **Activés** : `/etc/apache2/sites-enabled/`
- **Modules disponibles** : `/etc/apache2/mods-available/`
- **Répertoire des journaux** : `/var/log/apache2/`

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Installation complète d'Apache avec apt**

<br>

<div style="font-size:30px">

### **Résumé des commandes principales**
- **Installer Apache** : `sudo apt install apache2`
- **Activer un module** : `sudo a2enmod <module>`
- **Activer un site** : `sudo a2ensite <site>`
- **Vérifier la configuration** : `sudo apache2ctl configtest`
- **Redémarrer Apache** : `sudo systemctl restart apache2`

</div>


---

#### Apache HTTPD 2.4 : rappels et nouveautés

###### **Comparatif dans les installations**

| **Méthode**                | **Avantages**                      | **Inconvénients**                     | **Cas d'usage**                             |
|----------------------------|------------------------------------|---------------------------------------|---------------------------------------------|
| Gestionnaire de paquets    | Facile, stable, rapide            | Peu de personnalisation               | Serveurs de production standard             |
| Compilation (sources)      | Personnalisation, flexibilité     | Maintenance manuelle, complexe        | Besoins spécifiques, environnements avancés |
| Conteneurs Docker          | Isolation, portabilité            | Complexité initiale                   | Microservices, CI/CD, Cloud-native          |
| Outils d’automatisation    | Automatisation, réplicabilité     | Nécessite des outils tiers            | Environnements à grande échelle             |
| Services Cloud préconfigurés| Simplicité, haute disponibilité   | Coût élevé, dépendance                | Startups, mise à l’échelle rapide           |


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Comparatif dans les installations**

<div style="font-size:24px">


##### **La meilleure façon de faire**

#### **Cas généraux : Utilisation du gestionnaire de paquets**
- Simple, rapide et stable.
- Idéal pour des environnements de production nécessitant peu de personnalisation.

#### **Cas spécifiques : Compilation des sources**
- Permet de répondre à des besoins particuliers (versions spécifiques, optimisations).
- Nécessite un bon niveau en administration système.

#### **Environnements modernes : Docker et Automatisation**
- Docker pour des environnements flexibles, réplicables et isolés.
- Automatisation (Ansible, Terraform) pour les infrastructures complexes à grande échelle.

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Configuration générale du serveur:**

<div style="font-size:18px">

La localisation des dossiers principaux d’**Apache HTTPD** dépend de la méthode d’installation et du système d’exploitation. 

### **1. Installation par les gestionnaires de paquets**
Si Apache est installé via un gestionnaire de paquets (comme `apt` ou `yum`), les fichiers principaux se trouvent généralement sous **`/etc/`** pour la configuration et **`/var/www/`** pour les fichiers web.

#### **Debian/Ubuntu**
- **Fichiers de configuration** : Localisés sous **`/etc/apache2/`**.
  - `/etc/apache2/apache2.conf` : Fichier principal de configuration.
  - `/etc/apache2/sites-available/` : Configurations des Virtual Hosts disponibles.
  - `/etc/apache2/sites-enabled/` : Liens symboliques vers les Virtual Hosts activés.
  - `/etc/apache2/mods-available/` : Modules disponibles.
  - `/etc/apache2/mods-enabled/` : Liens symboliques vers les modules activés.

- **Répertoire des sites web** : **`/var/www/`**
  - Par défaut, `/var/www/html` est le répertoire racine pour les fichiers accessibles par le serveur.



</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Configuration générale du serveur:**

<div style="font-size:23px">

La localisation des dossiers principaux d’**Apache HTTPD** dépend de la méthode d’installation et du système d’exploitation. 

### **1. Installation par les gestionnaires de paquets**

#### **CentOS/RHEL**
- **Fichiers de configuration** : Localisés sous **`/etc/httpd/`**.
  - `/etc/httpd/conf/httpd.conf` : Fichier principal de configuration.
  - `/etc/httpd/conf.d/` : Configurations additionnelles (ex. SSL, Virtual Hosts).
  - `/etc/httpd/conf.modules.d/` : Liste des modules disponibles ou chargés.

- **Répertoire des sites web** : **`/var/www/`**
  - Par défaut, `/var/www/html` est utilisé comme racine des sites.



</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Configuration générale du serveur:**

<div style="font-size:23px">

### **2. Installation à partir des sources**
Si Apache est installé à partir des **sources**, l’organisation par défaut diffère. En général, les fichiers principaux sont placés sous **`/usr/local/`**, sauf si des options spécifiques sont définies lors de la compilation.

#### **Répertoires typiques** :
- **Configuration** : **`/usr/local/apache2/conf/`**
  - `/usr/local/apache2/conf/httpd.conf` : Fichier principal de configuration.
- **Modules** : **`/usr/local/apache2/modules/`**
- **Logs** : **`/usr/local/apache2/logs/`**
- **Répertoire des sites web** : **`/usr/local/apache2/htdocs/`**


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Configuration générale du serveur:**

<div style="font-size:28px">

### **3. Comparaison des répertoires principaux**

| **Installation**            | **Configuration**             | **Modules**               | **Sites web**         | **Logs**               |
|-----------------------------|-------------------------------|---------------------------|------------------------|------------------------|
| **Debian/Ubuntu**           | `/etc/apache2/`              | `/etc/apache2/mods-available/` | `/var/www/html/`      | `/var/log/apache2/`    |
| **CentOS/RHEL**             | `/etc/httpd/`                | `/etc/httpd/modules/`     | `/var/www/html/`      | `/var/log/httpd/`      |
| **Compilation (sources)**   | `/usr/local/apache2/conf/`   | `/usr/local/apache2/modules/` | `/usr/local/apache2/htdocs/` | `/usr/local/apache2/logs/` |


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<br>

<div style="font-size:35px">


- Le choix du MPM et la gestion des limites sont cruciaux pour optimiser les performances d’Apache en fonction de la charge de travail et des ressources système. 
- Apache HTTPD offre plusieurs MPMs. Chaque MPM a ses propres caractéristiques qui influencent les performances et l’utilisation des ressources

</div>


---



#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Gérer la charge et les limites :**

| **MPM**     | **Description**                                                                                             | **Avantages**                                                                                     | **Limites**                                                                                 |
|-------------|-------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| **prefork** | Basé sur des processus, chaque requête est traitée par un processus distinct.                                | - Stable.<br>- Compatible avec les modules non thread-safe.<br>- Simple à configurer.             | - Consomme beaucoup de mémoire.<br>- Performances limitées pour les charges élevées.        |
| **worker**  | Combinaison de processus et threads. Un processus peut traiter plusieurs requêtes via des threads.           | - Performances élevées.<br>- Meilleure utilisation des ressources.<br>- Moins de mémoire utilisée.| - Modules non thread-safe incompatibles.<br>- Configuration plus complexe.                 |

</div>


---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Gérer la charge et les limites :**

<br>


| **MPM**     | **Description**                                                                                             | **Avantages**                                                                                     | **Limites**                                                                                 |
|-------------|-------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| **event**   | Extension de worker. Optimisé pour les connexions persistantes (Keep-Alive).                                | - Très performant pour les connexions Keep-Alive.<br>- Faible consommation des ressources.        | - Modules non thread-safe incompatibles.<br>- Configuration avancée requise.               |
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:25px">


### **2. Choisir le bon MPM**

#### **Critères de choix :**

1. **Charge prévue :**
   - **Faible charge ou faible trafic** : `prefork` est suffisant.
   - **Charge moyenne à élevée** : `worker` ou `event` est plus adapté.
   - **Connexions persistantes (Keep-Alive)** : `event` est recommandé.

2. **Compatibilité des modules :**
   - **Modules non thread-safe** (par exemple certaines extensions PHP) : Utilisez `prefork`.
   - **Modules thread-safe** : Préférez `worker` ou `event`.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:27px">

### **3. Configurer les limites pour chaque MPM**

#### **a. Configuration du MPM `prefork`**
Dans le fichier de configuration Apache (`httpd.conf` ou un fichier spécifique selon votre système), ajustez les paramètres suivants :

```apache
<IfModule mpm_prefork_module>
    StartServers         5        # Nombre initial de processus.
    MinSpareServers      5        # Processus minimum en attente.
    MaxSpareServers     10        # Processus maximum en attente.
    MaxRequestWorkers   150       # Limite maximale de processus simultanés.
    MaxConnectionsPerChild 0      # Nombre de requêtes par processus (0 = illimité).
</IfModule>
```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:27px">

### **3. Configurer les limites pour chaque MPM**


#### **b. Configuration du MPM `worker`**
Les paramètres à ajuster pour `worker` :

```apache
<IfModule mpm_worker_module>
    StartServers         2        # Processus de démarrage.
    MinSpareThreads     25        # Threads minimum en attente.
    MaxSpareThreads     75        # Threads maximum en attente.
    ThreadLimit         64        # Limite de threads par processus.
    ThreadsPerChild     25        # Threads gérés par chaque processus.
    MaxRequestWorkers   150       # Nombre total maximum de threads (processeurs * threads).
    MaxConnectionsPerChild 0      # Nombre de requêtes avant de recréer un processus.
</IfModule>
```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:25px">

### **3. Configurer les limites pour chaque MPM**


#### **c. Configuration du MPM `event`**
Les paramètres pour `event` sont similaires à ceux de `worker`, avec des optimisations pour les connexions Keep-Alive :

```apache
<IfModule mpm_event_module>
    StartServers         2
    MinSpareThreads     25
    MaxSpareThreads     75
    ThreadLimit         64
    ThreadsPerChild     25
    MaxRequestWorkers   150
    MaxConnectionsPerChild 0
</IfModule>
```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:27px">

### **4. Gérer la charge : Limites et optimisations**

#### **a. MaxRequestWorkers**
- Définit le nombre maximum de requêtes simultanées que le serveur peut traiter.
- Adaptez ce paramètre en fonction de la mémoire disponible et de la charge prévue.

#### **b. Timeout**
- Contrôle le temps maximum qu’Apache attend pour qu'une requête soit traitée.
- Réduisez cette valeur pour éviter des connexions bloquantes :
  ```apache
  Timeout 60
  ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:20px">

### **4. Gérer la charge : Limites et optimisations**

#### **c. Keep-Alive**
- Permet aux clients de réutiliser une connexion TCP pour plusieurs requêtes.
- Réglez les paramètres pour optimiser les connexions persistantes :
  ```apache
  KeepAlive On
  MaxKeepAliveRequests 100
  KeepAliveTimeout 5
  ```

#### **d. Gestion de la mémoire**
- Surveillez l’utilisation de la mémoire avec des outils comme `top` ou `htop`.
- Augmentez les limites système si nécessaire :
  - Exemple pour augmenter les fichiers ouverts :
    ```bash
    ulimit -n 4096
    ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:30px">

### **4. Gérer la charge : Limites et optimisations**

#### **e. Surveillance et monitoring**
- Utilisez des outils comme **Prometheus**, **Nagios**, ou **Grafana** pour surveiller la charge et les performances.
- Analysez les fichiers de logs pour détecter les goulots d’étranglement :
  ```bash
  tail -f /var/log/apache2/access.log
  tail -f /var/log/apache2/error.log
  ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:25px">

### **5. Tests et ajustements**
1. **Testez vos paramètres avec des outils de charge** :
   - Exemple : **Apache Benchmark** (ab) :
     ```bash
     ab -n 1000 -c 100 http://localhost/
     ```
   - Cela permet de tester comment Apache gère un grand nombre de requêtes simultanées.

2. **Ajustez les paramètres** :
   - Si Apache atteint la limite de `MaxRequestWorkers`, vous devrez augmenter cette valeur ou optimiser l'application.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:25px">

### **6. Résumé : Quelle stratégie adopter ?**

| **Scénario**                     | **MPM recommandé** | **Stratégie**                                                                                  |
|-----------------------------------|--------------------|-----------------------------------------------------------------------------------------------|
| **Site avec faible trafic**       | `prefork`          | Paramètres par défaut, Keep-Alive activé.                                                     |
| **Site avec trafic modéré/élevé** | `worker`           | Ajustez `ThreadsPerChild` et `MaxRequestWorkers` pour gérer la charge.                        |
| **API ou connexions Keep-Alive**  | `event`            | Optimisez Keep-Alive et les threads pour maximiser les performances avec des connexions longues. |
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Choisir le bon MPM (Multi-Processing Module) :**

<div style="font-size:25px">


### **2. Choisir le bon MPM**

#### **Critères de choix :**

3. **Ressources disponibles :**
   - **Mémoire limitée** : Évitez `prefork` et optez pour `worker` ou `event`.
   - **Environnements virtualisés/cloud** : Préférez `worker` ou `event`.

4. **Type d’application :**
   - Applications lourdes en PHP : `prefork`.
   - API ou sites modernes : `worker` ou `event`.


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<br>

<div style="font-size:35px">

- Apache HTTPD est un serveur modulaire, ce qui signifie que ses fonctionnalités peuvent être étendues ou réduites en activant ou désactivant des modules. 
- Le choix des modules à activer dépend de vos besoins spécifiques, comme la sécurité, les performances, ou les fonctionnalités nécessaires pour votre application.

</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<div style="font-size:27px">

### **1. Gestion des modules dans Apache**

#### **b. Activer un module**
- Sur **Ubuntu/Debian** :
  ```bash
  sudo a2enmod module_name
  ```
  Exemple : Activer le module `rewrite` :
  ```bash
  sudo a2enmod rewrite
  sudo systemctl reload apache2
  ```
</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<div style="font-size:27px">

### **1. Gestion des modules dans Apache**

- Sur **CentOS/RHEL** :
  - Modifiez le fichier principal de configuration (`httpd.conf`) ou les fichiers sous `/etc/httpd/conf.modules.d/`.
  - Exemple :
    ```apache
    LoadModule rewrite_module modules/mod_rewrite.so
    ```
  - Rechargez la configuration :
    ```bash
    sudo systemctl reload httpd
    ````

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<div style="font-size:29px">

### **1. Gestion des modules dans Apache**

#### **c. Désactiver un module**
- Sur **Ubuntu/Debian** :
  ```bash
  sudo a2dismod module_name
  sudo systemctl reload apache2
  ```

- Sur **CentOS/RHEL** :
  - Commentez ou supprimez la ligne `LoadModule` correspondante.
``

</div>


---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Chargement des modules et modules à activer :**

##### **2. Modules essentiels et leurs rôles : Module base**


| **Module**       | **Description**                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `mod_core`       | Module de base, chargé automatiquement.                                         |
| `mod_mpm_*`      | Multi-Processing Modules (choisir `prefork`, `worker` ou `event`).              |
| `mod_log_config` | Permet de configurer les journaux (access logs, error logs).                    |
| `mod_status`     | Fournit une page de statut pour surveiller les performances et connexions actives. |
| `mod_alias`      | Permet de créer des alias d'URL (par exemple, rediriger `/docs` vers un dossier spécifique). |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Chargement des modules et modules à activer :**

##### **2. Modules essentiels et leurs rôles : Modules de performance**

<br>

| **Module**       | **Description**                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `mod_deflate`    | Compression des réponses HTTP pour économiser de la bande passante.             |
| `mod_headers`    | Permet de modifier les en-têtes HTTP.                                           |
| `mod_cache`      | Met en cache les contenus pour accélérer les réponses.                         |
| `mod_expires`    | Définit les politiques de cache basées sur la durée de vie des ressources.       |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Chargement des modules et modules à activer :**

##### **2. Modules essentiels et leurs rôles : Modules de sécurité**


| **Module**       | **Description**                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `mod_ssl`        | Ajoute le support HTTPS (SSL/TLS).                                              |
| `mod_security`   | Améliore la sécurité en bloquant les requêtes malveillantes (WAF - Web Application Firewall). |
| `mod_auth_basic` | Authentification basique par mot de passe.                                      |
| `mod_auth_digest`| Authentification plus sécurisée que `mod_auth_basic`.                          |
| `mod_evasive`    | Protège contre les attaques DDoS ou brute-force en limitant les connexions répétées. |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Chargement des modules et modules à activer :**

##### **2. Modules essentiels et leurs rôles : Redirections et réécritures**

<br>

| **Module**       | **Description**                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `mod_rewrite`    | Permet la réécriture des URL (très utilisé pour les sites dynamiques et les SEO).|
| `mod_redirect`   | Configure des redirections HTTP simples.                                        |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Chargement des modules et modules à activer :**

##### **2. Modules essentiels et leurs rôles : Applications et contenu**


| **Module**       | **Description**                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `mod_cgi`        | Exécute des scripts CGI.                                                        |
| `mod_php`        | Intègre le support PHP directement dans Apache (souvent remplacé par PHP-FPM).  |
| `mod_proxy`      | Configure un proxy ou une passerelle pour d'autres serveurs.                    |
| `mod_proxy_http` | Support pour le proxy HTTP (souvent utilisé avec `mod_proxy`).                  |
| `mod_proxy_fcgi` | Permet la communication avec PHP-FPM via FastCGI.                               |
| `mod_wsgi`       | Permet d'exécuter des applications Python (par exemple, Django).                |

</div>

---


#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Chargement des modules et modules à activer :**

<br>

##### **2. Modules essentiels et leurs rôles : Modules de diagnostic**


| **Module**       | **Description**                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `mod_info`       | Fournit des informations sur la configuration actuelle d'Apache.               |
| `mod_status`     | Fournit des statistiques en temps réel sur le serveur (ex. requêtes actives).   |

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<div style="font-size:27px">

### **3. Quels modules activer selon les besoins**

#### **a. Serveur web classique (HTML/CSS/JS uniquement)**
- Modules recommandés :
  - `mod_deflate`
  - `mod_headers`
  - `mod_alias`
  - `mod_expires`
  - `mod_status`


</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<div style="font-size:27px">

### **3. Quels modules activer selon les besoins**

#### **b. Serveur pour applications dynamiques (PHP, Python, etc.)**
- Modules recommandés :
  - `mod_php` ou `mod_proxy_fcgi` (pour PHP-FPM)
  - `mod_wsgi` (pour Python)
  - `mod_rewrite`
  - `mod_deflate`
  - `mod_headers`
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<div style="font-size:27px">

### **3. Quels modules activer selon les besoins**

#### **c. Serveur sécurisé avec HTTPS**
- Modules recommandés :
  - `mod_ssl`
  - `mod_security`
  - `mod_evasive`
  - `mod_headers`
  - `mod_rewrite`

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**

<div style="font-size:27px">

### **3. Quels modules activer selon les besoins**

#### **d. Serveur proxy/reverse proxy**
- Modules recommandés :
  - `mod_proxy`
  - `mod_proxy_http`
  - `mod_proxy_fcgi`
  - `mod_cache`
  - `mod_headers`

</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**


<div style="font-size:30px">

### **4. Désactiver les modules inutiles**

<br>

Pour des raisons de performance et de sécurité, désactivez les modules non utilisés. Par exemple :
- Modules CGI si vous n'exécutez pas de scripts CGI :
  ```bash
  sudo a2dismod cgi
  sudo systemctl reload apache2
  ```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**


<div style="font-size:30px">

### **5. Vérification après activation/désactivation**

- **Tester la configuration Apache :**
  ```bash
  apache2ctl configtest
  ```
  Exemple de sortie si tout est correct :
  ```
  Syntax OK
  ```

- **Vérifier les modules activés :**
  ```bash
  apache2ctl -M
  ```

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Chargement des modules et modules à activer :**


<div style="font-size:30px">

### **Résumé**

1. **Modules essentiels** : `mod_ssl`, `mod_rewrite`, `mod_headers`, `mod_status`.
2. **Modules à activer selon les besoins spécifiques** :
   - Performance : `mod_deflate`, `mod_expires`, `mod_cache`.
   - Sécurité : `mod_security`, `mod_evasive`.
   - Applications dynamiques : `mod_php`, `mod_wsgi`, `mod_proxy_fcgi`.
3. Désactivez les modules inutiles pour améliorer la sécurité et les performances.

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<br>

<div style="font-size:32px">

- Apache HTTPD 2.4 introduit des fonctionnalités permettant une gestion plus fine des configurations grâce à de nouveaux **types de contextes**. 
- Ces contextes permettent de personnaliser les comportements selon des conditions ou des environnements spécifiques.

- Ces contextes sont une manière de structurer la configuration du serveur en fonction de l’endroit ou du niveau où les directives s’appliquent.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<br>

<div style="font-size:25px">

### **1. Contextes standards existants dans Apache**

#### **a. Serveur global (`server config`)**
- **Description** :
  - Les directives s'appliquent à l'ensemble du serveur.
  - Configuré directement dans le fichier principal (`httpd.conf` ou `apache2.conf`).
- **Exemple** :
  ```apache
  ServerRoot "/etc/httpd"
  Listen 80
  ```
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<br>

<div style="font-size:25px">

### **1. Contextes standards existants dans Apache**

#### **a. Serveur global (`server config`)**
- **Description** :
  - Les directives s'appliquent à l'ensemble du serveur.
  - Configuré directement dans le fichier principal (`httpd.conf` ou `apache2.conf`).
- **Exemple** :
  ```apache
  ServerRoot "/etc/httpd"
  Listen 80
  ```
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:27px">

#### **b. Virtual Host (`virtual host`)**
- **Description** :
  - Les directives s'appliquent à un hôte virtuel spécifique (une configuration pour chaque domaine ou sous-domaine).
  - Définies dans les fichiers comme `/etc/apache2/sites-available/example.conf`.
- **Exemple** :
  ```apache
  <VirtualHost *:80>
      ServerName www.example.com
      DocumentRoot /var/www/example
  </VirtualHost>
  ```
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:28px">

#### **c. Répertoires (`directory`)**
- **Description** :
  - Les directives s'appliquent à des répertoires ou chemins spécifiques du système de fichiers.
- **Exemple** :
  ```apache
  <Directory "/var/www/html">
      AllowOverride All
      Require all granted
  </Directory>
  ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:30px">

#### **d. Emplacement URL (`location`)**
- **Description** :
  - Les directives s'appliquent à des parties spécifiques de l'URL.
- **Exemple** :
  ```apache
  <Location "/admin">
      Require ip 192.168.1.0/24
  </Location>
  ```


</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:30px">

#### **e. Fichiers (`files`)**
- **Description** :
  - Les directives s'appliquent à des fichiers spécifiques.
- **Exemple** :
  ```apache
  <Files "secret.html">
      Require all denied
  </Files>
  ```


</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**


<div style="font-size:30px">

#### **f. Module spécifique (`ifmodule`)**
- **Description** :
  - Les directives ne s’appliquent que si un module spécifique est activé.
- **Exemple** :
  ```apache
  <IfModule mod_ssl.c>
      SSLEngine On
  </IfModule>
  ```
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:19px">

### **2. Nouveaux types ou usages de contextes dans Apache**

Avec les versions modernes d'Apache HTTPD (2.4+), de nouveaux types de contextes ou des fonctionnalités avancées ont émergé.

#### **a. Contexte conditionnel (`if`)**
- **Description** :
  - Permet de conditionner les directives en fonction d'expressions dynamiques (comme des variables d'environnement, des conditions IP, etc.).
- **Exemple** :
  ```apache
  <If "%{HTTP_HOST} == 'example.com'">
      DocumentRoot "/var/www/example"
  </If>
  ```
- **Utilité** :
  - Flexibilité accrue dans les configurations.
  - Simplifie les scénarios complexes comme la gestion de plusieurs domaines ou configurations spécifiques.

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:18px">

### **2. Nouveaux types ou usages de contextes dans Apache**

Avec les versions modernes d'Apache HTTPD (2.4+), de nouveaux types de contextes ou des fonctionnalités avancées ont émergé.

#### **b. Contexte utilisateur (`userdir`)**
- **Description** :
  - Définit les règles pour les répertoires personnels des utilisateurs.
- **Exemple** :
  ```apache
  <IfModule mod_userdir.c>
      UserDir public_html
      <Directory "/home/*/public_html">
          AllowOverride FileInfo AuthConfig
          Require all granted
      </Directory>
  </IfModule>
  ```
- **Utilité** :
  - Permet aux utilisateurs du système d'avoir leurs propres sites web accessibles via `/~username`.

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:22px">

### **2. Nouveaux types ou usages de contextes dans Apache**

Avec les versions modernes d'Apache HTTPD (2.4+), de nouveaux types de contextes ou des fonctionnalités avancées ont émergé.

#### **c. Contexte fichier spécifique (`filesmatch`)**
- **Description** :
  - S'applique aux fichiers correspondant à des motifs spécifiques (par exemple, via regex).
- **Exemple** :
  ```apache
  <FilesMatch "\.(txt|log)$">
      Require all denied
  </FilesMatch>
  ```
- **Utilité** :
  - Utile pour protéger certains types de fichiers sensibles (comme `.env`, `.htpasswd`, etc.).
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:22px">

### **2. Nouveaux types ou usages de contextes dans Apache**

Avec les versions modernes d'Apache HTTPD (2.4+), de nouveaux types de contextes ou des fonctionnalités avancées ont émergé.

#### **d. Contexte dynamique pour les proxys (`proxy`)**
- **Description** :
  - Utilisé pour gérer des directives spécifiques aux connexions proxy ou reverse proxy.
- **Exemple** :
  ```apache
  <Proxy "http://backend.example.com">
      Require ip 192.168.0.0/24
  </Proxy>
  ```
- **Utilité** :
  - Permet un contrôle précis des connexions aux proxys.

</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:22px">

### **2. Nouveaux types ou usages de contextes dans Apache**

Avec les versions modernes d'Apache HTTPD (2.4+), de nouveaux types de contextes ou des fonctionnalités avancées ont émergé.

#### **d. Contexte dynamique pour les proxys (`proxy`)**
- **Description** :
  - Utilisé pour gérer des directives spécifiques aux connexions proxy ou reverse proxy.
- **Exemple** :
  ```apache
  <Proxy "http://backend.example.com">
      Require ip 192.168.0.0/24
  </Proxy>
  ```
- **Utilité** :
  - Permet un contrôle précis des connexions aux proxys.

</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:17px">

### **2. Nouveaux types ou usages de contextes dans Apache**

#### **e. Contexte dynamique par moteur de règles (`ifdefine` et `ifenv`)**

- **Description** :
  - Conditionne les directives en fonction des variables définies ou de l'environnement.
- **Exemples** :
  - **IfDefine** :
    ```apache
    <IfDefine DEBUG>
        LogLevel debug
    </IfDefine>
    ```
    - Utilisé pour activer/désactiver des configurations spécifiques en ligne de commande :
      ```bash
      apache2ctl -D DEBUG
      ```

  - **IfEnv** :
    ```apache
    <IfEnv TEST_ENV>
        DocumentRoot "/var/www/test"
    </IfEnv>
    ```
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:28px">

### **2. Nouveaux types ou usages de contextes dans Apache**

#### **f. Contexte par balises HTML intégrées (`htaccess inline`)**

- **Description** :
  - Directives placées directement dans un fichier `.htaccess`.
- **Exemple** :
  ```apache
  RewriteEngine On
  RewriteRule ^about$ /about.html [L]
  ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:28px">

### **3. Bonnes pratiques dans l’utilisation des contextes**

1. **Minimisez l’utilisation des fichiers `.htaccess`** :
   - Préférez la configuration dans les fichiers de Virtual Host ou dans le fichier principal (`httpd.conf`).
   - `.htaccess` ajoute une surcharge, car Apache le charge à chaque requête.

2. **Utilisez des directives conditionnelles (`<If>` et `<IfModule>`)** pour les configurations dynamiques ou spécifiques.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:28px">

### **3. Bonnes pratiques dans l’utilisation des contextes**

3. **Protégez les fichiers sensibles avec des contextes `FilesMatch`** :
   - Exemple pour protéger `.htpasswd` :
     ```apache
     <FilesMatch "^\.ht">
         Require all denied
     </FilesMatch>
     ```

4. **Planifiez et organisez vos Virtual Hosts** :
   - Placez un fichier distinct par domaine dans `/etc/apache2/sites-available/` ou `/etc/httpd/conf.d/`.
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Nouveaux types de contextes dans Apache HTTPD 2.4**

<div style="font-size:28px">

### **Résumé des nouveaux types de contextes**

| **Contexte**       | **Usage principal**                                                                 |
|---------------------|-------------------------------------------------------------------------------------|
| `<If>`             | Conditions dynamiques basées sur des variables ou expressions (ex. domaine, IP).   |
| `<FilesMatch>`     | S'applique aux fichiers correspondant à un motif spécifique (regex).               |
| `<Proxy>`          | Contrôle des proxys et reverse-proxys.                                             |
| `<IfDefine>`       | Activé/désactivé en fonction de variables définies au démarrage d'Apache.          |
| `<IfEnv>`          | Activé/désactivé en fonction des variables d'environnement.  

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Panorama des modules d'Apache 2.4**


<div style="font-size:30px">

Les modules étendent les fonctionnalités d'Apache HTTPD et permettent de répondre à des besoins spécifiques en matière de sécurité, performances, gestion des contenus, proxy, etc. Nous allons faire un petit panorama des modules par catégorie.


</div>


---

### Apache HTTPD 2.4 : rappels et nouveautés

#### **Panorama des modules d'Apache 2.4**


<div style="font-size:18px">

### **1. Modules de base**

Ces modules sont nécessaires au fonctionnement principal du serveur Apache.

| **Module**       | **Description**                                                                 |
|-------------------|---------------------------------------------------------------------------------|
| `core`           | Gère les fonctionnalités principales (traitement des requêtes, gestion des connexions). |
| `mpm_prefork`    | Multi-Processing Module basé sur des processus (mode non threadé).              |
| `mpm_worker`     | Multi-Processing Module basé sur des threads et processus combinés.             |
| `mpm_event`      | Extension de `worker`, optimisé pour les connexions persistantes (Keep-Alive).  |
| `mod_log_config` | Permet la configuration des journaux d'accès et d'erreurs.                     |
| `mod_version`    | Active des directives en fonction de la version d'Apache.   
</div>

---


#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules de sécurité**

<div style="font-size:16px">

##### **2. Modules de sécurité**

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_ssl`          | Ajoute le support HTTPS via SSL/TLS.                                           |
| `mod_auth_basic`   | Permet l'authentification basique par mot de passe.                            |
| `mod_auth_digest`  | Authentification plus sécurisée que `mod_auth_basic`.                         |
| `mod_authz_host`   | Contrôle l'accès basé sur les adresses IP.                                     |
| `mod_security`     | Module tiers pour un pare-feu d'application web (WAF).                        |
| `mod_evasive`      | Protège contre les attaques DoS/DDoS.                                          |
| `mod_headers`      | Ajoute ou modifie les en-têtes HTTP (utile pour CSP, CORS, etc.).              |
| `mod_remoteip`     | Remplace l'adresse IP client par celle envoyée dans un en-tête (proxy).        |  
</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules de contenus**

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_dir`          | Permet de définir le fichier par défaut d'un répertoire (exemple : `index.html`).|
| `mod_autoindex`    | Génère automatiquement une liste de fichiers pour les répertoires sans index.   |
| `mod_alias`        | Crée des alias d'URL (exemple : `/docs` vers un autre répertoire).              |
| `mod_negotiation`  | Permet la négociation de contenu (choix du fichier basé sur les préférences du client). |
| `mod_mime`         | Détermine le type MIME des fichiers en fonction de leur extension.              |
| `mod_rewrite`      | Permet la réécriture dynamique des URL.                                         |
| `mod_deflate`      | Comprime les réponses HTTP pour réduire l'utilisation de la bande passante.     |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules de proxy et reverse proxy**

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_proxy`        | Active les fonctionnalités de proxy de base.                                   |
| `mod_proxy_http`   | Permet le proxying des requêtes HTTP.                                           |
| `mod_proxy_fcgi`   | Gère les connexions avec des applications FastCGI (exemple : PHP-FPM).          |
| `mod_proxy_balancer`| Gère l'équilibrage de charge entre plusieurs serveurs backend.                 |
| `mod_proxy_wstunnel`| Supporte les connexions WebSocket via un proxy.                               |
| `mod_proxy_ftp`    | Permet le proxying des connexions FTP.                                          |

</div>

---


#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules pour les applications dynamiques**

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_cgi`          | Exécute des scripts CGI traditionnels.                                         |
| `mod_php`          | Intègre PHP directement dans Apache (souvent remplacé par PHP-FPM).            |
| `mod_perl`         | Permet d'exécuter des scripts Perl.                                            |
| `mod_wsgi`         | Exécute des applications Python (exemple : Django).                           |
| `mod_lua`          | Permet d'intégrer des scripts Lua dans Apache.                                |

</div>

---


#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules conditionnels**


<br>

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_status`       | Fournit une page de statut avec des informations sur les performances et connexions.|
| `mod_info`         | Affiche la configuration complète d'Apache pour diagnostic.                    |
| `mod_watchdog`     | Fournit un mécanisme pour surveiller les services d'arrière-plan d'Apache.      |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules de diagnostic**


<br>

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_if`           | Permet d'utiliser des conditions dynamiques basées sur des variables.           |
| `mod_define`       | Définit des variables utilisées dans les directives conditionnelles.            |
| `mod_env`          | Permet de définir ou d'utiliser des variables d'environnement.                  |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules spécifiques à HTTP/2**


<br>

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_http2`        | Ajoute le support HTTP/2, améliorant la vitesse et les performances des connexions. |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4 : Modules tiers populaires**


<br>

| **Module**         | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `mod_pagespeed`    | Optimise automatiquement les performances des sites (cache, minification, etc.).|
| `mod_security`     | Fournit une protection WAF (pare-feu d'application web).                       |
| `mod_cluster`      | Permet le clustering avec Apache Tomcat ou JBoss.                              |

</div>

---

#### Apache HTTPD 2.4 : rappels et nouveautés

##### **Panorama des modules d'Apache 2.4**

<div style="font-size:20px">

### **Résumé des modules essentiels à activer selon les besoins**

| **Type de serveur**            | **Modules essentiels à activer**                                                    |
|--------------------------------|-------------------------------------------------------------------------------------|
| **Serveur de base (HTTP)**      | `mod_dir`, `mod_alias`, `mod_mime`, `mod_deflate`, `mod_headers`, `mod_rewrite`    |
| **Serveur HTTPS**               | `mod_ssl`, `mod_headers`, `mod_deflate`, `mod_expires`                             |
| **Proxy ou reverse proxy**      | `mod_proxy`, `mod_proxy_http`, `mod_proxy_fcgi`, `mod_proxy_balancer`              |
| **Applications dynamiques**     | `mod_php`, `mod_proxy_fcgi`, `mod_wsgi`, `mod_perl`                                |
| **Serveur haute performance**   | `mod_http2`, `mod_deflate`, `mod_expires`, `mod_cache`                             |
| **Sécurité avancée**            | `mod_security`, `mod_evasive`, `mod_headers`, `mod_remoteip`                       |

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<br>

<div style="font-size:30px">

- Apache HTTPD 2.4 introduit le support natif du protocole **HTTP/2** via le module `mod_http2`. 
- HTTP/2 est conçu pour améliorer les performances des sites web modernes.
- Le protocole **HTTP/2** est une amélioration significative de HTTP/1.1. 
- Il améliore les performances, la latence et la gestion des connexions réseau. 
- Apache HTTPD prend en charge HTTP/2 à partir de la version **2.4.17**, mais il nécessite une configuration spécifique.


</div>


---


## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:25px">

### **1. Avantages du protocole HTTP/2**
- **Multiplexage** : Plusieurs requêtes et réponses peuvent être échangées simultanément sur une seule connexion TCP.
- **Compression des en-têtes** : Réduit la taille des en-têtes HTTP, ce qui améliore les performances.
- **Priorisation des ressources** : Permet de prioriser certains contenus, comme les fichiers CSS ou JavaScript critiques.
- **Connexions persistantes** : Réduit la surcharge en limitant l'ouverture de connexions multiples.
- **Performances accrues** : Idéal pour les sites modernes, notamment ceux avec beaucoup de fichiers (CSS, JS, images).
</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:25px">

### **2. Prérequis pour activer HTTP/2**

1. **Version d'Apache** : HTTP/2 est pris en charge à partir de la version **2.4.17**.
   - Vérifiez votre version avec :
     ```bash
     apache2 -v
     ```

2. **SSL/TLS activé** : HTTP/2 nécessite HTTPS pour être activé dans Apache.

3. **Module requis** : Le module `mod_http2` doit être activé.

4. **Version d’OpenSSL** : OpenSSL **1.0.2** ou supérieur est requis.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:25px">

### **3. Étapes pour activer HTTP/2**

#### **a. Vérifier et activer le module HTTP/2**
- **Ubuntu/Debian** :
  Activez le module `http2` :
  ```bash
  sudo a2enmod http2
  sudo systemctl reload apache2
  ```

#### **b. Modifier la configuration SSL pour activer HTTP/2**

Ajoutez ou modifiez la directive `Protocols` dans le fichier de configuration du Virtual Host HTTPS.

</div>


---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:20px">

### **3. Étapes pour activer HTTP/2**

#### **b. Modifier la configuration SSL pour activer HTTP/2**

Exemple de configuration dans `/etc/apache2/sites-available/default-ssl.conf` (Ubuntu) ou `/etc/httpd/conf.d/ssl.conf` (CentOS) :

```apache
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/html

    SSLEngine On
    SSLCertificateFile /etc/apache2/ssl/example.crt
    SSLCertificateKeyFile /etc/apache2/ssl/example.key

    # Activer HTTP/2
    Protocols h2 h2c http/1.1

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:24px">

### **3. Étapes pour activer HTTP/2**

#### **c. Vérifier et activer MPM event**
HTTP/2 fonctionne mieux avec le MPM **event**. Activez-le si ce n’est pas déjà fait :

- Désactivez le MPM prefork (si activé) :
  ```bash
  sudo a2dismod mpm_prefork
  ```

- Activez le MPM event :
  ```bash
  sudo a2enmod mpm_event
  ```

- Rechargez Apache :
  ```bash
  sudo systemctl reload apache2
  ```

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:24px">

### **4. Tester HTTP/2**

#### **a. Avec un navigateur**
- Accédez à votre site via HTTPS (`https://example.com`).
- Ouvrez les outils de développement (F12), allez dans l'onglet **Network**, et vérifiez la colonne **Protocol**. Si HTTP/2 est activé, vous verrez `h2`.

#### **b. Avec `curl`**
Utilisez l'option `--http2` :
```bash
curl -I --http2 https://example.com
```

#### **c. Avec un outil en ligne**
Utilisez des outils comme [HTTP/2 Test](https://tools.keycdn.com/http2-test) pour vérifier si HTTP/2 est activé sur votre site.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:24px">

### **5. Résolution des problèmes courants**

1. **HTTP/2 non activé malgré la configuration**
   - Vérifiez que le module `http2` est chargé :
     ```bash
     apache2ctl -M | grep http2
     ```
     Vous devriez voir :
     ```
     http2_module (shared)
     ```

2. **Erreurs liées à OpenSSL**
   - Assurez-vous que votre version d’OpenSSL est **1.0.2** ou supérieure :
     ```bash
     openssl version
     ```
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<br>
<div style="font-size:30px">

### **5. Résolution des problèmes courants**

 3. **Problème de compatibilité avec MPM prefork**
   - HTTP/2 ne fonctionne pas avec `mpm_prefork`. Utilisez `mpm_event` ou `mpm_worker`.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:35px">

### **6. Résumé des directives importantes pour HTTP/2**

| **Directive**       | **Description**                                                                 |
|---------------------|---------------------------------------------------------------------------------|
| `Protocols`         | Définit les protocoles pris en charge (exemple : `h2 h2c http/1.1`).           |
| `ProtocolsHonorOrder` | Priorise les protocoles dans l'ordre spécifié dans `Protocols`.                |
</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

#### **Support du protocole HTTP/2**

<div style="font-size:30px">

### **7. Bénéfices pratiques de HTTP/2**

1. **Sites à fort trafic** : HTTP/2 réduit la latence et améliore les temps de chargement.
2. **Applications modernes** : Idéal pour les sites avec beaucoup de fichiers (CSS, JS, images).
3. **Meilleures performances SEO** : Les moteurs de recherche favorisent les sites rapides et sécurisés.

</div>

---

## Apache HTTPD 2.4 : rappels et nouveautés

<br>
<br>
<center>
<img src="./assets/demo.jpg" width="400px">
</center>


---

<!-- _class: lead -->
<!-- _paginate: false -->

## Héberger des applications PHP

---

## Héberger des applications PHP

<div style="font-size:40px">

<br>
<br>

Héberger des applications PHP et faire cohabiter **PHP 5** et **PHP 7** sur le même serveur Apache peut être utile lorsque vous devez prendre en charge des applications héritées utilisant PHP 5 tout en supportant des applications modernes développées avec PHP 7 ou une version ultérieure.

</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:29px">

### **1. Concepts clés**

1. **Apache et PHP-FPM** :
   - Apache peut exécuter plusieurs versions de PHP en utilisant **PHP-FPM** (FastCGI Process Manager).
   - Chaque version de PHP est configurée dans un pool distinct.

2. **Virtual Hosts** :
   - Vous configurez Apache pour assigner des versions spécifiques de PHP à des **Virtual Hosts** (ou des répertoires spécifiques).

</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:25px">

### **2. Installation des versions PHP**

#### **Installer PHP 5.6 et PHP 7.4 (ou autres versions)**

Sur Ubuntu/Debian :
1. Installez les dépôts nécessaires :
   ```bash
   sudo apt update
   sudo apt install software-properties-common
   sudo add-apt-repository ppa:ondrej/php
   sudo apt update
   ```

2. Installez PHP 5.6 et PHP 7.4 avec leurs modules FPM :
   ```bash
   sudo apt install php5.6 php5.6-fpm php7.4 php7.4-fpm
   ```
</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:18px">

### **3. Configurer PHP-FPM**

#### **Configurer les pools pour chaque version**
Les pools PHP-FPM permettent de séparer les applications exécutées sous différentes versions de PHP.

1. **Fichier de configuration PHP-FPM pour PHP 5.6** :
   - Chemin : `/etc/php/5.6/fpm/pool.d/www.conf` (ou similaire selon l'OS).
   - Modifiez le port ou le socket pour éviter les conflits.
     ```ini
     [www]
     listen = /run/php/php5.6-fpm.sock
     ```

2. **Fichier de configuration PHP-FPM pour PHP 7.4** :
   - Chemin : `/etc/php/7.4/fpm/pool.d/www.conf`.
   - Exemple :
     ```ini
     [www]
     listen = /run/php/php7.4-fpm.sock
     ```
</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:30px">

<br>

### **3. Configurer PHP-FPM**

#### **Configurer les pools pour chaque version**

3. Redémarrez les services PHP-FPM :
   ```bash
   sudo systemctl restart php5.6-fpm
   sudo systemctl restart php7.4-fpm
   ```
</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:30px">

<br>

### **4. Configurer Apache pour cohabiter PHP 5 et PHP 7**

#### **Activer les modules Apache nécessaires**
Activez les modules requis pour PHP-FPM et FastCGI :
```bash
sudo a2enmod proxy_fcgi setenvif
sudo a2enmod actions
sudo systemctl reload apache2
```

</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:35px">

### **4. Configurer Apache pour cohabiter PHP 5 et PHP 7**

#### **Activer les modules Apache nécessaires**
Activez les modules requis pour PHP-FPM et FastCGI :
```bash
sudo a2enmod proxy_fcgi setenvif
sudo a2enmod actions
sudo systemctl reload apache2
```


</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:18px">

#### **Configurer des Virtual Hosts**
Créez un Virtual Host pour chaque version de PHP.

1. **Virtual Host pour PHP 5.6** :
   Fichier : `/etc/apache2/sites-available/php5.6.example.com.conf`.
   ```apache
   <VirtualHost *:80>
       ServerName php5.6.example.com
       DocumentRoot /var/www/php5.6

       <Directory /var/www/php5.6>
           AllowOverride All
           Require all granted
       </Directory>

       <FilesMatch \.php$>
           SetHandler "proxy:unix:/run/php/php5.6-fpm.sock|fcgi://localhost"
       </FilesMatch>

       ErrorLog ${APACHE_LOG_DIR}/php5.6-error.log
       CustomLog ${APACHE_LOG_DIR}/php5.6-access.log combined
   </VirtualHost>
   ```
</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:18px">

#### **Configurer des Virtual Hosts**
Créez un Virtual Host pour chaque version de PHP.

2. **Virtual Host pour PHP 7.4** :
   Fichier : `/etc/apache2/sites-available/php7.4.example.com.conf`.
   ```apache
   <VirtualHost *:80>
       ServerName php7.4.example.com
       DocumentRoot /var/www/php7.4

       <Directory /var/www/php7.4>
           AllowOverride All
           Require all granted
       </Directory>

       <FilesMatch \.php$>
           SetHandler "proxy:unix:/run/php/php7.4-fpm.sock|fcgi://localhost"
       </FilesMatch>

       ErrorLog ${APACHE_LOG_DIR}/php7.4-error.log
       CustomLog ${APACHE_LOG_DIR}/php7.4-access.log combined
   </VirtualHost>
   ```
</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:30px">

<br>

#### **Configurer des Virtual Hosts**
Créez un Virtual Host pour chaque version de PHP.

3. Activez les Virtual Hosts :
   ```bash
   sudo a2ensite php5.6.example.com.conf
   sudo a2ensite php7.4.example.com.conf
   sudo systemctl reload apache2
   ```

</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:28px">

### **5. Tester la configuration**

1. **Créer des répertoires de test**
   - Répertoire pour PHP 5.6 :
     ```bash
     sudo mkdir -p /var/www/php5.6
     echo "<?php phpinfo(); ?>" | sudo tee /var/www/php5.6/index.php
     ```

   - Répertoire pour PHP 7.4 :
     ```bash
     sudo mkdir -p /var/www/php7.4
     echo "<?php phpinfo(); ?>" | sudo tee /var/www/php7.4/index.php
     ```

</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:30px">

### **5. Tester la configuration**

2. **Accéder aux Virtual Hosts**
   - Testez les URLs :
     - [http://php5.6.example.com](http://php5.6.example.com) : Vous devriez voir la page PHP Info indiquant **PHP 5.6**.
     - [http://php7.4.example.com](http://php7.4.example.com) : Vous devriez voir la page PHP Info indiquant **PHP 7.4**.


</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:26px">

### **6. Résolution des problèmes**

- **Erreur 500 ou aucun contenu PHP exécuté :**
  - Vérifiez que les sockets PHP-FPM sont actifs :
    ```bash
    sudo systemctl status php5.6-fpm
    sudo systemctl status php7.4-fpm
    ```
  - Vérifiez les logs Apache pour plus de détails :
    ```bash
    tail -f /var/log/apache2/error.log
    ```

- **Conflits entre les versions PHP :**
  - Assurez-vous que chaque version utilise un socket ou un port distinct.

</div>

---

## Héberger des applications PHP

#### Faire cohabiter PHP5 et PHP7.

<div style="font-size:32px">

### **Résumé**

1. Installez plusieurs versions de PHP via PHP-FPM.
2. Configurez des pools PHP-FPM distincts pour chaque version.
3. Configurez Apache avec des Virtual Hosts assignés à chaque version PHP.
4. Testez les sites pour vérifier que chaque Virtual Host utilise la version PHP appropriée.

</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<br>

<div style="font-size:43px">

Ces technologies sont utilisées pour exécuter des applications dynamiques (comme des scripts PHP) sur des serveurs web tels qu'Apache.

</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM


<div style="font-size:28px">

### **1. Common Gateway Interface (CGI)**

#### **Description**
- Le **Common Gateway Interface (CGI)** est une norme permettant à un serveur web de communiquer avec des scripts ou programmes externes pour générer des pages dynamiques.
- Fonctionnement :
  - Pour chaque requête, le serveur web exécute un nouveau processus pour le script.
  - Le script retourne une réponse au serveur qui est ensuite envoyée au client.
</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:23px">

### **1. Common Gateway Interface (CGI)**

#### **Avantages**
- Simplicité : Facile à implémenter.
- Compatibilité : Supporte de nombreux langages (Perl, Python, PHP, etc.).
- Isolation : Chaque requête étant un processus distinct, les erreurs affectent moins le reste du système.

#### **Inconvénients**
- Performance médiocre : Un nouveau processus est créé pour chaque requête, ce qui est coûteux en ressources.
- Non adapté à une charge élevée : Problèmes de scalabilité dans des environnements à fort trafic.

</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:23px">

### **2. CGI Daemon (CGID)**

#### **Description**
- **CGID** est une variante de CGI introduite dans Apache.
- Fonctionnement :
  - Les scripts CGI sont gérés par un daemon dédié au lieu d'être directement exécutés par le serveur web.
  - Cela réduit la surcharge liée à la création et destruction de processus à chaque requête.

#### **Avantages**
- Réduction de la charge serveur par rapport à CGI classique.
- Toujours isolé entre chaque requête.
</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:28px">

### **2. CGI Daemon (CGID)**

#### **Inconvénients**
- Toujours limité en termes de performance par rapport à des solutions modernes comme FastCGI.
- Légèrement plus complexe à configurer que CGI.

#### **Cas d'usage**
- Environnements où CGI est nécessaire mais avec une meilleure gestion des ressources.
</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:31px">

### **3. FastCGI**

#### **Description**
- **FastCGI** est une amélioration significative de CGI.
- Fonctionnement :
  - Au lieu de créer un nouveau processus pour chaque requête, un **processus persistant** est créé.
  - Ce processus persistant gère plusieurs requêtes, réduisant la surcharge.


</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:21px">

### **3. FastCGI**

#### **Avantages**
- Performance : Réutilisation des processus permet une gestion beaucoup plus rapide des requêtes.
- Support de plusieurs langages : Fonctionne avec PHP, Python, Ruby, etc.
- Scalabilité : Convient aux sites à fort trafic.
- Séparation : Les scripts peuvent être exécutés sur un serveur distant ou dédié.

#### **Inconvénients**
- Configuration plus complexe que CGI.
- Nécessite un module spécifique sur le serveur web (comme `mod_proxy_fcgi` pour Apache).

#### **Cas d'usage**
- Applications web dynamiques à fort trafic nécessitant de meilleures performances que CGI classique.


</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:29px">

### **4. PHP-FPM (FastCGI Process Manager)**

#### **Description**
- **PHP-FPM** est une implémentation de FastCGI spécialement conçue pour PHP.
- Fonctionnement :
  - Crée des processus persistants pour traiter les scripts PHP.
  - Offre des fonctionnalités avancées comme la gestion des pools, des limites de ressources, et des logs.


</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:20px">

### **4. PHP-FPM (FastCGI Process Manager)**

#### **Avantages**
- Performance : Plus rapide que CGI ou CGID.
- Gestion des ressources : Vous pouvez configurer des pools distincts avec des limites spécifiques (CPU, RAM) pour chaque application ou domaine.
- Fiabilité : Gère les scripts PHP de manière isolée pour minimiser les effets des erreurs.
- Flexibilité : Compatible avec plusieurs serveurs web (Apache, Nginx, etc.).

#### **Inconvénients**
- Limité à PHP.
- Configuration initiale légèrement complexe.

#### **Cas d'usage**
- Hébergement d'applications PHP modernes avec des besoins en performances et scalabilité.
- Idéal pour les serveurs web combinant Apache ou Nginx avec PHP.

</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:20px">

### **Comparaison des technologies**

| **Critère**          | **CGI**           | **CGID**         | **FastCGI**      | **PHP-FPM**       |
|-----------------------|-------------------|------------------|------------------|-------------------|
| **Performance**      | Faible            | Moyenne          | Élevée           | Très élevée       |
| **Compatibilité**     | Tous les langages | Tous les langages| Tous les langages| PHP uniquement    |
| **Scalabilité**       | Faible            | Moyenne          | Bonne            | Excellente        |
| **Complexité**        | Simple            | Moyenne          | Moyenne          | Moyenne à élevée  |
| **Usage recommandé** | Sites très simples| Sites avec faible à modérée charge| Sites dynamiques ou API modernes | Applications PHP performantes |

</div>

---


## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:30px">

<br>

### **Exemple de configuration**

#### **FastCGI avec PHP-FPM dans Apache**

1. **Activer les modules nécessaires**
   ```bash
   sudo a2enmod proxy_fcgi setenvif
   sudo systemctl restart apache2
   ```

</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:20px">

2. **Configurer un Virtual Host**
   Fichier : `/etc/apache2/sites-available/example.conf`
   ```apache
   <VirtualHost *:80>
       ServerName example.com
       DocumentRoot /var/www/html

       <Directory /var/www/html>
           AllowOverride All
           Require all granted
       </Directory>

       # Utiliser PHP-FPM via FastCGI
       <FilesMatch \.php$>
           SetHandler "proxy:unix:/run/php/php7.4-fpm.sock|fcgi://localhost"
       </FilesMatch>

       ErrorLog ${APACHE_LOG_DIR}/example-error.log
       CustomLog ${APACHE_LOG_DIR}/example-access.log combined
   </VirtualHost>
   ```
</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:30px">

3. **Redémarrer les services**
   ```bash
   sudo systemctl restart php7.4-fpm
   sudo systemctl reload apache2
   ```

4. **Tester**
   - Créez un fichier `info.php` :
     ```php
     <?php phpinfo(); ?>
     ```
   - Accédez à `http://example.com/info.php` et vérifiez que PHP fonctionne via FastCGI.

</div>

---

## Héberger des applications PHP

#### CGI, CGID, Fast CGI et PHP-FPM

<div style="font-size:30px">

### **Résumé**

1. **CGI** : Simple mais obsolète, adapté à des environnements avec peu de trafic.
2. **CGID** : Une légère amélioration de CGI pour des cas spécifiques.
3. **FastCGI** : Une solution moderne et performante pour exécuter des scripts dynamiques.
4. **PHP-FPM** : Implémentation de FastCGI optimisée pour PHP, offrant des performances et une scalabilité excellentes.
</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:35px">

<br>

Lors de l'hébergement d'applications PHP avec Apache et PHP-FPM, il est important de :
1. Configurer des **droits et identités dédiées** pour isoler les applications.
2. Gérer les **sessions PHP** pour assurer la sécurité et le bon fonctionnement des utilisateurs.

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:24px">

### **1. Droits et identités dédiées**

Chaque application hébergée doit fonctionner sous une identité utilisateur différente pour :
- Isoler les applications (éviter qu'une application compromise n'accède aux données d'une autre).
- Améliorer la sécurité globale du serveur.

#### **Étapes pour configurer des identités dédiées**

##### **a. Créer des utilisateurs système pour chaque application**

1. Créez un utilisateur système pour chaque application :
   ```bash
   sudo adduser --system --no-create-home --group phpapp1
   sudo adduser --system --no-create-home --group phpapp2
   ```
</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:25px">

### **1. Droits et identités dédiées**

#### **Étapes pour configurer des identités dédiées**

##### **a. Créer des utilisateurs système pour chaque application**

2. Attribuez les permissions des répertoires des applications :
   - Application 1 :
     ```bash
     sudo chown -R phpapp1:phpapp1 /var/www/phpapp1
     ```
   - Application 2 :
     ```bash
     sudo chown -R phpapp2:phpapp2 /var/www/phpapp2
     ```

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:23px">

### **1. Droits et identités dédiées**

#### **Étapes pour configurer des identités dédiées**

##### **b. Configurer PHP-FPM avec des pools dédiés**

1. Modifiez le fichier de configuration des pools PHP-FPM pour chaque application :

- **Pour l'application 1** : `/etc/php/7.4/fpm/pool.d/phpapp1.conf`
  ```ini
  [phpapp1]
  user = phpapp1
  group = phpapp1
  listen = /run/php/phpapp1.sock
  listen.owner = www-data
  listen.group = www-data
  listen.mode = 0660
  ```

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:23px">

### **1. Droits et identités dédiées**

#### **Étapes pour configurer des identités dédiées**

##### **b. Configurer PHP-FPM avec des pools dédiés**

1. Modifiez le fichier de configuration des pools PHP-FPM pour chaque application :

- **Pour l'application 2** : `/etc/php/8.1/fpm/pool.d/phpapp2.conf`
  ```ini
  [phpapp2]
  user = phpapp2
  group = phpapp2
  listen = /run/php/phpapp2.sock
  listen.owner = www-data
  listen.group = www-data
  listen.mode = 0660
  ```

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:17px">

### **1. Droits et identités dédiées**

#### **Étapes pour configurer des identités dédiées**

##### **b. Configurer PHP-FPM avec des pools dédiés**

1. Modifiez le fichier de configuration des pools PHP-FPM pour chaque application :

- **Pour l'application 2** : `/etc/php/8.1/fpm/pool.d/phpapp2.conf`
  ```ini
  [phpapp2]
  user = phpapp2
  group = phpapp2
  listen = /run/php/phpapp2.sock
  listen.owner = www-data
  listen.group = www-data
  listen.mode = 0660
  ```
2. Redémarrez PHP-FPM pour appliquer les changements :
   ```bash
   sudo systemctl restart php7.4-fpm
   sudo systemctl restart php8.1-fpm
   ```

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:19px">

### **1. Droits et identités dédiées**

#### **Étapes pour configurer des identités dédiées**

##### **c. Configurer Apache pour utiliser les pools dédiés**

Modifiez les Virtual Hosts pour chaque application afin d'utiliser les sockets spécifiques :

- **Application 1** : `/etc/apache2/sites-available/phpapp1.conf`
  ```apache
  <VirtualHost *:80>
      ServerName phpapp1.example.com
      DocumentRoot /var/www/phpapp1

      <Directory /var/www/phpapp1>
          AllowOverride All
          Require all granted
      </Directory>

      <FilesMatch \.php$>
          SetHandler "proxy:unix:/run/php/phpapp1.sock|fcgi://localhost"
      </FilesMatch>
  </VirtualHost>
  ```
</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:19px">

### **1. Droits et identités dédiées**

#### **Étapes pour configurer des identités dédiées**

##### **c. Configurer Apache pour utiliser les pools dédiés**

Modifiez les Virtual Hosts pour chaque application afin d'utiliser les sockets spécifiques :

- **Application 2** : `/etc/apache2/sites-available/phpapp2.conf`
  ```apache
  <VirtualHost *:80>
      ServerName phpapp2.example.com
      DocumentRoot /var/www/phpapp2

      <Directory /var/www/phpapp2>
          AllowOverride All
          Require all granted
      </Directory>

      <FilesMatch \.php$>
          SetHandler "proxy:unix:/run/php/phpapp2.sock|fcgi://localhost"
      </FilesMatch>
  </VirtualHost>
  ```
</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:26px">

### **1. Droits et identités dédiées**

#### **Étapes pour configurer des identités dédiées**

##### **c. Configurer Apache pour utiliser les pools dédiés**

Activez les Virtual Hosts et redémarrez Apache :
```bash
sudo a2ensite phpapp1.conf phpapp2.conf
sudo systemctl reload apache2
```

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:23px">

### **2. Gérer les sessions PHP**

Les sessions sont une fonctionnalité clé pour les applications web dynamiques. Une mauvaise gestion des sessions peut exposer les utilisateurs à des risques (vol de session, accès non autorisé).

#### **a. Configurer un répertoire de sessions dédié pour chaque application**

1. Créez des répertoires distincts pour stocker les fichiers de session :
   ```bash
   sudo mkdir -p /var/lib/php/sessions/phpapp1
   sudo mkdir -p /var/lib/php/sessions/phpapp2
   ```

2. Attribuez les permissions appropriées :
   ```bash
   sudo chown -R phpapp1:phpapp1 /var/lib/php/sessions/phpapp1
   sudo chown -R phpapp2:phpapp2 /var/lib/php/sessions/phpapp2
   sudo chmod 700 /var/lib/php/sessions/phpapp1 /var/lib/php/sessions/phpapp2
   ```


</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:23px">

### **2. Gérer les sessions PHP**

Les sessions sont une fonctionnalité clé pour les applications web dynamiques. Une mauvaise gestion des sessions peut exposer les utilisateurs à des risques (vol de session, accès non autorisé).

#### **b. Configurer PHP-FPM pour utiliser ces répertoires**

- **Pour l'application 1** :
  Modifiez `/etc/php/7.4/fpm/pool.d/phpapp1.conf` :
  ```ini
  php_value[session.save_path] = /var/lib/php/sessions/phpapp1
  ```

- **Pour l'application 2** :
  Modifiez `/etc/php/8.1/fpm/pool.d/phpapp2.conf` :
  ```ini
  php_value[session.save_path] = /var/lib/php/sessions/phpapp2
  ```

3. Redémarrez PHP-FPM :
   ```bash
   sudo systemctl restart php7.4-fpm
   sudo systemctl restart php8.1-fpm
   ```

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:29px">

<br>

### **3. Sécurisation supplémentaire des sessions**

#### **a. Configurer des options sécurisées pour les sessions PHP**
Ajoutez ces directives dans les fichiers de configuration PHP (`php.ini`) :

```ini
session.cookie_httponly = 1      ; Empêche l'accès aux cookies via JavaScript.
session.cookie_secure = 1        ; Force l'utilisation de HTTPS pour les cookies.
session.use_strict_mode = 1      ; Rejette les sessions non valides.
session.use_only_cookies = 1     ; Interdit l'utilisation d'URL contenant des identifiants de session.
```

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:30px">

<br>

### **3. Sécurisation supplémentaire des sessions**

#### **b. Utiliser une session basée sur une base de données**
Pour une application complexe, vous pouvez stocker les sessions dans une base de données sécurisée. Cela facilite la gestion et améliore la résilience des sessions.

</div>

---

## Héberger des applications PHP

#### Droits et identité dédiée, sessions

<div style="font-size:23px">

### **Résumé de la configuration**

1. **Identités dédiées** :
   - Chaque application a son propre utilisateur et groupe système.
   - Chaque application fonctionne dans un pool PHP-FPM dédié.

2. **Gestion des sessions** :
   - Les sessions sont stockées dans des répertoires séparés pour éviter toute interférence.
   - Les sessions sont sécurisées avec des options PHP appropriées.

3. **Résultat** :
   - Applications isolées en termes de droits et sessions.
   - Sécurité renforcée pour les utilisateurs et données.

</div>

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


