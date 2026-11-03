# Corrigé — Reconstituer la course patch / exploit

> Corrigé de [l'exercice « Reconstituer la course patch / exploit »](../09-timeline-cve.md).
> Comparez d'abord votre travail aux **étapes** ci-dessous : la méthode compte
> autant que le résultat. Les dates citées sont celles des journaux fournis dans
> l'énoncé, et les heures y sont exprimées en UTC (heure universelle coordonnée).

---

## Partie 1 — Reconstituer la course

**Éléments attendus :**

| Date | Volet attaquant | Volet défenseur |
| --- | --- | --- |
| 20 octobre 2026 | lecture de l'annonce, analyse des deux failles | lecture de l'alerte de veille à 14:00 UTC |
| 21 octobre 2026, 13:30 UTC | le correctif est lu comme une documentation de la faille | le correctif est disponible |
| 21 octobre 2026, fin de journée | des preuves de concept circulent publiquement | — |
| 22 octobre 2026 | exploitation possible à partir des preuves de concept | l'inventaire établit les serveurs concernés |
| 23 octobre 2026 | une troisième faille est annoncée sur le même produit | — |
| 24 octobre 2026 | — | le correctif est testé sur la recette |
| 28 octobre 2026 | — | le correctif est déployé en production |

**Ce qu'il faut retenir :** les deux volets ne se lisent pas l'un après l'autre.
Le fait marquant est le décalage entre le 21 octobre 2026 (les preuves de
concept sont publiques) et le 28 octobre 2026 (le correctif est en production) :
pendant cette période, l'attaquant dispose de tout ce qu'il faut et le
défenseur non. Une frise à un seul volet — celle du fournisseur — donne
l'impression que la course s'arrête au correctif : elle ne fait que commencer.

**Erreurs fréquentes :** placer la publication du correctif comme la fin de
l'incident. Le correctif est un **point de départ** pour le défenseur (tester,
déployer, vérifier) et une **documentation** pour l'attaquant.

---

## Partie 2 — Situer où l'équipe perd du temps

**Éléments attendus :**

| Période | Type d'étape | Ce qui aurait pu raccourcir l'étape |
| --- | --- | --- |
| du 20 au 22 octobre 2026 | **inventaire** | un inventaire des composants déjà à jour, qui répond « sommes-nous concernés ? » sans campagne de recherche |
| du 22 au 24 octobre 2026 | **test** | un environnement de recette maintenu proche de la production, et un scénario de test de correctif déjà écrit |
| du 24 au 28 octobre 2026 | **fenêtre de déploiement** | une procédure de déploiement d'urgence, décidée à l'avance, avec un responsable et un seuil de déclenchement |

**Ce qu'il faut retenir :** aucune de ces étapes n'est anormale — tester un
correctif avant de le mettre en production est un bon réflexe. Le problème est
qu'elles s'enchaînent **en série** alors que la menace, elle, avance en
parallèle. Les trois mesures proposées ont un point commun : elles se
préparent **pendant le calme**, pas pendant l'urgence. C'est le sens du temps de
réduction d'exposition — la durée qui compte est celle entre l'annonce et le
moment où vos systèmes ne sont plus exploitables.

**Erreurs fréquentes :** conclure « il faut supprimer les tests pour aller plus
vite ». Réduire la durée d'une étape, ce n'est pas la sauter : c'est l'avoir
préparée. Un correctif non testé qui arrête la plateforme de reporting coûte
souvent plus cher que la faille qu'il corrige.

---

## Partie 3 — Écrire un avis d'exposition

**Éléments attendus** (structure indicative — l'important est que les trois
questions soient traitées et que le distinguo savoir / supposer soit visible) :

- **Sommes-nous concernés ?** Oui : l'inventaire n'est pas encore fait à cette
  date, mais la plateforme de reporting repose sur PHP (moteur d'exécution des
  pages web dynamiques). Ce qui est **su** : le produit est utilisé. Ce qui est
  **supposé** : l'étendue exacte (quels serveurs, quelles versions) reste à
  établir — c'est la première action demandée.
- **Que faites-vous en attendant ?** Réduire l'exposition de ce qui n'a pas
  besoin d'être joignable, surveiller les journaux des serveurs concernés,
  préparer le correctif et son test pour le déployer dès que possible — sans
  attendre la prochaine fenêtre planifiée, puisque les preuves de concept
  circulent.
- **Que changez-vous pour la suite ?** Tenir un inventaire des composants,
  écrire la procédure de déploiement d'urgence, et prévoir un retour
  d'expérience après chaque correctif urgent.

**Ce qu'il faut retenir :** un avis d'exposition utile n'est pas un résumé de
l'advisory — c'est une **décision** argumentée. Il distingue ce qui est établi
de ce qui est supposé, il nomme les actions en cours, et il dit ce qui changera
pour que la prochaine annonce soit moins coûteuse à traiter.

**Erreurs fréquentes :** répondre uniquement « nous sommes concernés, le
correctif est disponible ». C'est vrai, mais cela ne dit ni quoi faire, ni dans
quel ordre, ni qui décide.

---

## Points à retenir

- La course se lit sur **deux volets** : le correctif n'arrête pas la menace, il
  arme les deux camps.
- La fenêtre dangereuse s'ouvre quand les preuves de concept circulent et se
  ferme quand le correctif est **déployé**, pas quand il est publié.
- Le temps se perd dans des étapes normales — inventaire, test, fenêtre — qui
  deviennent coûteuses parce qu'elles ne sont pas préparées.
- Un avis d'exposition vaut par sa **décision** : savoir / supposer, actions
  immédiates, amélioration structurelle.
- Tout cela se prépare pendant le calme : c'est le travail de veille qui rend
  l'urgence supportable.
