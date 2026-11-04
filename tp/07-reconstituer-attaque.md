# Exercice — Reconstituer les étapes d'une attaque

> **Nature :** exercice · **Durée :** 25 min · **Notion :** modèles d'attaquant — reconstituer les étapes d'une attaque
> 🟢 **Débutant** · **Durée :** 25 min · **Lien :** chapitre [Modèles d'attaquant](../07-modeles-attaquant.md)
> **Objectif :** remettre dans l'ordre les étapes d'une attaque réelle, identifier
> le profil d'attaquant concerné, puis repérer où la défense pouvait agir.

---

## Contexte

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

## Consignes

### Partie 1 — Remettre la chronologie en ordre

Classez les lettres dans l'ordre chronologique plausible. Pour chaque étape,
ajoutez le nom de l'étape de la kill chain qu'elle correspond (reconnaissance,
accès initial, escalade de privilèges, mouvement latéral, persistance, actions
sur les objectifs, exfiltration).

### Partie 2 — Identifier le profil d'attaquant

Quel profil parmi ceux du cours décrit le mieux cette attaque ? Justifiez votre
réponse en vous appuyant sur :

- la **motivation** apparente ;
- les **moyens** observés (temps, outils, connaissance du système).

Dites ensuite quelle conséquence ce profil a sur la défense à construire en
priorité.

### Partie 3 — Où la défense pouvait agir

Pour les étapes d'**accès initial**, de **persistance** et d'**exfiltration**,
proposez la mesure qui les aurait coupées ou rendues visibles. Une phrase par
mesure, formulée comme une action.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Mon ordre chronologique est **complet** : chaque lettre apparaît une fois.
- [ ] Chaque étape porte le **nom de la kill chain** correspondant.
- [ ] Mon profil d'attaquant est justifié par la motivation **et** par les moyens.
- [ ] J'ai dit ce que ce profil change pour la **défense à construire en
      priorité**.
- [ ] Mes mesures sont formulées comme des **actions**, pas comme des souhaits.

---

## Pour aller plus loin

*Facultatif.* Parmi les étapes de cette attaque, laquelle aurait été la plus
simple à **détecter** avec des journaux, et pourquoi ? Répondez en deux lignes.
