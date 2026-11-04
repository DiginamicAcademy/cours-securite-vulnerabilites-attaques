[← Chapitre précédent](13-tests-intrusion.md) · [Sommaire](Readme.md)

# Glossaire et ressources

> **Nature :** annexe de référence · **Usage :** tout au cours

Dans les chapitres, chaque acronyme est défini à sa première occurrence. Cette page rassemble toutes ces définitions au même endroit, les termes techniques clés du cours, et les ressources de référence — à garder sous la main pendant les deux journées.

---

## Acronymes

| Sigle | Signification | En clair |
| --- | --- | --- |
| **A01 – A10** | les dix codes des familles du Top 10 de l'OWASP | la carte de référence des failles web — détail ci-dessous |
| **ANSSI** | Agence nationale de la sécurité des systèmes d'information | l'autorité française de cybersécurité : guides, recommandations, retours d'expérience |
| **API** | Application Programming Interface | un point d'entrée programmatique d'une application |
| **ATT&CK** | Adversarial Tactics, Techniques and Common Knowledge | la base de connaissances des gestes des attaquants, maintenue par le MITRE |
| **BDD** | base de données | le stockage structuré de l'application |
| **BC** | bloc de compétences | un ensemble de compétences certifiées (ici, le bloc BC03 de la certification RNCP40573) |
| **CERT** | Computer Emergency Response Team | l'équipe qui publie les avis de sécurité : CERT-EU (Union européenne), CERT-FR (France) |
| **CK** | Common Knowledge | la troisième partie du sigle ATT&CK |
| **CSP** | Content Security Policy | la politique qui dit au navigateur quels scripts exécuter |
| **CSRF** | Cross-Site Request Forgery | faire exécuter une action à un utilisateur connecté, sans qu'il le sache |
| **CSV** | Comma-Separated Values | un fichier de valeurs séparées par des virgules |
| **CVE** | Common Vulnerabilities and Exposures | l'identifiant normalisé d'une faille connue |
| **CVSS** | Common Vulnerability Scoring System | la note de gravité d'une faille, de 0 à 10 |
| **DOM** | Document Object Model | la représentation de la page que le navigateur tient en mémoire |
| **EDR** | Endpoint Detection and Response | l'outil qui surveille postes et serveurs : processus, fichiers, connexions |
| **GHSA** | GitHub Security Advisory | les avis de sécurité publiés par GitHub |
| **HTML** | HyperText Markup Language | le langage de balisage des pages web |
| **HTTP** | HyperText Transfer Protocol | le protocole d'échange des pages web |
| **HTTPS** | HTTP Secure | HTTP chiffré |
| **IA** | intelligence artificielle | |
| **JSON** | JavaScript Object Notation | le format d'échange de données structurées le plus courant |
| **MD5** | Message-Digest Algorithm 5 | une fonction de hachage — obsolète pour les mots de passe |
| **MFA** | Multi-Factor Authentication | l'authentification multifacteur : une seconde preuve d'identité |
| **MITRE** | (nom d'organisme, ce n'est pas un sigle) | l'organisme de recherche américain qui maintient ATT&CK et la base CWE |
| **NIST** | National Institute of Standards and Technology | l'organisme américain de référence : recommandations, notamment sur les mots de passe |
| **NVD** | National Vulnerability Database | la base publique des CVE |
| **ORM** | Object-Relational Mapping | la couche qui écrit les requêtes SQL à votre place |
| **OWASP** | Open Web Application Security Project | la communauté de référence en sécurité applicative : le Top 10 |
| **PHP** | PHP: Hypertext Preprocessor | un langage de serveur web |
| **PME** | petite et moyenne entreprise | |
| **PoC** | Proof of Concept | la preuve de concept : le premier exploit fonctionnel publié |
| **QCM** | questionnaire à choix multiples | un format d'évaluation où l'on coche la bonne réponse |
| **RGPD** | Règlement général sur la protection des données | le cadre européen des données personnelles |
| **RNCP** | Répertoire national des certifications professionnelles | le registre des certifications : RNCP40573 est la certification visée par ce cours |
| **SAST** | Static Application Security Testing | l'analyse automatique du code source, sans l'exécuter |
| **SBOM** | Software Bill of Materials | le registre des dépendances d'un logiciel |
| **SCA** | Software Composition Analysis | l'analyse automatique des dépendances |
| **SI** | système d'information | l'ensemble des systèmes et des données d'une organisation |
| **SOC** | Security Operations Center | le centre de sécurité qui surveille et répond |
| **SFTP** | SSH File Transfer Protocol | le transfert de fichiers sur SSH |
| **SQL** | Structured Query Language | le langage d'interrogation des bases de données |
| **SSH** | Secure Shell | l'accès distant chiffré |
| **SSRF** | Server-Side Request Forgery | faire émettre une requête par le serveur lui-même |
| **STRIDE** | Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege | un modèle de catégorisation des menaces, développé par Microsoft |
| **TD** | travaux dirigés | une série d'exercices, souvent en séquence |
| **TP** | travaux pratiques | une mise en pratique — ici, le TP intégrateur est un projet de bout en bout |
| **UTC** | Temps universel coordonné | le temps de référence des journaux et des avis de sécurité |
| **WAF** | Web Application Firewall | le pare-feu applicatif, qui filtre les requêtes |
| **XDR** | eXtended Detection and Response | la détection étendue : postes, serveurs, réseau, cloud |
| **XML** | eXtensible Markup Language | le langage de balisage structuré : factures, échanges de données |
| **XSS** | Cross-Site Scripting | injecter un script dans une page vue par d'autres |
| **XXE** | XML External Entity | l'abus d'entités externes dans un document XML |

---

## Les dix familles du Top 10 (A01 – A10)

| Code | Famille | Ce qu'elle recouvre |
| --- | --- | --- |
| **A01** | Broken Access Control | faire ce qu'on n'a pas le droit de faire |
| **A02** | Cryptographic Failures | des données mal protégées, chiffrées mal ou pas |
| **A03** | Injection | confondre les données et les instructions |
| **A04** | Insecure Design | un choix de conception dangereux, pas un bug |
| **A05** | Security Misconfiguration | réglages par défaut, debug laissé actif, pages ou secrets inutiles |
| **A06** | Vulnerable and Outdated Components | une dépendance connue pour être fragile |
| **A07** | Identification and Authentication Failures | des comptes et des mots de passe mal protégés |
| **A08** | Software and Data Integrity Failures | des mises à jour ou des données jamais vérifiées |
| **A09** | Security Logging and Monitoring Failures | ne pas voir l'attaque arriver |
| **A10** | Server-Side Request Forgery | faire émettre une requête par le serveur lui-même |

---

## Termes techniques clés

- **Advisory (avis de sécurité)** — la publication qui annonce une faille : produit concerné, versions affectées, correctif disponible.
- **Attaquant (profil d'attaquant)** — qui attaque, avec quels moyens, quel temps et quelle connaissance du système visé.
- **Cause racine** — la raison pour laquelle la faille existe ; la corriger évite qu'elle ne revienne sous une autre forme.
- **Correctif (patch)** — la modification qui supprime la faille.
- **Dépendance** — un composant externe sur lequel l'application s'appuie.
- **Détection** — rendre l'attaque visible pendant qu'elle se produit.
- **Escalade de privilèges** — obtenir plus de droits que ceux dont on disposait.
- **Exfiltration** — sortir des données du système visé.
- **Exploit** — la mise en œuvre concrète d'une faille : la requête ou le geste qui la déclenche.
- **Faille / vulnérabilité** — une faiblesse exploitable. « Vulnérabilité » est le terme général ; « faille » désigne la faiblesse effective dans un composant.
- **Gravité** — ce que la faille coûte, dans le contexte précis de l'application : exposition et perte pour l'organisation.
- **Jour zéro (zero-day)** — une faille exploitée avant que le correctif n'existe.
- **Kill chain (chaîne de compromission)** — la suite des étapes d'une attaque, de la reconnaissance à l'exfiltration.
- **Menace** — ce qu'un attaquant pourrait faire, et avec quelles conséquences.
- **Modélisation des menaces** — l'exercice qui liste ces scénarios avant qu'ils n'arrivent.
- **Mouvement latéral** — se déplacer d'un système à un autre à l'intérieur du réseau.
- **Non-régression (test de)** — la vérification automatique qui empêche une faille corrigée de revenir.
- **Point de confiance** — un endroit où un élément en croit un autre sans le vérifier.
- **Prévention** — empêcher l'attaque de fonctionner.
- **Reconnaissance** — la première étape d'une attaque : repérer ce qui existe et ce qui est exposé.
- **Revue de code** — la lecture du code pour y repérer les failles avant la production.
- **Risque** — la gravité d'une faille, pondérée par la probabilité qu'elle soit effectivement exploitée.
- **Surface d'attaque** — l'ensemble des points par lesquels l'application peut être atteinte.
- **Test d'intrusion (pentest)** — une simulation d'attaque, sous contrat et dans un périmètre défini.
- **Top 10 de l'OWASP** — la classification de référence des dix familles de failles web.

---

## Ressources utiles

### Classification et référentiels

- **Top 10 de l'OWASP** — <https://owasp.org/www-project-top-ten/> — la carte des familles de failles web.
- **OWASP Cheat Sheet Series** — <https://cheatsheetseries.owasp.org/> — les gestes de correction, fiche par fiche.
- **MITRE ATT&CK** — <https://attack.mitre.org/> — les gestes des attaquants, décrits un par un.
- **MITRE CWE** — <https://cwe.mitre.org/> — les types de faiblesses, en amont des CVE.
- **Modèle STRIDE** (Microsoft) — la catégorisation des menaces en six familles.
- **NIST** — <https://csrc.nist.gov/> — les référentiels américains, dont la série SP 800-63 sur l'authentification.
- **ANSSI** — <https://cyber.gouv.fr/> — guides, recommandations et retours d'expérience en français.

### Bases de failles et avis de sécurité

- **NVD** — <https://nvd.nist.gov/> — les CVE et leur score CVSS.
- **CVE** — <https://www.cve.org/> — le registre des identifiants de failles.
- **GHSA** — <https://github.com/advisories> — les avis de sécurité des écosystèmes GitHub.
- **CERT-FR** — <https://cert.cyber.gouv.fr/> — les avis et alertes pour la France.
- **CERT-EU** — <https://cert.europa.eu/> — les avis pour les institutions de l'Union européenne.
- **FIRST (CVSS)** — <https://www.first.org/cvss/> — la définition de l'échelle de gravité.

### Environnements d'entraînement

À pratiquer sur **vos propres conteneurs**, comme le lab du cours :

- **OWASP Juice Shop** — <https://owasp.org/www-project-juice-shop/>
- **DVWA** (Damn Vulnerable Web Application) — <https://dvwa.co.uk/>
- **WebGoat** — <https://owasp.org/www-project-webgoat/>

### Cadre légal

- **Articles 323-1 et suivants du Code pénal** — l'accès et le maintien sans droit dans un système de traitement automatisé de données.
- **RGPD** — <https://www.cnil.fr/> — les obligations en cas de fuite de données personnelles, dont la notification.

### Dans ce cours

- Les exercices, les TD et le TP intégrateur : le dossier `tp/`.
- Les manipulations : le lab Docker du cours, sur vos propres conteneurs.

---

[← Chapitre précédent](13-tests-intrusion.md) · [Sommaire](Readme.md)
