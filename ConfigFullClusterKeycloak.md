
# Configuration Docker full Cluster  

1. **Docker Compose pour MySQL Galera Cluster (3 nœuds)**
2. **Docker Compose pour Keycloak Node 1**
3. **Docker Compose pour Keycloak Node 2**

### 1. Docker Compose pour MySQL Galera Cluster (3 nœuds)

Ce fichier est destiné à un serveur unique où les trois nœuds MySQL Galera sont déployés.

```yaml
version: '3.8'

services:
  mysql-galera-node1:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: keycloak
      MYSQL_USER: keycloak
      MYSQL_PASSWORD: password
      MYSQL_ROOT_HOST: '%'
    volumes:
      - mysql_node1_data:/var/lib/mysql
      - ./my-node1.cnf:/etc/mysql/my.cnf
    networks:
      - galera_net
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      retries: 3
    user: "1000:1000"

  mysql-galera-node2:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: keycloak
      MYSQL_USER: keycloak
      MYSQL_PASSWORD: password
      MYSQL_ROOT_HOST: '%'
    volumes:
      - mysql_node2_data:/var/lib/mysql
      - ./my-node2.cnf:/etc/mysql/my.cnf
    networks:
      - galera_net
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      retries: 3
    user: "1000:1000"

  mysql-galera-node3:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: keycloak
      MYSQL_USER: keycloak
      MYSQL_PASSWORD: password
      MYSQL_ROOT_HOST: '%'
    volumes:
      - mysql_node3_data:/var/lib/mysql
      - ./my-node3.cnf:/etc/mysql/my.cnf
    networks:
      - galera_net
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 30s
      retries: 3
    user: "1000:1000"

volumes:
  mysql_node1_data:
    driver: local
    driver_opts:
      type: 'none'
      device: '/path/to/storage/mysql_node1'
      o: 'bind'
  mysql_node2_data:
    driver: local
    driver_opts:
      type: 'none'
      device: '/path/to/storage/mysql_node2'
      o: 'bind'
  mysql_node3_data:
    driver: local
    driver_opts:
      type: 'none'
      device: '/path/to/storage/mysql_node3'
      o: 'bind'

networks:
  galera_net:
    driver: bridge
```

### 2. Docker Compose pour Keycloak Node 1

Ce fichier est destiné au serveur 2 pour déployer le premier nœud Keycloak.

```yaml
version: '3.8'

services:
  keycloak-node1:
    image: quay.io/keycloak/keycloak:21.0.0
    environment:
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: admin
      DB_VENDOR: MYSQL
      DB_ADDR: mysql-galera-node1
      DB_PORT: 3306
      DB_DATABASE: keycloak
      DB_USER: keycloak
      DB_PASSWORD: password
    ports:
      - "8080:8080"
    networks:
      - keycloak_net
    restart: always
    resources:
      limits:
        cpus: "1.0"
        memory: "2G"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/auth/realms/master"]
      interval: 30s
      retries: 3
    user: "1000:1000"

networks:
  keycloak_net:
    driver: bridge
```

### 3. Docker Compose pour Keycloak Node 2

Ce fichier est destiné au serveur 3 pour déployer le deuxième nœud Keycloak.

```yaml
version: '3.8'

services:
  keycloak-node2:
    image: quay.io/keycloak/keycloak:21.0.0
    environment:
      KEYCLOAK_USER: admin
      KEYCLOAK_PASSWORD: admin
      DB_VENDOR: MYSQL
      DB_ADDR: mysql-galera-node1
      DB_PORT: 3306
      DB_DATABASE: keycloak
      DB_USER: keycloak
      DB_PASSWORD: password
    ports:
      - "8081:8080"
    networks:
      - keycloak_net
    restart: always
    resources:
      limits:
        cpus: "1.0"
        memory: "2G"
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/auth/realms/master"]
      interval: 30s
      retries: 3
    user: "1000:1000"

networks:
  keycloak_net:
    driver: bridge
```

### Répartition des Ressources

1. **Serveur 1 :**
   - Déploie les trois nœuds MySQL Galera (en utilisant le fichier Docker Compose pour MySQL Galera Cluster).

2. **Serveur 2 :**
   - Déploie Keycloak Node 1 (en utilisant le fichier Docker Compose pour Keycloak Node 1).

3. **Serveur 3 :**
   - Déploie Keycloak Node 2 (en utilisant le fichier Docker Compose pour Keycloak Node 2).

En utilisant ces fichiers Docker Compose, vous pouvez déployer et gérer chaque composant de votre infrastructure de manière indépendante tout en suivant les meilleures pratiques pour la configuration des conteneurs Docker.


### Intégration avec les Autres Serveurs Docker

Pour gérer vos autres serveurs Docker (où MySQL Galera et Keycloak sont déployés) à partir de Portainer, vous avez deux principales méthodes : ajouter des environnements Docker distants ou configurer des agents Portainer. Voici une explication détaillée pour chaque méthode.

#### 1. Ajouter des Environnements Docker Distants

Cette méthode vous permet d'ajouter des serveurs Docker distants directement à l'interface Portainer, ce qui vous permet de gérer tous vos serveurs à partir d'une seule interface.

**Étapes pour Ajouter des Environnements Docker Distants :**

1. **Accéder à l'Interface Portainer :**
   - Connectez-vous à votre interface Portainer à l'adresse `http://<adresse_ip_serveur>:9000`.

2. **Ajouter un Environnement Docker :**
   - Dans le menu principal, cliquez sur **"Environments"**.
   - Cliquez sur **"Add environment"** pour ajouter un nouveau serveur Docker distant.

3. **Choisir le Type d’Environnement :**
   - Sélectionnez **"Docker"** comme type d'environnement.

4. **Configurer les Informations de Connexion :**
   - **URL de l'API Docker** : Entrez l'URL de l'API Docker de votre serveur distant. L'URL doit inclure le port de l'API Docker, souvent `2375` pour les connexions non sécurisées ou `2376` pour les connexions sécurisées (HTTPS).
     - Exemple : `tcp://<adresse_ip_serveur_distant>:2376`
   - **Jeton d'Authentification (si nécessaire)** : Si l'API Docker est protégée par un jeton ou une clé d'authentification, vous devrez entrer ces informations dans les champs appropriés.
     - Ces informations peuvent inclure un **certificat client**, un **certificat CA**, et une **clé client** si vous utilisez HTTPS.

5. **Configurer les Options de Sécurité :**
   - Si vous utilisez HTTPS, vous devrez ajouter le certificat CA, le certificat client et la clé client pour sécuriser la connexion. Ces options se trouvent généralement sous la section de configuration de sécurité.

6. **Sauvegarder les Paramètres :**
   - Cliquez sur **"Add environment"** pour finaliser l'ajout de votre serveur Docker distant.

7. **Vérifier la Connexion :**
   - Une fois l'environnement ajouté, Portainer tentera de se connecter au serveur Docker distant. Vous verrez les conteneurs et les images disponibles sur le serveur distant dans l'interface de Portainer.

#### 2. Configurer les Agents Portainer

Les agents Portainer permettent de gérer plusieurs environnements Docker à partir d'une seule instance de Portainer en déployant un conteneur agent sur chaque serveur Docker.

**Étapes pour Configurer les Agents Portainer :**

1. **Déployer l'Agent Portainer sur Chaque Serveur Docker :**
   - Connectez-vous à chaque serveur Docker où vous souhaitez déployer l'agent Portainer.
   - Créez un fichier `docker-compose.yml` pour l'agent Portainer :

     ```yaml
     version: '3.8'

     services:
       portainer-agent:
         image: portainer/agent:latest
         container_name: portainer_agent
         volumes:
           - /var/run/docker.sock:/var/run/docker.sock
           - /var/lib/docker/volumes:/var/lib/docker/volumes
         environment:
           - AGENT_CLUSTER_ADDR=tasks.portainer_agent
         restart: always
         networks:
           - portainer_net

     networks:
       portainer_net:
         driver: bridge
     ```

   - Exécutez `docker-compose up -d` dans le répertoire contenant le fichier `docker-compose.yml` pour lancer l'agent Portainer.

2. **Ajouter l'Agent Portainer à Portainer :**
   - Retournez à votre interface Portainer principale.
   - Allez dans **"Environments"** et cliquez sur **"Add environment"**.
   - Sélectionnez **"Agent"** comme type d'environnement.
   - Entrez le nom de l'environnement et l'adresse IP de l'agent Portainer déployé sur chaque serveur Docker.
   - Cliquez sur **"Add environment"** pour connecter l'agent Portainer à votre instance principale de Portainer.

3. **Configurer les Options de Sécurité :**
   - Si nécessaire, configurez les paramètres de sécurité, tels que les jetons d'authentification ou les certificats pour sécuriser la connexion entre Portainer et les agents.

4. **Vérifier la Connexion :**
   - Une fois l'agent ajouté, Portainer affichera les conteneurs et les images de ce serveur dans l'interface principale.

### Résumé

- **Ajouter des Environnements Docker Distants** : Vous permet de gérer des serveurs Docker distants en utilisant l'URL de l'API Docker de chaque serveur. Vous devrez fournir des informations de connexion, et éventuellement des certificats, pour sécuriser l'accès.
- **Configurer les Agents Portainer** : Déployez un conteneur agent sur chaque serveur Docker pour centraliser la gestion via Portainer. Ajoutez chaque agent à Portainer en entrant l'adresse IP de l'agent et configurez les paramètres de sécurité si nécessaire.

En utilisant ces méthodes, vous pouvez gérer et administrer efficacement vos conteneurs Docker déployés sur différents serveurs depuis une interface centralisée avec Portainer.
