# Charte des dépôts WINDEV — GSInformatique SA

> **Version 1.0** · Document vivant. Toute proposition d'amendement passe par une PR sur ce fichier.
> Complète les chartes [GitHub](charte-github-gsinformatique.md), [Claude](charte-claude-gsinformatique.md) et [BMad](charte-bmad-gsinformatique.md) — elles s'appliquent ensemble.
> En cas de tension, cette charte précise **comment ces règles s'appliquent au cas particulier des projets de la suite WINDEV** (WINDEV, WEBDEV, WINDEV Mobile) — elle ne les remplace pas.

## Objectif

Donner des réflexes communs pour versionner proprement un projet de la suite PC SOFT sur GitHub. Les projets WINDEV ne sont pas des projets texte : leurs sources sont des **fichiers binaires opaques**, et l'environnement génère en permanence des artefacts (exécutables, sauvegardes, snapshots) qui n'ont rien à faire dans un dépôt. Cette charte évite les pièges qui ont coûté du temps à la mise en place — `.gitignore` PC SOFT incomplet, binaires de 150 Mo refusés par GitHub, sauvegardes complètes dupliquées dans l'historique.

**Trois principes directeurs :**

1. **Le dépôt ne contient que les sources.** Jamais les binaires de compilation, jamais les sauvegardes locales.
2. **Le `.gitignore` PC SOFT est un point de départ, pas une fin.** On le durcit dès la création du dépôt.
3. **Les sources WINDEV sont au format texte, largement lisibles.** Le code et la structure se diffent et se mergent ; seul l'habillage visuel (chiffré) reste opaque — on coordonne surtout les retouches d'apparence d'un même écran.

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

## 4. Sources au format texte : diff lisible, habillage chiffré

Les éléments WinDev (`.wdw`, `.wde`, `.wdc`, `.wdg`, `.ana`…) sont stockés au **format texte**. **Ils ne sont pas opaques** : sur un projet réel, **~95 % des lignes sont lisibles** et **100 % du code WLanguage est en clair**.

**En clair** (diffable, mergeable) :

- tout le **code WLanguage** (procédures, méthodes, événements) ;
- la **structure** des fenêtres : noms de contrôles, hiérarchie, géométrie (`x`/`y`/largeur/hauteur), ancrage, ordre de tabulation ;
- les **libellés de fenêtres** (`caption: \n fr-FR: …`), les mappings de classes (`<mapping=…>`).

**Chiffré** (base64, clé WinDev — illisible hors IDE) : **un blob `internal_properties` par contrôle/élément (~5 % des lignes)**, qui porte l'apparence fine (couleurs, polices, styles) et le **texte statique des états** (en-têtes, titres). Modifier ces éléments-là se fait dans l'**IDE WinDev**.

**Conséquences concrètes :**

- **Le diff de review EST utile** : on relit réellement le code et les changements de structure dans la PR (cf. [Charte GitHub §2](charte-github-gsinformatique.md#2-pull-requests--reviews)). Pour un changement **purement visuel** (couleur, police d'un état), le diff ne montre qu'un blob opaque → on complète par une **preuve fonctionnelle** (capture, aperçu PDF).
- **Le merge Git fonctionne** pour le code et la structure. La coordination reste prudente surtout sur l'**apparence d'un même écran** : deux retouches visuelles concurrentes sur un même contrôle ne se fusionnent pas (un seul blob).
- **Bonnes pratiques** : **branches courtes** mergées vite (cf. [Charte GitHub §1](charte-github-gsinformatique.md#1-branches--commits)) ; éviter que deux personnes retouchent **l'habillage** du même élément en parallèle. Le **GDS (Gestionnaire De Sources) WinDev** reste un confort utile pour le travail concurrent au niveau élément, **pas une nécessité** : Git suffit pour l'essentiel.
- **Squash merge** recommandé : un historique `main` lisible, une PR = une entrée.
- **Extraction texte (optionnelle)** : une extraction `.txt` du code WLanguage peut être versionnée en parallèle pour faciliter recherche et revue de masse — utile, mais **plus indispensable** puisque les sources sont déjà lisibles.

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
