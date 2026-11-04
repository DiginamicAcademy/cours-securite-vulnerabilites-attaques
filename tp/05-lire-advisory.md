# Exercice — Lire un advisory et rédiger un avis d'exposition

> **Nature :** exercice · **Durée :** 35 min · **Notion :** les failles connues — lire un advisory et rédiger un avis d'exposition
> 🟡 **Intermédiaire** · **Durée :** 35 min · **Lien :** chapitre [Configuration,
> dépendances et failles connues](../04-owasp-config-deps.md)
> **Objectif :** lire un avis de sécurité réel, en extraire les informations qui
> décident d'une action, puis rédiger un avis d'exposition pour un système
> d'information data.

---

## Contexte

Le 20 octobre 2026, le **CERT** (Computer Emergency Response Team) de l'Union
européenne publie l'avis de référence **2026-018** sur **PHP** (PHP: Hypertext
Preprocessor), le langage qui fait tourner une bonne part des applications web.
Voici l'essentiel de l'avis, tel qu'il a été relayé aux équipes :

> **Advisory CERT-EU 2026-018 — PHP**
>
> - **Produit** : PHP (PHP: Hypertext Preprocessor).
> - **Versions concernées** : les versions antérieures au correctif du 21 octobre
> 2026.
> - **20 octobre 2026** : deux failles d'exécution de code à distance non
> authentifiées sont annoncées — score **CVSS** (Common Vulnerability Scoring
> System) de 9.2.
> - **21 octobre 2026, 13:30 UTC** (temps universel coordonné) : le correctif est
> publié.
> - **Dans la foulée** : des preuves de concept — les premiers exploits
> fonctionnels — circulent publiquement.
> - **23 octobre 2026** : une troisième faille est annoncée sur le même produit.
> - **Recommandation** : appliquer le correctif dès que possible ; à défaut,
> restreindre l'accès aux serveurs concernés.

Vous travaillez pour un opérateur de mobilité, sur son système d'information
**data** — la plateforme qui produit les statistiques de fréquentation et les
tableaux de bord ouverts aux collectivités. Voici l'inventaire tenu par l'équipe :

| Élément du système | Technologie | Exposition |
| --- | --- | --- |
| portail de reporting interne | PHP | intranet, accès authentifié |
| tableaux de bord publics | PHP | internet, derrière un reverse-proxy |
| orchestration des traitements | Airflow | intranet, accès authentifié |
| stockage objet des exports | MinIO | intranet |
| API de données | Python | internet |

---

## Consignes

### Partie 1 — Extraire les champs de l'avis

À partir du seul extrait ci-dessus, remplissez une fiche en cinq lignes : produit,
versions concernées, gravité, date du correctif, recommandation. Ne recopiez pas
l'avis : reformulez chaque champ en une phrase.

### Partie 2 — Lire la chronologie

Placez les quatre dates de l'avis sur une frise, puis répondez :

- combien de temps sépare l'annonce des failles de la publication du correctif ;
- ce que signifie, pour une équipe qui exploite du PHP, la phrase « des preuves de
  concept circulent dans la foulée » ;
- ce que change l'annonce d'une troisième faille le 23 octobre, alors que le
  correctif du 21 n'est peut-être pas encore déployé.

### Partie 3 — Évaluer l'exposition du système

Pour chacun des cinq éléments de l'inventaire, répondez **concerné**, **non
concerné** ou **à vérifier**, avec une justification d'une ligne. Puis classez les
éléments concernés par ordre de traitement, en explicitant votre critère.

### Partie 4 — Rédiger l'avis d'exposition

En une dizaine de lignes destinées au responsable du système d'information,
rédigez l'avis d'exposition : ce qui est touché, ce que cela peut coûter, ce qui
doit être fait, dans quel ordre et pour quand. Un lecteur non technique doit
pouvoir décider en le lisant.

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] Ma fiche de l'avis contient les **cinq champs**, reformulés, sans copier le
      texte d'origine.
- [ ] Ma chronologie donne les **quatre dates** et ce que chacune impose à
      l'équipe.
- [ ] J'ai donné un verdict pour **les cinq éléments** de l'inventaire, avec une
      justification — y compris pour ceux que je juge non concernés.
- [ ] Mon classement affiche un **critère explicite** (exposition, données
      touchées, nombre de personnes concernées…).
- [ ] Mon avis d'exposition se lit **sans connaître l'advisory** : un décideur
      peut trancher en le lisant.

---

## Pour aller plus loin

*Facultatif.* L'avis annonce une troisième faille après la publication du
correctif. Décrivez la procédure que vous mettriez en place pour que la prochaine
annonce de ce type soit traitée en une journée, sans réunion de crise.
