# Exercice — Diagnostiquer l'authentification d'une application

> 🟢 **Débutant** · **Durée :** 25 min · **Lien :** chapitre [Authentifier n'est pas autoriser](../01-authentifier-nest-pas-autoriser.md)
> **Objectif :** repérer les failles d'authentification et d'autorisation dans un scénario, et proposer les corrections.

---

## Contexte

**DIGICHEESE** (la fromagerie du fil rouge) ouvre un extranet
à ses revendeurs (épiceries, restaurants) et une console d'administration à son
équipe. L'agence qui a livré
l'application vous fait une démonstration. Voici ce que vous observez :

1. Un revendeur se connecte avec son adresse électronique et son mot de passe. Il
   arrive sur la page de sa dernière commande, `/commandes/42`. En remplaçant `42`
   par `43` dans l'adresse, il affiche la commande d'un autre revendeur.
2. La console d'administration `/admin/clients` liste tous les clients de la
   fromagerie. La page est protégée : il faut être connecté pour l'ouvrir. Un
   compte revendeur, une fois connecté, y accède sans problème.
3. En base de données, les mots de passe des revendeurs sont stockés en clair.
   L'équipe de support les lit pour « dépanner » un client oubliieux.
4. Quand un revendeur clique sur « Se déconnecter », la page d'accueil s'affiche.
   Mais son jeton de session reste valable : rejoué dans le navigateur, il rouvre
   la session.
5. L'application mobile des livreurs de DIGICHEESE envoie l'adresse électronique et
   le mot de passe à chaque requête, sans chiffrement : les identifiants transitent
   en clair sur le réseau.

---

## Consignes

### Partie 1 — Distinguer authentification, autorisation et session

Pour chacun des cinq constats, dites s'il relève de l'**authentification** (prouver
qui on est), de l'**autorisation** (avoir le droit) ou de la **gestion de session**
(la preuve de connexion qui circule). Un constat peut toucher plusieurs catégories :
justifiez votre choix en une phrase.

### Partie 2 — Repérer les failles

Parmi ces constats, lesquels sont des **failles** ? Pour chacune, nommez-la en une
ligne — en désignant **ce qui manque**, pas seulement la conséquence — puis dites ce
qu'un attaquant peut en faire.

### Partie 3 — Classer les vecteurs d'identité

Le scénario fait intervenir plusieurs moyens de prouver son identité. Classez-les
selon les trois familles du chapitre : ce qu'on **sait**, ce qu'on **a**, ce qu'on
**est**. Précisez pour chacun ce qui le rend fragile, et signalez les familles qui
n'interviennent pas.

### Partie 4 — Proposer les corrections

Listez les corrections à apporter, classez-les par urgence (de la plus urgente à la
moins urgente), et expliquez votre classement en deux lignes.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai classé chaque constat dans la bonne catégorie (authentification, autorisation ou session) et **justifié** mon choix.
- [ ] Mes failles nomment **ce qui manque** (le contrôle absent), pas seulement la conséquence.
- [ ] J'ai classé les moyens de preuve dans les **trois familles** de vecteurs du chapitre, et signalé celles qui n'interviennent pas.
- [ ] Mon classement d'urgence affiche un **critère explicite** (données exposées, personnes touchées, facilité d'exploitation…).
- [ ] Mes corrections sont **applicables** (une action concrète), pas des vœux pieux du type « il faudrait sécuriser ».

---

## Pour aller plus loin

*Facultatif.* Le scénario suppose un attaquant revendeur, donc authentifié. Refaites
l'exercice en supposant qu'il ne possède **aucun compte** : quelles failles lui
restent accessibles, et lesquelles disparaissent ?
