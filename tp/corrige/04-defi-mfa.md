# Grille de vérification — Politique MFA accessible et conforme

> Grille de vérification de [l'exercice « Proposer une politique MFA accessible et
> conforme »](../04-defi-mfa.md). Un défi n'a pas de correction pas-à-pas :
> confrontez votre proposition aux critères ci-dessous. Ils couvrent la MFA
> (authentification multifacteur), l'accessibilité, le RGPD et la gestion
> d'incident. Plusieurs politiques se
> défendent — ce qui est attendu, c'est que chaque choix soit **dit, justifié et
> applicable**.

---

## Partie 1 — La politique

**Critères de vérification :**

- [ ] Le tableau couvre les **trois populations** du contexte, et distingue au
      moins un niveau de sensibilité (la comptabilité et le forum interne ne
      demandent pas le même facteur).
- [ ] Chaque ligne nomme un facteur imposé, un facteur de repli et un cas
      d'exception — une politique sans repli est incomplète, pas stricte.
- [ ] Les comptes de service reçoivent un **régime distinct** : pas de MFA (sans
      objet), mais une credential dédiée, à périmètre minimal et à rotation.
- [ ] La justification de chaque ligne renvoie à une **donnée exposée** ou à un
      usage, pas à une règle uniforme.

**Exemples de réponses acceptables :** application TOTP (Time-based One-Time
Password) imposée pour la
comptabilité avec clé matérielle en repli ; facteur unique renforcé pour le forum
interne ; clé de sécurité matérielle pour les administrateurs de la plateforme.

**Écueils fréquents :** « MFA partout, sans exception » — cela viole la
contrainte d'accessibilité et la proportionnalité. Ou un tableau qui ne parle que
des salariés, oubliant les prestataires et les pipelines.

---

## Partie 2 — L'accessibilité

**Critères de vérification :**

- [ ] Le parcours de la personne **malvoyante** est décrit pas à pas, et
      fonctionne avec ses outils d'assistance (lecteur d'écran, contraste,
      saisie vocale) — un code à usage unique affiché en image ne passe pas.
- [ ] Le parcours de la personne **sans téléphone personnel** ne repose pas sur
      un appareil qui ne lui appartient pas : clé matérielle fournie par
      l'organisme, code à usage unique imprimé, ou facteur alternatif.
- [ ] Aucun parcours ne rend la personne **dépendante d'un collègue** pour se
      connecter (« demandez à quelqu'un de vous lire le code » est un échec).
- [ ] Le repli est **disponible au quotidien**, pas seulement en dépannage : une
      solution de secours jamais testée ne fonctionne pas le jour de l'incident.

**Exemples de réponses acceptables :** une clé de sécurité matérielle détectée
par le lecteur d'écran, validée par un simple toucher ; une liste de codes à usage
unique, imprimée en gros caractères, renouvelée sur demande.

**Écueils fréquents :** proposer la biométrie comme solution universelle
d'accessibilité — elle pose d'autres problèmes (section suivante), et une
empreinte ne se révoque pas.

---

## Partie 3 — Le RGPD

**Critères de vérification :**

- [ ] Les données en jeu sont **identifiées** : numéro de téléphone — SMS (Short
      Message Service), donnée biométrique, journal de connexion, adresse IP.
- [ ] Chaque donnée est accompagnée de sa **finalité**, de sa **durée de
      conservation** et de sa **minimisation** — on ne collecte pas « au cas où ».
- [ ] La biométrie est traitée comme une **donnée sensible** : refusée quand un
      facteur moins intrusif suffit, acceptée seulement quand elle est stockée
      **sur l'appareil de la personne** (et non dans une base centrale).
- [ ] Le refus d'un facteur pour raison d'accessibilité n'entraîne **aucune
      trace stigmatisante** dans le dossier de la personne.

**Exemples de réponses acceptables :** « la biométrie reste sur le téléphone de
la personne, le serveur ne reçoit qu'une preuve ; nous ne conservons aucune
empreinte ; alternative disponible pour qui ne peut ou ne veut pas l'utiliser ».

**Écueils fréquents :** une base centrale d'empreintes « pour simplifier les
reconnexions » — c'est exactement la donnée qu'il ne faut pas centraliser. Ou une
durée de conservation indéfinie par défaut.

---

## Partie 4 — L'incident

**Critères de vérification :**

- [ ] La procédure nomme **qui agit** (la personne, le service desk, l'équipe
      sécurité) et dans quel ordre.
- [ ] Le compte est **suspendu ou son facteur révoqué** dès la déclaration —
      jamais laissé ouvert « le temps de vérifier ».
- [ ] Les **sessions déjà ouvertes** sont invalidées : un facteur perdu ne coupe
      pas une session volée avec lui.
- [ ] La promesse des **moins de 4 heures** est tenue par un mécanisme, pas par
      un engagement moral : permanence, procédure automatisée, ou délégation
      préparée.
- [ ] Le cas des **comptes de service** liés à l'appareil perdu est traité :
      rotation de la credential concernée, pas seulement le compte humain.

**Exemples de réponses acceptables :** déclaration en libre-service qui suspend
immédiatement le facteur, réactivation sur un nouvel appareil après vérification
d'identité en visio ou en présentiel, journal de l'incident conservé pour
l'audit.

**Écueils fréquents :** une procédure qui exige le téléphone perdu pour se
déclarer — ou qui repose sur une seule personne « qui a les droits ».

---

## Synthèse

Une politique MFA se juge à trois questions :

1. **Est-elle tenable au quotidien** par toutes les personnes concernées, y
   compris celles qui ne correspondent pas au profil moyen ?
2. **Est-elle proportionnée** aux données exposées, et le dit-elle ?
3. **Se termine-t-elle proprement** — incident, départ, perte de facteur — sans
   laisser de session ni de credential orpheline ?

Si une question reste sans réponse dans votre proposition, c'est le point à
reprendre.
