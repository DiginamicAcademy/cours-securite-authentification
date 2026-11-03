# Exercice — Concevoir l'authentification d'une API de données

> 🔵 **Intégration** · **Durée :** 55 min · **Lien :** chapitre [OAuth2 et OpenID Connect](../05-oauth2-oidc.md)
> **Objectif :** concevoir le schéma d'authentification d'une API de données servie à des humains et à des machines, côté client comme côté serveur.

---

## Contexte

Un opérateur logistique met en place une **API de données** qui expose ses
indicateurs : `/stocks`, `/ventes`, `/expeditions`. Trois consommateurs se
présentent :

1. **L'ERP (Enterprise Resource Planning) d'un partenaire** — un logiciel de gestion qui synchronise ses propres
   stocks avec ceux de l'opérateur. Personne ne l'utilise directement : il tourne
   tout seul.
2. **Un tableau de bord web** utilisé par les équipes du partenaire pour suivre
   les expéditions en temps réel. Les utilisateurs ont déjà une identité dans
   l'entreprise du partenaire (annuaire unique).
3. **Les pipelines internes de la plateforme data** de l'opérateur — des
   traitements d'automatisation qui lisent l'API pour alimenter l'entrepôt de
   données, et y déposent des agrégats.

Contraintes validées avec la direction :

- l'ERP synchronise ses données **une fois par nuit** ;
- le tableau de bord est utilisé **8 heures par jour** par les équipes ;
- les pipelines internes s'exécutent **toutes les heures** ;
- un accès compromis doit pouvoir être révoqué en **moins de 15 minutes** ;
- aucune clé en clair dans le code : les secrets sont stockés dans un coffre,
  injectés à l'exécution ;
- tout passe par HTTPS (HyperText Transfer Protocol Secure) et chaque appel est
  journalisé avec l'identité appelante.

---

## Consignes

### Partie 1 — Cartographier les acteurs

Pour chacun des trois consommateurs, identifiez les quatre acteurs du chapitre :
propriétaire de la ressource, client, serveur d'autorisation, serveur de
ressources. Précisez pour qui parle le jeton — une personne, un programme, ou les
deux.

### Partie 2 — Choisir le mécanisme

Proposez, pour chaque consommateur, le mécanisme d'authentification retenu
(authorization code, OpenID Connect, client credentials, compte de service avec
clé, etc.) et justifiez votre choix en deux ou trois lignes. Un même mécanisme
peut servir à plusieurs consommateurs — dites pourquoi.

### Partie 3 — Définir les jetons

Remplissez un tableau à cinq colonnes : consommateur, type de jeton, périmètres
(scopes), durée de vie, stockage côté client. Les durées de vie doivent respecter
les contraintes chiffrées du contexte — montrez comment.

### Partie 4 — Contrôler côté serveur

Listez les contrôles que l'API applique à chaque requête, et décrivez la
procédure de révocation qui tient la promesse des **moins de 15 minutes**. Ajoutez
les signaux d'alerte qui trahissent un accès compromis.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Chaque consommateur a un mécanisme **choisi et justifié** — pas une solution unique plaquée sur les trois.
- [ ] Mes périmètres sont **aussi étroits que l'usage** : lire les stocks n'est pas écrire les expéditions.
- [ ] Mes durées de vie respectent les **contraintes chiffrées** du contexte, et je sais dire laquelle borne quoi.
- [ ] Aucun secret n'apparaît dans mon schéma côté client : je nomme **où** il vit et **comment** il arrive au programme.
- [ ] Ma révocation est **explicable en une procédure**, pas en un vœu (« il faudrait pouvoir couper »).

---

## Pour aller plus loin

*Facultatif.* Le partenaire souhaite une version mobile du tableau de bord, sans
secret client embarqué. Quel mécanisme ajoutez-vous, et quelle menace spécifique
couvre-t-il ? Reliez votre réponse à la section « Aller plus loin » du chapitre.
