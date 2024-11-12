# Comparaison des Stratégies d'Intégration de Keycloak avec une Application Java Spring Boot : HTTP vs HTTPS, avec ou sans Docker

## Introduction

L'intégration d'une application Java Spring Boot avec Keycloak est une solution populaire pour gérer l'authentification et l'autorisation de manière centralisée et sécurisée. Cependant, la configuration de cette intégration dépend fortement du protocole (HTTP ou HTTPS) et du mode de déploiement (avec ou sans Docker). En effet, les configurations HTTPS nécessitent des certificats SSL/TLS pour sécuriser les échanges entre les services, et l'intégration avec Docker ajoute des couches supplémentaires de complexité, notamment dans la gestion des certificats et des réseaux au sein des conteneurs.

Dans ce contexte, il est courant de rencontrer des difficultés lors du passage de HTTP à HTTPS en environnement Docker. Ce document explore les différentes configurations possibles, compare les avantages et les inconvénients de chaque approche et fournit un aperçu des configurations et du code minimal pour intégrer Keycloak avec Spring Boot dans ces différentes situations.

---

## Comparaison des Configurations Keycloak-Spring Boot en HTTP et HTTPS, avec ou sans Docker

Nous allons maintenant examiner les quatre scénarios d’intégration principaux en termes de configuration et de code.

### 1. **Intégration Keycloak avec Spring Boot en HTTP sans Docker**

#### Configuration minimale dans `application.properties`
```properties
keycloak.auth-server-url=http://localhost:8080/auth
keycloak.realm=example-realm
keycloak.resource=example-client
keycloak.public-client=true
keycloak.bearer-only=true
spring.security.oauth2.client.provider.keycloak.issuer-uri=http://localhost:8080/auth/realms/example-realm
```

#### Code Spring Boot minimal
Dans la classe de configuration de sécurité :
```java
import org.keycloak.adapters.springsecurity.KeycloakSecurityConfigurerAdapter;
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;

@EnableWebSecurity
public class SecurityConfig extends KeycloakSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .oauth2ResourceServer()
            .jwt();
    }
    
    @Bean
    public KeycloakConfigResolver KeycloakConfigResolver() {
        return new KeycloakSpringBootConfigResolver();
    }
}
```

### 2. **Intégration Keycloak avec Spring Boot en HTTPS sans Docker**

#### Pré-requis
- **Certificat SSL** : Configurez un certificat SSL/TLS valide pour sécuriser les échanges.
- **Configuration HTTPS** dans `application.properties` :
  ```properties
  server.port=8443
  server.ssl.key-store=classpath:keystore.p12
  server.ssl.key-store-password=password
  server.ssl.key-store-type=PKCS12
  keycloak.auth-server-url=https://localhost:8443/auth
  ```

#### Configuration Keycloak dans Spring Boot
Même code que pour le HTTP, mais le `auth-server-url` doit être en HTTPS.

### 3. **Intégration Keycloak avec Spring Boot en HTTP avec Docker**

#### Dockerfile minimal pour Spring Boot avec Keycloak en HTTP
```dockerfile
FROM openjdk:17-jdk-alpine
COPY target/myapp.jar myapp.jar
ENTRYPOINT ["java", "-jar", "/myapp.jar"]
```

#### Configuration Keycloak en HTTP dans `application.properties`
```properties
keycloak.auth-server-url=http://keycloak:8080/auth
keycloak.realm=example-realm
keycloak.resource=example-client
keycloak.public-client=true
keycloak.bearer-only=true
spring.security.oauth2.client.provider.keycloak.issuer-uri=http://keycloak:8080/auth/realms/example-realm
```

#### Exemple `docker-compose.yml`
```yaml
version: '3'
services:
  myapp:
    build: .
    ports:
      - "8080:8080"
    environment:
      - KEYCLOAK_URL=http://keycloak:8080/auth

  keycloak:
    image: jboss/keycloak
    environment:
      - KEYCLOAK_USER=admin
      - KEYCLOAK_PASSWORD=admin
    ports:
      - "8080:8080"
```

### 4. **Intégration Keycloak avec Spring Boot en HTTPS avec Docker**

#### Pré-requis
- **Certificats SSL dans Docker** : Montez les certificats dans le conteneur.

#### Dockerfile avec Keystore
```dockerfile
FROM openjdk:17-jdk-alpine
COPY target/myapp.jar myapp.jar
COPY keystore.p12 /etc/ssl/certs/keystore.p12
ENTRYPOINT ["java", "-jar", "/myapp.jar"]
```

#### Configuration Keycloak en HTTPS dans `application.properties`
```properties
server.ssl.key-store=/etc/ssl/certs/keystore.p12
server.ssl.key-store-password=password
server.ssl.key-store-type=PKCS12
keycloak.auth-server-url=https://keycloak:8443/auth
```

#### Exemple `docker-compose.yml`
```yaml
version: '3'
services:
  myapp:
    build: .
    ports:
      - "8443:8443"
    environment:
      - KEYCLOAK_URL=https://keycloak:8443/auth
    volumes:
      - ./keystore.p12:/etc/ssl/certs/keystore.p12

  keycloak:
    image: jboss/keycloak
    environment:
      - KEYCLOAK_USER=admin
      - KEYCLOAK_PASSWORD=admin
    ports:
      - "8443:8443"
    volumes:
      - ./keystore.p12:/etc/ssl/certs/keystore.p12
```

---

### Comparaison Récapitulative

| Option                                   | Avantages                                                    | Inconvénients                                                     | Niveau de sécurité |
|------------------------------------------|--------------------------------------------------------------|-------------------------------------------------------------------|---------------------|
| **HTTP sans Docker**                     | Simplicité, facilité de débogage                             | Faible sécurité, non recommandé pour production                   | Faible              |
| **HTTPS sans Docker**                    | Sécurité, support de production                              | Configuration des certificats, renouvellement et gestion          | Élevé               |
| **HTTP avec Docker**                     | Déploiement rapide, idéal pour dev/test                      | Faible sécurité, transition difficile vers HTTPS                  | Faible              |
| **HTTPS avec Docker**                    | Sécurité de production, isolation                            | Configuration complexe des certificats, diagnostic plus difficile | Élevé               |

---

### Conclusion

Pour une utilisation en production, **HTTPS avec Docker** est la configuration la plus sûre, bien que plus complexe. Assurez-vous que les certificats SSL sont correctement configurés et montés dans les conteneurs Docker. Pour des tests rapides, HTTP sans Docker ou en Docker peut suffire, mais pour la production, HTTPS est nécessaire pour protéger les échanges entre les services.

--------------------------------

Pour sécuriser une application Java Spring Boot avec Keycloak en HTTPS, il est nécessaire de configurer un **keystore** qui contient les certificats SSL/TLS. Ce keystore peut être au format `.p12` (PKCS#12), un format couramment utilisé pour stocker les clés et les certificats. Voici les étapes pour générer les différentes configurations de keystore et les intégrer dans une application.

## 1. Création d'un Keystore `.p12` avec `keytool`

Java fournit un outil intégré appelé `keytool` pour gérer les certificats et les keystores. La commande suivante crée un keystore au format `.p12` avec un certificat auto-signé.

### Étape 1 : Générer un Keystore `.p12`

Utilisez la commande suivante pour créer un fichier keystore `.p12` :
```bash
keytool -genkeypair -alias myapp-key -keyalg RSA -keysize 2048 \
  -storetype PKCS12 -keystore keystore.p12 -validity 365 \
  -storepass password
```

- **`-alias`** : le nom de l’alias de la clé (ex. `myapp-key`).
- **`-keyalg`** : l'algorithme de chiffrement de la clé (ex. `RSA`).
- **`-keysize`** : la taille de la clé (ex. `2048` bits).
- **`-storetype`** : le type de keystore (ici `PKCS12`).
- **`-keystore`** : le nom du fichier keystore (ex. `keystore.p12`).
- **`-validity`** : la durée de validité du certificat en jours (ex. `365`).
- **`-storepass`** : le mot de passe pour le keystore (ex. `password`).

Cette commande génère un fichier `keystore.p12` dans le répertoire courant.

### Étape 2 : Ajouter le Keystore dans Spring Boot

Pour que votre application Spring Boot utilise ce keystore en HTTPS, ajoutez les configurations suivantes dans `application.properties` ou `application.yml` :

#### Configuration dans `application.properties`
```properties
server.port=8443
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=password
server.ssl.key-store-type=PKCS12
```

- **`server.port`** : le port HTTPS de l’application (par défaut `8443`).
- **`server.ssl.key-store`** : le chemin vers le keystore.
- **`server.ssl.key-store-password`** : le mot de passe du keystore.
- **`server.ssl.key-store-type`** : le type de keystore (ici `PKCS12`).

### Configuration dans `application.yml`
```yaml
server:
  port: 8443
  ssl:
    key-store: classpath:keystore.p12
    key-store-password: password
    key-store-type: PKCS12
```

### Étape 3 : Vérifier l’accès en HTTPS

Lancez votre application Spring Boot et vérifiez l'accès via HTTPS en utilisant `https://localhost:8443`.

---

## 2. Création d'un Keystore avec un Certificat Signé par une Autorité de Certification (CA)

Pour un environnement de production, un certificat auto-signé n'est généralement pas recommandé. À la place, utilisez un certificat signé par une autorité de certification (CA).

### Étape 1 : Créer une demande de certificat (CSR)

Générez un fichier CSR (Certificate Signing Request) que vous soumettrez à la CA :
```bash
keytool -certreq -alias myapp-key -file myapp.csr -keystore keystore.p12 -storepass password
```

### Étape 2 : Obtenir le certificat signé

Envoyez le fichier `myapp.csr` à une CA (par exemple, Let's Encrypt, DigiCert). La CA vous fournira un certificat signé.

### Étape 3 : Importer le certificat dans le Keystore

Une fois que vous avez reçu le certificat signé, importez-le dans le keystore :
```bash
keytool -importcert -trustcacerts -alias myapp-key -file myapp-cert.pem \
  -keystore keystore.p12 -storepass password
```

Cela ajoute le certificat signé au keystore existant `keystore.p12`.

---

## 3. Utiliser le Keystore dans Docker

Pour intégrer le keystore dans un environnement Docker, suivez ces étapes :

### Étape 1 : Copier le keystore dans le Conteneur

Si vous avez un Dockerfile, copiez le keystore dans le conteneur :
```dockerfile
FROM openjdk:17-jdk-alpine
COPY target/myapp.jar /app/myapp.jar
COPY keystore.p12 /app/keystore.p12
ENTRYPOINT ["java", "-jar", "/app/myapp.jar"]
```

### Étape 2 : Configurer Spring Boot pour Utiliser le Keystore

Dans votre fichier `application.properties`, réglez le chemin pour pointer vers l'emplacement du keystore dans le conteneur :
```properties
server.port=8443
server.ssl.key-store=/app/keystore.p12
server.ssl.key-store-password=password
server.ssl.key-store-type=PKCS12
```

### Étape 3 : Exemple de `docker-compose.yml`

Si vous utilisez Docker Compose, montez le keystore en volume pour faciliter la gestion des certificats :
```yaml
version: '3'
services:
  myapp:
    image: myapp-image
    ports:
      - "8443:8443"
    volumes:
      - ./keystore.p12:/app/keystore.p12
```

Avec cette configuration, Docker montera le fichier `keystore.p12` de votre hôte vers le conteneur, ce qui simplifie la mise à jour ou le renouvellement des certificats.

---

## Conclusion

La configuration du keystore dépend de l'environnement cible (développement ou production) et du mode de déploiement (local ou Docker). Un keystore auto-signé est adapté pour le développement et les tests locaux, tandis qu'un certificat signé par une autorité de certification est nécessaire pour la production. Pour Docker, montez le keystore en volume pour simplifier la gestion des certificats dans le conteneur.

-----------------------------------

Il est possible de simplifier cette configuration en utilisant quelques alternatives qui facilitent le processus de configuration HTTPS pour une application Spring Boot intégrée à Keycloak. Voici quelques solutions plus simples :

### 1. Utiliser le Keystore en Auto-signé pour les Tests Locaux

Pour les environnements de développement et de test, un certificat auto-signé peut suffire sans nécessiter une configuration complexe. De plus, Spring Boot permet de générer le certificat directement dans le conteneur Docker pour éviter de manipuler un fichier keystore.

**Exemple de simplification :**

1. Ajoutez la commande `keytool` directement dans le Dockerfile pour générer un certificat auto-signé à chaque démarrage du conteneur.

   ```dockerfile
   FROM openjdk:17-jdk-alpine
   COPY target/myapp.jar /app/myapp.jar

   RUN keytool -genkeypair -alias myapp-key -keyalg RSA -keysize 2048 \
       -dname "CN=localhost" \
       -keystore /app/keystore.p12 -storepass password -keypass password -validity 365 \
       -storetype PKCS12

   ENTRYPOINT ["java", "-jar", "/app/myapp.jar"]
   ```

2. Ensuite, configurez Spring Boot pour utiliser ce keystore généré :

   ```properties
   server.port=8443
   server.ssl.key-store=/app/keystore.p12
   server.ssl.key-store-password=password
   server.ssl.key-store-type=PKCS12
   ```

### 2. Passer par un Proxy Inverse HTTPS

Une autre solution simple consiste à utiliser un serveur proxy, comme **Nginx** ou **Traefik**, qui gère le HTTPS. Cela simplifie la configuration Spring Boot et évite de manipuler des certificats directement dans l’application.

- **Étapes :**
  1. Configurez l’application Spring Boot pour fonctionner uniquement en HTTP.
  
     ```properties
     server.port=8080
     ```

  2. Utilisez un proxy, comme Nginx, pour servir le contenu de l’application Spring Boot via HTTPS.

     **Exemple de configuration Nginx :**
     ```nginx
     server {
         listen 443 ssl;
         server_name myapp.local;

         ssl_certificate /etc/nginx/ssl/myapp.crt;
         ssl_certificate_key /etc/nginx/ssl/myapp.key;

         location / {
             proxy_pass http://localhost:8080;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
         }
     }
     ```

- **Avantages :** Cette solution délègue la gestion du certificat HTTPS à un proxy, réduisant ainsi la complexité de configuration côté Spring Boot.

### 3. Utiliser Let’s Encrypt pour le Certificat Automatique (Production)

Pour les environnements de production, vous pouvez automatiser l’obtention et le renouvellement des certificats via **Let’s Encrypt** (par exemple, avec Certbot) et les intégrer dans un conteneur Docker.

1. **Installer Certbot** et configurer un certificat SSL valide.

2. Utilisez Nginx (ou un autre proxy) pour servir le contenu de Spring Boot en HTTPS en intégrant automatiquement les certificats Let’s Encrypt.

### 4. Utiliser Spring Boot Devtools (pour un HTTPS simple en Dev)

Si c’est uniquement pour le développement, **Spring Boot Devtools** peut lancer automatiquement votre application avec une configuration HTTPS locale sans keystore explicite. Cependant, cela n’est pas adapté pour les environnements de production.

---

Ces méthodes vous permettent de gérer plus simplement la configuration HTTPS pour Spring Boot sans nécessiter de configurations complexes dans Docker ou de gestion de certificats détaillée dans l’application elle-même.
