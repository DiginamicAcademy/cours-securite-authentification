# Exercice — Proposer une politique MFA accessible et conforme

> 🔴 **Défi** · **Facultatif** · **Durée :** ≈ 30 min · **Lien :** chapitre [MFA et identité data](../06-mfa-et-identite-data.md)
> **Objectif :** proposer une politique d'authentification multifacteur pour une organisation réelle, en tenant compte de l'accessibilité, du RGPD et de la proportionnalité.

---

## Contexte

Un éditeur de logiciels de **200 personnes** veut généraliser la MFA
(authentification multifacteur) à toute son organisation. Le dossier est confié à
l'équipe sécurité — la vôtre. Trois populations coexistent :

1. **Les salariés** : développement, comptabilité, support, direction. La
   comptabilité manipule des données bancaires ; le support accède aux comptes
   clients.
2. **Les prestataires externes** : consultants en mission de quelques semaines,
   souvent sur leur propre matériel.
3. **Les comptes de service** : les pipelines d'automatisation de la plateforme
   data (traitements planifiés, sans personne derrière).

La direction a posé ses contraintes :

- la politique doit être **accessible** : parmi les salariés, une personne
  malvoyante et une personne sans téléphone personnel doivent pouvoir se
  connecter au quotidien, sans dépendre d'un collègue ;
- elle doit être **conforme au RGPD** (Règlement général sur la protection des
  données) : toute donnée biométrique est une donnée sensible, à justifier et à
  minimiser ;
- elle doit être **proportionnée** : pas de facteur imposé là où le risque ne le
  justifie pas — le forum interne n'est pas la comptabilité ;
- un incident courant — téléphone perdu — doit être traité en **moins de 4
  heures**, sans laisser le compte ouvert pendant la procédure.

---

## Consignes

Vous rédigez la proposition qui sera présentée à la direction. Aucun gabarit
imposé : c'est à vous de choisir la forme qui porte le mieux votre raisonnement.

### Partie 1 — La politique

Proposez un tableau de politique : pour chaque population (et, si vous le jugez
utile, par niveau de sensibilité à l'intérieur d'une population), indiquez le
facteur imposé, le facteur de repli, et le cas d'exception. Chaque ligne est
accompagnée d'une justification d'une phrase.

### Partie 2 — L'accessibilité

Décrivez le parcours de connexion des deux salariés cités dans le contexte, du
quotidien jusqu'à la perte d'un facteur. Expliquez pourquoi votre solution évite
de les rendre dépendants d'une autre personne — c'est un critère de dignité, pas
un détail de confort.

### Partie 3 — Le RGPD

Identifiez les données à caractère personnel que votre politique collecte ou
utilise, et dites pour chacune : ce qui la justifie, combien de temps elle est
conservée, et ce qui la rend minimale. Argumentez le cas de la biométrie : où
l'acceptez-vous, où la refusez-vous, et pourquoi.

### Partie 4 — L'incident

Rédigez la procédure « téléphone perdu » : qui fait quoi, dans quel ordre, et
comment la promesse des **moins de 4 heures** est tenue. Précisez ce qui arrive
aux sessions déjà ouvertes et aux comptes de service concernés.

---

## ✅ Auto-évaluation

Avant de confronter votre proposition à la grille de vérification, vérifiez que :

- [ ] Chaque ligne de ma politique nomme un **facteur imposé**, un **facteur de repli** et un **cas d'exception** — pas seulement « MFA obligatoire ».
- [ ] Mes deux salariés se connectent **eux-mêmes**, au quotidien, sans intermédiaire humain.
- [ ] Chaque donnée personnelle citée est **justifiée, datée de conservation et minimisée**.
- [ ] Ma position sur la biométrie est **argumentée** (où, pourquoi, et quelle alternative).
- [ ] Ma procédure d'incident tient les **moins de 4 heures** en nommant qui agit, et coupe les sessions ouvertes.

---

## Grille de vérification

Ce défi n'a pas de correction pas-à-pas : confrontez votre travail à la
[grille de vérification](corrige/04-defi-mfa.md), qui liste les critères
attendus et les écueils fréquents.
