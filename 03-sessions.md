[← Chapitre précédent](02-mots-de-passe.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-jwt-et-tokens.md)

# Sessions — le badge de connexion

> Vous consultez votre compte bancaire en ligne. Entre la connexion et la
> dernière opération, le serveur ne vous a pas redemandé votre mot de passe à
> chaque page : il vous a remis un **badge** — la session. Ce badge vous suit
> partout, et s'il est volé, l'attaquant devient vous, sans jamais avoir connu
> votre secret.

**Durée :** ≈ 16 min — lecture ≈ 11 min + atelier 5 min · **Prérequis :** [Authentifier n'est pas autoriser](01-authentifier-nest-pas-autoriser.md)

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
> données. Dans ce lab, le jeton expire après **1 heure** — retrouvez ce chiffre
> dans le code du service `secure`.

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

Le lab fixe une seule limite, d'une heure. Une application bancaire combine les
deux ; un outil interne peut être plus laxiste. C'est un arbitrage de
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

[← Chapitre précédent](02-mots-de-passe.md) · [Sommaire](Readme.md) · [Chapitre suivant →](04-jwt-et-tokens.md)
