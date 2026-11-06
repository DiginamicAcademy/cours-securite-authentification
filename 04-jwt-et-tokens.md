[← Chapitre précédent](03-sessions.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-oauth2-oidc.md)

# JWT et tokens — la lettre signée

> Un serveur qui vous a identifié vous remet parfois une **lettre signée** :
> chacun peut la lire, personne ne peut la modifier sans casser la signature.
> C'est le principe d'un JWT (JSON Web Token — JSON pour JavaScript Object
> Notation) — et la source de ses
> mésaventures : une signature non vérifiée, une clé faible ou un jeton sans
> expiration, et la lettre devient un laissez-passer contrefait.

**Durée :** ≈ 20 min — lecture ≈ 12 min + atelier 8 min · **Prérequis :** [Sessions — le badge de connexion](03-sessions.md)

> 📖 Vocabulaire et acronymes : [glossaire du cours](A-Glossaire-et-ressources.md).

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Décomposer** un JWT (JSON Web Token) en ses trois parties et expliquer le rôle de chacune.
- **Distinguer** signature et chiffrement : un JWT est lisible par tous, vérifiable par le détenteur de la clé.
- **Repérer** les erreurs classiques d'usage : algorithme non vérifié, clé faible, stockage web, durée de vie excessive.
- **Contrôler** un jeton reçu côté serveur selon une liste de vérifications explicite.

---

## I. Structure

### Trois morceaux séparés par des points

Un JWT se lit de gauche à droite :

```text
eyJhbGciOiJIUzI1NiJ9 . eyJzdWIiOiJ6b2UiLCJleHAiOjE3fQ . SflKxwRJSMeKKF2QT4fwpM
└────── en-tête ─────┘   └──────── charge utile ────────┘   └──── signature ────┘
   { "alg": "HS256" }       { "sub": "zoe", "exp": … }      empreinte de l'en-tête
                                                            et de la charge utile
```

- **L'en-tête** annonce l'algorithme de signature (`alg`) et le type de jeton.
- **La charge utile** (payload) porte les **déclarations** (claims) : `sub` (le
  sujet, l'identifiant de l'utilisateur), `exp` (la date d'expiration), `iss`
  (l'émetteur), `aud` (le destinataire), et les rôles ou périmètres utiles.
- **La signature** est une empreinte calculée sur les deux premières parties avec
  une clé. Modifier un seul caractère de la charge utile la rend invalide.

Les deux premières parties sont en **base64url** : c'est un encodage, pas un
chiffrement. Tout le monde peut les relire — c'est voulu : un serveur qui reçoit
le jeton doit pouvoir consulter les déclarations sans appeler qui que ce soit.

> ⚠️ **Attention**
> Ne mettez jamais d'information sensible dans la charge utile d'un JWT : un
> numéro de sécurité sociale, un mot de passe, une donnée personnelle. Le jeton
> est **lisible**, il n'est pas **confidentiel**.

### Signature n'est pas chiffrement

| Opération | Qui peut lire | Qui peut modifier | Usage |
| --- | --- | --- | --- |
| Signature — HMAC (Hash-based Message Authentication Code) ou RSA (Rivest, Shamir et Adleman) | tout le monde | personne sans la clé | prouver l'origine et l'intégrité |
| Chiffrement | le seul destinataire | personne sans la clé | protéger la confidentialité |

Un JWT signé répond à « ce jeton vient-il bien de mon serveur, et n'a-t-il pas
été trafiqué ? ». Il ne répond pas à « qui d'autre l'a lu ? ».

> 🧪 **Atelier rapide (8 min)**
>
> Fabriquez un jeton signé à la main, puis cassez-le. Ce script crée une charge
> utile, la signe avec une clé, puis vérifie la signature :

```python
import base64, hashlib, hmac

cle = b"cle-du-lab"

def encodage(brut: bytes) -> bytes:
    return base64.urlsafe_b64encode(brut).rstrip(b"=")

entete = encodage(b'{"alg":"HS256"}')
charge = encodage(b'{"sub":"zoe","role":"user"}')
message = entete + b"." + charge
signature = encodage(hmac.new(cle, message, hashlib.sha256).digest())
jeton = message + b"." + signature
print(jeton.decode())

# Vérification : même clé, même message -> la signature colle.
print(hmac.compare_digest(signature, encodage(hmac.new(cle, message, hashlib.sha256).digest())))
```

> Modifiez la charge utile (`"role":"user"` devient `"role":"admin"`) sans
> changer la clé : la vérification renvoie `False`. Re-signez avec la bonne clé :
> elle renvoie `True`. C'est toute la logique d'un JWT — la signature lie le
> contenu à la clé.

---

## II. Erreurs d'usage

La commodité des JWT a popularisé quelques erreurs, toutes spectaculaires.

- **`alg: none`** — l'en-tête annonce « pas de signature ». Un serveur qui fait
  confiance à l'en-tête du jeton accepte alors n'importe quel contenu fabriqué à
  la main. L'algorithme attendu se décide **côté serveur**, en liste blanche, et
  jamais dans le jeton.
- **Ne vérifier que le décodage** — décoder la charge utile sans vérifier la
  signature revient à accepter un document non signé.
- **Clé faible ou clé par défaut** — une signature HMAC avec la clé `secret` se
  casse par devinette. La clé est un
  secret au sens du chapitre [Mots de passe](02-mots-de-passe.md) : longue,
  tirée au sort, stockée dans un coffre.
- **Durée de vie excessive, aucune révocation** — un jeton valable un an, volé
  aujourd'hui, reste bon un an. La révocation d'un JWT signé sans état est
  difficile : on compense par des durées courtes et des jetons de
  rafraîchissement.
- **Stockage web à risque** — dans `localStorage`, le jeton est lisible par
  n'importe quel script injecté — XSS (cross-site scripting). Un cookie
  `HttpOnly` le cache au
  JavaScript ; le choix se discute selon l'architecture (chapitre
  [Sessions](03-sessions.md)).
- **Le jeton dans l'adresse** — passé en paramètre d'URL (Uniform Resource
  Locator), il se retrouve dans les journaux du serveur, l'historique du
  navigateur et les en-têtes `Referer`.

> ⛔ **Piège de sécurité**
> Un JWT n'est pas « sécurisé par nature ». Il est sécurisé **si et seulement si**
> le serveur vérifie la signature, l'algorithme, l'expiration et le destinataire —
> et si la clé reste secrète.

---

## III. Contrôler un jeton reçu

La vérification côté serveur tient en une liste courte, à appliquer dans l'ordre :

| Contrôle | Pourquoi | Si le contrôle manque |
| --- | --- | --- |
| Signature valide, algorithme en liste blanche | le jeton vient bien de l'émetteur | jeton fabriqué à la main |
| `exp` (expiration) et `nbf` (validité) | le jeton est encore dans sa période | jeton volé exploitable indéfiniment |
| `iss` (émetteur) et `aud` (destinataire) | le jeton est bien pour moi | jeton d'un autre service accepté ici |
| Périmètre ou rôle (`scope`) | l'identité a le droit demandé | confusion authentification / autorisation |
| Révocation | le jeton n'a pas été invalidé | un jeton compromis reste bon |

Sur la révocation, deux écoles :

- **Durées courtes + rafraîchissement** : le jeton d'accès vit quelques minutes,
  un jeton de rafraîchissement (refresh token) en obtient des neufs. Compromis
  détecté ? on coupe le jeton de rafraîchissement.
- **Liste noire** : le serveur garde une liste de jetons invalidés, vérifiée à
  chaque requête. Cela réintroduit un état serveur — et donc une base à
  interroger vite.

Un détail d'implémentation qui coûte cher : tolérer une petite différence
d'horloge entre serveurs (quelques secondes de marge sur `exp`), sans quoi des
jetons valides sont refusés au hasard.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **Jetons de référence** (opaque tokens) : le serveur ne stocke qu'un identifiant
  aléatoire, les données vivent côté serveur. Révocation immédiate, au prix d'un
  appel à chaque vérification. Le chapitre suivant montre quand choisir l'un ou
  l'autre.
- **OAuth2** (Open Authorization 2) : les JWT sont souvent le véhicule des jetons
  distribués par un serveur d'autorisation — c'est le sujet du chapitre
  [OAuth2 et OpenID Connect](05-oauth2-oidc.md).
- **Rotation des clés** : une clé de signature se change sans casser les jetons en
  circulation en publiant un identifiant de clé (`kid`) dans l'en-tête, et en
  acceptant temporairement les deux clés.

---

## Points clés

- Un JWT se lit de tous : **base64url n'est pas du chiffrement**. La signature
  prouve l'origine, elle ne cache rien.
- La signature lie la charge utile à une **clé** : sans elle, rien ne se
  fabrique ; avec une clé faible, tout se fabrique.
- Les erreurs classiques sont connues : `alg: none`, signature non vérifiée,
  jeton dans `localStorage`, durée de vie excessive.
- La vérification côté serveur est une **liste** : signature, algorithme,
  expiration, émetteur, destinataire, périmètre, révocation.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre. Les réponses se déplient juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un collègue veut écrire l'adresse postale de l'utilisateur dans la charge utile « de toute façon c'est encodé en base64, donc personne ne le lit ». Que lui répondez-vous ?

- A. Il a raison : base64url est une forme de chiffrement, sans la clé personne ne relit la charge utile
- B. Il a tort : base64url est un encodage, la charge utile est lisible par tous
- C. Il a raison si la clé de signature est assez longue : la charge utile devient illisible sans elle
- D. Il a tort : l'adresse doit aller dans l'en-tête à la place, que seul le serveur est censé lire

**2.** Votre API accepte un jeton dont l'en-tête annonce `{"alg":"none"}` et laisse son auteur choisir son propre rôle. Quelle règle a été violée ?

- A. La clé de signature était trop courte : une clé robuste aurait rendu la fabrication impossible
- B. L'algorithme attendu se décide côté serveur, en liste blanche, jamais dans le jeton
- C. Le jeton voyageait dans une URL, où un tiers a pu le lire puis modifier le rôle avant l'appel
- D. Le jeton ne portait pas de déclaration `aud` : n'importe quel destinataire pouvait donc l'utiliser

**3.** Votre API accepte un jeton parfaitement signé, non expiré, émis par un service partenaire — et il ouvre des données qui ne lui étaient pas destinées. Quelle déclaration n'a pas été contrôlée ?

- A. `exp`
- B. `nbf`
- C. `aud`
- D. `scope`

**4.** Votre front web range le jeton d'accès dans `localStorage`. Quel risque, et quelle alternative le chapitre met en face ?

- A. Risque : le jeton est renvoyé à chaque requête — alternative : le mettre dans le corps des requêtes POST (celles qui envoient des données)
- B. Risque : n'importe quel script injecté le lit — alternative : un cookie `HttpOnly`
- C. Risque : il expire plus vite dans `localStorage` — alternative : une variable de session
- D. Aucun risque dès lors que le site est en HTTPS : le stockage local du navigateur est hors d'atteinte

**5.** *Question ouverte (2-3 lignes).* Écrivez la liste des contrôles que votre API applique à un JWT reçu, dans l'ordre, et dites ce que chacun attrape.

---

<details>

<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** Base64url est un encodage : il rend la charge utile lisible par quiconque reçoit le jeton, et c'est voulu pour que le serveur vérificateur puisse la consulter sans appeler personne.
**Pourquoi :** signature ne veut pas dire chiffrement — un JWT répond à « d'où vient ce jeton et a-t-il été trafiqué ? », jamais à « qui l'a lu ? ».

**2. B.** Faire confiance à l'en-tête du jeton, c'est laisser l'émetteur présumé choisir les règles : l'algorithme se décide côté serveur, en liste blanche.
**Pourquoi :** un serveur qui lit l'algorithme dans le jeton accepte n'importe quel contenu fabriqué à la main, signature comprise.

**3. C.** `aud` est le destinataire attendu : sans ce contrôle, un jeton émis pour un autre service est accepté ici.
**Pourquoi :** `iss` dit qui a émis, `aud` dit pour qui — les deux évitent le jeton « valable mais pas pour moi » ; `exp` et `nbf` ne portent que sur la période de validité.

**4. B.** Dans `localStorage`, le jeton est lisible par tout script injecté ; un cookie `HttpOnly` le cache au JavaScript, le choix se discutant selon l'architecture.
**Pourquoi :** le stockage fait partie de la surface d'attaque — un jeton volé par script est un jeton utilisable tel quel.

**5.** Attendu : signature valide avec algorithme en liste blanche (jeton fabriqué à la main) ; `exp` et `nbf` (jeton hors période, donc volé et exploitable indéfiniment) ; `iss` et `aud` (jeton d'un autre service ou d'un autre destinataire) ; périmètre ou rôle (confusion authentification / autorisation) ; révocation (jeton compromis), compensée par des durées courtes avec jeton de rafraîchissement ou par une liste noire.
**Pourquoi :** un JWT n'est pas sécurisé par nature — il l'est si et seulement si ces contrôles sont faits et la clé reste secrète.

</details>

---

[← Chapitre précédent](03-sessions.md) · [Sommaire](Readme.md) · [Chapitre suivant →](05-oauth2-oidc.md)
