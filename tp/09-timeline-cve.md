# Exercice — Reconstituer la course patch / exploit

> 🟡 **Intermédiaire** · **Durée :** 40 min · **Lien :** chapitre [Failles connues et course contre la montre](../09-cve-et-course.md)
> **Objectif :** reconstituer la course entre correctif et exploitation à partir
> d'un cas réel, puis situer les étapes où l'équipe de défense a perdu du temps.

---

## Contexte

Le groupe de distribution **Vial & Fils** exploite une plateforme de reporting
qui repose sur PHP. L'équipe technique suit la veille de sécurité, mais comme
beaucoup d'équipes, elle découvre les annonces au fil de l'eau et planifie ses
mises à jour de production une fois par mois.

Voici les faits dont vous disposez, tels qu'ils ont été documentés.

**Faits publics — advisory CERT-EU 2026-018 :**

| Date | Fait |
| --- | --- |
| 20 octobre 2026 | le projet PHP annonce deux failles d'exécution de code à distance non authentifiées, notées CVSS 9.2 |
| 21 octobre 2026, 13:30 UTC | le correctif est publié |
| 21 octobre 2026, fin de journée | des preuves de concept circulent publiquement |
| 23 octobre 2026 | une troisième faille est annoncée sur le même produit |

**Journal de l'équipe de Vial & Fils :**

| Date | Fait |
| --- | --- |
| 20 octobre 2026, 14:00 UTC | l'alerte de veille est lue par l'équipe |
| 22 octobre 2026 | l'inventaire établit que deux serveurs de production et un serveur de recette sont concernés |
| 24 octobre 2026 | le correctif est testé sur le serveur de recette |
| 28 octobre 2026 | le correctif est déployé en production |

---

## Consignes

### Partie 1 — Reconstituer la course

Construisez la frise de la course à **deux volets** : un volet « attaquant » et
un volet « défenseur ». Placez chaque fait à sa date, en indiquant pour chacun
s'il relève de l'attaquant, du défenseur ou des deux. La frise peut être un
tableau, un schéma ou une liste datée : l'important est que les deux volets se
lisent en parallèle.

### Partie 2 — Situer où l'équipe perd du temps

Identifiez les étapes où l'équipe de Vial & Fils a perdu du temps entre
l'annonce et la mise en production. Pour chacune, donnez :

- la période concernée (avec les dates du journal) ;
- le type d'étape, parmi celles du cours : **inventaire**, **test**,
  **fenêtre de déploiement**, **vérification** ;
- une phrase sur ce qui aurait pu raccourcir cette étape **avant** l'urgence.

### Partie 3 — Écrire un avis d'exposition

Vous êtes l'ingénieur qui lit l'alerte le 20 octobre 2026 à 14:00 UTC. Rédigez
un avis d'exposition de dix lignes maximum à destination de votre responsable,
qui réponde aux trois questions suivantes :

- sommes-nous concernés, et sur quoi en êtes-vous sûr à cette date ?
- que faites-vous entre l'annonce et le déploiement du correctif ?
- que changez-vous dans votre organisation pour la prochaine annonce ?

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Ma frise comporte **les deux volets** (attaquant et défenseur) et pas
      seulement la chronologie du fournisseur.
- [ ] Chaque fait est placé à la **date** indiquée dans les journaux.
- [ ] J'ai identifié **au moins deux** étapes où l'équipe perd du temps, chacune
      rattachée à un type d'étape du cours.
- [ ] Pour chaque étape lente, j'ai proposé une mesure qui se prépare **avant**
      l'urgence, pas une urgence supplémentaire.
- [ ] Mon avis d'exposition répond aux **trois questions**, et distingue ce que
      je sais de ce que je suppose.

---

## Pour aller plus loin

*Facultatif.* La troisième faille est annoncée le 23 octobre 2026, alors que le
correctif de la précédente n'est pas encore déployé. Que change cette annonce
dans votre plan : reprenez-vous l'analyse depuis le début, ou traitez-vous les
deux sujets en parallèle ? Justifiez votre choix en deux lignes.
