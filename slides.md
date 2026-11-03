---
marp: true
theme: default
RNCPBloc: [RNCP40573BC05]
style: |
  section {background-color: #121114}
  h1,h2,h3 {color: #8393f0}
  p,ul,li,td,th {color: #ccc}
  section table {font-size: 0.85em}
  section table td {background-color: #121114}
  section table th {background-color: #272133; font-weight: bolder}
  pre {background-color: #17151a; color: #ccc}
  blockquote {border: 2px green solid; border-left-width: 15px; padding: 0.5em 15px;font-style: italic }
paginate: true
header: "Sécurisation des accès par l'authentification"
footer: "![height:20px](https://raw.githubusercontent.com/DiginamicInternal/PublicAssets/refs/heads/main/Logo-diginamic-color-blk.png)"
---

# Sécurisation des accès par l'authentification

## Qui êtes-vous ? Avez-vous le droit ?

<!-- Diapositive d'ouverture. La journée pose la première question de toute
     sécurité — l'identité — puis la seconde : les droits. Fil rouge DIGICHEESE,
     la fromagerie du fil rouge. -->

---

## 🎯 Objectifs

À l'issue de ce cours, vous saurez :

1. Distinguer authentification, autorisation et session
2. Identifier les vecteurs d'identité et leur fragilité
3. Repérer une confusion entre authentification et autorisation
4. Expliquer pourquoi « être connecté » ne donne pas tous les droits
5. Proposer les corrections d'un contrôle d'accès défaillant

---

## Le parcours

| # | Étape | Durée |
| --- | --- | --- |
| 01 | Authentifier n'est pas autoriser | 45 min |
| TP | Diagnostiquer l'authentification d'une application | 25 min |

Le chapitre se lit, l'exercice se fait : chaque notion est pratiquée tout de suite,
puis confrontée au corrigé.

---

## 1. Authentifier n'est pas autoriser

---

## Deux questions, deux portes

Deux questions se posent à chaque accès, toujours dans le même ordre :

1. **AuthN** (authentification) — « qui êtes-vous ? » → une **preuve** d'identité
2. **AuthZ** (autorisation) — « avez-vous le droit ? » → les **droits** de cette identité

Entre les deux circule la **session** : la preuve de l'authentification, remise
après la connexion et présentée à chaque requête.

> Le badge du musée prouve que vous êtes entré par la caisse, pas que vous pouvez ouvrir les réserves.

---

## La fromagerie, deux portes

Chez **DIGICHEESE** (la fromagerie du fil rouge), la boutique en ligne et la
console d'administration sont deux mondes différents :

| Question | Client de la boutique | Administrateur |
| --- | --- | --- |
| Qui êtes-vous ? (AuthN) | un compte client, avec son mot de passe | un compte de l'équipe, mot de passe et code sur téléphone |
| Avez-vous le droit ? (AuthZ) | consulter **ses** commandes et son profil | tout le catalogue, tous les clients, les stocks |

Un client authentifié qui ouvre la console d'administration n'a pas « trop de
droits » : il n'a **aucun** droit là-dessus.

---

## 🧪 Atelier rapide (5 min)

Voici une route de la console d'administration de DIGICHEESE :

```python
@app.route("/admin/clients")
def liste_clients():
    return jsonify(base.tous_les_clients())
```

Que fait cette route ? Que ne vérifie-t-elle pas ?

> Notez le contrôle manquant, et dites s'il relève de l'authentification
> ou de l'autorisation.

<!-- Attendu : la route renvoie tous les clients sans vérifier l'identité ni le
     rôle. Le contrôle manquant est un contrôle de droits. -->

---

## 2. Vecteurs d'identité

---

## Trois familles de preuves

Pour répondre à « qui êtes-vous ? », on s'appuie sur trois familles de preuves :

| Famille | Exemples | Fragilité principale |
| --- | --- | --- |
| Ce qu'on **sait** | mot de passe, question secrète | fuite, réutilisation, devinette |
| Ce qu'on **a** | téléphone (code reçu), carte, clé de sécurité | vol, perte, prêt à un tiers |
| Ce qu'on **est** | empreinte digitale, visage | ne se change pas : une donnée biométrique fuitée l'est pour toujours |

Aucune famille n'est parfaite : un mot de passe se devine, un téléphone se vole,
une empreinte ne se révoque pas.

---

## Combiner les preuves

Le réflexe à prendre : devant chaque mécanisme de connexion, demandez **quelle
famille de preuve il utilise**, et ce qui se passe si cette preuve est volée.

- On combine les familles : c'est le principe de la **MFA**
  (authentification multifacteur)
- Une preuve volée n'est pas une fatalité si une autre lui résiste
- Choisir une preuve, c'est choisir ce qui se passe en cas de vol

---

## 3. Les erreurs classiques

---

## « Connecté = autorisé »

Les formes les plus fréquentes de la confusion :

- **« Connecté = autorisé »** — on vérifie l'identité, puis on ouvre tout
- **Le droit vérifié côté interface, pas côté serveur** — le bouton est masqué,
  mais l'**API** (interface de programmation) répond quand même
- **La donnée confiée à l'identifiant de l'adresse** — `/commandes/42` : si le
  serveur ne vérifie pas le propriétaire, chaque numéro est une porte d'entrée
- **La session oubliée** — pas d'expiration, pas d'invalidation à la déconnexion

---

## Le contrôle qui manque

Ces erreurs ont un point commun : le contrôle absent porte sur les **droits**,
pas sur l'identité.

> Ce que je viens de vérifier, est-ce « qui c'est » ou « ce qu'il peut faire » ?

La question se pose ligne après ligne — c'est elle qui sépare une application
ouverte d'une application protégée.

---

## 4. Exercice — Diagnostiquer l'authentification

---

## Le scénario

**DIGICHEESE** ouvre un extranet à ses revendeurs et une console d'administration
à son équipe. L'agence qui a livré l'application vous fait une démonstration :

1. `/commandes/42` → en remplaçant `42` par `43`, la commande d'un autre revendeur
2. `/admin/clients` ouvert à tout compte connecté
3. mots de passe stockés en clair, lus par le support
4. déconnexion… mais le jeton de session reste valable
5. l'application mobile envoie identifiants et mot de passe sans chiffrement

---

## Les quatre parties

| Partie | Ce que vous faites |
| --- | --- |
| 1 | Distinguer authentification, autorisation et session — et justifier |
| 2 | Repérer les failles — nommer **ce qui manque**, pas la conséquence |
| 3 | Classer les vecteurs d'identité dans les trois familles |
| 4 | Proposer les corrections, classées par urgence |

> 🟢 Débutant · 25 min · le corrigé est fourni, à comparer **après** avoir travaillé.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- chaque constat est classé dans la bonne catégorie, avec justification
- les failles nomment **ce qui manque**, pas seulement la conséquence
- les moyens de preuve sont classés dans les trois familles
- le classement d'urgence affiche un **critère explicite**
- les corrections sont **applicables**, pas des vœux pieux

---

## Points clés

- **AuthN** répond à « qui êtes-vous ? », **AuthZ** à « avez-vous le droit ? » :
  deux contrôles distincts, jamais déductibles l'un de l'autre
- La **session** est la preuve qui circule : expiration, invalidation,
  transport chiffré
- Trois familles de preuves : ce qu'on **sait**, ce qu'on **a**, ce qu'on
  **est** — chacune se combine, aucune ne suffit
- « Connecté » ne veut pas dire « autorisé » : confondre les deux est la faille
  la plus fréquente

---

## Aller plus loin

*Ces approfondissements sont facultatifs.*

- **Zero Trust** (« ne jamais faire confiance, toujours vérifier ») — chaque
  accès est revérifié, une connexion validée ce matin ne vaut pas autorisation
- **NIST** (National Institute of Standards and Technology), SP 800-207 — la
  référence publiée de cette architecture
- **ANSSI** (Agence nationale de la sécurité des systèmes d'information) —
  guides d'hygiène informatique : gestion des comptes et des accès

---

## Des questions ?

### Le dépôt reste la référence

Pour toute question [adressez-vous au service pédagogie](mailto:pedagogie@diginamic.fr)
