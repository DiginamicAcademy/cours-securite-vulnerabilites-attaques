# Corrigé — Reconstituer les étapes d'une attaque

> Corrigé de [l'exercice « Reconstituer les étapes d'une attaque »](../07-reconstituer-attaque.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat.

---

## Partie 1 — Remettre la chronologie en ordre

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

## Partie 2 — Identifier le profil d'attaquant

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

## Partie 3 — Où la défense pouvait agir

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

## Points à retenir

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
