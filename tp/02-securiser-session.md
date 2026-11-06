# Exercice — Sécuriser la session d'une application

> 🟡 **Intermédiaire** · **Durée :** 45 min · **Lien :** chapitre [Sessions — le badge de connexion](../03-sessions.md)
> **Objectif :** repérer les failles de gestion de session dans un extrait de code, puis le corriger (drapeaux, expiration, invalidation).

---

## Contexte

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

## Consignes

### Partie 1 — Repérer les failles

Listez les failles de gestion de session du code ci-dessus. Pour chacune, nommez
**ce qui manque** (le contrôle absent ou le choix dangereux), puis décrivez ce
qu'un attaquant peut en faire. Il y en a au moins quatre.

### Partie 2 — Corriger le code

Réécrivez les trois routes (`/connexion`, `/mon-compte`, `/deconnexion`) pour
corriger les failles identifiées. Le jeton doit être tiré au sort par le serveur,
régénéré à chaque connexion, et vérifié à chaque requête. Vous ajouterez les
importations nécessaires.

### Partie 3 — Drapeaux et expiration

Donnez la ligne `Set-Cookie` exacte que doit envoyer la billetterie, et justifiez
chaque drapeau par la menace qu'il couvre. Puis décrivez comment implémenter les
deux expirations (inactivité et maximale) avec les données déjà présentes dans le
dictionnaire de sessions.

### Partie 4 — Vérifier

Proposez au moins trois tests qui prouvent que vos corrections fonctionnent.
Chaque test décrit une action et le résultat attendu.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Chaque faille nomme **ce qui manque**, pas seulement la conséquence observée.
- [ ] Mon code de connexion **régénère** le jeton à chaque connexion, avec un tirage aléatoire cryptographique.
- [ ] Ma ligne `Set-Cookie` porte les **trois drapeaux** du chapitre, et je sais dire ce que chacun ne protège pas.
- [ ] Mes deux expirations (inactivité et maximale) sont **vérifiées côté serveur**, pas seulement dans le navigateur.
- [ ] Ma déconnexion supprime la session **côté serveur**, et mes tests le prouvent.

---

## Pour aller plus loin

*Facultatif.* La billetterie passe sur deux serveurs derrière un répartiteur de
charge. Où placer les données de session, et que se passe-t-il si un utilisateur
est routé vers l'autre serveur en pleine réservation ? Comparez les deux modèles
du chapitre (session côté serveur, jeton signé).
