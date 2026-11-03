# Exercice — Modéliser la kill chain d'une fuite de données

> 🔴 **Défi** · **Durée :** libre · **Lien :** chapitre [La kill chain](../08-kill-chain.md)
> **Objectif :** modéliser de bout en bout la chaîne de compromission d'une
> fuite de données sur une plateforme data, puis identifier les points de rupture
> et les signaux qui l'auraient rendue visible.

---

## Contexte

Le réseau de pharmacies **Léonie** consolide ses ventes dans un entrepôt de
données. Des exports quotidiens alimentent un lac de données, une chaîne de
planification exécute les traitements, et un tableau de bord expose les agrégats
aux responsables de région.

Un incident vient d'éclater : des données de programme de fidélité — noms,
adresses, historique d'achats — ont été retrouvées en circulation. Personne ne
sait encore par où elles sont sorties, ni depuis quand. L'équipe technique
dispose de ce qui suit :

- les journaux de la chaîne de planification des traitements ;
- les journaux d'accès au lac de données ;
- les journaux du tableau de bord ;
- les journaux du réseau de l'entreprise.

C'est un **défi** : il n'y a pas de solution unique, et vous travaillez en
autonomie. L'objectif est de produire un modèle défendable, pas de retrouver une
réponse attendue.

---

## Consignes

### Partie 1 — Modéliser la kill chain

Retracez la chaîne de compromission plausible de cette fuite, étape par étape,
de la reconnaissance à l'exfiltration. Pour chaque étape, décrivez :

- ce que l'attaquant a **fait**, concrètement, dans ce contexte précis ;
- le **composant** de la plateforme impliqué (exports, lac de données, chaîne de
  planification, tableau de bord, réseau) ;
- ce que l'attaquant a **obtenu**, qui lui sert pour l'étape suivante.

### Partie 2 — Points de rupture

Pour chaque étape de votre modèle, indiquez la mesure qui l'aurait **coupée** ou
**rendue plus coûteuse**. Précisez pour chacune si elle relève de la prévention
ou de la détection, et à quel moment elle devait être en place — avant
l'incident, bien sûr, mais à quel stade de la vie de la plateforme.

### Partie 3 — Signaux de détection

Parmi les journaux dont dispose l'équipe, identifiez ceux qui auraient permis de
**voir** l'attaque, et à quelle étape. Pour chacun, formulez le signal recherché
sous la forme d'une règle : « quand X arrive, alerter ».

Terminez par une estimation honnête : à quelle étape pensez-vous que
l'attaque aurait pu être **détectée** avec les journaux existants, et que
manquait-il pour la détecter plus tôt ?

---

## ✅ Auto-évaluation

Avant de comparer avec le modèle de référence, vérifiez que :

- [ ] Ma kill chain couvre **toutes les étapes**, de la reconnaissance à
      l'exfiltration, sans sauter d'étape.
- [ ] Chaque étape nomme un **composant concret** de la plateforme, pas une
      abstraction.
- [ ] Chaque étape dit ce que l'attaquant **obtient** pour la suivante — c'est
      ce qui fait une chaîne.
- [ ] Mes points de rupture distinguent **prévention** et **détection**.
- [ ] Mes signaux de détection sont formulés comme des **règles**, et je dis à
      quelle étape ils auraient agi.

---

## Pour aller plus loin

*Facultatif.* Si vous ne deviez retenir qu'**un seul** signal de détection à
installer en premier sur cette plateforme, lequel choisiriez-vous, et pourquoi
celui-là plutôt qu'un autre ? Justifiez en quelques lignes.
