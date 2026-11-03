# Exercice — Lister les vulnérabilités et leur gravité sur un extrait de code

> 🔵 **Intégration** · **Durée :** 55 min · **Lien :** chapitres [Les failles web —
> quand le navigateur se retourne contre vous](../03-owasp-web.md) et
> [Configuration, dépendances et failles connues](../04-owasp-config-deps.md)
> **Objectif :** mener une revue de code sécurité sur une petite application —
> lister les constats, les rattacher à une famille, évaluer leur gravité et
> ordonner les corrections.

---

## Contexte

L'association qui organise le festival « Les Nuits Sonores » fait évoluer son
application d'inscription. Le développeur historique est parti ; le nouveau
mainteneur reçoit le fichier principal, écrit en Python avec FastAPI, et le
fichier de dépendances. Avant toute évolution, l'association demande une **revue
de code sécurité** : qu'est-ce qui, dans ce code, expose les inscrits ou
l'application ?

Votre mission n'est pas de corriger : c'est de **lister**, **classer** et
**ordonner**. Les corrections viendront ensuite.

Pour vous repérer, voici les familles du Top 10 de l'**OWASP** (Open Web
Application Security Project) que cette revue est susceptible de rencontrer :

| Code | Famille | Ce qu'elle recouvre |
| --- | --- | --- |
| **A01** | Broken Access Control | faire ce qu'on n'a pas le droit de faire |
| **A02** | Cryptographic Failures | des données mal protégées, chiffrées mal ou pas |
| **A03** | Injection | confondre les données et les instructions |
| **A05** | Security Misconfiguration | réglages par défaut, debug laissé actif, secrets exposés |
| **A06** | Vulnerable and Outdated Components | une dépendance connue pour être fragile |
| **A07** | Identification and Authentication Failures | des comptes et des mots de passe mal protégés |

### Le code à relire — `application.py`

```python
# application.py — inscriptions au festival « Les Nuits Sonores »
import hashlib
import sqlite3

from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse

app = FastAPI(debug=True)
db = sqlite3.connect("festival.db", check_same_thread=False)

# clé de service utilisée pour appeler la billetterie partenaire
CLES_API = {"billetterie": "CLE_A_PLACER_EN_VARIABLE_D_ENVIRONNEMENT"}


@app.get("/recherche")
def recherche(q: str):
    # recherche d'un artiste dans la programmation
    lignes = db.execute(
        "SELECT artiste, scene FROM concerts WHERE artiste LIKE '%" + q + "%'"
    ).fetchall()
    return lignes


@app.get("/concerts/{id_concert}", response_class=HTMLResponse)
def fiche(id_concert: int):
    artiste, description = db.execute(
        "SELECT artiste, description FROM concerts WHERE id = ?",
        (id_concert,),
    ).fetchone()
    return f"<h1>{artiste}</h1><div>{description}</div>"


@app.post("/connexion")
def connexion(identifiant: str, mot_de_passe: str):
    empreinte = hashlib.md5(mot_de_passe.encode()).hexdigest()
    ligne = db.execute(
        "SELECT id, role FROM comptes WHERE identifiant = ? AND empreinte = ?",
        (identifiant, empreinte),
    ).fetchone()
    return {"compte": ligne}


@app.get("/admin/inscriptions")
def inscriptions():
    # réservé à l'organisation : l'interface n'affiche ce lien qu'aux admins
    return db.execute("SELECT nom, courriel FROM inscriptions").fetchall()


@app.post("/inscriptions/annuler")
def annuler(id_inscription: int, request: Request):
    # le cookie de session suffit à prouver l'identité du demandeur
    db.execute("DELETE FROM inscriptions WHERE id = ?", (id_inscription,))
    return {"annulee": id_inscription}
```

### Le fichier de dépendances — `requirements.txt`

```text
fastapi==0.95.0
requests==2.19.0
pillow==8.0.0
```

L'équipe n'a jamais vérifié si des failles publiques concernaient ces versions.

---

## Consignes

### Partie 1 — Inventorier les constats

Listez **tous** les constats de sécurité que vous repérez. Pour chacun, donnez :

- l'endroit concerné (la fonction ou la construction de code, pas un numéro de
  ligne) ;
- la famille OWASP correspondante, par son code ;
- la cause racine, en une phrase.

### Partie 2 — Évaluer la gravité

Pour chaque constat, évaluez la gravité **dans ce contexte** : l'application
contient des noms, des courriels et des paiements d'inscrits au festival, et elle
est accessible depuis internet. Appuyez votre évaluation sur deux éléments :
l'exposition et ce que l'organisation y perd.

### Partie 3 — Ordonner les corrections

Classez les constats par ordre de traitement, et justifiez les deux premiers en
deux lignes chacun : quelle mesure de cause racine traitez-vous, et pourquoi
celle-là d'abord ?

### Partie 4 — Les limites de la revue

Une revue de code ne voit pas tout. Citez deux points que ce code ne permet pas
de trancher et qui demandent un autre type de vérification (test, configuration
du serveur, environnement d'exécution…).

---

## ✅ Auto-évaluation

Avant de passer au corrigé, vérifiez que :

- [ ] J'ai listé **au moins six constats**, et je peux dire pourquoi chacun en est
      un.
- [ ] Chaque constat porte une **famille OWASP** et une **cause racine**, pas
      seulement une description du risque.
- [ ] Mon évaluation de gravité s'appuie sur **l'exposition** et sur **ce que
      l'organisation y perd**, pour chaque constat.
- [ ] Mon ordre de traitement est **justifié**, et les deux premiers sont traités
      par une mesure de cause racine.
- [ ] J'ai nommé **deux limites** de la revue de code — ce que le texte du code
      ne dit pas.

---

## Pour aller plus loin

*Facultatif.* Parmi vos constats, l'un d'eux ne se voit pas dans le code de
l'application mais dans ce qui l'entoure. Lequel, et quelle vérification
l'écarterait ou le confirmerait ?
