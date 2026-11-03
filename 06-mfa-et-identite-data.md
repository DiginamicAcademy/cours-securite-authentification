[← Chapitre précédent](05-oauth2-oidc.md) · [Sommaire](Readme.md)

# MFA et identité data — humains et machines

> Une administratrice de plateforme de données se connecte à l'entrepôt avec un
> mot de passe unique. Son pipeline d'automatisation, lui, utilise un compte de
> service avec une clé stockée dans un fichier de configuration. Lequel des deux
> est le plus dangereux ? La réponse dérange : le pipeline, lui, ne se fait pas
> hameçonner.

**Durée de lecture :** 50 min · **Prérequis :** [Authentifier n'est pas autoriser](01-authentifier-nest-pas-autoriser.md)

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Expliquer** ce que la MFA (authentification multifacteur) change — et ce qu'elle ne change pas.
- **Arbitrer** les facteurs d'authentification selon la population et la sensibilité des données, en tenant compte de l'accessibilité.
- **Distinguer** une identité humaine d'une identité de service, et leurs règles respectives.
- **Encadrer** un compte de service : périmètre minimal, stockage des secrets, rotation, journalisation.

---

## I. La MFA (authentification multifacteur)

### Ce que ça change

Le chapitre [Authentifier n'est pas autoriser](01-authentifier-nest-pas-autoriser.md)
a classé les preuves d'identité en trois familles : ce qu'on **sait**, ce qu'on
**a**, ce qu'on **est**. La **MFA** (authentification multifacteur) exige des
preuves de **familles différentes** — un mot de passe et un code sur le téléphone,
pas deux mots de passe.

Effet : le vol d'une preuve ne suffit plus. Un mot de passe fuité reste inutilisable
sans le second facteur ; un téléphone volé ne se combine pas avec un mot de passe
inconnu.

| Second facteur | Famille | Ce qu'il apporte | Sa faiblesse |
| --- | --- | --- | --- |
| Code par SMS (Short Message Service) | ce qu'on a | universel, aucun matériel | interception du message, échange de carte SIM (Subscriber Identity Module) |
| Application TOTP (Time-based One-Time Password) | ce qu'on a | codes à durée courte, hors réseau | téléphone perdu ou volé |
| Notification push | ce qu'on a | validation en un geste | « fatigue de validation » : on approuve sans lire |
| Clé de sécurité matérielle | ce qu'on a | résiste à l'hameçonnage (la clé vérifie le site) | coût, perte de la clé |
| Biométrie | ce qu'on est | ne se prête pas | ne se révoque pas : une empreinte fuitée l'est pour toujours |

### Ce que ça ne change pas

La MFA n'est pas un rempart universel. Elle ne protège pas contre une session
déjà ouverte (chapitre [Sessions](03-sessions.md)), contre un malware installé sur
le poste, ni contre les relais d'hameçonnage en temps réel qui transmettent le
code à l'instant où vous le tapez. Et le NIST (National Institute of Standards and
Technology) considère le SMS comme un facteur **restreint** : à réserver quand les
autres ne sont pas possibles.

### Le coût de la friction

Chaque facteur ajouté est une étape de plus à la connexion — et un motif de
contournement. La règle de proportionnalité du cours s'applique : la MFA se
décide selon les **données exposées**, pas par uniformité. Une console
d'administration d'entrepôt de données, oui ; un forum de discussion interne,
probablement pas.

> 🧪 **Atelier rapide (5 min)**
>
> Pour chacun de ces trois scénarios, notez « MFA pertinente » ou « MFA
> excessive », et une phrase de justification :
>
> 1. la comptabilité en ligne d'une PME (petite et moyenne entreprise) —
>    salaires, relevés bancaires ;
> 2. le blog de l'entreprise (articles publics, aucun compte lecteur) ;
> 3. la console d'administration de l'entrepôt de données (accès à toutes les
>    tables, y compris les données personnelles).
>
> Comparez vos réponses à ce critère : *ce que l'attaquant obtient s'il réussit*.

### L'accessibilité, contrainte de conception

Une politique MFA qui suppose « un smartphone à portée de main » exclut des
personnes — employées malvoyantes, sans téléphone personnel, en situation de
handicap — PSH (personnes en situation de handicap). Une politique réelle prévoit
**un facteur de repli** (clé matérielle, code imprimé à usage unique, assistance
du service desk avec procédure de vérification) et documente la procédure
d'exception. C'est le sujet de l'exercice de fin de journée.

---

## II. Passkeys — la fin du mot de passe

Une **passkey** remplace le couple identifiant / mot de passe par une paire de
clés cryptographiques :

- la clé privée reste sur l'appareil (téléphone, ordinateur, clé matérielle) et
  ne sort jamais ;
- la clé publique est enregistrée par le service ;
- la déverrouillage de l'appareil (empreinte, code) débloque la signature.

Conséquences heureuses : rien à mémoriser, rien à deviner, et surtout une
propriété que les mots de passe n'ont pas — la passkey est **liée au site**. Un
faux site d'hameçonnage ne reçoit rien : l'appareil refuse de signer pour un
domaine qui n'est pas celui enregistré.

| | Mot de passe | Passkey |
| --- | --- | --- |
| Secret partagé avec le serveur | oui | non (clé publique seulement) |
| Résistance à l'hameçonnage | non | oui (origine vérifiée par l'appareil) |
| Réutilisation entre sites | fréquente | impossible par construction |
| Perte de l'appareil | sans conséquence | procédure de récupération nécessaire |

La limite se déplace : la **récupération de compte** devient le point sensible —
un appareil perdu ne se remplace pas par « j'oublie mon mot de passe ». Il faut
un second appareil enregistré, ou une procédure de vérification d'identité.

---

## III. L'identité des pipelines

Un pipeline d'automatisation — un DAG (Directed Acyclic Graph) Airflow qui charge
les ventes du jour, un job Spark qui agrège les logs — n'a personne derrière lui.
Pas de MFA possible, pas d'hameçonnage non plus : son identité **est** sa
credential. D'où des règles différentes.

### Les règles d'un compte de service

- **Un compte par usage** : le pipeline de chargement des ventes n'est pas celui
  de la synchronisation des référentiels. Un incident se limite au périmètre du
  compte.
- **Le moindre privilège** : écrire dans une table, pas administrer la base. Le
  compte du chargement quotidien a le droit d'insérer dans `ventes_journalieres`,
  pas de lire la table des salaires.
- **Les secrets hors du code** : une clé dans un dépôt Git l'est pour toujours —
  l'historique ne pardonne pas. Les secrets vivent dans un coffre (vault), injectés
  à l'exécution.
- **La rotation** : une clé se change périodiquement et **immédiatement** en cas
  de suspicion — d'où l'intérêt de clés à durée de vie courte, renouvelées
  automatiquement.
- **La journalisation** : chaque appel du compte est tracé. Un compte de service
  qui se met à lister des tables inconnues est un signal, pas un détail.

> ⚠️ **Attention**
> Un compte de service doté des droits d'administrateur « pour ne pas être bloqué »
> est la porte la plus large de l'infrastructure : personne ne surveille un
> pipeline comme on surveille une personne.

### Humains et machines, deux régimes

| | Identité humaine | Identité de service |
| --- | --- | --- |
| Preuve | ce qu'on sait, ce qu'on a, ce qu'on est | une credential (clé, certificat) |
| MFA | oui, selon la sensibilité | sans objet — la credential est unique |
| Durée de vie | session courte, reconnexion | clé longue ou renouvelée automatiquement |
| Révocation | désactiver le compte | supprimer ou faire tourner la clé |
| Signal d'alerte | connexion inhabituelle | appel hors périmètre, volume anormal |

Les **clés d'API** (API keys) appartiennent à ce second régime : elles identifient
un programme appelant, pas une personne. Le chapitre
[OAuth2 et OpenID Connect](05-oauth2-oidc.md) détaille quand les préférer à un
jeton délégué.

---

## IV. Aller plus loin

*Ces approfondissements sont facultatifs : ils ne sont jamais exigés pour terminer
la journée.*

- **FIDO2** (Fast Identity Online 2) et **WebAuthn** (Web Authentication) : les
  standards derrière les passkeys et les clés de sécurité. Le navigateur parle à
  l'appareil d'authentification, le site ne voit qu'une signature — le détail des
  échanges explique pourquoi l'origine est vérifiée à chaque tentative.
- **L'attestation** : une entreprise peut exiger que les passkeys de ses employés
  proviennent de matériel dont elle contrôle la provenance — un sujet de gestion
  de parc plus que de sécurité applicative.
- **Zero Trust** appliqué aux machines : un pipeline authentifié n'est pas
  « de confiance » pour autant — chaque appel reste vérifié, borné et journalisé.

---

## Points clés

- La MFA combine des preuves de **familles différentes** : elle annule le vol
  d'une seule preuve, pas une session déjà ouverte ni un poste compromis.
- Le facteur se choisit selon les **données exposées**, le coût de friction et
  l'**accessibilité** — une politique sans facteur de repli exclut des personnes.
- Les passkeys suppriment le secret partagé et résistent à l'hameçonnage ; la
  difficulté se déplace vers la **récupération de compte**.
- Un compte de service n'a pas de MFA : sa protection tient au **moindre
  privilège**, aux secrets hors du code, à la **rotation** et à la **journalisation**.

---

[← Chapitre précédent](05-oauth2-oidc.md) · [Sommaire](Readme.md)
