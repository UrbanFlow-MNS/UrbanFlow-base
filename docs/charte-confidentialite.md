# Charte de confidentialité — UrbanFlow

**Dernière mise à jour : 16 septembre 2026**

UrbanFlow est une plateforme de gestion de la mobilité urbaine (suivi des trajets, gestion du réseau de transport, signalement d'incidents) développée dans le cadre d'un projet étudiant. Cette charte explique quelles données nous traitons, pourquoi, et comment elles sont protégées. Elle couvre l'ensemble des services de la plateforme : passerelle API, authentification, gestion des comptes, trajets, incidents et notifications.

Nous avons voulu que ce document reflète ce que le système fait réellement, pas une liste générique de bonnes intentions. Si une pratique décrite ici change, la date en haut sera mise à jour.

## 1. Qui traite vos données

UrbanFlow est opéré par l'équipe projet UrbanFlow composée de Théo Sementa, Mathias Dumas, Ethan Collignon et Logan Pallara. Pour toute question relative à vos données, vous pouvez nous écrire à **urban.flow.moselle@gmail.com**.

## 2. Données que nous collectons

### Compte utilisateur
Lors de la création d'un compte (y compris les comptes gérés par un administrateur pour le personnel d'agence), nous enregistrons : nom, prénom, adresse email, mot de passe et rôle applicatif. Le mot de passe n'est jamais stocké en clair, il est haché avant d'être écrit en base et n'est jamais renvoyé dans les réponses de l'API. Pour les comptes rattachés à une agence, l'identifiant de l'agence et l'identifiant de la personne ayant créé le compte sont également conservés, à des fins de traçabilité administrative.

### Connexion et session
L'authentification repose sur des jetons JWT : un jeton d'accès de courte durée (1 heure) et un jeton de renouvellement plus long (30 jours), ce dernier étant lui aussi haché avant stockage. Une réinitialisation de mot de passe génère un lien signé, envoyé par email, qui contient temporairement votre adresse email dans le jeton. Ce lien expire rapidement et ne peut pas être réutilisé une fois le mot de passe changé.

### Signalements d'incidents
Si vous signalez un incident sur le réseau, nous conservons le titre, la description, le statut, la priorité, l'identifiant du site concerné (avec ses coordonnées géographiques et un contact de site, qui n'est pas nécessairement vous) et votre identifiant en tant qu'auteur du signalement. Les photos ou pièces jointes associées à un incident sont stockées avec l'identifiant de la personne qui les a déposées.

### Notifications
Les notifications que vous recevez (email ou autre canal) sont associées à votre identifiant utilisateur, ainsi qu'au contenu et au statut d'envoi du message. Les emails transactionnels (réinitialisation de mot de passe, notifications) transitent par un serveur SMTP Gmail (voir la section « Sous-traitants » ci-dessous).

### Journalisation technique
Chaque connexion réussie génère un événement de log contenant votre adresse email, à des fins de suivi opérationnel et de diagnostic. Ces journaux sont internes, ne sont pas exposés publiquement et n'incluent ni adresse IP ni identifiant d'appareil.

### Données de trajet et de véhicule
Les données de trajets, horaires et positions de véhicules (au format proche GTFS) ne sont pas rattachées à un compte utilisateur individuel : ce sont des données opérationnelles sur le réseau de transport lui-même, pas sur les usagers.

## 3. Pourquoi nous traitons ces données

- **Gestion de compte et authentification** : vous identifier, sécuriser l'accès à la plateforme, permettre la réinitialisation de mot de passe.
- **Fonctionnement du service** : afficher les trajets et le réseau, permettre le signalement et le suivi des incidents, vous notifier des événements pertinents.
- **Administration** : permettre aux gestionnaires d'agence de créer et suivre les comptes de leur personnel.
- **Sécurité et diagnostic** : détecter les anomalies de connexion et diagnostiquer les incidents techniques via les journaux applicatifs.

Nous ne traitons pas vos données à des fins publicitaires et nous ne les vendons pas.

## 4. Base légale

Le traitement de vos données repose sur l'exécution du service que vous nous avez demandé (création et gestion de votre compte, traitement de vos signalements) et, pour la journalisation technique, sur notre intérêt légitime à assurer la sécurité et la fiabilité de la plateforme.

## 5. Qui reçoit vos données

### En interne
Les différents services d'UrbanFlow (authentification, utilisateurs, incidents, notifications, journalisation, supervision) communiquent entre eux via une file de messages interne (RabbitMQ) et des bases PostgreSQL dédiées, hébergées dans notre propre infrastructure. Aucune donnée ne transite par un tiers pour ces échanges internes.

### Sous-traitants
- **Google (Gmail SMTP)** : l'envoi des emails transactionnels (réinitialisation de mot de passe, notifications par email) passe par un compte Gmail dédié au projet. L'adresse email du destinataire et le contenu du message transitent donc par les serveurs de Google.
- **Projet OSRM (routing public)** : pour le calcul d'itinéraires piétons, le service d'aide à la planification interroge l'API publique `router.project-osrm.org` avec des coordonnées géographiques de départ et d'arrivée. Ces coordonnées ne sont pas accompagnées d'un identifiant utilisateur ou de compte dans cette requête.

Nous ne partageons vos données avec aucun autre tiers, et en particulier pas à des fins commerciales.

## 6. Durée de conservation

Vos données de compte sont conservées tant que votre compte est actif. Les signalements d'incidents et leurs pièces jointes sont conservés le temps nécessaire au suivi opérationnel du réseau. Les journaux techniques ont vocation à être conservés sur une durée limitée, suffisante pour le diagnostic, avant purge. Si vous demandez la suppression de votre compte, nous supprimons ou anonymisons les données qui ne sont pas nécessaires à une obligation légale ou à un intérêt légitime documenté (par exemple, un signalement d'incident déjà traité peut être conservé sans être nominativement rattaché à vous).

## 7. Sécurité

- Les mots de passe et les jetons de renouvellement sont hachés avant stockage, jamais conservés en clair.
- Les jetons d'accès ont une durée de vie courte (1 heure) pour limiter l'impact d'une fuite.
- L'accès aux fonctionnalités d'administration est restreint par rôle.
- La passerelle API est le seul point d'entrée HTTP exposé publiquement. Les services internes ne sont pas accessibles directement depuis l'extérieur.

Aucun système n'est infaillible : si nous identifions un incident de sécurité affectant vos données, nous vous en informerons conformément à nos obligations légales.

## 8. Vos droits

Conformément au RGPD, vous disposez d'un droit d'accès, de rectification, d'effacement et de limitation du traitement de vos données, ainsi que d'un droit d'opposition et de portabilité lorsque ces droits s'appliquent. Pour les exercer, écrivez-nous à **urban.flow.moselle@gmail.com** en précisant votre demande. Nous vous répondrons dans les meilleurs délais.

Vous avez également le droit d'introduire une réclamation auprès de la CNIL (www.cnil.fr) si vous estimez que vos droits ne sont pas respectés.

## 9. Cookies et traceurs

Ce document couvre le fonctionnement des services backend d'UrbanFlow. Les applications frontend (web, mobile) qui consomment cette API peuvent avoir leur propre politique de gestion des cookies et du stockage local. Reportez-vous à la charte spécifique de l'application que vous utilisez le cas échéant.

## 10. Évolution de cette charte

Cette charte peut évoluer avec le projet, notamment si de nouvelles fonctionnalités impliquant des données personnelles sont ajoutées (par exemple, la géolocalisation des utilisateurs ou les notifications push). Toute modification substantielle sera reflétée par une mise à jour de la date en haut de ce document.
