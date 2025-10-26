# 📝 Laboratoire AWS - site web de traitement d'images

Ce rapport documente les phases de la mise en place du projet du Site Web de Traitement d'Images.

***

## 🛡️ Phase 1 : Fondation du Réseau (VPC)

### Objectif
Mettre en place un réseau VPC hautement segmenté pour isoler le trafic public (serveur Web) du trafic privé (base de données et composants d'application Lambda).

### Composants Réseau

| Composant | Rôle | Contrainte de l'Énoncé | Statut |
| :--- | :--- | :--- | :--- |
| **VPC Principal** | Conteneur du réseau | `10.0.0.0/16` | ✅ Réalisé |
| **Sous-réseau Public** | Pour le serveur web et la NAT-GW | `10.0.1.0/24` | ✅ Réalisé |
| **Sous-réseau Privé** | Pour Lambda et RDS | `10.0.2.0/24` | ✅ Réalisé |

### Routage et Passerelles

| Composant | Emplacement | Détails de la Configuration |
| :--- | :--- | :--- |
| **Passerelle Internet (IGW)** | Attachée au VPC | Route Table Publique configurée : `0.0.0.0/0` ciblant l'IGW. |
| **Passerelle NAT (NAT-GW)** | Sous-réseau Public | Route Table Privée configurée : `0.0.0.0/0` ciblant la NAT-GW (avec EIP allouée). |

### Isolation et Sécurité (NACLs et Security Groups)

* **Sous-réseau Public (`10.0.1.0/24`) :**
    * **Objectif :** Autoriser le trafic web standard et l'administration.
    * **Règles Clés :** NACLs et SGs autorisent HTTP (Port 80) et SSH (Port 22) entrant, et gèrent le trafic sortant, y compris le correctif HTTPS (Port 443) pour les mises à jour.

* **Sous-réseau Privé (`10.0.2.0/24`) :**
    * **Objectif :** Restriction maximale.
    * **Règles Clés :** Tout accès entrant de l'extérieur est bloqué. Les règles sont limitées à l'accès fonctionnel interne (ex: Lambda → RDS).

***

## 🖼️ Phase 2 : Stockage et Livraison de Médias (CDN)

### Objectifs
1.  Stocker les images dans un service objet hautement disponible (S3).
2.  Accélérer la livraison des images à l'échelle mondiale (CloudFront).
3.  Sécuriser le trafic vers le domaine média avec HTTPS (ACM).
4.  Relier l'URL conviviale (`cdn-media.eazimusicschoolandevents.com`) au CDN (Route 53).

### Services AWS Utilisés

| Service | Rôle dans l'Architecture |
| :--- | :--- |
| **S3** (Simple Storage Service) | Stockage principal des images (`public/for images`). C'est l'Origine du contenu. |
| **CloudFront** | Content Delivery Network (CDN) pour la mise en cache mondiale et la livraison rapide. |
| **ACM** (Certificate Manager) | Crée le certificat SSL/TLS (région N. Virginia) pour garantir la connexion HTTPS. |
| **Route 53** | Gère le CNAME final, liant le domaine alternatif au CDN. |

### Configuration Détaillée

1.  **Configuration du Stockage (S3)**
    * **Bucket :** Créé avec la structure de dossier `public/for images`.
    * **Permissions :** L'accès est restreint via **OAC (Origin Access Control)** sur CloudFront (méthode de sécurité préférée).

2.  **Sécurisation HTTPS (ACM)**
    * **Région :** Le certificat SSL/TLS pour le CDN a été créé dans USA Est (Virginie du Nord).
    * **Certificat :** Demande pour `*.eazimusicschoolandevents.com` validée via l'ajout d'un enregistrement CNAME dans Route 53.

3.  **Distribution de Contenu (CloudFront)**
    * **Origine :** La distribution pointe vers le Bucket S3.
    * **CNAME :** `cdn-media.eazimusicschoolandevents.com` est configuré comme nom de domaine alternatif.
    * **Certificat :** Le certificat ACM est sélectionné pour activer HTTPS.

4.  **Configuration DNS Finale (Route 53)**
    * **Lien Final :** Un enregistrement de type **A Alias** a été créé pour lier `cdn-media.eazimusicschoolandevents.com` à la Distribution CloudFront.

### Résultat de l'Étape

Toutes les images sont désormais accessibles de manière sécurisée et rapide via l'URL :

$$\text{https://cdn-media.eazimusicschoolandevents.com/<nom\_image>.jpg}$$
#  Phase 3 : Configuration du Serveur Web EC2

## Objectif

L'objectif de cette phase était de provisionner la couche d'hébergement frontal (frontend) de l'application. Cette couche repose sur une instance EC2 dans le sous-réseau public, configurée avec un serveur Apache (`httpd`), et accessible via le nom de domaine principal.

***

## Actions Effectuées

| # | Instruction | Détails de l'Implémentation | Statut |
| :-: | :--- | :--- | :--- |
| 1 | **Lancement de l'Instance EC2** | Instance `audrey-web-server` (`i-0bd5dc0780ef72d83`) lancée dans le Sous-réseau public. | ✅ |
| 2 | **Installation et Configuration Web** | Le serveur Apache (`httpd`) a été installé, démarré et activé. Le code du frontend (`index.html`) a été déployé dans `/var/www/html/`. | ✅ |
| 3 | **Correction de Connectivité** | Problème de *timeout* (délai d'expiration) lors du `yum update` résolu par l'ajout de la règle sortante **HTTPS (Port 443)** dans la NACL Publique. | ✅ |
| 4 | **Configuration Route 53** | L'Enregistrement A pour le domaine racine (`eazimusicschoolandevents.com`) a été mis à jour pour pointer vers l'IP publique de l'EC2. | ✅ |

***

## Détails Techniques Clés

| Composant | Valeur |
| :--- | :--- |
| **Instance EC2** | `i-0bd5dc0780ef72d83` (`audrey-web-server`) |
| **IP Publique** | `13.60.69.149` |
| **Service Web** | Apache (`httpd`), statut `active (running)` |
| **Nom de Domaine** | `http://eazimusicschoolandevents.com` |

***

## Vérification et Validation

Pour confirmer que la Phase 3 est terminée avec succès :

### 1. Accessibilité du Service Apache :

* **Action :** Exécutez sur l'EC2 : `sudo systemctl status httpd`
* **Résultat attendu :** Statut `Active: active (running)`.

### 2. Accessibilité par Nom de Domaine (Frontend) :

* **Action :** Ouvrez un navigateur et accédez à : `http://eazimusicschoolandevents.com`
* **Résultat attendu :** La page du frontend doit s'afficher. Elle affichera le titre et le message de **"Chargement des métadonnées..."** (l'échec de l'appel à l'API Gateway est attendu à ce stade, car l'Étape 6 n'est pas encore implémentée).


***
## 💾 Phase 4 : Stockage des Métadonnées avec RDS

### Objectif
L'objectif de cette phase était de provisionner une base de données **MySQL gérée (RDS)** dans le **sous-réseau privé** pour stocker les métadonnées des images de manière **sécurisée et scalable**.

Les **identifiants de connexion** devaient être stockés dans **AWS Secrets Manager**.

***

### Actions Effectuées

| # | Instruction | Détails de l'Implémentation | Statut |
| :--- | :--- | :--- | :--- |
| **1** | **Lancement de l'Instance RDS** | Instance `audrey-database-rds` lancée dans le sous-réseau privé. Accessibilité publique désactivée. | ✅ |
| **2** | **Configuration du Groupe de Sécurité** | Groupe de sécurité RDS modifié pour autoriser le trafic entrant sur le port **3306 (MySQL)** uniquement depuis le groupe de sécurité de l'EC2 et les groupes de sécurité des fonctions Lambda. | ✅ |
| **3** | **Connexion du Client EC2 au Serveur RDS** | Connexion depuis l'EC2 vers le point de terminaison RDS établie avec succès en utilisant l'utilisateur **admin**. | ✅ |
| **4** | **Création du Schéma (Base de Données)** | Schéma de base de données d'application **`image_project_db`** créé sur le serveur RDS. | ✅ |
| **5** | **Création de la Table de Métadonnées** | Table **`media_metadata`** créée dans le schéma `image_project_db`. | ✅ |
| **6** | **Stockage des Identifiants DB** | Identifiants de l'utilisateur admin stockés/liés dans **Secrets Manager** pour un accès sécurisé par les services Lambda. | ✅ |

***

### Détails Techniques Clés

| Composant | Valeur |
| :--- | :--- |
| **Instance RDS** | `audrey-database-rds` |
| **Type de Moteur** | MySQL Community (8.0.42) |
| **Base de Données (Schéma)** | `image_project_db` |
| **Identifiant Secret** | `image-project/rds-credentials` |

***


## Vérification et Validation



### 1. Création du Schéma et de la Table :

* **Action :** Exécutez sur l'EC2, puis dans le client MySQL :
  ```sql
  CREATE DATABASE image_project_db; 
  USE image_project_db; 
  CREATE TABLE media_metadata ( 
      id SERIAL PRIMARY KEY, 
      filename VARCHAR(255), 
      size BIGINT, 
      content_type VARCHAR(100), 
      upload_time TIMESTAMP 
  );
* **Résultat attendu :** Les commandes SQL doivent s'exécuter sans erreur.
  
### 2. Accessibilité de la Table :

* **Action :** Ouvrez une connexion MySQL et exécutez la commande :
  ```sql
  SHOW TABLES FROM image_project_db;
* **Résultat attendu :** La table media_metadata doit s'afficher (confirme la persistance des données sur le RDS).
