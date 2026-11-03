<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/Logo-diginamic-color-blk.png">
    <img src="assets/Logo-diginamic-color.svg" alt="Diginamic Formation" width="358">
  </picture>
</p>

# Sécurisation des accès par l'authentification

> Cours d'**autoformation** · **RNCP40573** Expert en informatique et systèmes d'information · spécialisation **4B Big Data & IA**
> 1 journée(s) · supports **rédigés par IA**, relus et validés.

---

## Pourquoi ce cours

« Qui êtes-vous ? » est la première question de toute sécurité, et « avez-vous le droit ? » vient juste après. Les confondre est la faille la plus fréquente du web : un client connecté qui ouvre la console d'administration, une commande qui se lit en changeant un numéro dans l'adresse, un mot de passe stocké en clair dans la base.

Ce cours sépare **AuthN** (authentification), **AuthZ** (autorisation) et session, puis met en œuvre les mécanismes qui les font respecter : mots de passe hachés, sessions protégées, jetons signés, délégation d'autorisation, authentification multifacteur. Il suit le fil rouge de **DIGICHEESE** (la fromagerie du fil rouge), dont la boutique en ligne, l'extranet revendeurs et la console d'administration servent de terrain d'observation, du diagnostic des failles à leur correction.

Ces réflexes servent au-delà du web : une **API** (interface de programmation) de données, un pipeline d'automatisation ou un compte de service posent les mêmes questions d'identité et de droits.

## À l'issue de ce cours

Vous serez capable de :

- **Distinguer** authentification (AuthN), autorisation (AuthZ) et session, et repérer leur confusion dans un scénario ou un extrait de code.
- **Identifier** les vecteurs d'identité — ce qu'on sait, ce qu'on a, ce qu'on est — et la fragilité propre à chacun.
- **Mettre en œuvre** un stockage de mots de passe robuste : hachage adapté (bcrypt, argon2), sel, politique fondée sur la longueur.
- **Sécuriser** une session web : drapeaux de cookie (HttpOnly, Secure, SameSite), expiration, invalidation à la déconnexion.
- **Comparer** les mécanismes d'authentification — cookies de session, JWT (JSON Web Token), OAuth 2.0 (délégation d'autorisation), OpenID Connect, clés d'API — et choisir le schéma adapté à un contexte : application web, API, machine à machine, pipeline data.
- **Mettre en œuvre** la **MFA** (authentification multifacteur) et appliquer des règles distinctes aux comptes humains et aux comptes de service.

## Sommaire

1. [Authentifier n'est pas autoriser — AuthN, AuthZ et vecteurs d'identité](01-authentifier-nest-pas-autoriser.md)

---

## Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer le cours.*

- **OWASP (Open Web Application Security Project) — Authentication Cheat Sheet** : les bonnes pratiques de connexion, de mots de passe et de gestion de session. <https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html>
- **NIST (National Institute of Standards and Technology) — SP 800-63B, Digital Identity Guidelines** : la référence pour la politique de mots de passe — longueur, vérification des fuites, pas de rotation forcée. <https://pages.nist.gov/800-63-3/sp800-63b.html>
- **ANSSI (Agence nationale de la sécurité des systèmes d'information) — Guide d'hygiène informatique** : les mesures de gestion des comptes et des accès, pour auditer une application existante. <https://cyber.gouv.fr/publications/guide-dhygiene-informatique>
- **OAuth 2.0 — RFC (Request for Comments) 6749, publiée par l'IETF (Internet Engineering Task Force)** : le texte de référence du protocole de délégation d'autorisation. <https://datatracker.ietf.org/doc/html/rfc6749>
- **France Compétences — fiche RNCP40573 (Répertoire national des certifications professionnelles)** : le référentiel du titre visé par ce cours. <https://www.francecompetences.fr/recherche/rncp/40573/>
