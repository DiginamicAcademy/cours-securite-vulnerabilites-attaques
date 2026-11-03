# Exercice — Écrire une analyse menace / prévention

> 🔵 **Avancé** · **Durée :** 55 min · **Lien :** chapitre [Prévention et détection](../12-prevention-et-detection.md)
> **Objectif :** produire une analyse menace / prévention exploitable sur une
> application réelle : scénarios d'attaque, mesures de coupe, signaux de
> détection, premiers gestes de réponse.

---

## Contexte

La société **Altis Data** exploite une plateforme de reporting : une API
(*application programming interface*) alimente un tableau de bord consulté par
les responsables de magasins. La plateforme expose les points d'entrée suivants :

| Point d'entrée | Rôle | Qui y accède |
| --- | --- | --- |
| `POST /login` | ouverture de session | tous les utilisateurs |
| `GET /reports` | consultation des rapports | utilisateurs connectés |
| `GET /reports/export` | export des données de vente | utilisateurs connectés |
| `GET /admin/users` | gestion des comptes | administrateurs |
| `GET /debug/config` | diagnostic technique | équipe technique |

L'équipe vient de mettre la plateforme en production. Personne n'a encore écrit
ce qui pourrait mal tourner, ni ce qui permettrait de le voir arriver : c'est
votre travail.

---

## Consignes

### Partie 1 — Cartographier les menaces

Listez **six scénarios d'attaque** plausibles sur cette plateforme. Pour chacun,
donnez :

- le scénario, en une phrase concrète (qui fait quoi, et ce qu'il obtient) ;
- l'étape de la kill chain concernée (accès initial, escalade, mouvement
      latéral, exfiltration…) ;
- le point d'entrée impliqué parmi ceux du tableau ci-dessus.

Visez la variété : tous les scénarios ne doivent pas passer par le même point
d'entrée ni par la même étape de la chaîne.

### Partie 2 — Prévenir et détecter

Complétez votre cartographie avec deux colonnes :

- **prévention** : la mesure qui coupe le scénario, formulée comme une action
  (« paramétrer la requête », « limiter le débit »…) ;
- **détection** : l'événement qui rendrait le scénario visible, formulé comme
  une information journalisée (« série d'échecs de connexion sur un même
  compte »…).

Une mesure peut couvrir plusieurs scénarios : signalez-le plutôt que de la
recopier.

### Partie 3 — Répondre et prioriser

Parmi vos six scénarios, choisissez celui que vous traiteriez **en premier**.
Justifiez votre choix avec un critère explicite (exposition, données touchées,
nombre de personnes concernées, difficulté de détection…).

Puis écrivez les **premiers gestes de réponse** pour ce scénario, dans l'ordre,
en vous appuyant sur les cinq temps du cours (conserver, isoler, qualifier,
informer, corriger).

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai **six** scénarios, et ils ne passent pas tous par le même point
      d'entrée.
- [ ] Chaque scénario est rattaché à une **étape de la kill chain** nommée.
- [ ] Chaque scénario a **les deux colonnes** : prévention **et** détection.
- [ ] Ma priorité affiche un **critère explicite**, pas seulement un classement.
- [ ] Mes premiers gestes de réponse suivent un **ordre**, et commencent par la
      conservation des traces.

---

## Pour aller plus loin

*Facultatif.* Parmi vos mesures de détection, laquelle déclencherait le plus
d'alertes sans être la plus utile ? Expliquez en quelques lignes comment vous
réduiriez ce bruit sans perdre le signal.
