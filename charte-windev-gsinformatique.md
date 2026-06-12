# Charte des dépôts WINDEV — GSInformatique SA

> **Version 1.0** · Document vivant. Toute proposition d'amendement passe par une PR sur ce fichier.
> Complète les chartes [GitHub](charte-github-gsinformatique.md), [Claude](charte-claude-gsinformatique.md) et [BMad](charte-bmad-gsinformatique.md) — elles s'appliquent ensemble.
> En cas de tension, cette charte précise **comment ces règles s'appliquent au cas particulier des projets de la suite WINDEV** (WINDEV, WEBDEV, WINDEV Mobile) — elle ne les remplace pas.

## Objectif

Donner des réflexes communs pour versionner proprement un projet de la suite PC SOFT sur GitHub. Les projets WINDEV ne sont pas des projets texte : leurs sources sont des **fichiers binaires opaques**, et l'environnement génère en permanence des artefacts (exécutables, sauvegardes, snapshots) qui n'ont rien à faire dans un dépôt. Cette charte évite les pièges qui ont coûté du temps à la mise en place — `.gitignore` PC SOFT incomplet, binaires de 150 Mo refusés par GitHub, sauvegardes complètes dupliquées dans l'historique.

**Trois principes directeurs :**

1. **Le dépôt ne contient que les sources.** Jamais les binaires de compilation, jamais les sauvegardes locales.
2. **Le `.gitignore` PC SOFT est un point de départ, pas une fin.** On le durcit dès la création du dépôt.
3. **Les sources WINDEV sont binaires.** Pas de diff lisible, pas de merge textuel : on s'organise pour *éviter* les conflits, pas pour les résoudre.

---

## 1. Périmètre & nommage

- **Périmètre** : tout projet WINDEV, WEBDEV ou WINDEV Mobile versionné sur GitHub.
- **Nommage** *(amende la [Charte GitHub §4](charte-github-gsinformatique.md#4-gouvernance--accès))* : préfixe **`wd-<projet>`** en minuscules pour situer un dépôt de la suite WinDev d'un coup d'œil (ex. `wd-venteauxencheres`). Le préfixe complète `gsi-` (apps/socles) et `poc-` (preuves de concept).
- **Code client** : un projet client part en dépôt **privé** par défaut (code propriétaire). On ne publie jamais le source d'un client sans accord explicite.

---

## 2. Le `.gitignore` — obligatoire dès la création

Le point de départ est le **`.gitignore` modèle fourni par PC SOFT** (sous-dossier `Personal` du répertoire d'installation de WINDEV/WEBDEV/WINDEV Mobile). Il couvre l'essentiel des fichiers de travail :

```gitignore
Backup
Bin
cache.gestion de sources
Corbeille
Groupware
Lastsave
Sauvegarde
.tmp
*.cache
*.cpl
*.env
*.rep
*.waudit
*.wdcoverage
*.wpf
HstDuree.log
```

> ⚠️ **Le modèle PC SOFT est incomplet.** Il **n'exclut PAS** les exécutables générés, les packages d'installation, ni les dossiers de sauvegarde datés. Laissés tels quels, ces dossiers font entrer dans l'historique des binaires de plusieurs centaines de Mo — au-delà des limites GitHub — et le push est **refusé**.

**Ajouts obligatoires** à coller dans le `.gitignore` de tout dépôt WinDev :

```gitignore
# Artefacts de build & sauvegardes locales (ajout GSI — hors modèle PC SOFT)
Exe/
Install_*/
Versions/
Svg_*/
```

- `Exe/` — exécutable généré **et runtime WinDev** (`wd300*.dll`, `wd300zip64.zip`… plusieurs dizaines de Mo). Régénéré à chaque compilation.
- `Install_*/` — package d'installation (`Install_Store_*.exe`, souvent > 100 Mo).
- `Versions/` — archives ZIP de versions produites par l'IDE.
- `Svg_*/` — **snapshots de sauvegarde automatiques** : ce sont des **copies intégrales du projet** (sources + Exe + Install). Versionnés, ils triplent le dépôt et ré-injectent les gros binaires qu'on vient d'exclure.

---

## 3. Ce qui ne va JAMAIS dans le dépôt

| Catégorie | Exemples | Pourquoi |
| --------- | -------- | -------- |
| Exécutables & runtime | `Exe/`, `*.dll` PC SOFT | Régénérés à la compilation — pas une source |
| Packages d'installation | `Install_*/`, `*.exe` d'install | Volumineux, dérivés |
| Sauvegardes / snapshots | `Sauvegarde`, `Versions/`, `Svg_*/`, `Backup`, `Lastsave` | Copies redondantes ; gonflent l'historique |
| Caches & fichiers de travail | `*.cache`, `.tmp`, `cache.gestion de sources` | Spécifiques à un poste |
| Fichiers volumineux | tout fichier ≥ 50 Mo | GitHub **alerte à 50 Mo**, **refuse le push à 100 Mo** |

**Sur les fichiers volumineux** : un binaire lourd *légitime et stable* (gros visuel, thème, base modèle) peut passer par **Git LFS** — mais c'est l'exception, à justifier. Un binaire **régénérable** (exe, dll, package) ne va jamais dans le dépôt, ni en LFS.

> Si un push est refusé pour cause de gros fichier, **ne pas forcer ni passer en LFS par réflexe** : vérifier d'abord que le fichier n'est pas un artefact à exclure (c'est presque toujours le cas).

---

## 4. Sources binaires : diff illisible, pas de merge

Les éléments WinDev (`.wdw`, `.wde`, `.wdc`, `.wdg`, `.ana`…) sont des fichiers **binaires** : leurs propriétés visuelles sont stockées en base64 **chiffré**, illisible hors de l'IDE WinDev. Conséquences concrètes :

- **Aucun diff de relecture utile** sur ces fichiers, et **aucun merge automatique possible** — un conflit sur un élément WinDev ne se résout pas à la main dans Git.
- **On évite les conflits au lieu de les résoudre** :
  - **branches courtes**, mergées vite (cf. [Charte GitHub §1](charte-github-gsinformatique.md#1-branches--commits)) ;
  - **un seul développeur à la fois** sur un même élément (fenêtre, état, requête) ; se répartir le travail par fichier, pas par ligne ;
  - utiliser le **GDS (Gestionnaire De Sources) de WinDev** quand plusieurs personnes travaillent en parallèle — il gère le check-out/réintégration au niveau élément, là où Git ne voit qu'un blob.
- **La review porte sur le *quoi* et le *comment tester*, pas sur le diff** (cf. [Charte GitHub §2](charte-github-gsinformatique.md#2-pull-requests--reviews)). La PR décrit le changement fonctionnel ; le reviewer valide en ouvrant le projet dans WinDev, pas en lisant le binaire. Une **preuve fonctionnelle** (capture, scénario testé) remplace la lecture de diff.
- **Squash merge** systématique : l'historique `main` reste une suite de changements fonctionnels lisibles, à défaut de diffs lisibles.
- **Extraction texte** : pour garder une trace relisible/diffable du code WLanguage, on peut versionner en parallèle une extraction `.txt` du code (cf. outillage d'extraction du projet) — utile pour la review et la recherche, sans remplacer les sources binaires.

---

## 5. Secrets & connexions

Les règles de la [Charte GitHub §3](charte-github-gsinformatique.md#3-sécurité--secrets) s'appliquent intégralement. Points d'attention propres à WinDev :

- **Connexions HFSQL / bases de données** : chaînes de connexion, identifiants, mots de passe → jamais en dur dans le code WLanguage ni dans l'analyse. Variables d'environnement ou paramétrage externe non versionné.
- **Analyse (`.ana`)** et fichiers de configuration : vérifier qu'aucun credential n'y est figé avant le premier commit.
- `*.env` est déjà exclu par le modèle PC SOFT — le garder.
- **Secret Scanning + Push Protection** activés sur l'organisation (cf. Charte GitHub §3).

---

## 6. Gouvernance & adoption

- **Branch protection sur `main`** (push direct interdit, PR + 1 review, historique linéaire) comme tout dépôt GSI — cf. [Charte GitHub §4](charte-github-gsinformatique.md#4-gouvernance--accès).
- **Référent** : le même que pour les autres chartes ; valide les dérogations (LFS, brique hors standard) via ADR.
- **Revue annuelle**, alignée sur les autres chartes.

Pour démarrer un nouveau dépôt WinDev :

1. Créer le dépôt **privé** `wd-<projet>` dans l'organisation.
2. Copier le `.gitignore` PC SOFT **+ les ajouts du §2** *avant* le premier commit.
3. Premier commit : vérifier qu'aucun dossier `Exe/`, `Install_*/`, `Versions/`, `Svg_*/` ni fichier ≥ 50 Mo n'est suivi (`git ls-files` + contrôle des tailles).
4. Activer la branch protection et le secret scanning.
