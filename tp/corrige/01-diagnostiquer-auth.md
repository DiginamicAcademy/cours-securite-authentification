# Corrigé — Diagnostiquer l'authentification d'une application

> Corrigé de [l'exercice « Diagnostiquer l'authentification d'une application »](../01-diagnostiquer-auth.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

## Partie 1 — Distinguer authentification, autorisation et session

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

## Partie 2 — Repérer les failles

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

## Partie 3 — Classer les vecteurs d'identité

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

## Partie 4 — Proposer les corrections

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

## Points à retenir

- Authentification et autorisation sont deux contrôles distincts : le second ne se
  déduit jamais du premier.
- Une donnée protégée par la connexion ne l'est pas pour autant : vérifiez **à qui
  elle appartient**.
- Les secrets (mots de passe, jetons) se protègent **au repos** (hachage), **en
  transit** (chiffrement) et **dans le temps** (expiration, invalidation).
- Un classement de priorités n'a de valeur que si son **critère est explicite**.
