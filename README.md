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

Exception : Les Security Groups sont paramétrés pour n'autoriser que la connexion de Lambda vers RDS. Cela permet à notre future fonction Lambda de communiquer avec la base de données sans l'exposer.

Prochaine Étape (Phase 2)
Maintenant que le réseau est stable et sécurisé, la prochaine étape consistera à configurer le stockage des médias et la diffusion de contenu en déployant le bucket S3 et la distribution CloudFront.
