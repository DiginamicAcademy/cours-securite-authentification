[← Chapitre précédent](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md) · [Sommaire](Readme.md) · [Chapitre suivant →](02-Exemple%20de%20cours%20en%20multi-fichier.md)

# Authentifier n'est pas autoriser — AuthN, AuthZ et vecteurs d'identité

> Un client de **DIGICHEESE** (la fromagerie du fil rouge) se connecte à son compte
> et découvre la liste de toutes les commandes de la boutique. Il était bien authentifié — mais
> avait-il le droit de voir ça ? Deux questions différentes, deux portes
> différentes : les confondre est la faille la plus fréquente du web.

**Durée de lecture :** 45 min · **Prérequis :** aucun

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Distinguer** authentification, autorisation et session.
- **Identifier** les vecteurs d'identité : ce qu'on sait, ce qu'on a, ce qu'on est.
- **Repérer** une confusion entre authentification et autorisation dans un scénario ou un extrait de code.
- **Expliquer** pourquoi « être connecté » ne donne pas automatiquement tous les droits.

---

## I. La différence

Deux questions se posent à chaque accès, toujours dans le même ordre :

1. **AuthN** (authentification) — « qui êtes-vous ? ». On demande une **preuve**
   d'identité : un mot de passe, un code reçu par téléphone, une empreinte. La
   réponse dit *qui* est en face de l'écran.
2. **AuthZ** (autorisation) — « avez-vous le droit ? ». On consulte les **droits**
   de cette identité : ce qu'elle peut lire, modifier, supprimer.

Entre les deux circule la **session** : la preuve de l'authentification, remise à
l'utilisateur après la connexion et présentée à chaque requête. C'est le badge du
musée — il prouve que vous êtes entré par la caisse, pas que vous pouvez ouvrir
les réserves.

### La fromagerie, deux portes

Chez DIGICHEESE, la boutique en ligne et la console d'administration sont deux
mondes différents :

| Question | Client de la boutique | Administrateur de DIGICHEESE |
| --- | --- | --- |
| Qui êtes-vous ? (AuthN) | un compte client, avec son mot de passe | un compte de l'équipe, avec son mot de passe et un code sur son téléphone |
| Avez-vous le droit ? (AuthZ) | consulter **ses** commandes et son profil | tout le catalogue, tous les clients, les stocks |

Un client authentifié qui ouvre la console d'administration n'a pas « trop de
droits » : il n'a **aucun** droit là-dessus. Si la page s'ouvre quand même, ce
n'est pas lui qui a triché — c'est l'application qui a oublié de vérifier.

> 🧪 **Atelier rapide (5 min)**
>
> Voici une route de l'application, écrite pour la console d'administration de
> DIGICHEESE :

```python
@app.route("/admin/clients")
def liste_clients():
    return jsonify(base.tous_les_clients())
```

> Que fait cette route ? Que ne vérifie-t-elle pas ? Notez le contrôle manquant,
> et dites s'il relève de l'authentification ou de l'autorisation.

---

## II. Vecteurs d'identité

Pour répondre à « qui êtes-vous ? », on s'appuie sur trois familles de preuves :

| Famille | Exemples | Fragilité principale |
| --- | --- | --- |
| Ce qu'on **sait** | mot de passe, question secrète | fuite, réutilisation, devinette |
| Ce qu'on **a** | téléphone (code reçu), carte, clé de sécurité | vol, perte, prêt à un tiers |
| Ce qu'on **est** | empreinte digitale, visage | ne se change pas : une donnée biométrique fuitée l'est pour toujours |

Aucune famille n'est parfaite. Un mot de passe se devine, un téléphone se vole,
une empreinte ne se révoque pas. C'est pourquoi on combine les preuves : c'est le
principe de la **MFA** (authentification multifacteur), que nous retrouverons en
fin de journée.

Le réflexe à prendre : devant chaque mécanisme de connexion, demandez **quelle
famille de preuve il utilise**, et ce qui se passe si cette preuve est volée.

---

## III. Les erreurs classiques

La confusion entre authentification et autorisation ne se lit pas toujours aussi
clairement que dans l'extrait de code ci-dessus. Les formes les plus fréquentes :

- **« Connecté = autorisé »** — on vérifie l'identité, puis on ouvre tout. Un
  client DIGICHEESE connecté qui consulte les commandes des autres : la
  vérification d'identité fonctionnait, le contrôle des droits n'existait pas.
- **Le droit vérifié côté interface, pas côté serveur** — le bouton
  « Administration » est masqué pour le client, mais l'**API** (interface de
  programmation) répond quand même à la requête. Ce qui n'est pas affiché n'est
  pas protégé.
- **La donnée confiée à l'identifiant de l'adresse** — `/commandes/42` : si le
  serveur ne vérifie pas que la commande `42` appartient au compte connecté, chaque
  numéro devient une porte d'entrée.
- **La session oubliée** — pas d'expiration, pas d'invalidation à la déconnexion,
  jeton transmis sans chiffrement : la preuve de connexion survit à celui qui l'a
  obtenue.

Ces erreurs ont un point commun : le contrôle qui manque porte sur les **droits**,
pas sur l'identité. La question à se poser, ligne après ligne : *ce que je viens
de vérifier, est-ce « qui c'est » ou « ce qu'il peut faire » ?*

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **Zero Trust** (« ne jamais faire confiance, toujours vérifier ») : appliquée à
  l'identité, cette architecture refuse les blancs-seings. Une connexion validée ce
  matin ne vaut pas autorisation pour la requête de l'après-midi : chaque accès est
  revérifié. La référence est publiée par le **NIST** (National Institute of
  Standards and Technology), sous le numéro SP 800-207.
- **ANSSI** (Agence nationale de la sécurité des systèmes d'information) : ses
  guides d'hygiène informatique détaillent la gestion des comptes et des accès —
  un bon cadre pour auditer une application existante.

---

## Points clés

- **AuthN** (authentification) répond à « qui êtes-vous ? », **AuthZ** (autorisation)
  à « avez-vous le droit ? » : deux contrôles distincts, jamais déductibles l'un de
  l'autre.
- La **session** est la preuve de l'authentification qui circule : elle se protège
  (expiration, invalidation, transport chiffré).
- Les vecteurs d'identité sont de trois familles : ce qu'on **sait**, ce qu'on
  **a**, ce qu'on **est** — chacune se combine, aucune ne suffit.
- « Connecté » ne veut pas dire « autorisé » : confondre les deux est la faille la
  plus fréquente.

---

[← Chapitre précédent](01-Exemple%20de%20cours%20en%20un%20seul%20fichier.md) · [Sommaire](Readme.md) · [Chapitre suivant →](02-Exemple%20de%20cours%20en%20multi-fichier.md)
