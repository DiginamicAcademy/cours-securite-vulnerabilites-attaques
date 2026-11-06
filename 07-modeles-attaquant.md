[← Chapitre précédent](06-lecture-de-code-vulnerable.md) · [Sommaire](Readme.md) · [Chapitre suivant →](08-kill-chain.md)

# Modèles d'attaquant — Qui attaque, et avec quoi

> Un ancien employé mécontent, un adolescent qui lance un outil trouvé en ligne,
> une cellule financée par un État. Trois attaquants, trois budgets, trois
> objectifs. Devant la même application, ils ne passeront pas par la même porte —
> et la barrière qui les arrête tous les trois d'un seul geste n'existe pas.
> Savoir qui vous vise, c'est déjà savoir quoi construire en premier.

**Durée de lecture :** 35 min (noyau exigé, atelier compris) · **Aller plus
loin :** +15 min (facultatif, pas exigé pour finir la journée) · **Prérequis :**
la journée « Vulnérabilités applicatives »

---

## Objectifs

À l'issue de ce chapitre, vous serez capable de :

- **Distinguer** les grands profils d'attaquants et leurs motivations.
- **Relier** un profil à ses moyens : temps, outils, argent, connaissance du système.
- **Évaluer** la menace la plus probable pour une application donnée.
- **Choisir** des mesures de défense proportionnées à cette menace.

---

## I. Qui attaque ?

Trois histoires, tirées de ce que les équipes de sécurité rencontrent chaque
année.

Un ancien administrateur système, licencié par une PME (petite et moyenne entreprise) de logistique, conserve
un accès qu'on a oublié de lui retirer. Il ne casse rien : il consulte les
fiches de paie, puis propose ses services à un concurrent. Un adolescent, de
son côté, télécharge un outil de scan publié sur un forum et le lance contre
des centaines de sites, sans viser personne en particulier — il cherche une
capture d'écran à publier. Enfin, une cellule financée par un État passe des
mois sur une seule cible : un laboratoire de recherche, pour ses données
d'essais.

Ces trois personnes n'ont ni les mêmes motivations, ni les mêmes moyens, ni la
même patience. Les regrouper sous le seul mot « pirate » empêche de choisir une
défense : c'est comme soigner tous les patients avec le même médicament.

### Les motivations

Une attaque commence presque toujours par une motivation. Les plus fréquentes :

| Motivation | Ce que cherche l'attaquant | Exemple typique |
| --- | --- | --- |
| **Argent** | rançon, revente de données, fraude | un hôpital paralysé par un rançongiciel |
| **Revanche** | nuire à une personne ou à une organisation | un employé parti en mauvais termes |
| **Espionnage** | des informations stratégiques | des plans industriels, des données de recherche |
| **Gloire ou curiosité** | la prouesse, la reconnaissance des pairs | un site défait « pour montrer que c'est possible » |
| **Idéologie** | faire plier un discours, punir une position | un site institutionnel défiguré |

La motivation décide de la **cible** et de la **persévérance**. Un attaquant
motivé par l'argent cherche le chemin le plus rentable : si votre application
est trop difficile à percer, il passe à la suivante. Un attaquant motivé par
l'espionnage, lui, accepte de revenir pendant des mois.

### Les profils

À chaque motivation correspondent des profils récurrents :

- **l'opportuniste automatisé** : il ne vous connaît pas. Un script scanne des
  milliers d'applications à la recherche d'une version obsolète ou d'un mot de
  passe par défaut. Vous n'êtes pas visé : vous êtes dans la liste ;
- **l'initié** (*insider*) : il a déjà un accès légitime — employé, prestataire,
  stagiaire. Son avantage est de connaître le système ; son point faible est
  qu'il laisse des traces dans les journaux comme les autres ;
- **le criminel organisé** : une équipe structurée, qui achète des accès, loue
  des outils, revend les données. Elle travaille par lots et automatise ce
  qu'elle peut ;
- **l'espion industriel ou étatique** : un budget important, une patience
  importante, une cible précise. C'est le profil le plus rare et le plus
  difficile à arrêter ;
- **le militant** (*hacktiviste*) : il agit par conviction, souvent de manière
  visible, pour que l'attaque soit **vue**.

> [!TIP]
> La question utile n'est pas « qui pourrait m'attaquer ? » mais « qui a
> intérêt à m'attaquer, et avec quel budget ? ». Une application de gestion de
> stock de fromagerie n'attire pas la même patience qu'un laboratoire de
> recherche — mais elle reste dans la liste de balayage de l'opportuniste.

> 🧪 **Atelier rapide (5 min)**
>
> Associez chaque scénario au profil le plus probable, et dites en une phrase
> quelle motivation le pousse :
>
> 1. Un script parcourt toute la nuit les boutiques en ligne pour trouver une version de bibliothèque obsolète, sans viser personne.
> 2. Un ancien administrateur, parti depuis un mois, utilise encore son compte pour consulter les fiches de paie.
> 3. Un concurrent veut les tarifs fournisseurs d'un fabricant de composants.
> 4. Un adolescent publie sur un forum la capture d'un site qu'il a défait pour s'amuser.

---

## II. Les moyens

Derrière la motivation, il y a des **moyens**, et ils se mesurent simplement :
du temps, de l'argent, des outils, et une connaissance plus ou moins fine de
votre système d'information (SI).

| Profil | Temps | Outils | Connaissance du SI | Défense proportionnée |
| --- | --- | --- | --- | --- |
| Opportuniste automatisé | minutes | scripts prêts à l'emploi | nulle : il balaye | mises à jour, mots de passe forts, pas de réglage par défaut |
| Initié | semaines | ce qu'il a déjà | très bonne | moindre privilège, journalisation, revue des accès au départ |
| Criminel organisé | jours à semaines | achetés ou loués | moyenne, acquise sur place | segmentation, détection des mouvements latéraux, sauvegardes |
| Espion industriel ou étatique | mois | sur mesure | ciblée, préparée | défense en profondeur, surveillance, cloisonnement des données sensibles |

Deux enseignements se dégagent de ce tableau. **La connaissance du SI vaut plus
cher que l'outil** : un script acheté en ligne ne dit pas où sont vos données
sensibles, un employé, lui, le sait — d'où la revue des accès au départ, une
des mesures les plus rentables qui existent. Et **la menace probable n'est pas
la menace possible** : il est théoriquement possible qu'un État vise votre
petite structure, il est beaucoup plus probable qu'un script automatisé trouve
votre page d'administration oubliée. On construit sa défense contre la menace
probable, et on garde un œil sur les autres — c'est le principe de la sécurité
proportionnée.

> [!WARNING]
> Une erreur fréquente : calquer sa défense sur la menace la plus spectaculaire
> (l'espionnage étatique) et négliger la plus courante (le script automatisé).
> Les statistiques d'incidents sont dominées par les attaques de masse : c'est
> là que le retour sur investissement défensif est le plus élevé.

### Exercice 1 — Reconstituer les étapes d'une attaque (25 min)

#### Contexte

L'éditeur **Meridian Software** publie une application de gestion de stock pour
les commerces de proximité. Un matin, l'équipe découvre que l'historique de
commandes de ses clients a été mis en vente sur un forum. Personne n'a vu
l'attaque se dérouler.

L'enquête reconstitue les gestes suivants — mais ils vous parviennent dans le
désordre :

| Lettre | Geste observé |
| --- | --- |
| **A** | Il copie les adresses de courriel et les historiques de commande. |
| **B** | Il télécharge un outil de scan et l'exécute contre des milliers de sites. |
| **C** | Il utilise un compte de service pour lire la base de commandes. |
| **D** | Il revend la liste sur un forum. |
| **E** | Il trouve une page d'administration oubliée, accessible sans mot de passe. |
| **F** | Il sort ces données par un transfert déguisé en sauvegarde. |
| **G** | Il dépose sur cette page un script qui lui donnera un accès permanent. |
| **H** | Il relève les comptes utilisateurs listés sur cette page. |

---

#### Consignes

##### Partie 1 — Remettre la chronologie en ordre

Classez les lettres dans l'ordre chronologique plausible. Pour chaque étape,
ajoutez le nom de l'étape de la kill chain qu'elle correspond (reconnaissance,
accès initial, escalade de privilèges, mouvement latéral, persistance, actions
sur les objectifs, exfiltration).

##### Partie 2 — Identifier le profil d'attaquant

Quel profil parmi ceux du cours décrit le mieux cette attaque ? Justifiez votre
réponse en vous appuyant sur :

- la **motivation** apparente ;
- les **moyens** observés (temps, outils, connaissance du système).

Dites ensuite quelle conséquence ce profil a sur la défense à construire en
priorité.

##### Partie 3 — Où la défense pouvait agir

Pour les étapes d'**accès initial**, de **persistance** et d'**exfiltration**,
proposez la mesure qui les aurait coupées ou rendues visibles. Une phrase par
mesure, formulée comme une action.

---

#### ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Mon ordre chronologique est **complet** : chaque lettre apparaît une fois.
- [ ] Chaque étape porte le **nom de la kill chain** correspondant.
- [ ] Mon profil d'attaquant est justifié par la motivation **et** par les moyens.
- [ ] J'ai dit ce que ce profil change pour la **défense à construire en
      priorité**.
- [ ] Mes mesures sont formulées comme des **actions**, pas comme des souhaits.

---

#### Pour aller plus loin

*Facultatif.* Parmi les étapes de cette attaque, laquelle aurait été la plus
simple à **détecter** avec des journaux, et pourquoi ? Répondez en deux lignes.

<details>
<summary>Voir la correction</summary>

> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte autant que le résultat.

#### Partie 1 — Remettre la chronologie en ordre

**Éléments attendus :**

| Ordre | Lettre | Étape de la kill chain | Justification |
| --- | --- | --- | --- |
| 1 | **B** | reconnaissance | un scan de masse ne vise personne : il cherche ce qui est exposé |
| 2 | **E** | accès initial | la page d'administration oubliée est la porte trouvée par le scan |
| 3 | **H** | reconnaissance interne | les comptes listés disent ce qui est utilisable dans le système |
| 4 | **G** | persistance | le script déposé permet de revenir sans repasser par la porte d'entrée |
| 5 | **C** | escalade de privilèges | le compte de service ouvre des données qu'un compte ordinaire ne lit pas |
| 6 | **A** | actions sur les objectifs | la copie des données est le but recherché |
| 7 | **F** | exfiltration | le transfert déguisé fait sortir les données sans éveiller l'attention |
| 8 | **D** | usage des données | la revente est la finalité économique de l'attaque |

**Ce qu'il faut retenir :** l'ordre se justifie par les **dépendances** entre les
étapes — chaque geste fournit ce dont le suivant a besoin. Une chronologie qui
ne dit pas ces dépendances reste une liste ; c'est le lien entre les étapes qui
fait la chaîne.

**Erreurs fréquentes :** placer la persistance en fin d'attaque. Un attaquant
installe son accès permanent **tôt**, précisément pour ne pas dépendre de la
porte d'entrée qui pourrait être refermée à tout moment.

---

#### Partie 2 — Identifier le profil d'attaquant

**Éléments attendus :**

Le profil est celui de **l'opportuniste automatisé**, devenu criminel dans
l'usage de ce qu'il trouve.

- **Motivation** : l'argent — la revente de la liste sur un forum est la
  finalité, et non la prouesse ni la revanche.
- **Moyens** : un outil de scan prêt à l'emploi, lancé contre des milliers de
  sites, sans connaissance préalable de Meridian Software. Le temps passé par
  cible est faible : l'attaquant ne cherche pas une entreprise précise, il
  exploite ce qui est trouvé.

**Ce qu'il faut retenir :** la conséquence défensive est directe. Face à un
attaquant automatisé, la mesure la plus rentable est celle qui **retire la cible
de la liste** — fermer les pages et les services inutiles, corriger les
expositions banales. Ce n'est pas la défense la plus spectaculaire, mais c'est
celle qui traite le volume réel des attaques.

**Erreurs fréquentes :** conclure « attaque ciblée » parce que les données ont
été choisies avec soin. La **sélection** des données est opportuniste ; c'est la
**cible** qui ne l'était pas. Le distinguo change entièrement la défense à
prioriser.

---

#### Partie 3 — Où la défense pouvait agir

**Éléments attendus :**

| Étape | Mesure |
| --- | --- |
| Accès initial | retirer les pages d'administration inutiles, et protéger celles qui restent par une authentification vérifiée côté serveur |
| Persistance | journaliser et alerter sur tout dépôt de fichier ou création de tâche sur un serveur applicatif |
| Exfiltration | plafonner les volumes de transfert sortants et alerter sur un transfert qui sort de l'habitude |

**Ce qu'il faut retenir :** les trois mesures relèvent de registres différents —
la première est de la **prévention** (supprimer la porte), les deux autres de la
**détection** (voir le geste). Une attaque comme celle-ci se coupe rarement par
une seule barrière : elle se rend coûteuse en rendant chaque étape visible.

**Erreurs fréquentes :** proposer « un pare-feu » pour l'exfiltration. Un
pare-feu filtre des destinations, pas des volumes : le transfert déguisé en
sauvegarde sort par un canal légitime. C'est la comparaison à une habitude qui
le révèle.

---

#### Points à retenir

- Une attaque se reconstitue par les **dépendances** entre ses étapes : chaque
  geste fournit ce dont le suivant a besoin.
- La persistance s'installe **tôt** — un attaquant ne laisse pas sa rentrée
  dépendre d'une porte qui pourrait se fermer.
- Un scan de masse ne vise personne : la bonne défense est de **sortir de la
  liste**, pas de se prémunir d'une attaque sur mesure.
- Les mesures se répartissent entre **prévention** (supprimer la porte) et
  **détection** (voir le geste) — et les deux sont nécessaires.
- Ce qui se voit dans les journaux n'est pas l'attaque entière, mais ses gestes
  les plus anormaux : dépôt de fichier, changement de droits, transfert
  inhabituel.

</details>

---

## IV. Aller plus loin

*Contenu facultatif, pas exigé pour finir la journée : ces approfondissements
ne sont jamais exigés pour terminer le cours.*

- **L'ordre des priorités, concrètement** — pour une application de gestion
  interne d'une PME : 1. couvrir l'opportuniste (mises à jour, configuration,
  mots de passe), le lot le plus courant ; 2. limiter les dégâts de l'initié
  (moindre privilège, journaux) ; 3. prévoir la persistance du criminel
  organisé (sauvegardes testées, plan de réponse) ; 4. accepter qu'une menace
  étatique bien préparée ne se bloque pas à l'écran : on la **détecte** et on
  réduit ce qu'elle peut emporter.

- **Le modèle STRIDE** (Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege), développé par Microsoft : une manière systématique de
  parcourir les menaces d'une application en six catégories — usurpation
  d'identité (*spoofing*), altération de données (*tampering*), répudiation,
  divulgation d'information, déni de service, élévation de privilège. Chaque
  catégorie pose une question au concepteur : « quelqu'un peut-il se faire
  passer pour un autre ? », « quelqu'un peut-il modifier ce qu'il ne devrait
  pas ? ». C'est un outil d'**analyse de conception**, pas une liste de failles.
- **Les modèles de menace par secteur** : santé, banque, industrie publient des
  référentiels de menaces qui décrivent les attaquants observés dans leur
  domaine. Consulter celui de votre secteur est souvent plus utile qu'une
  classification générique.
- **La notion d'acteur de menace** (*threat actor*) : les éditeurs de solutions
  de sécurité publient des fiches par acteur — motivations, techniques
  observées, secteurs visés. Utile pour objectiver une discussion en comité.

---

## Points clés

- « Pirate » n'est pas un profil : **motivation, moyens et patience** changent
  tout, et donc la défense à construire.
- Les motivations courantes sont l'argent, la revanche, l'espionnage, la gloire
  et l'idéologie — chacune produit des attaques de forme différente.
- La **connaissance du système** est la ressource la plus précieuse pour un
  attaquant : c'est pourquoi les accès internes mal gérés coûtent cher.
- On construit sa défense contre la **menace probable** (souvent automatisée, de
  masse), pas contre la menace la plus spectaculaire.
- La sécurité est **proportionnée** : l'objectif est d'aligner les mesures sur
  le profil réellement visé, pas d'empiler des barrières au hasard.

---

## 🧠 Vérifiez vos acquis

*Formatif, jamais noté. Répondez sans relire le chapitre, puis dépliez « Vérifier ses réponses » pour vous corriger : ce qui compte est de repérer où vous vacillez, pas de faire 5 sur 5.*

**1.** Un ancien administrateur parti depuis un mois consulte encore les fiches de paie avec son compte, jamais révoqué. Quel profil, et quelles mesures proportionnées ?

- A. Opportuniste automatisé : mises à jour, mots de passe forts, aucun réglage par défaut
- B. Initié : moindre privilège, journalisation et revue des accès au départ
- C. Criminel organisé : segmentation, détection des mouvements latéraux, sauvegardes
- D. Espion industriel : défense en profondeur, surveillance, cloisonnement des données sensibles

**2.** « La connaissance du SI vaut plus cher que l'outil. » Que veut dire cette phrase ?

- A. Qu'un outil acheté en ligne coûte trop cher pour une petite structure, qui doit d'abord budgéter les salaires
- B. Qu'il faut investir dans de meilleurs outils avant de former les équipes, l'outil étant ce qui manque le plus
- C. Qu'un script ne sait pas où sont vos données sensibles, alors qu'un employé le sait : d'où la revue des accès au départ
- D. Qu'un initié n'utilise aucun outil, seulement ses accès : la connaissance du SI se passerait donc de technique

**3.** Une PME hésite entre se protéger contre un espionnage étatique bien préparé et corriger ses pages d'administration oubliées et ses comptes par défaut. Que retient-elle, selon le chapitre ?

- A. L'espionnage étatique : c'est le profil le plus difficile à arrêter, il doit donc passer avant les autres
- B. Les deux au même niveau : la menace possible compte autant que la menace probable, on ne choisit pas
- C. Les pages oubliées et les comptes par défaut : les statistiques d'incidents sont dominées par les attaques de masse automatisées
- D. Ni l'un ni l'autre : la probabilité ne change rien, seule la gravité technique de la faille décide

**4.** Un attaquant dont la motivation est l'argent se heurte à une application difficile à percer. Que fait-il, en général ?

- A. Il revient pendant des mois : la patience accompagne toutes les motivations
- B. Il passe à la suivante : il cherche le chemin le plus rentable
- C. Il publie la capture de sa tentative pour la reconnaissance
- D. Il bascule vers l'idéologie et défigure la page d'accueil

**5.** *Question ouverte (2-3 lignes).* Reprenez la question utile du chapitre — « qui a intérêt à m'attaquer, et avec quel budget ? » — pour la boutique en ligne de la fromagerie DIGICHEESE. Quel profil est le plus probable, et quelles mesures proportionnées en découlent ?

---

<details>
<summary>Vérifier ses réponses</summary>

### Réponses

**1. B.** L'initié n'a rien cassé : il disposait déjà d'un accès légitime, et son avantage est de connaître le système.
**Pourquoi :** son point faible est aussi le nôtre — il laisse des traces dans les journaux comme les autres, d'où journalisation, moindre privilège et revue des accès au départ, l'une des mesures les plus rentables qui existent.

**2. C.** Un script prêt à l'emploi ne dit pas où sont les données sensibles ; un employé, si.
**Pourquoi :** c'est la première des deux leçons du tableau des moyens — et la raison pour laquelle les accès internes mal gérés coûtent si cher.

**3. C.** On construit sa défense contre la menace **probable**, pas contre la plus spectaculaire ; on garde un œil sur les autres.
**Pourquoi :** c'est le principe de la sécurité proportionnée, et le retour sur investissement défensif est le plus élevé contre les attaques de masse.

**4. B.** La motivation décide de la cible et de la **persévérance** : un attaquant motivé par l'argent cherche le chemin le plus rentable.
**Pourquoi :** c'est ce qui le distingue de l'espion, capable d'accepter de revenir pendant des mois sur une cible précise.

**5.** Attendu : le profil le plus probable est l'opportuniste automatisé — la boutique n'est pas visée, elle est dans la liste de balayage ; mesures attendues : mises à jour, mots de passe forts, aucun réglage par défaut, page d'administration non exposée. Peut être complété par l'initié (prestataire, stagiaire) : moindre privilège et journaux.
**Pourquoi :** une application de gestion de stock de fromagerie n'attire pas la même patience qu'un laboratoire de recherche ; aligner les mesures sur le profil réellement visé, c'est la sécurité proportionnée.

</details>

---

[← Chapitre précédent](06-lecture-de-code-vulnerable.md) · [Sommaire](Readme.md) · [Chapitre suivant →](08-kill-chain.md)
