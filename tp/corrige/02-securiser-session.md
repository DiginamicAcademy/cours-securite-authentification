# Corrigé — Sécuriser la session d'une application

> Corrigé de [l'exercice « Sécuriser la session d'une application »](../02-securiser-session.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

## Partie 1 — Repérer les failles

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

## Partie 2 — Corriger le code

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

## Partie 3 — Drapeaux et expiration

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

## Partie 4 — Vérifier

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

## Points à retenir

- Le jeton de session est un secret : il se **tire au sort** côté serveur, jamais
  dérivé d'une donnée connue.
- Il se **régénère** à la connexion — c'est la défense contre la fixation.
- Les drapeaux du cookie protègent chacun une menace précise ; aucun ne remplace
  le chiffrement du transport.
- L'expiration et l'invalidation se vérifient **côté serveur** : le navigateur
  n'est pas un garde.
