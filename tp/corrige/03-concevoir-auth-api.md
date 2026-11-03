# Corrigé — Concevoir l'authentification d'une API de données

> Corrigé de [l'exercice « Concevoir l'authentification d'une API de données »](../03-concevoir-auth-api.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat. Plusieurs schémas se défendent — ce qui est attendu,
> c'est que chaque choix soit **justifié par une contrainte du contexte**.

---

## Partie 1 — Cartographier les acteurs

**Éléments attendus :**

| Consommateur | Propriétaire de la ressource | Client | Serveur d'autorisation | Serveur de ressources | Pour qui parle le jeton |
| --- | --- | --- | --- | --- | --- |
| ERP (Enterprise Resource Planning) du partenaire | l'opérateur logistique (le donneur d'ordre) | l'ERP | le serveur d'autorisation de l'opérateur | l'API de données | le **programme** (l'ERP), pas une personne |
| Tableau de bord web | chaque utilisateur du partenaire | l'application du tableau de bord | le fournisseur d'identité du partenaire | l'API de données | la **personne** derrière la session |
| Pipelines internes | la plateforme data de l'opérateur | chaque pipeline | le serveur d'autorisation interne | l'API de données | le **compte de service** du pipeline |

**Ce qu'il faut retenir :** la question « pour qui parle le jeton ? » décide de
tout le reste. Un jeton qui parle pour un programme ne porte pas de droits
délégués par une personne — et ne doit pas en porter.

**Erreurs fréquentes :** donner au ERP un jeton « au nom du responsable logistique
du partenaire ». Personne n'est derrière la synchronisation nocturne : prétendre
le contraire fabrique une identité qui n'existe pas, impossible à auditer.

---

## Partie 2 — Choisir le mécanisme

**Éléments attendus :**

- **ERP du partenaire — client credentials.** Aucun utilisateur à représenter, un
  seul programme appelant : le flow authorization code n'apporterait rien. Le
  client s'authentifie auprès du serveur d'autorisation avec sa propre preuve et
  reçoit un jeton à périmètre borné.
- **Tableau de bord web — authorization code + OIDC (OpenID Connect).** Les
  utilisateurs ont déjà une identité chez le partenaire : OIDC la réutilise
  (annuaire unique), OAuth2 délègue l'accès à l'API sans jamais partager le mot
  de passe. Le départ d'une personne du partenaire coupe l'accès au même moment.
- **Pipelines internes — compte de service avec credential dédiée.** Pas de
  personne derrière, pas de MFA (authentification multifacteur) possible : chaque
  pipeline reçoit son propre compte, à périmètre minimal, avec une credential
  stockée dans le coffre de secrets et renouvelée automatiquement.

**Sur la cohérence :** les deux premiers passent par le même serveur
d'autorisation — c'est le troisième qui mérite un régime distinct, parce que sa
menace n'est pas l'hameçonnage mais la fuite de credential.

---

## Partie 3 — Définir les jetons

**Éléments attendus** (une réponse équivalente, justifiée, est acceptée) :

| Consommateur | Type de jeton | Périmètres | Durée de vie | Stockage côté client |
| --- | --- | --- | --- | --- |
| ERP du partenaire | jeton d'accès + jeton de rafraîchissement | `lecture:stocks` | jeton d'accès court (minutes), rafraîchissement renouvelé à chaque synchronisation **nocturne** | coffre de secrets de l'ERP, jamais dans le code |
| Tableau de bord web | jeton d'accès OIDC + jeton de rafraîchissement | `lecture:stocks`, `lecture:expeditions` | couvre une journée de travail (**8 heures**) au maximum, puis reconnexion | cookie `HttpOnly` côté serveur, pas de `localStorage` |
| Pipelines internes | credential de compte de service, jeton court obtenu à l'exécution | `lecture:ventes` pour la lecture, `ecriture:agregats` pour le dépôt — **un périmètre par usage** | durée d'une exécution (**toutes les heures**), credential renouvelée par rotation | coffre de secrets, injectée à l'exécution |

**Comment les contraintes bornent les durées :** la promesse de révocation en
**moins de 15 minutes** interdit tout jeton longue durée sans contrôle — un jeton
d'accès qui vit des heures ne peut pas être coupé en minutes sans liste de
révocation. D'où des jetons d'accès courts, un rafraîchissement révocable, et une
liste noire vérifiée pour ce qui reste en circulation.

**Erreurs fréquentes :** une seule clé partagée par tous les pipelines « pour
simplifier » — l'incident devient alors total et l'audit impossible. Ou des
périmètres larges (`admin`, `*`) donnés « pour ne pas être bloqué ».

---

## Partie 4 — Contrôler côté serveur

**Éléments attendus :**

Contrôles appliqués à chaque requête :

1. **Vérifier le jeton** — un JWT (JSON Web Token) : signature, algorithme en
   liste blanche, expiration, émetteur, destinataire (la liste du chapitre
   [JWT et tokens](../04-jwt-et-tokens.md)).
2. **Vérifier le périmètre** — `lecture:stocks` ne débloque pas
   `ecriture:agregats`. C'est de l'autorisation, pas de l'authentification.
3. **Vérifier l'identité appelante** — le compte de service ou la personne
   rattachée au jeton, pour la journalisation.
4. **Limiter les appels** (rate limiting) par identité appelante : un pipeline qui
   se met à multiplier les requêtes est un signal.
5. **Journaliser** chaque appel : identité, périmètre, ressource, horodatage.

Procédure de révocation en **moins de 15 minutes** :

- supprimer ou faire tourner la credential du compte de service concerné (coffre
  de secrets, propagation immédiate à la prochaine exécution) ;
- révoquer le jeton de rafraîchissement du client concerné — les jetons d'accès
  en cours meurent seuls à leur expiration courte ;
- ajouter les jetons encore valides à la liste noire, vérifiée à chaque appel ;
- couper côté API si nécessaire : le périmètre du client est désactivé, tout
  appel est refusé même avec un jeton encore valide.

Signaux d'alerte d'un accès compromis : appel hors périmètre habituel, volume
anormal, exécution hors créneau (l'ERP qui se met à appeler en pleine journée),
tentatives répétées avec un jeton refusé.

**Ce qu'il faut retenir :** la révocation rapide se prépare **avant** l'incident.
Sans jetons courts ni liste noire, la promesse des **moins de 15 minutes** est
tenable uniquement en coupant le client entier — ce qui reste une option
d'urgence, pas une procédure.

---

## Points à retenir

- Le mécanisme suit **pour qui parle le jeton** : personne (OIDC), programme
  (client credentials), pipeline (compte de service).
- Les périmètres bornent le jeton à l'usage réel ; un périmètre large annule le
  bénéfice de la délégation.
- Les durées de vie se déduisent des **contraintes opérationnelles** — ici la
  synchronisation nocturne, la journée de travail et la promesse de révocation.
- La révocation est un dispositif complet (credential, rafraîchissement, liste
  noire, coupure d'urgence), pas un bouton.
