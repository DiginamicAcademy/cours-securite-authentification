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

Ce cours suit le fil rouge de **DIGICHEESE** (la fromagerie du fil rouge) : sa boutique en ligne, son extranet revendeurs et sa console d'administration servent de terrain d'observation. On y sépare **AuthN** (authentification), **AuthZ** (autorisation) et session, on classe les preuves d'identité, puis on diagnostique les failles d'une application réelle.

Ces réflexes servent au-delà du web : une **API** (interface de programmation) de données, un pipeline d'automatisation ou un compte de service posent les mêmes questions d'identité et de droits.

## À l'issue de ce cours

Vous serez capable de :

- **Distinguer** authentification, autorisation et session, et nommer le contrôle en jeu dans un cas concret.
- **Identifier** les vecteurs d'identité — ce qu'on sait, ce qu'on a, ce qu'on est — et leur fragilité.
- **Repérer** une confusion entre authentification et autorisation dans un scénario ou un extrait de code.
- **Expliquer** pourquoi « être connecté » ne donne pas automatiquement tous les droits.
- **Proposer** les corrections d'un contrôle d'accès défaillant, classées par urgence selon un critère explicite.

## Sommaire

1. [Authentifier n'est pas autoriser — AuthN, AuthZ et vecteurs d'identité](01-authentifier-nest-pas-autoriser.md) — chapitre, 45 min
2. [Exercice — Diagnostiquer l'authentification d'une application](tp/01-diagnostiquer-auth.md) — 25 min, avec son [corrigé](tp/corrige/01-diagnostiquer-auth.md)

---

## Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer le cours.*

- **Zero Trust** (« ne jamais faire confiance, toujours vérifier ») — appliquée à l'identité, cette architecture refuse les blancs-seings : chaque accès est revérifié.
- **NIST** (National Institute of Standards and Technology), SP 800-207 — la référence publiée de cette architecture.
- **ANSSI** (Agence nationale de la sécurité des systèmes d'information) — ses guides d'hygiène informatique détaillent la gestion des comptes et des accès : un bon cadre pour auditer une application existante.
