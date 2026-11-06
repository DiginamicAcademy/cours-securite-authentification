[Sommaire](Readme.md) · [Chapitre suivant →](02-mots-de-passe.md)

# Authentifier n'est pas autoriser — AuthN, AuthZ et vecteurs d'identité

> Un client de **DIGICHEESE** (la fromagerie du fil rouge) se connecte à son compte
> et découvre la liste de toutes les commandes de la boutique. Il était bien authentifié — mais
> avait-il le droit de voir ça ? Deux questions différentes, deux portes
> différentes : les confondre est la faille la plus fréquente du web.

**Durée :** ≈ 15 min — lecture ≈ 10 min + atelier 5 min · **Prérequis :** aucun

> 📖 Vocabulaire et acronymes : [glossaire du cours](A-Glossaire-et-ressources.md).

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

### Exercice 1 — Diagnostiquer l'authentification d'une application (25 min)

#### Contexte

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

#### Consignes

##### Partie 1 — Distinguer authentification, autorisation et session

Pour chacun des cinq constats, dites s'il relève de l'**authentification** (prouver
qui on est), de l'**autorisation** (avoir le droit) ou de la **gestion de session**
(la preuve de connexion qui circule). Un constat peut toucher plusieurs catégories :
justifiez votre choix en une phrase.

##### Partie 2 — Repérer les failles

Parmi ces constats, lesquels sont des **failles** ? Pour chacune, nommez-la en une
ligne — en désignant **ce qui manque**, pas seulement la conséquence — puis dites ce
qu'un attaquant peut en faire.

##### Partie 3 — Classer les vecteurs d'identité

Le scénario fait intervenir plusieurs moyens de prouver son identité. Classez-les
selon les trois familles du chapitre : ce qu'on **sait**, ce qu'on **a**, ce qu'on
**est**. Précisez pour chacun ce qui le rend fragile, et signalez les familles qui
n'interviennent pas.

##### Partie 4 — Proposer les corrections

Listez les corrections à apporter, classez-les par urgence (de la plus urgente à la
moins urgente), et expliquez votre classement en deux lignes.

---

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai classé chaque constat dans la bonne catégorie (authentification, autorisation ou session) et **justifié** mon choix.
- [ ] Mes failles nomment **ce qui manque** (le contrôle absent), pas seulement la conséquence.
- [ ] J'ai classé les moyens de preuve dans les **trois familles** de vecteurs du chapitre, et signalé celles qui n'interviennent pas.
- [ ] Mon classement d'urgence affiche un **critère explicite** (données exposées, personnes touchées, facilité d'exploitation…).
- [ ] Mes corrections sont **applicables** (une action concrète), pas des vœux pieux du type « il faudrait sécuriser ».

---

#### Prolongement de l'exercice

*Facultatif.* Le scénario suppose un attaquant revendeur, donc authentifié. Refaites
l'exercice en supposant qu'il ne possède **aucun compte** : quelles failles lui
restent accessibles, et lesquelles disparaissent ?

<details>

<summary>Voir la correction</summary>

> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

#### Partie 1 — Distinguer authentification, autorisation et session

**Éléments attendus :**

Rappel : **AuthN** (authentification) pose la question « qui êtes-vous ? »,
**AuthZ** (autorisation) pose la question « avez-vous le droit ? », et la
**session** est la preuve de connexion qui circule entre les deux.

| Constat | Catégorie | Justification |
| --- | --- | --- |
| 1. `42` remplacé par `43` dans l'adresse | **Autorisation** | Le revendeur est bien authentifié : c'est le contrôle de propriété de la donnée qui manque. |
| 2. Console d'administration ouverte à tout compte connecté | **Autorisation** | L'authentification fonctionne (il faut être connecté), le contrôle de rôle est absent. |
| 3. Mots de passe lus en clair par le support | **Authentification** | Le secret qui sert à prouver l'identité est lui-même exposé. |
| 4. Jeton de session valable après déconnexion | **Session** | La preuve de connexion n'est pas invalidée quand la session se termine. |
| 5. Identifiants envoyés sans chiffrement | **Authentification** (et transport) | Le secret transite en clair : quiconque capte le réseau l'obtient. |

**Ce qu'il faut retenir :** un même constat peut toucher plusieurs catégories —
ici le constat 5 mêle le secret d'authentification et son transport. Ce qui compte,
c'est de nommer **le contrôle en jeu**, pas la page où le problème se voit.

**Erreurs fréquentes :** classer le constat 2 en « authentification » parce qu'il y a
un écran de connexion. La question à se poser : *le contrôle qui manque porte sur
l'identité, ou sur les droits ?*

---

#### Partie 2 — Repérer les failles

**Éléments attendus :** les cinq constats sont des failles, mais pas du même type.

- **Accès aux données d'autrui** (constat 1) — le serveur ne vérifie pas que la
  commande demandée appartient au compte connecté : un revendeur lit les commandes
  des autres, leurs adresses et leur historique d'achats.
- **Contrôle de rôle absent** (constat 2) — n'importe quel compte connecté
  administre la fromagerie : clients, commandes, peut-être suppression.
- **Secrets stockés en clair** (constat 3) — une fuite de la base expose tous les
  mots de passe, et la réutilisation des mots de passe fait le reste ailleurs.
- **Session non invalidée** (constat 4) — un jeton volé, oublié sur un poste
  partagé ou conservé dans un navigateur reste exploitable indéfiniment.
- **Identifiants en clair sur le réseau** (constat 5) — le vol de mot de passe
  devient une capture réseau, sans la moindre faille applicative.

**Ce qu'il faut retenir :** la faille la plus spectaculaire n'est pas forcément la
plus grave. Ici, la lecture des commandes des autres (constat 1) se fait sans outil
et sans faille technique — juste en changeant un numéro dans l'adresse.

---

#### Partie 3 — Classer les vecteurs d'identité

**Éléments attendus :**

| Moyen de preuve | Famille | Fragilité principale |
| --- | --- | --- |
| Mot de passe du revendeur | Ce qu'on **sait** | Fuite, réutilisation, devinette — et il transite en clair dans le scénario. |
| Jeton de session | Ce qu'on **a** | Volable (navigateur, poste partagé, réseau) et non invalidé à la déconnexion. |

La famille « ce qu'on **est** » n'intervient pas dans ce scénario : aucune donnée
biométrique n'y est utilisée. C'est un constat, pas un oubli — et ce n'est pas un
problème en soi : la biométrie n'est pas obligatoire, elle apporte surtout une
preuve difficile à prêter.

**Erreurs fréquentes :** classer le jeton de session en « ce qu'on sait » parce
qu'il s'écrit dans le navigateur. Ce qu'on détient (un jeton, une carte, un
téléphone) n'est pas ce qu'on connaît.

---

#### Partie 4 — Proposer les corrections

**Éléments attendus** (classement indicatif — l'important est le critère explicite) :

1. **Vérifier la propriété de la donnée** (constat 1) — chaque commande demandée est
   rattachée au compte qui la demande, côté serveur.
2. **Ajouter un contrôle de rôle** sur la console d'administration (constat 2) — le
   droit est vérifié à chaque requête, pas seulement dans l'interface.
3. **Chiffrer le transport** (constat 5) — les identifiants ne transitent plus en
   clair sur le réseau.
4. **Hacher les mots de passe** côté serveur (constat 3) — le support ne lit plus
   jamais un mot de passe, et une fuite de base ne livre pas les secrets.
5. **Invalider la session à la déconnexion** (constat 4) — et lui donner une
   expiration, pour qu'un jeton oublié ne vive pas indéfiniment.

**Sur le classement :** deux critères se défendent — l'**exposition des données**
(ici les commandes des revendeurs et la console d'administration, d'où les contrôles
d'accès en tête) ou la **facilité d'exploitation** (le changement de numéro dans
l'adresse est immédiat, la capture réseau demande une position sur le réseau). Ce
qui est attendu, c'est que le critère soit **dit** : un classement sans critère
affiché ne se discute pas.

---

#### Points à retenir

- Authentification et autorisation sont deux contrôles distincts : le second ne se
  déduit jamais du premier.
- Une donnée protégée par la connexion ne l'est pas pour autant : vérifiez **à qui
  elle appartient**.
- Les secrets (mots de passe, jetons) se protègent **au repos** (hachage), **en
  transit** (chiffrement) et **dans le temps** (expiration, invalidation).
- Un classement de priorités n'a de valeur que si son **critère est explicite**.

</details>

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

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre. Les réponses se déplient juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un client connecté de DIGICHEESE ouvre `/admin/clients` et récupère la liste complète des clients de la boutique. Qu'est-ce qui a manqué ?

- A. Une authentification : le client n'était pas identifié par la boutique, aucune session n'aurait dû être ouverte
- B. Une autorisation : personne n'a vérifié ce qu'il avait le droit de voir
- C. Une MFA : un second facteur sur le téléphone du client aurait bloqué la requête avant l'affichage
- D. Une session plus courte : la session aurait expiré avant que le client ouvre cette adresse

**2.** Un client authentifié ouvre `/commandes/42` et tombe sur la commande d'un autre. Quelle vérification manque ?

- A. Le serveur n'a pas vérifié que la commande 42 appartient bien au compte connecté
- B. Le serveur a vérifié l'identité du compte au lieu de vérifier les droits : il fallait contrôler le rôle avant de renvoyer la commande
- C. Le compte aurait dû passer par un second facteur : la commande d'un autre n'aurait pas été servie sans validation sur le téléphone
- D. Les numéros de commande n'auraient pas dû se suivre : un identifiant imprévisible aurait fermé toutes les portes

**3.** Quel moyen de preuve appartient à la même famille qu'un mot de passe ?

- A. Une empreinte digitale
- B. Une question secrète
- C. Un code reçu sur le téléphone
- D. Une carte d'agent

**4.** Face à une fuite de données, pourquoi la biométrie est-elle la plus fragile des trois familles ?

- A. Elle se devine plus vite qu'un mot de passe, les empreintes les plus courantes étant déjà cataloguées
- B. Elle ne se change pas : une empreinte fuitée l'est pour toujours
- C. Elle se prête à un tiers comme un téléphone : on peut la prêter pour une connexion puis la reprendre
- D. Elle ne peut pas être combinée avec un autre facteur, ce qui interdit d'en faire un second facteur

**5.** *Question ouverte (2-3 lignes).* Le chapitre compare la session au badge d'un musée : « il prouve que vous êtes entré par la caisse, pas que vous pouvez ouvrir les réserves ». Que prouve ce badge, que ne prouve-t-il pas ? Illustrez avec un exemple pris dans ce chapitre.

---

<details>

<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** L'identité était établie, le contrôle des droits n'existait pas.
**Pourquoi :** AuthN et AuthZ sont deux portes distinctes, et « connecté » ne veut pas dire « autorisé ».

**2. A.** C'est l'erreur « la donnée confiée à l'identifiant de l'adresse » : chaque numéro devient une porte d'entrée si l'appartenance n'est pas vérifiée.
**Pourquoi :** savoir *qui* a fait la requête ne dit rien sur le droit de voir *cette* ressource-là.

**3. B.** Question secrète et mot de passe sont deux preuves de « ce qu'on sait ».
**Pourquoi :** même famille, mêmes fragilités — fuite, réutilisation, devinette.

**4. B.** Une empreinte ne se révoque pas, contrairement à un mot de passe qu'on change ou à une carte qu'on bloque.
**Pourquoi :** c'est la fragilité principale de la famille « ce qu'on est », telle que la pose le tableau des vecteurs.

**5.** Attendu : le badge prouve que l'authentification a réussi et permet d'être reconnu à chaque requête ; il ne dit rien des droits. Exemple attendu : le client authentifié qui voit les commandes de toute la boutique, ou le bouton « Administration » masqué dans l'interface alors que l'API répond.
**Pourquoi :** la session est la preuve de l'authentification qui circule — elle se protège, mais elle ne remplace jamais la consultation des droits.

</details>

---

[Sommaire](Readme.md) · [Chapitre suivant →](02-mots-de-passe.md)
