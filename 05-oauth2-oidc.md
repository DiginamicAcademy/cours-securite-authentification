[← Chapitre précédent](04-jwt-et-tokens.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-mfa-et-identite-data.md)

# OAuth2 et OpenID Connect — donner un accès limité

> Avant OAuth, une application qui voulait accéder à vos photos vous demandait
> votre **mot de passe** — et se retrouvait avec tous vos droits, sans que vous
> puissiez rien lui retirer. OAuth2 (Open Authorization 2) inverse la logique : on
> ne partage pas son secret, on accorde un **accès limité** dans le temps et dans
> le périmètre. OpenID Connect ajoute par-dessus la réponse à « qui êtes-vous ? ».

**Durée de lecture :** 70 min · **Prérequis :** [JWT et tokens](04-jwt-et-tokens.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Expliquer** le problème que résout OAuth2 : partage de mot de passe, permissions excessives, absence de révocation.
- **Décrire** le flow authorization code : acteurs, consentement, code, jeton d'accès.
- **Distinguer** l'autorisation (OAuth2) de l'authentification (OpenID Connect).
- **Choisir** entre clé d'API et jeton délégué selon le consommateur : humain, application, pipeline.

---

## I. Le problème

Une plateforme de business intelligence veut afficher les ventes d'un entrepôt de
données dans son tableau de bord. Solutions d'avant OAuth :

1. **Donner le mot de passe** de la base ou du compte. L'outil peut alors tout
   faire — y compris ce que personne n'a demandé. Et si le mot de passe change,
   tout casse.
2. **Ouvrir un compte dédié** avec un mot de passe partagé. Qui l'utilise
   réellement ? Impossible de le savoir, et la suppression du compte casse tout le
   monde.
3. **Copier les données** dans l'outil. La donnée se retrouve en double, hors de
   votre contrôle.

Le point commun : on confie un **secret global** là où on voudrait accorder un
**droit limité**. OAuth2 sépare les deux rôles — un serveur d'autorisation décide
qui a droit à quoi, l'API se contente de vérifier le jeton.

---

## II. Le flow authorization code

### Les quatre acteurs

| Acteur | Rôle | Exemple |
| --- | --- | --- |
| Propriétaire de la ressource | la personne qui accorde | la responsable des ventes |
| Client | l'application qui demande | le tableau de bord |
| Serveur d'autorisation | il distribue les jetons | le service d'identité de l'entreprise |
| Serveur de ressources | l'API protégée | l'API de l'entrepôt de données |

### Le déroulement

```text
 navigateur            client (tableau de bord)     serveur d'autorisation      API
     │  1. « afficher mes ventes » │                       │                      │
     │────────────────────────────►│  2. redirection vers la demande de droits   │
     │◄────────────────────────────│──────────────────────►│                      │
     │  3. consentement : « lire les ventes »              │                      │
     │────────────────────────────►│  4. code d'autorisation│                     │
     │                             │◄──────────────────────│                      │
     │                             │  5. code + preuve du client                 │
     │                             │──────────────────────►│                      │
     │                             │  6. jeton d'accès (+ jeton de rafraîchissement)
     │                             │◄──────────────────────│                      │
     │                             │  7. appel API avec le jeton                 │
     │                             │─────────────────────────────────────────────►│
```

L'idée centrale : le **code** d'autorisation transite par le navigateur, mais le
**jeton** est échangé directement entre le client et le serveur d'autorisation,
avec une preuve d'identité du client. Un code intercepté ne suffit pas.

### Les périmètres (scopes)

Le jeton d'accès porte des **périmètres** — des mots qui décrivent ce qui est
autorisé : `lecture:ventes`, `ecriture:expeditions`. Le consentement les affiche,
l'API les vérifie à chaque appel. Un jeton `lecture:ventes` ne débloque jamais
`ecriture:expeditions`, même volé.

### Trois sortes de jetons

| Jeton | À quoi il sert | Durée de vie typique |
| --- | --- | --- |
| Jeton d'accès (access token) | appeler l'API | courte — minutes à heures |
| Jeton de rafraîchissement (refresh token) | obtenir un nouveau jeton d'accès | plus longue, révocable |
| Jeton d'identité (ID token) | dire **qui** est l'utilisateur | courte — voir section III |

### Clé d'API ou jeton délégué ?

| | Clé d'API (API key) | Jeton délégué (OAuth2) |
| --- | --- | --- |
| Elle identifie | le programme appelant | la personne **et** le programme |
| Elle porte des droits | fixes, attachés à la clé | des périmètres, consentis |
| Elle se révoque | en supprimant la clé | jeton par jeton, sans casser le reste |
| Usage idéal | machine à machine, partenaire technique | accès délégué au nom d'un utilisateur |

Une clé d'API n'est **pas** un moyen d'authentifier une personne : elle prouve
qu'un programme est autorisé à appeler, pas qui est derrière.

---

## III. OpenID Connect — l'identité par-dessus l'autorisation

OAuth2 répond à « qu'est-ce que cette application a le droit de faire ? ». Il ne
répond pas à « qui est l'utilisateur ? » — et beaucoup d'applications ont essayé
de le deviner à partir du jeton d'accès. OIDC (OpenID Connect) formalise la
réponse : c'est une couche **au-dessus** d'OAuth2, pas un concurrent.

- Le client demande le périmètre `openid` ; en plus du jeton d'accès, il reçoit
  un **jeton d'identité** (ID token) — un JWT (JSON Web Token) signé.
- Ce jeton porte des déclarations : `sub` (l'identifiant de l'utilisateur),
  `email`, `iss` (l'émetteur), `aud` (le client destinataire), `exp`.
- C'est le mécanisme derrière les boutons « se connecter avec… » : votre
  fournisseur d'identité atteste de qui vous êtes, l'application n'a jamais vu
  votre mot de passe.

> ⚠️ **Attention**
> Un ID token n'est pas un jeton d'accès : il est fait pour être **lu par le
> client**, pas présenté à une API. Et comme tout JWT, il se vérifie — signature,
> `iss`, `aud`, `exp` — avant de faire confiance à son contenu.

### Le cas du tableau de bord

Pour notre plateforme de business intelligence, la séparation devient nette :

- les **personnes** se connectent via OIDC avec l'identité de leur entreprise —
  le tableau de bord sait qui consulte quoi, et le départ d'une personne coupe
  son accès au même moment ;
- le **tableau de bord** appelle l'API avec un jeton d'accès à périmètre
  `lecture:ventes` ;
- l'**API** ne vérifie ni mot de passe ni identité : elle vérifie le jeton et ses
  périmètres.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **PKCE** (Proof Key for Code Exchange) : les applications mobiles et les
  applications web riches ne peuvent pas garder de secret client. PKCE remplace le
  secret par une paire de valeurs aléatoires calculées à chaque connexion, ce qui
  neutralise l'interception du code d'autorisation. C'est aujourd'hui recommandé
  pour tous les clients, même ceux qui ont un secret.
- **La rotation des jetons de rafraîchissement** : chaque usage en délivre un
  nouveau et invalide l'ancien. Un jeton de rafraîchissement volé devient alors
  détectable — deux utilisations simultanées trahissent le vol.
- **Les jetons dans les URL** : jamais. Un jeton transmis en paramètre d'adresse
  se retrouve dans les journaux, l'historique et les en-têtes `Referer`. Le jeton
  voyage dans un en-tête `Authorization`, sur HTTPS.

---

## Points clés

- OAuth2 sépare le **droit** (ce que l'application peut faire) du **secret** (votre
  mot de passe) : on accorde un accès limité, révocable, sans rien partager.
- Le flow authorization code fait transiter un **code** par le navigateur, et
  échange le **jeton** directement entre le client et le serveur d'autorisation.
- Les **périmètres** bornent le jeton ; l'API les vérifie à chaque appel — un
  jeton volé reste limité à ce qui a été consenti.
- OpenID Connect ajoute l'**identité** (ID token) à l'autorisation ; une clé
  d'API, elle, identifie un programme — jamais une personne.

---

[← Chapitre précédent](04-jwt-et-tokens.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-mfa-et-identite-data.md)
