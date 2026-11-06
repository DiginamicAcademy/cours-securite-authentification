[← Chapitre précédent](06-mfa-et-identite-data.md) · [Sommaire](Readme.md)

# Glossaire et ressources

> **Nature :** annexe de référence · **Usage :** tout au cours

Cette page rassemble les acronymes et les termes techniques du cours, avec leurs
définitions, et les ressources de référence — à garder sous la main pendant la
journée.

---

## Acronymes

| Sigle | Signification | En clair |
| --- | --- | --- |
| **ANSSI** | Agence nationale de la sécurité des systèmes d'information | l'autorité française de cybersécurité : ses guides détaillent la gestion des comptes et des accès |
| **API** | Application Programming Interface | l'interface de programmation d'une application, appelée à distance — ici, une API de données |
| **AuthN** | authentication (authentification) | « qui êtes-vous ? » : la vérification de l'identité |
| **AuthZ** | authorization (autorisation) | « avez-vous le droit ? » : la vérification des droits de cette identité |
| **CSRF** | Cross-Site Request Forgery | la requête forgée depuis un autre site : le navigateur envoie votre cookie à votre insu |
| **DAG** | Directed Acyclic Graph | le graphe sans cycle d'un pipeline d'automatisation, par exemple un DAG Airflow |
| **FIDO2** | Fast Identity Online 2 | le standard derrière les passkeys et les clés de sécurité matérielles |
| **GPU** | Graphics Processing Unit | le processeur graphique : ce qui rend le craquage des hachages rapides bon marché |
| **HMAC** | Hash-based Message Authentication Code | la signature à clé partagée d'un message — ici, celle d'un JWT |
| **HTTPS** | HyperText Transfer Protocol Secure | le transport chiffré du web : sans lui, un jeton se capte sur le réseau |
| **JSON** | JavaScript Object Notation | le format de données lisible dans lequel est écrit un JWT |
| **JWT** | JSON Web Token | la « lettre signée » : lisible par tous, vérifiable par qui détient la clé |
| **kid** | key id (identifiant de clé) | le repère de clé publié dans l'en-tête d'un JWT, qui permet de la changer sans casser les jetons en circulation |
| **MFA** | Multi-Factor Authentication | l'authentification multifacteur : des preuves de familles différentes, pas deux mots de passe |
| **MD5** | Message Digest 5 | un algorithme de hachage trop rapide et cassé : proscrit pour les mots de passe |
| **NIST** | National Institute of Standards and Technology | l'organisme américain de référence, dont deux documents sont cités dans le cours |
| **OAuth2** | Open Authorization 2 | le protocole qui accorde un accès limité et révocable sans partager son mot de passe |
| **OIDC** | OpenID Connect | la couche identité posée au-dessus d'OAuth2 : le mécanisme des boutons « se connecter avec… » |
| **PKCE** | Proof Key for Code Exchange | la preuve calculée à chaque connexion, qui neutralise l'interception du code d'autorisation |
| **PME** | petite et moyenne entreprise | |
| **PSH** | Personnes en Situation de Handicap | l'accessibilité est une contrainte de conception : une politique MFA sans facteur de repli exclut |
| **RAM** | Random Access Memory (mémoire vive) | ce que consomme argon2 : la mémoire coûte plus cher que le temps de calcul |
| **RSA** | Rivest, Shamir et Adleman | un algorithme de signature à paire de clés, alternative à HMAC pour les JWT |
| **SHA** | Secure Hash Algorithm | la famille de hachage dont SHA-1 est issue : trop rapide pour un mot de passe |
| **SIM** | Subscriber Identity Module | la carte d'abonné téléphonique : échangeable, donc facteur fragile |
| **SMS** | Short Message Service | le texto : facteur « ce qu'on a » universel, que le NIST considère comme restreint |
| **SP** | Special Publication (publication du NIST) | leurs documents de référence : SP 800-63B pour les mots de passe, SP 800-207 pour le Zero Trust |
| **SQL** | Structured Query Language | le langage des bases de données : une injection SQL lit les secrets qu'elles contiennent |
| **TOTP** | Time-based One-Time Password | le code à usage unique généré par une application : court, et hors réseau |
| **URL** | Uniform Resource Locator | l'adresse d'une ressource : jamais de jeton dedans, elle atterrit dans les journaux |
| **WebAuthn** | Web Authentication | le dialogue du navigateur avec l'appareil d'authentification, pièce du standard FIDO2 |
| **XSS** | Cross-Site Scripting | le script injecté dans une page : il lit cookies et localStorage |

---

## Termes techniques clés

- **`alg: none`** — l'en-tête qui annonce « pas de signature » : un serveur qui se fie à l'en-tête du jeton accepte alors n'importe quel contenu fabriqué à la main.
- **argon2** — fonction de hachage de mots de passe (2015) qui ajoute la mémoire au calcul : le choix recommandé pour un nouveau projet.
- **base64url** — l'encodage des deux premières parties d'un JWT : lisible par tous, ce n'est pas un chiffrement.
- **bcrypt** — fonction de hachage de mots de passe (1999) qui embarque le sel et le coût dans l'empreinte, reconnaissable à son préfixe `$2b$`.
- **Biométrie** — la preuve « ce qu'on est » (empreinte, visage) : elle ne se prête pas, et une empreinte fuitée l'est pour toujours.
- **Client (OAuth2)** — l'application qui demande l'accès, ici le tableau de bord — et non la personne qui l'utilise.
- **Clé d'API (API key)** — le secret qui identifie un programme appelant avec des droits fixes : elle n'authentifie jamais une personne.
- **Clé publique / clé privée** — la paire sur laquelle repose une passkey : la privée ne quitte jamais l'appareil, le service ne reçoit que la publique.
- **Coût (work factor)** — le réglage de lenteur de bcrypt : à chaque cran, le temps de calcul double.
- **Compte de service** — l'identité d'un pipeline, sans personne derrière : moindre privilège, secrets hors du code, rotation, journalisation.
- **Consentement** — l'écran qui liste ce que l'application obtient : pour être éclairé, il doit afficher les périmètres.
- **Cookie de session** — le petit enregistrement qui porte l'identifiant de session et repart automatiquement à chaque requête.
- **Credential** — la preuve d'identité au sens large ; pour un compte de service, elle est l'identité elle-même.
- **Déni de service** — le blocage d'un service légitime : une limitation de tentatives ne doit pas se retourner contre la victime.
- **Détournement de session** — le vol du jeton d'une session ouverte : `HttpOnly`, `Secure` et HTTPS sont les défenses.
- **Drapeaux du cookie (flags)** — `HttpOnly` contre le vol par script, `Secure` contre l'envoi en clair, `SameSite` contre les requêtes forgées : aucun ne remplace le chiffrement du transport.
- **Facteur de repli** — la solution prévue pour ceux qui n'ont pas le facteur principal : clé matérielle, code imprimé à usage unique, assistance vérifiée.
- **Fatigue de validation** — approuver les notifications sans les lire : le point faible du facteur push.
- **Fournisseur d'identité** — le service qui atteste de qui vous êtes auprès de l'application, laquelle ne voit jamais votre mot de passe.
- **Fixation de session** — l'attaquant impose son identifiant de session à la victime : on s'en protège en régénérant l'identifiant à la connexion.
- **Flow authorization code** — le parcours OAuth2 en sept étapes : le code transite par le navigateur, le jeton s'échange en direct avec le serveur d'autorisation.
- **Gestionnaire de mots de passe** — l'outil qui génère, stocke chiffré et remplit un secret unique par service : d'où une politique qui accepte le copier-coller.
- **Hachage** — la transformation à sens unique en une empreinte de longueur fixe : un mot de passe se vérifie, il ne se relit jamais.
- **Hameçonnage (phishing)** — le faux site ou le faux message qui soutire le secret ; les relais en temps réel captent le code à l'instant où on le tape.
- **Jeton (token)** — la preuve présentée à chaque requête : elle est porteuse, donc volable — un mot de passe volé se change, une session volée est déjà ouverte.
- **Jeton d'accès (access token)** — celui qui débloque l'API : durée courte, périmètre borné.
- **Jeton d'identité (ID token)** — le JWT livré par OIDC, qui dit qui est l'utilisateur : fait pour être lu par le client, jamais présenté à une API.
- **Jeton opaque (jeton de référence)** — un identifiant aléatoire sans contenu, dont les données vivent côté serveur : révocation immédiate, mais un appel à chaque vérification.
- **Jeton de rafraîchissement (refresh token)** — celui qui permet d'obtenir de nouveaux jetons d'accès : plus long, révocable, et à faire tourner à chaque usage.
- **Journalisation** — la trace de chaque appel : un compte de service qui se met à lister des tables inconnues est un signal, pas un détail.
- **localStorage** — le stockage local du navigateur, lisible par n'importe quel script injecté : mauvais endroit pour un jeton.
- **Moindre privilège (least privilege)** — le strict nécessaire et rien de plus : écrire dans une table, pas administrer la base.
- **Passkey** — la paire de clés qui remplace le couple identifiant / mot de passe, liée au site : résistante à l'hameçonnage, sans secret partagé.
- **Périmètre (scope)** — le droit nommé porté par le jeton, affiché au consentement et vérifié à chaque appel : `lecture:ventes` ne débloque jamais `ecriture:expeditions`.
- **Phrase de passe (passphrase)** — trois mots ordinaires alignés : plus mémorisable et plus robuste qu'un mot court « complexe ».
- **Politique de mots de passe** — ce que l'application accepte ou refuse : la longueur plutôt que la complexité, pas de rotation forcée, secrets déjà fuités refusés.
- **Préfixe `__Host-`** — un nom de cookie qu'un sous-domaine ne peut pas poser : une protection contre la fixation par sous-domaine.
- **Propriétaire de la ressource** — celui qui accorde l'accès, ici la responsable des ventes.
- **Proportionnalité** — le niveau de sécurité se décide selon les données exposées et l'impact métier : chaque contrainte ajoutée est un motif d'abandon ou de contournement.
- **Révocation** — rendre une preuve caduque avant son échéance : facile côté serveur, difficile pour un JWT signé sans état, d'où les durées courtes.
- **Rotation** — le changement périodique d'une clé ou d'un jeton, et immédiat en cas de suspicion.
- **Sans état (stateless)** — le jeton porte lui-même ses informations : aucune base à interroger, mais la révocation demande alors un montage supplémentaire.
- **Sel (salt)** — la donnée aléatoire unique ajoutée avant le hachage : deux comptes qui choisissent le même mot de passe n'ont plus la même empreinte.
- **Serveur d'autorisation** — celui qui décide qui a droit à quoi et distribue les jetons.
- **Serveur de ressources** — l'API protégée : elle ne vérifie ni mot de passe ni identité, seulement le jeton et ses périmètres.
- **Session** — la preuve de l'authentification, remise après la connexion et présentée à chaque requête : le badge du musée, pas la clé des réserves.
- **Signature (et chiffrement)** — la signature prouve l'origine et l'intégrité, elle ne cache rien ; seul le chiffrement protège la confidentialité.
- **Tables arc-en-ciel (rainbow tables)** — les milliards d'empreintes déjà calculées pour les mots de passe courants : le sel les rend inutilisables.
- **Vecteurs d'identité** — les trois familles de preuves : ce qu'on sait, ce qu'on a, ce qu'on est ; aucune ne suffit, on les combine.
- **Zero Trust** — « ne jamais faire confiance, toujours vérifier » : une connexion validée ce matin ne vaut pas autorisation pour la requête de l'après-midi.

---

## Ressources utiles

### Mots de passe et leur stockage

- **NIST SP 800-63B** (Digital Identity Guidelines) — <https://pages.nist.gov/800-63-3/> — la politique de mots de passe telle que le cours la présente : longueur, pas de rotation forcée, secrets fuités refusés.
- **Password Storage Cheat Sheet** (OWASP) — <https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html> — bcrypt, argon2, sel et coût : ce que retient un développeur au moment d'écrire le code.
- **Password Hashing Competition / argon2** — <https://www.password-hashing.net/> — le site de référence d'argon2, lauréat de la compétition.

### Sessions, jetons et API

- **Authentication Cheat Sheet** (OWASP) — <https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html> — la liste des contrôles à appliquer à une connexion, de la limitation des tentatives à la déconnexion.
- **RFC 7519** (JSON Web Token) — <https://datatracker.ietf.org/doc/html/rfc7519> — la spécification du JWT : ses parties, ses déclarations, l'identifiant de clé `kid`.
- **jwt.io** — <https://jwt.io/> — un décodeur de JWT, pour un jeton fabriqué en atelier : un jeton réel ne se colle jamais dans un site tiers.

### OAuth2 et OpenID Connect

- **OpenID Connect** — <https://openid.net/connect/> — la couche identité posée sur OAuth2, et ce qu'elle ajoute au jeton d'accès.
- **RFC 6749** (OAuth 2.0) — <https://datatracker.ietf.org/doc/html/rfc6749> — le flow authorization code, les quatre acteurs et les périmètres, décrits dans le détail.
- **RFC 7636** (PKCE) — <https://datatracker.ietf.org/doc/html/rfc7636> — la preuve calculée à chaque connexion, recommandée aujourd'hui pour tous les clients.

### MFA, passkeys et identités de service

- **Multifactor Authentication Cheat Sheet** (OWASP) — <https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html> — le choix des facteurs, leurs faiblesses et les procédures de repli.
- **FIDO Alliance** — <https://fidoalliance.org/> — le standard derrière les passkeys et les clés de sécurité.
- **WebAuthn, niveau 2** (W3C) — <https://www.w3.org/TR/webauthn-2/> — le dialogue du navigateur avec l'appareil, et pourquoi l'origine est vérifiée à chaque tentative.

### Autorités et cadres de référence

- **ANSSI** — <https://cyber.gouv.fr/> — les recommandations de sécurité en français, dont la gestion des comptes et des accès.
- **NIST SP 800-207** (Zero Trust Architecture) — <https://csrc.nist.gov/pubs/sp/800/207/final> — l'architecture qui refuse les blancs-seings, appliquée aux humains comme aux pipelines.

---

## Dans ce cours

- Les quatre exercices de la journée : le dossier `tp/`, avec leurs corrigés dans `tp/corrige/`.
- Le lab des ateliers des chapitres 02 à 04 : `cd lab && docker compose up --build -d`, comme indiqué dans les chapitres — le service `db` pour les stockages de mots de passe, le service `secure` pour le jeton et la session.

---

[← Chapitre précédent](06-mfa-et-identite-data.md) · [Sommaire](Readme.md)
