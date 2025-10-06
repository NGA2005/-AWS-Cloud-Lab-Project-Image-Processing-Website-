Laboratoire AWS - Sécurisation de l'Architecture VPC (Phase 1/8)
Projet : Site Web de Traitement d'Images
Ce rapport concerne la première phase critique de notre projet : la mise en place de l'infrastructure réseau de base (VPC) sur AWS. Cette base assure l'isolation et la sécurité de toutes nos futures ressources (Serveur Web, Lambda, RDS) dès le démarrage du projet.

- Objectifs de la Phase 1 : Fondation du Réseau
L'objectif de cette phase était de mettre en place un réseau VPC hautement segmenté afin de séparer le trafic public (accès Web) du trafic privé (composants d'application et base de données).

Composant		              Rôle			                               Contraintes de l'Énoncé	     Statut
VPC Principal	         	Conteneur du réseau			                	10.0.0.0/16                    ✅ Réalisé
                                        
Sous-réseau Public	    Pour le serveur web et la NAT-GW			     10.0.1.0/24                  ✅ Réalisé
			
Sous-réseau Privé		    Pour Lambda et RDS			                   10.0.2.0/24 	                 ✅ Réalisé		

Routage		              Contrôle du flux de trafic		             Public → IGW, Privé → NAT-GW 	✅ Configuré
		

- Configuration du Réseau

La configuration des composants réseau et des règles de routage a été effectuée via la console AWS.

Routage et Passerelles
Composant		                 Emplacement		          Détails de la Configuration

Passerelle Internet (IGW)		Attachée au VPC 			    Route Table Publique configurée : 0.0.0.0/0 ciblant l'IGW.

Passerelle NAT (NAT-GW)		Sous-réseau Public 			    Route Table Privée configurée : 0.0.0.0/0 ciblant la NAT-GW. (Requiert une EIP allouée).


- Isolation et Sécurité (NACLs et Security Groups)

Une double couche de sécurité a été appliquée (NACL pour le sous-réseau et SG pour la ressource) pour répondre aux exigences de la phase 6.

Sous-réseau Public (10.0.1.0/24)

  Objectif : Autoriser le trafic web standard et l'administration.

  Règles Clés : Les NACLs et SGs sont configurés pour autoriser le trafic HTTP (Port 80) et SSH (Port 22) entrant.  

Sous-réseau Privé (10.0.2.0/24)

   Objectif : Restriction maximale, avec une exception fonctionnelle.

   Règles Clés :

Restriction : Tout accès entrant de l'extérieur est bloqué (y compris depuis le sous-réseau public, grâce aux SGs).

ÉTAPE 2 : Stockage et Livraison de Médias (CDN)

L'objectif principal de cette étape est de mettre en place une architecture de distribution de contenu. Cela implique le stockage sécurisé des images et leur diffusion à l'échelle mondiale via un réseau de diffusion de contenu rapide (CDN) sécurisé par HTTPS.

 Objectifs de l'Étape

1. Stocker les images dans un service objet hautement disponible.
2. Accélérer la livraison des images à l'échelle mondiale.
3. Sécuriser tout le trafic vers le domaine média avec un certificat SSL/TLS (HTTPS).
4. Relier l'URL conviviale (`cdn-media.aurorasocialcloud.org`) au CDN.

 Services AWS Utilisés

 Service                                            Rôle dans l'Architecture 

S3 (Simple Storage Service)     Stockage principal et fiable des images (`public/for images`). C'est l'Origine du contenu. 
CloudFront                       Content Delivery Network (CDN). Il met en cache les images sur des points de présence mondiaux pour une livraison rapide. 
ACM (Certificate Manager)     Crée le certificat SSL/TLS pour garantir la connexion HTTPS. 
Route 53                        Gère le domaine (`aurorasocialcloud.org`) et crée le lien final entre le CNAME et le CDN.


 Instructions Détaillées (Rappel)

 1. Configuration du Stockage (S3)

 Instruction                 Détails 

Création du Bucket        Bucket S3 créé (Ex : `mon-projet-media-audrey`). |
Dossier                  Structure de dossier : `public/for images`. |
Permissions                Les permissions de lecture publique sont activées, ou l'accès est restreint via **OAC (Origin Access Control)** sur CloudFront (méthode de sécurité préférée).

2. Sécurisation HTTPS (ACM)

 Instruction  Détails 
 
|Région   Le certificat SSL/TLS pour le CDN doit être créé dans  USA Est (Virginie du Nord). 
Certificat  Demande de certificat  pouR`*.aurorasocialcloud.org` (recommandé pour la flexibilité). 
Validation  Validation du certificat par l'ajout de l'enregistrement **CNAME** fourni par ACM dans la zone hébergée Route 53.

3. Distribution de Contenu (CloudFront)

 Instruction    Détails 

 Origine        Pointage de la distribution vers le Bucket S3. 
  CNAME         `cdn-media.aurorasocialcloud.org` est configuré comme nom de domaine alternatif. 
Certificat   Le certificat ACM (pour `*.aurorasocialcloud.org`) est sélectionné pour activer HTTPS.

 4. Configuration DNS Finale (Route 53)

 Instruction      Détails 

Zone Hébergée      La zone    `aurorasocialcloud.org`   est créée et gérée dans Route 53. 
Enregistrement A   Un enregistrement de type  A Alias est créé pour lier le CNAME au CDN. 
Lien Final         `cdn-media.aurorasocialcloud.org` → Cible :  Distribution CloudFront .


- Résultat de l'Étape

À la fin de cette étape, toutes les images stockées dans S3 seront accessibles de manière rapide et sécurisée via l'URL suivante :

`https://cdn-media.aurorasocialcloud.org/<nom_image>.jpg`





Prochaine Étape (Phase 2)
Maintenant que le réseau est stable et sécurisé, la prochaine étape consistera à configurer le stockage des médias et la diffusion de contenu en déployant le bucket S3 et la distribution CloudFront.
