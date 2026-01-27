# SAE 34 – Projet Infrastructure

SAE34 – Infrastructure Réseau avec Docker

Ce projet met en place une infrastructure réseau complète conteneurisée avec Docker Compose, comprenant les services suivants :

DNS : BIND9

NTP : Chrony

RADIUS : FreeRADIUS (+ PostgreSQL)

VPN : OpenVPN

Tous les services communiquent via un réseau Docker dédié (lab_net).

1️/Démarrage de l’infrastructure

Se placer à la racine du projet :

docker compose down
docker compose up -d --build
docker ps

Explication

docker compose up -d --build démarre tous les services.

docker ps permet de vérifier que tous les conteneurs sont en cours d’exécution (Up).

2/ Service DNS – BIND9
Rôle du DNS

Le DNS permet la résolution de noms de domaine (ex : google.fr → adresse IP).
Le serveur DNS utilisé est BIND9, qui écoute sur le port 53 à l’intérieur du conteneur.

Vérification que BIND écoute sur le port 53
docker exec dns_server ss -lunpt | findstr ":53"


 Cela montre que le service DNS écoute bien sur le port standard.

Test de résolution DNS

 Sous PowerShell, le caractère @ doit être protégé par des guillemets.

docker exec dns_server dig "@127.0.0.1" -p 53 google.fr

Résultat attendu

Le serveur DNS répond à la requête

Si le statut est :

NOERROR → résolution réussie

SERVFAIL → le serveur fonctionne mais la récursion DNS (Internet) n’est pas configurée

Explication orale possible

« Le serveur DNS répond bien aux requêtes.
Le code SERVFAIL indique que la récursion n’est pas configurée, mais cela prouve que le service BIND fonctionne et traite les requêtes DNS. »

3️/ Service NTP – Chrony
Rôle du NTP

Le service NTP permet la synchronisation de l’heure, essentielle pour :

les logs

la sécurité

l’authentification réseau

Test du service NTP
docker exec ntp_server chronyc tracking

Informations importantes affichées

Stratum : niveau de la source de temps

Reference ID : serveur NTP utilisé

Last offset / System time : écart de synchronisation

Explication orale possible

« Le serveur NTP est synchronisé avec une source de temps externe. Les informations affichées montrent que la synchronisation fonctionne correctement. »

4️/ Service RADIUS – FreeRADIUS
Rôle de RADIUS

RADIUS permet l’authentification centralisée des utilisateurs (ex : Wi-Fi, VPN, accès réseau).

Test d’authentification RADIUS
docker exec radius_server radtest steve testing localhost 0 testing123

Résultat attendu
Received Access-Accept

Explication orale possible

« Le serveur RADIUS valide l’utilisateur steve avec son mot de passe.
La réponse Access-Accept confirme que l’authentification fonctionne. »

5️/ Service VPN – OpenVPN
Rôle du VPN

Le VPN permet de créer un tunnel réseau sécurisé pour accéder au réseau interne via une interface virtuelle (tun0).

Vérification du démarrage du VPN
docker logs vpn_server

Éléments à repérer dans les logs

TUN/TAP device tun0 opened

UDP link local ... :1194

absence d’erreurs critiques

Vérification de l’interface VPN
docker exec vpn_server ip a


La présence de l’interface tun0 confirme que le tunnel VPN est actif.

6️/ Vérification du réseau Docker (bonus)
Rôle

Tous les services sont connectés au même réseau Docker avec des adresses IP fixes.

docker network inspect sae34-groupe-5_lab_net


Cela permet de vérifier :

le subnet (ex : 172.28.0.0/24)

les IP attribuées à chaque service

7️/ Arrêt de l’infrastructure
docker compose down


Arrête proprement tous les services.

Conclusion

Cette infrastructure démontre :

une orchestration multi-services avec Docker Compose

un serveur DNS fonctionnel

un serveur NTP synchronisé

un service RADIUS validant l’authentification

un serveur VPN opérationnel

Chaque service est validé par des commandes de test concrètes, garantissant le bon fonctionnement de l’ensemble.


Problèmes

Secrets présents en clair dans :

docker-compose.yml (POSTGRES_PASSWORD, etc.)

configs RADIUS (secret = testing123)

utilisateurs RADIUS (Cleartext-Password)

Risque : fuite GitHub → compromission immédiate.

Améliorations


Éviter les mots de passe en clair : préférer des hash adaptés 
