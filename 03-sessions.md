[← Chapitre précédent](02-mots-de-passe.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-jwt-et-tokens.md)

# Sessions — le badge de connexion

> Vous consultez votre compte bancaire en ligne. Entre la connexion et la
> dernière opération, le serveur ne vous a pas redemandé votre mot de passe à
> chaque page : il vous a remis un **badge** — la session. Ce badge vous suit
> partout, et s'il est volé, l'attaquant devient vous, sans jamais avoir connu
> votre secret.

**Durée :** ≈ 16 min — lecture ≈ 11 min + atelier 5 min · **Prérequis :** [Authentifier n'est pas autoriser](01-authentifier-nest-pas-autoriser.md)

> 📖 Vocabulaire et acronymes : [glossaire du cours](A-Glossaire-et-ressources.md).

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Décrire** le cycle de vie d'une session : création, transmission, expiration, invalidation.
- **Distinguer** la fixation de session du détournement de session, et nommer la défense de chacun.
- **Configurer** les drapeaux d'un cookie de session (HttpOnly, Secure, SameSite) selon la menace visée.
- **Sécuriser** une session web : transport chiffré, expiration, invalidation à la déconnexion.

---

## I. Le principe

### Ce que le serveur remet

À la connexion, le serveur vérifie l'identité (chapitre
[Mots de passe](02-mots-de-passe.md)), puis crée une **session** et remet au
navigateur un identifiant. À chaque requête suivante, le navigateur renvoie cet
identifiant : le serveur retrouve la session et n'a plus besoin du mot de passe.

Deux modèles coexistent :

| Modèle | Où vivent les données de session | Avantage | Limite |
| --- | --- | --- | --- |
| Session côté serveur | le serveur garde un dictionnaire identifiant → données | révocation immédiate | état à partager entre les serveurs d'un cluster |
| Jeton signé (sans état) | le jeton porte lui-même les informations | aucun état serveur, monte en charge | la révocation demande des mesures supplémentaires |

### Un jeton que l'on porte

La session est une preuve de la famille « ce qu'on **a** » : quiconque présente le
jeton est traité comme l'utilisateur. C'est pratique — et c'est exactement ce qui
la rend fragile. Un mot de passe volé se change ; une session volée est déjà
ouverte.

> 🧪 **Atelier rapide (5 min)**
>
> Sur le lab (`secure`, port 8081), connectez-vous et observez le jeton renvoyé :

```bash
curl -s -X POST -F "username=zoe" -F "password=zoe123" \
  http://localhost:8081/login
```

> La réponse contient un `token` en trois morceaux séparés par des points. Le
> premier est lisible — décodez-le **en local** (un jeton réel ne se colle jamais
> dans un site tiers) :

```bash
python3 -c "import base64,sys; p=sys.argv[1].split('.')[0]; print(base64.urlsafe_b64decode(p + '=' * (-len(p) % 4)))" "VOTRE_JETON"
```

> Vous y lisez le nom d'utilisateur. Le reste du jeton est une **signature** : le
> serveur vérifie que personne n'a modifié le contenu, sans interroger de base de
> données. Dans ce lab, le jeton expire **1 heure après son émission** — retrouvez
> ce chiffre dans le code du service `secure`.

---

## II. Fixation et détournement

Deux attaques, deux mécanismes — et deux défenses différentes.

### Le détournement : voler le badge

L'attaquant récupère le jeton d'une session ouverte. Par où ? Un script injecté —
XSS (cross-site scripting) — qui lit le cookie, un réseau en clair, un poste
partagé où la session reste ouverte, un log qui contient le jeton. Une fois le
jeton en main, il n'a plus rien à deviner : la requête qu'il envoie est
indiscernable de la vôtre.

### La fixation : imposer le badge

L'attaquant choisit lui-même l'identifiant de session et le fait attribuer à sa
victime — un lien contenant un identifiant de session, un cookie posé sur un
sous-domaine qu'il contrôle. La victime se connecte ; si l'application conserve
le même identifiant après la connexion, la copie de l'attaquant est maintenant
authentifiée.

La défense tient en un geste : **régénérer l'identifiant de session à la
connexion** (et à chaque changement de privilège). L'identifiant attribué avant
la connexion ne doit jamais survivre à celle-ci.

> ⛔ **Piège de sécurité**
> Un identifiant de session calculé de façon prévisible — le MD5 du nom
> d'utilisateur, par exemple — cumule les deux problèmes : il se devine, et il ne
> change jamais. L'identifiant doit être tiré au sort par le serveur.

---

## III. Flags et expiration

### Les drapeaux du cookie

Un cookie de session se protège par des **drapeaux** (flags), trois en pratique :

| Drapeau | Ce qu'il protège | Sa limite |
| --- | --- | --- |
| `HttpOnly` | le vol par script : le JavaScript ne lit pas le cookie | rien contre le vol réseau ou le poste partagé |
| `Secure` | l'envoi en clair : le cookie ne part qu'en HTTPS (HyperText Transfer Protocol Secure) | exige HTTPS partout, y compris en développement |
| `SameSite` | les requêtes forgées depuis un autre site (CSRF, cross-site request forgery) | `Strict` casse les liens entrants (courriel, moteur de recherche) |

```http
Set-Cookie: session=VOTRE_JETON; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

Aucun drapeau ne remplace le chiffrement du transport : sans HTTPS, le jeton
transite en clair et se capte sur le réseau.

### L'expiration

Une session ouverte indéfiniment est un passif. Deux durées se combinent :

- l'**expiration par inactivité** : sans requête pendant un certain temps, la
  session se ferme — c'est la protection contre un poste oublié ouvert ;
- l'**expiration absolue** : au-delà d'une durée maximale, la session se termine
  même si elle est active — un jeton volé ne reste pas exploitable des semaines.

Le lab fixe une seule limite, d'une heure à compter de l'émission du jeton —
une limite absolue, qui court même si la session reste active. Une application
bancaire combine les deux ; un outil interne peut être plus laxiste. C'est un arbitrage de
proportionnalité, pas une constante universelle.

### La déconnexion

Se déconnecter, c'est **invalider la session côté serveur**, puis effacer le
cookie. Une déconnexion qui se contente d'effacer le cookie du navigateur laisse
le jeton vivre : copié avant la déconnexion, il rouvre la session.

> 💡 **Astuce**
> Testez votre déconnexion comme le ferait un attaquant : copiez le jeton avant de
> vous déconnecter, puis rejouez-le avec `curl`. S'il fonctionne encore, la
> session n'a pas été invalidée côté serveur.

---

### Exercice 1 — Sécuriser la session d'une application (45 min)

#### Contexte

Une billetterie en ligne confie la gestion des sessions de son espace membre à un
développeur pressé. Voici le code livré, en FastAPI :

```python
import hashlib
import time

from fastapi import FastAPI, Form, Request, Response

app = FastAPI(title="Billetterie — espace membre")

SESSIONS = {}  # jeton -> (identifiant, date_de_creation)


@app.post("/connexion")
def connexion(identifiant: str = Form(...), mot_de_passe: str = Form(...),
              response: Response):
    if not mot_de_passe_valide(identifiant, mot_de_passe):
        return {"erreur": "identifiants invalides"}
    jeton = hashlib.md5(identifiant.encode()).hexdigest()
    SESSIONS[jeton] = (identifiant, time.time())
    response.set_cookie("session", jeton)
    return {"message": f"bienvenue {identifiant}"}


@app.get("/mon-compte")
def mon_compte(request: Request):
    jeton = request.cookies.get("session")
    if jeton in SESSIONS:
        return {"compte": SESSIONS[jeton][0]}
    return {"erreur": "non connecté"}


@app.post("/deconnexion")
def deconnexion(request: Request, response: Response):
    response.delete_cookie("session")
    return {"message": "déconnecté"}
```

La fonction `mot_de_passe_valide` est considérée comme correcte (vérification
d'un hachage bcrypt) : elle n'est pas dans le périmètre de l'exercice.

La billetterie impose les règles suivantes :

- la session expire après **1 heure** sans activité, et **8 heures** au maximum
  après la connexion ;
- le cookie de session ne doit jamais être lu par le JavaScript de la page, ni
  voyager en clair sur le réseau ;
- la déconnexion doit invalider le jeton **immédiatement**, y compris pour un
  client qui en aurait gardé une copie.

---

#### Consignes

##### Partie 1 — Repérer les failles

Listez les failles de gestion de session du code ci-dessus. Pour chacune, nommez
**ce qui manque** (le contrôle absent ou le choix dangereux), puis décrivez ce
qu'un attaquant peut en faire. Il y en a au moins quatre.

##### Partie 2 — Corriger le code

Réécrivez les trois routes (`/connexion`, `/mon-compte`, `/deconnexion`) pour
corriger les failles identifiées. Le jeton doit être tiré au sort par le serveur,
régénéré à chaque connexion, et vérifié à chaque requête. Vous ajouterez les
importations nécessaires.

##### Partie 3 — Drapeaux et expiration

Donnez la ligne `Set-Cookie` exacte que doit envoyer la billetterie, et justifiez
chaque drapeau par la menace qu'il couvre. Puis décrivez comment implémenter les
deux expirations (inactivité et maximale) avec les données déjà présentes dans le
dictionnaire de sessions.

##### Partie 4 — Vérifier

Proposez au moins trois tests qui prouvent que vos corrections fonctionnent.
Chaque test décrit une action et le résultat attendu.

---

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Chaque faille nomme **ce qui manque**, pas seulement la conséquence observée.
- [ ] Mon code de connexion **régénère** le jeton à chaque connexion, avec un tirage aléatoire cryptographique.
- [ ] Ma ligne `Set-Cookie` porte les **trois drapeaux** du chapitre, et je sais dire ce que chacun ne protège pas.
- [ ] Mes deux expirations (inactivité et maximale) sont **vérifiées côté serveur**, pas seulement dans le navigateur.
- [ ] Ma déconnexion supprime la session **côté serveur**, et mes tests le prouvent.

---

#### Pour aller plus loin

*Facultatif.* La billetterie passe sur deux serveurs derrière un répartiteur de
charge. Où placer les données de session, et que se passe-t-il si un utilisateur
est routé vers l'autre serveur en pleine réservation ? Comparez les deux modèles
du chapitre (session côté serveur, jeton signé).

<details>

<summary>Voir la correction</summary>

> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

#### Partie 1 — Repérer les failles

**Éléments attendus :**

| Faille | Ce qui manque | Ce qu'un attaquant en fait |
| --- | --- | --- |
| Jeton calculé par MD5 à partir de l'identifiant | un tirage aléatoire côté serveur | il calcule le jeton de n'importe quel utilisateur sans jamais le mot de passe |
| Jeton identique à chaque connexion | une régénération à la connexion | il conserve un ancien jeton, ou fixe le jeton de sa victime (fixation de session) |
| Cookie sans drapeau | `HttpOnly`, `Secure`, `SameSite` | il lit le jeton par un script injecté, ou le capte sur le réseau |
| Aucune expiration vérifiée | un contrôle de durée côté serveur | un jeton volé reste exploitable indéfiniment |
| Déconnexion sans suppression serveur | l'invalidation de la session | il rejoue le jeton copié avant la déconnexion |

**Ce qu'il faut retenir :** la faille la plus grave est la première. Un jeton
dérivé de l'identifiant n'est pas un secret — c'est une adresse déguisée. Les
autres failles aggravent le problème, mais celle-là le rend trivial.

**Erreurs fréquentes :** ne voir que le cookie sans drapeau, parce que c'est le
plus visible dans le code. Regardez d'abord **comment le jeton est produit**, puis
comment il est transmis, puis comment il meurt.

---

#### Partie 2 — Corriger le code

**Éléments attendus :** les trois routes sont corrigées, les importations
complétées, et le jeton est tiré au sort.

```python
import secrets
from datetime import datetime, timedelta

from fastapi import FastAPI, Form, Request, Response

app = FastAPI(title="Billetterie — espace membre")

SESSIONS = {}  # jeton -> {"identifiant", "creation", "derniere_activite"}

INACTIVITE = timedelta(hours=1)   # 1 heure sans activité, comme imposé
MAXIMALE = timedelta(hours=8)     # 8 heures au total, comme imposé


@app.post("/connexion")
def connexion(identifiant: str = Form(...), mot_de_passe: str = Form(...),
              response: Response):
    if not mot_de_passe_valide(identifiant, mot_de_passe):
        return {"erreur": "identifiants invalides"}
    # Tirage aléatoire cryptographique : imprévisible et unique par connexion.
    jeton = secrets.token_urlsafe()
    maintenant = datetime.now()
    SESSIONS[jeton] = {
        "identifiant": identifiant,
        "creation": maintenant,
        "derniere_activite": maintenant,
    }
    # Le jeton est régénéré à chaque connexion : un ancien jeton ne vaut plus rien.
    response.set_cookie(
        "session",
        jeton,
        httponly=True,
        secure=True,
        samesite="lax",
        path="/",
        max_age=int(MAXIMALE.total_seconds()),
    )
    return {"message": f"bienvenue {identifiant}"}


@app.get("/mon-compte")
def mon_compte(request: Request):
    jeton = request.cookies.get("session")
    session = SESSIONS.get(jeton)
    if session is None:
        return {"erreur": "non connecté"}
    maintenant = datetime.now()
    expiree = (
        maintenant - session["derniere_activite"] > INACTIVITE
        or maintenant - session["creation"] > MAXIMALE
    )
    if expiree:
        del SESSIONS[jeton]
        return {"erreur": "session expirée"}
    session["derniere_activite"] = maintenant
    return {"compte": session["identifiant"]}


@app.post("/deconnexion")
def deconnexion(request: Request, response: Response):
    jeton = request.cookies.get("session")
    if jeton in SESSIONS:
        del SESSIONS[jeton]          # invalidation côté serveur, d'abord
    response.delete_cookie("session", path="/")
    return {"message": "déconnecté"}
```

**Points de vigilance :**

- `secrets.token_urlsafe()` s'appuie sur le générateur cryptographique de la
  bibliothèque standard — `random` ou `uuid1` ne sont pas des tirages imprévisibles.
- Les deux expirations sont testées **à chaque requête**, côté serveur : le
  `Max-Age` du cookie n'est qu'un confort d'affichage, il ne protège rien.
- La déconnexion supprime d'abord la session serveur, puis le cookie. Dans cet
  ordre, une erreur d'effacement du cookie ne laisse pas une session ouverte.

---

#### Partie 3 — Drapeaux et expiration

**Éléments attendus :**

```http
Set-Cookie: session=VOTRE_JETON; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=28800
```

(`Max-Age` s'exprime en secondes : `28800` correspond aux **8 heures** imposées
par l'énoncé.)

| Drapeau | Menace couverte | Ce qu'il ne couvre pas |
| --- | --- | --- |
| `HttpOnly` | la lecture du cookie par un script injecté — XSS (cross-site scripting) | le vol réseau, le poste partagé |
| `Secure` | l'envoi du cookie en clair sur le réseau | rien si le site n'est pas en HTTPS partout |
| `SameSite=Lax` | les requêtes forgées depuis un autre site — CSRF (cross-site request forgery) | les attaques depuis le site lui-même |
| `Path=/` | l'envoi du cookie aux chemins inutiles | — |
| `Max-Age` | la durée de vie côté navigateur | l'expiration réelle, qui doit être vérifiée côté serveur |

Pour les deux expirations, le dictionnaire de sessions porte déjà la date de
création : on y ajoute la date de dernière activité, mise à jour à chaque requête.
Une session est refusée si l'inactivité dépasse **1 heure** ou si l'âge total
dépasse **8 heures** — puis supprimée du dictionnaire.

**Erreurs fréquentes :** croire que `Max-Age` expire la session. Le navigateur
oublie le cookie, mais un jeton copié reste valable tant que le serveur ne l'a pas
refusé.

---

#### Partie 4 — Vérifier

**Éléments attendus** (au moins trois tests, chacun avec action et résultat
attendu) :

| Action | Résultat attendu |
| --- | --- |
| Se connecter, copier le jeton, se déconnecter, rejouer le jeton copié | refus : la session a été supprimée côté serveur |
| Se connecter deux fois de suite avec le même compte | les deux jetons sont différents, et le premier reste utilisable jusqu'à expiration |
| Calculer le MD5 d'un identifiant connu et l'envoyer comme cookie | refus : ce jeton n'a jamais été attribué |
| Laisser la session inactive pendant plus de **1 heure**, puis requêter | refus : expiration pour inactivité |
| Rejouer un jeton après **8 heures** d'activité continue | refus : expiration maximale |
| Lire le cookie depuis le JavaScript de la page | échec : `HttpOnly` |

**Ce qu'il faut retenir :** un test de session se rejoue **avec un jeton copié**.
Tant que ce scénario n'est pas dans la batterie de tests, la déconnexion reste
théorique.

---

#### Points à retenir

- Le jeton de session est un secret : il se **tire au sort** côté serveur, jamais
  dérivé d'une donnée connue.
- Il se **régénère** à la connexion — c'est la défense contre la fixation.
- Les drapeaux du cookie protègent chacun une menace précise ; aucun ne remplace
  le chiffrement du transport.
- L'expiration et l'invalidation se vérifient **côté serveur** : le navigateur
  n'est pas un garde.

</details>

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **CSRF** (cross-site request forgery) : le navigateur envoyant les cookies
  automatiquement, une page malveillante peut déclencher une requête authentifiée
  à votre place. Défenses : `SameSite`, jeton anti-CSRF dans les formulaires, et
  jamais d'action sensible déclenchée par une simple consultation.
- **`SameSite=Strict`** supprime les requêtes forgées venues d'un autre site,
  mais casse aussi la navigation légitime depuis un lien externe : l'utilisateur
  arrive sur le site « non connecté » le temps d'une page. `Lax` est le compromis
  courant.
- **Le préfixe `__Host-`** sur le nom du cookie : le navigateur refuse alors un
  cookie de ce nom posé par un sous-domaine — une protection contre la fixation
  par sous-domaine.

---

## Points clés

- La session est un **jeton porteur** : quiconque le détient est l'utilisateur —
  d'où le transport chiffré, les drapeaux et les durées de vie courtes.
- **Fixation** : l'attaquant choisit l'identifiant → on le régénère à la
  connexion. **Détournement** : l'attaquant vole l'identifiant → `HttpOnly`,
  `Secure`, HTTPS.
- `HttpOnly` limite le vol par script, `SameSite` limite les requêtes forgées
  depuis un autre site — aucun drapeau ne remplace HTTPS.
- Une session se termine **des deux côtés** : expiration côté serveur **et**
  invalidation à la déconnexion.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre. Les réponses se déplient juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un attaquant envoie à votre victime un lien contenant un identifiant de session. Elle se connecte depuis ce lien, et l'application conserve le même identifiant après la connexion. Quelle attaque, et quelle défense ?

- A. Détournement de session — ajouter le drapeau `HttpOnly`
- B. Fixation de session — régénérer l'identifiant à la connexion
- C. Détournement de session — passer tout le site en HTTPS
- D. Fixation de session — raccourcir l'expiration par inactivité

**2.** Un script injecté (XSS) sur votre site parvient à lire le cookie de session. Quel drapeau manquait ?

- A. `HttpOnly`
- B. `Secure`
- C. `SameSite=Lax`
- D. `Max-Age=3600`

**3.** Depuis trois semaines, une session reste ouverte sur un poste partagé : l'utilisatrice est active, le serveur ne la ferme jamais. Quelle durée aurait dû la fermer ?

- A. L'expiration par inactivité
- B. L'expiration absolue
- C. Le drapeau `Secure`
- D. Aucune : une session active n'a pas à être fermée

**4.** Votre déconnexion se contente d'effacer le cookie du navigateur. Un attaquant avait copié le jeton avant. Que se passe-t-il ?

- A. Rien : le jeton meurt en même temps que le cookie, puisque le serveur le rattache à ce cookie
- B. Le jeton fonctionne toujours : la session n'a pas été invalidée côté serveur
- C. Le serveur invalide le jeton automatiquement dès que le cookie part
- D. Le jeton devient inutilisable parce que l'identifiant est régénéré

**5.** *Question ouverte (2-3 lignes).* Deux modèles de session coexistent : session côté serveur et jeton signé sans état. Pour un cluster de dix serveurs qui doit monter en charge, lequel choisissez-vous, et qu'acceptez-vous en échange ?

---

<details>

<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** C'est la fixation : l'attaquant a choisi l'identifiant et l'a fait adopter par la victime. La défense est de régénérer l'identifiant à la connexion, et à chaque changement de privilège.
**Pourquoi :** un identifiant attribué avant la connexion ne doit jamais survivre à celle-ci ; les drapeaux et HTTPS protègent contre le vol, pas contre l'imposition.

**2. A.** `HttpOnly` retire le cookie au JavaScript, donc au script injecté.
**Pourquoi :** chaque drapeau a sa menace et sa limite — `Secure` ne protège que le transport, `SameSite` que les requêtes forgées venues d'un autre site, et aucun ne remplace HTTPS.

**3. B.** L'expiration absolue termine la session au-delà d'une durée maximale même si elle est active ; celle par inactivité ne ferme qu'un poste oublié.
**Pourquoi :** les deux se combinent — l'inactivité protège le poste laissé ouvert, l'absolue empêche un jeton volé de rester exploitable des semaines.

**4. B.** Le jeton copié rouvre la session : se déconnecter, c'est invalider côté serveur, puis effacer le cookie.
**Pourquoi :** le test du chapitre est justement de copier le jeton, de se déconnecter, puis de le rejouer avec `curl` — s'il passe, la déconnexion est fausse.

**5.** Attendu : le jeton signé sans état, parce qu'il porte lui-même ses informations et évite de partager un dictionnaire de sessions entre les serveurs ; en échange, la révocation n'est plus immédiate et demande des mesures supplémentaires (durées courtes, liste de jetons invalidés).
**Pourquoi :** la session côté serveur offre une révocation immédiate mais un état à partager dans le cluster — on choisit selon la contrainte qui domine.

</details>

---

[← Chapitre précédent](02-mots-de-passe.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-jwt-et-tokens.md)
