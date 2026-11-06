[← Chapitre précédent](04-jwt-et-tokens.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-mfa-et-identite-data.md)

# OAuth2 et OpenID Connect — donner un accès limité

> Avant OAuth, une application qui voulait accéder à vos photos vous demandait
> votre **mot de passe** — et se retrouvait avec tous vos droits, sans que vous
> puissiez rien lui retirer. OAuth2 (Open Authorization 2) inverse la logique : on
> ne partage pas son secret, on accorde un **accès limité** dans le temps et dans
> le périmètre. OpenID Connect ajoute par-dessus la réponse à « qui êtes-vous ? ».

**Durée :** ≈ 17 min — lecture ≈ 12 min + atelier 5 min · **Prérequis :** [JWT et tokens](04-jwt-et-tokens.md)

> 📖 Vocabulaire et acronymes : [glossaire du cours](A-Glossaire-et-ressources.md).

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

> 🧪 **Atelier rapide (5 min)**
>
> Reprenez le déroulement ci-dessus, étape par étape, et notez qui parle à qui —
> navigateur, client, serveur d'autorisation, API :
>
> 1. à quelle étape le mot de passe de la responsable des ventes est-il utilisé ?
> 2. un code d'autorisation intercepté par un tiers permet-il d'appeler l'API ?
>    Justifiez en une phrase.
> 3. que doit afficher l'écran de consentement pour que l'accord soit éclairé ?
>
> Comparez vos réponses à ce critère : *ce que chaque acteur apprend à chaque
> étape, et ce qu'il pourrait en faire*.

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

La ligne « machine à machine » du tableau a un nom dans OAuth2 : le flow
**client credentials** (identifiants du client). Le programme s'authentifie
*en son propre nom* — avec son `client_id` et son secret — et reçoit un jeton
d'accès de durée courte et de périmètre (scope) limité, sans consentement
humain. C'est l'alternative moderne à la clé d'API brute : même usage (pipeline,
service à service), mais le jeton est périssable et révocable. Les comptes de
service du chapitre 6 en appliquent les règles.

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
- **Les jetons dans les URL** : jamais — la règle est détaillée au chapitre [JWT et tokens](04-jwt-et-tokens.md).
  Le jeton voyage dans un en-tête `Authorization`, sur HTTPS.

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

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis cochez. Les réponses sont juste après le séparateur : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Ce qu'OAuth2 change par rapport à « donnez-moi votre mot de passe » :

- A. Il chiffre le mot de passe avant de le confier à l'application
- B. Il accorde un droit limité dans le temps et dans le périmètre, sans partager le secret, et il est révocable
- C. Il supprime les mots de passe
- D. Il centralise les mots de passe des applications dans un coffre commun

- [ ] ✓ je savais

**2.** Un tiers intercepte le code d'autorisation qui transite par le navigateur. Peut-il appeler l'API ?

- A. Oui : le code vaut jeton d'accès
- B. Non : le jeton est échangé directement entre le client et le serveur d'autorisation, avec une preuve d'identité du client
- C. Oui, mais seulement si le consentement portait un périmètre d'écriture
- D. Non : le code n'est jamais utilisable, pas même par le client légitime

- [ ] ✓ je savais

**3.** Un jeton d'accès portant le périmètre `lecture:ventes` est volé. Que peut faire l'attaquant ?

- A. Tout : un jeton volé donne le compte complet
- B. Rien : le vol invalide le jeton
- C. Lire les ventes uniquement — un jeton ne débloque jamais un périmètre qui n'a pas été consenti
- D. Écrire les expéditions, mais pas les supprimer

- [ ] ✓ je savais

**4.** Votre tableau de bord doit savoir **qui** consulte, en s'appuyant sur l'identité de l'entreprise, sans jamais voir les mots de passe. Qu'est-ce qui apporte cette réponse ?

- A. Un jeton d'accès OAuth2 avec un périmètre plus large
- B. Le code d'autorisation
- C. Le jeton d'identité (ID token) d'OpenID Connect, reçu avec le périmètre `openid`
- D. Une clé d'API nominative pour chaque utilisateur

- [ ] ✓ je savais

**5.** *Question ouverte (2-3 lignes).* Une équipe veut brancher son outil de business intelligence sur l'entrepôt de données et propose un compte commun `bi_tool` avec un mot de passe partagé. Ce qui ne va pas, ce que vous proposez à la place, qui vérifie quoi.

- [ ] ✓ je savais

---

### Réponses

**1. B.** On ne partage plus son secret : le serveur d'autorisation décide qui a droit à quoi, l'API se contente de vérifier le jeton.
**Pourquoi :** un secret global ne se retire pas sans tout casser, alors qu'un jeton se révoque jeton par jeton.

**2. B.** Le code transite par le navigateur, mais le jeton s'échange directement entre le client et le serveur d'autorisation, avec une preuve d'identité du client.
**Pourquoi :** c'est tout l'intérêt du flow authorization code — un code intercepté ne suffit pas à obtenir le jeton.

**3. C.** Les périmètres bornent le jeton et l'API les vérifie à chaque appel : `lecture:ventes` ne débloque jamais `ecriture:expeditions`, même volé.
**Pourquoi :** un jeton volé reste limité à ce qui a été consenti — c'est ce que le partage de mot de passe ne permettait pas.

**4. C.** OIDC est une couche au-dessus d'OAuth2 : avec le périmètre `openid`, le client reçoit un ID token, JWT signé qui dit qui est l'utilisateur.
**Pourquoi :** OAuth2 répond « qu'a le droit de faire cette application ? », pas « qui est l'utilisateur ? » — et une clé d'API identifie un programme, jamais une personne.

**5.** Attendu : un compte partagé est un secret global — impossible de savoir qui l'utilise réellement, les droits sont excessifs et la suppression du compte casse tout le monde. À la place : le flow authorization code avec un périmètre `lecture:ventes` pour l'outil (jeton court, révocable), et OIDC pour que les personnes se connectent avec l'identité de leur entreprise. Le serveur d'autorisation décide des droits, l'API ne vérifie ni mot de passe ni identité mais le jeton et ses périmètres.
**Pourquoi :** OAuth2 sépare le droit du secret — on retire un accès sans toucher au mot de passe de personne.

---

[← Chapitre précédent](04-jwt-et-tokens.md) · [Sommaire](Readme.md) · [Chapitre suivant →](06-mfa-et-identite-data.md)
