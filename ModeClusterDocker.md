# Déploiement d'un Environnement de Recette Minimaliste pour Keycloak avec MySQL Galera Cluster

## Introduction


Le déploiement d'un environnement de recette pour Keycloak avec un cluster MySQL Galera est une étape cruciale pour garantir que votre solution d'authentification et de gestion des accès est robuste, évolutive, et prête pour une production efficace. Ce type d'environnement de recette permet de tester et valider les configurations avant de passer en production, ce qui est essentiel pour éviter des erreurs coûteuses en déploiement réel.

**Enjeux :**
- **Validation des Configurations :** S'assurer que Keycloak et MySQL Galera fonctionnent correctement ensemble.
- **Scalabilité et Résilience :** Vérifier que l'architecture choisie peut supporter la montée en charge et la tolérance aux pannes.
- **Performance et Sécurité :** Tester la performance du système et sa sécurité avant la mise en production.

**Défis :**
- **Configuration Complexe :** Assurer une configuration correcte des trois nœuds MySQL Galera sur un seul serveur.
- **Gestion des Ressources :** Optimiser les ressources disponibles pour éviter les goulots d'étranglement.
- **Synchronisation des Services :** Garantir que Keycloak communique efficacement avec MySQL Galera et que les nœuds de Keycloak sont bien synchronisés.

**Objectifs :**
- **Déployer un Cluster MySQL Galera Minimaliste :** Installer et configurer un cluster MySQL Galera avec trois nœuds sur un seul serveur.
- **Configurer Deux Nœuds Keycloak :** Déployer deux instances de Keycloak sur des serveurs distincts pour garantir la haute disponibilité.
- **Assurer la Résilience et la Performance :** Tester et valider les performances et la résilience de l'ensemble de l'architecture.

**Risques :**
- **Monopole des Ressources :** Un serveur unique pour MySQL Galera pourrait devenir un goulet d'étranglement si mal configuré.
- **Complexité de Configuration :** La configuration incorrecte de MySQL Galera ou de Keycloak peut entraîner des erreurs difficiles à diagnostiquer.
- **Problèmes de Synchronisation :** Assurer que la synchronisation entre les nœuds MySQL Galera et Keycloak se fait correctement.

## Tableau de Synthèse des Serveurs

| **Serveur**           | **Rôle**                                | **CPU** | **RAM** | **Stockage** | **Réseau** | **Services**           |
|-----------------------|-----------------------------------------|---------|---------|--------------|------------|------------------------|
| **Serveur 1**         | MySQL Galera Cluster (3 nœuds)          | 4 cœurs | 8 Go    | 200 Go SSD    | 1 Gbps     | MySQL Galera (3 nœuds) |
| **Serveur 2**         | Keycloak Node 1                         | 2 cœurs | 4 Go    | 100 Go SSD    | 1 Gbps     | Keycloak               |
| **Serveur 3**         | Keycloak Node 2                         | 2 cœurs | 4 Go    | 100 Go SSD    | 1 Gbps     | Keycloak               |

## Configuration du Cluster MySQL Galera sur Serveur 1

### Exemple de Fichier `docker-compose.yml` pour MySQL Galera Cluster

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

### Exemple de Fichier de Configuration pour chaque nœud (`my-nodeX.cnf`)

Pour `my-node1.cnf` :
```ini
[mysqld]
binlog_format=row
default_storage_engine=innodb
innodb_autoinc_lock_mode=2
wsrep_on=ON
wsrep_cluster_name=galera_cluster
wsrep_cluster_address=gcomm://mysql-galera-node1,mysql-galera-node2,mysql-galera-node3
wsrep_node_address=mysql-gal

## Configuration des Nœuds Keycloak

### Exemple de Fichier `docker-compose.yml` pour Keycloak

#### Serveur 2 : Keycloak Node 1

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
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: "1"
          memory: "2G"
      restart_policy:
        condition: on-failure

networks:
  keycloak_net:
    driver: bridge
```

#### Serveur 3 : Keycloak Node 2

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
    deploy:
      replicas: 1
      resources:
        limits:
          cpus: "1"
          memory: "2G"
      restart_policy:
        condition: on-failure

networks:
  keycloak_net:
    driver: bridge
```

## Top 20 des Bonnes Pratiques pour le Déploiement

1. **Configuration Initiale :** Démarrer le cluster MySQL Galera avec un seul nœud pour initialiser le cluster avant d’ajouter les autres nœuds.
2. **Sécurisation des Connexions :** Utiliser des connexions sécurisées (SSL/TLS) pour les communications entre Keycloak et MySQL.
3. **Sauvegardes Régulières :** Mettre en place une stratégie de sauvegarde régulière pour MySQL et Keycloak.
4. **Surveillance :** Configurer des outils de surveillance pour surveiller les performances et la disponibilité des services.
5. **Limitation des Accès :** Restreindre les accès réseau aux nœuds Keycloak et MySQL pour des adresses IP spécifiques.
6. **Mises à Jour :** Appliquer régulièrement les mises à jour de sécurité et les correctifs pour Keycloak et MySQL.
7. **Tests de Charge :** Effectuer des tests de charge pour évaluer la capacité des nœuds Keycloak à gérer les demandes simultanées.
8. **Optimisation des Ressources :** Ajuster les configurations de mémoire et de CPU en fonction des besoins des applications.
9. **Configuration de Réplication :** S’assurer que la réplication des données entre les nœuds MySQL est correctement configurée.
10. **Plan de Reprise après Sinistre :** Établir un plan de reprise après sinistre pour la récupération des services en cas de défaillance.
11. **Gestion des Logs :** Configurer la gestion des logs pour une analyse facile des erreurs et des événements.
12. **Tests de Tolérance aux Pannes :** Vérifier que l'architecture résiste aux pannes en simulant des défaillances de nœuds.
13. **Isolation des Environnements :** Utiliser des environnements séparés pour les tests de recette et la production.
14. **Documentation :** Maintenir une documentation détaillée des configurations et des procédures de déploiement.
15. **Validation des Performances :** Effectuer des validations de performance pour garantir que l'environnement répond aux attentes.
16. **Automatisation des Déploiements :** Utiliser des outils d'automatisation pour simplifier les déploiements et les mises à jour.
17. **Gestion des Secrets :** Protéger les informations sensibles comme les mots de passe et les clés API.
18. **Plan de Test :** Développer un plan de test complet pour vérifier toutes les fonctionnalités de l'application.
19. **Réglages de Sécurité :** Configurer les paramètres de sécurité pour minimiser les risques d'accès non autorisé.
20. **Revue de Code :** Effectuer des revues de code régulières pour identifier et corriger les vulnérabilités potentielles.

## Conclusion

Le déploiement de cet environnement de recette minimaliste avec MySQL Galera et Keycloak est conçu pour offrir une solution robuste pour tester les configurations et les performances avant la mise en production. En suivant les bonnes pratiques recommandées et en veillant à la gestion des risques, vous pouvez assurer une transition fluide et sécurisée vers un environnement de production.

N'hésitez pas à adapter les configurations et les ressources en fonction des besoins spécifiques de votre organisation pour obtenir les meilleurs résultats possibles.


Pour s'assurer que chaque nœud redémarre automatiquement après un redémarrage du serveur, vous pouvez configurer les services Docker pour qu'ils se redémarrent automatiquement en cas d'échec ou de redémarrage du système. Voici comment procéder pour Keycloak et MySQL Galera sur Docker avec `docker-compose` :

### 1. Configuration de la Politique de Redémarrage dans `docker-compose.yml`

Vous pouvez utiliser les options de redémarrage intégrées de Docker pour garantir que les conteneurs redémarrent automatiquement. Voici comment ajouter ces options dans votre fichier `docker-compose.yml`.

#### Exemple pour MySQL Galera Cluster

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
    restart: always  # Assure le redémarrage automatique du conteneur

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
    restart: always  # Assure le redémarrage automatique du conteneur

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
    restart: always  # Assure le redémarrage automatique du conteneur

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

#### Exemple pour Keycloak

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
    restart: always  # Assure le redémarrage automatique du conteneur

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
    restart: always  # Assure le redémarrage automatique du conteneur

networks:
  keycloak_net:
    driver: bridge
```

### 2. Configuration des Services Système (Optionnelle)

En plus de la configuration de `docker-compose`, vous pouvez également configurer les services système pour qu'ils redémarrent automatiquement au démarrage du serveur. Voici comment faire cela sur un système Linux utilisant `systemd` :

#### Création de Fichiers de Service Systemd pour Docker

1. Créez un fichier de service pour chaque conteneur dans `/etc/systemd/system/` :

   - Pour MySQL Galera :
     ```ini
     [Unit]
     Description=MySQL Galera Cluster Node
     After=network.target

     [Service]
     Restart=always
     ExecStart=/usr/bin/docker-compose -f /path/to/your/docker-compose.yml up
     ExecStop=/usr/bin/docker-compose -f /path/to/your/docker-compose.yml down
     WorkingDirectory=/path/to/your
     User=your-user
     Group=your-group

     [Install]
     WantedBy=multi-user.target
     ```

   - Pour Keycloak :
     ```ini
     [Unit]
     Description=Keycloak Service
     After=network.target

     [Service]
     Restart=always
     ExecStart=/usr/bin/docker-compose -f /path/to/your/docker-compose.yml up
     ExecStop=/usr/bin/docker-compose -f /path/to/your/docker-compose.yml down
     WorkingDirectory=/path/to/your
     User=your-user
     Group=your-group

     [Install]
     WantedBy=multi-user.target
     ```

2. Rechargez les fichiers de configuration `systemd` et activez les services :

   ```sh
   sudo systemctl daemon-reload
   sudo systemctl enable mysql-galera.service
   sudo systemctl enable keycloak.service
   ```

3. Démarrez les services :

   ```sh
   sudo systemctl start mysql-galera.service
   sudo systemctl start keycloak.service
   ```

En suivant ces configurations, chaque nœud de votre environnement de recette sera automatiquement redémarré après un redémarrage du serveur ou en cas d'échec.


Pour assurer un déploiement efficace et sécurisé avec Docker, voici les 10 bonnes pratiques clés à mettre en œuvre dans vos configurations Docker. Ces pratiques aident à garantir la performance, la sécurité, et la résilience de vos conteneurs.

### Top 10 des Bonnes Pratiques pour la Configuration Docker

1. **Utilisation de l'Option `restart` :**
   - **Pratique :** Configurer le mode de redémarrage des conteneurs pour garantir leur disponibilité en cas d'échec ou de redémarrage du système.
   - **Exemple :** `restart: always` dans le fichier `docker-compose.yml`.

2. **Définition des Limites de Ressources :**
   - **Pratique :** Limiter l'utilisation de CPU et de mémoire pour éviter que les conteneurs consomment trop de ressources et impactent les autres services.
   - **Exemple :**
     ```yaml
     resources:
       limits:
         cpus: "1.0"
         memory: "2G"
     ```

3. **Gestion des Secrets et Variables d'Environnement :**
   - **Pratique :** Utiliser des fichiers `.env` ou des secrets Docker pour gérer les informations sensibles de manière sécurisée.
   - **Exemple :** `env_file: .env`

4. **Isolation des Réseaux :**
   - **Pratique :** Créer des réseaux Docker dédiés pour les différents groupes de services afin de minimiser les interférences et améliorer la sécurité.
   - **Exemple :**
     ```yaml
     networks:
       app_net:
         driver: bridge
     ```

5. **Volumes pour la Persistance des Données :**
   - **Pratique :** Utiliser des volumes Docker pour stocker les données persistantes afin qu'elles ne soient pas perdues lors du redémarrage des conteneurs.
   - **Exemple :**
     ```yaml
     volumes:
       data_volume:
     ```

6. **Optimisation des Images Docker :**
   - **Pratique :** Utiliser des images légères et optimiser le Dockerfile pour minimiser la taille des images et accélérer les temps de déploiement.
   - **Exemple :** Utiliser `alpine` comme image de base si possible.

7. **Gestion des Logs :**
   - **Pratique :** Configurer la gestion des logs pour une analyse facile et une surveillance efficace. Vous pouvez rediriger les logs vers des fichiers ou utiliser un système centralisé.
   - **Exemple :**
     ```yaml
     logging:
       driver: "json-file"
       options:
         max-size: "10m"
         max-file: "3"
     ```

8. **Vérification des Santé des Conteneurs :**
   - **Pratique :** Définir des vérifications de santé (`healthcheck`) pour surveiller l'état de vos conteneurs et assurer qu'ils fonctionnent correctement.
   - **Exemple :**
     ```yaml
     healthcheck:
       test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
       interval: 30s
       retries: 3
     ```

9. **Exécution de Conteneurs avec Moins de Privilèges :**
   - **Pratique :** Configurer les conteneurs pour qu'ils s'exécutent avec le moins de privilèges possibles en utilisant des utilisateurs non-root et des options de sécurité.
   - **Exemple :**
     ```yaml
     user: "1000:1000"
     ```

10. **Contrôle de Version et Validation des Images :**
    - **Pratique :** Utiliser des versions spécifiques des images Docker et valider les images avant leur déploiement en production pour éviter les surprises.
    - **Exemple :** Utiliser `image: quay.io/keycloak/keycloak:21.0.0` plutôt que `latest`.

### Exemple Complet d'un Fichier `docker-compose.yml` avec les Bonnes Pratiques

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
    user: "1000:1000"

volumes:
  mysql_node1_data:
    driver: local
    driver_opts:
      type: 'none'
      device: '/path/to/storage/mysql_node1'
      o: 'bind'

networks:
  galera_net:
    driver: bridge
  keycloak_net:
    driver: bridge
```

En appliquant ces bonnes pratiques, vous pouvez améliorer la gestion, la sécurité, et la performance de vos déploiements Docker.
