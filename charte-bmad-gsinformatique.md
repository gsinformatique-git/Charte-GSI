# Charte d'utilisation de BMad — GSInformatique SA

> **Version 1.0** · Document vivant. Toute proposition d'amendement passe par une PR sur ce fichier.
> Complète les chartes [GitHub](charte-github-gsinformatique.md), [Claude](charte-claude-gsinformatique.md) et [Stack web](charte-stack-web-gsinformatique.md) — les quatre s'appliquent ensemble.
> **BMad est obligatoire sur tout projet GSI livrant du code.**

## Objectif

Donner à toute l'équipe (Suisse / France / Algérie, travail asynchrone) un cadre commun pour développer avec la méthode BMad : artefacts de cadrage, cycle de story, suivi d'avancement. La charte officialise la pratique éprouvée sur le dépôt pilote `gsi-modeleMT` — North Star → stories → dev TDD → revue croisée → done — et la rend applicable à tous les dépôts. C'est ce cadrage qui rend possible l'autonomie des agents décrite dans la charte Claude : un agent ne peut travailler seul que si le quoi, le pourquoi et les règles sont écrits quelque part.

**Trois principes directeurs :**

1. **Pas de code sans story, pas de story sans North Star.** Tout changement se rattache à une intention écrite et traçable.
2. **Le cycle est standard, les personas sont libres.** Un dev qui change de projet retrouve le même cycle partout ; chaque projet choisit ses agents.
3. **Les artefacts BMad sont du code.** Versionnés, relus, maintenus — un artefact périmé est un bug de documentation.

---

## 1. Périmètre & installation

### Périmètre — tout code livré

- BMad s'applique à **tout dépôt qui produit du code maintenu** : socle (`gsi-modeleMT`), apps clientes (ERP, Conservatoire, GED, MyAccessWeb…), migrations WinDev, outils internes.
- **Seules exceptions** : scripts jetables et POC, à condition d'être **explicitement marqués comme tels** dans le README du dépôt. Un POC qui devient un produit entre dans le périmètre — avec rattrapage du North Star avant toute nouvelle fonctionnalité.

### Installation — dès le jour 1

- BMad s'installe **à la création du dépôt**, au même titre que le kit garde-fous de la [Charte Claude §4](charte-claude-gsinformatique.md#4-autonomie-des-agents) dont il fait partie.
- **Config** : `_bmad/config.toml` est géré par l'installeur — on ne l'édite jamais directement. Les choix d'équipe vivent dans `_bmad/custom/config.toml` (commité), les préférences personnelles dans `_bmad/custom/config.user.toml` (gitignoré).
- **Langue des artefacts : français** (`document_output_language = "French"`), conforme aux conventions GSI.
- Les artefacts vivent dans **`_bmad-output/`**, versionné dans le dépôt.

---

## 2. North Star — obligatoire avant le code

Aucune story ne se crée tant que les quatre artefacts North Star ne sont pas en place :

| Artefact | Rôle |
|---|---|
| **`project-brief.md`** | Le **pourquoi** du projet — besoin, valeur, périmètre |
| **`architecture.md`** | Le **comment** — structure cible, choix techniques |
| **`project-context.md`** | Les **règles cardinales** pour les agents IA — la source de vérité |
| **`epics.md`** | Le **backlog** structuré en épics et stories |

- **`project-context.md` fait autorité** : en cas de contradiction avec un autre document, c'est lui (et les ADRs) qu'on suit. Le `CLAUDE.md` du dépôt reste lean et **renvoie** vers lui — il ne le duplique pas (pattern gsi-modeleMT).
- Le North Star est **amendé par PR** comme le reste du code. Une décision qui le contredit passe par un **ADR**, pas par une dérive silencieuse.
- L'investissement de départ est assumé : c'est lui qui permet ensuite les runs autonomes et l'arrivée d'un nouveau dev (humain ou agent) sans transfert oral.

---

## 3. Cycle de story — le standard GSI

### Le cycle, identique sur tous les projets

```
create-story → dev (TDD) → revue LLM croisée → done → sprint-status à jour
```

- **L'orchestrateur crée les stories et ne code jamais** ; il délègue l'implémentation à l'agent dev. Séparation stricte des rôles.
- **L'agent dev implémente en TDD**, dans le respect du `project-context.md`.
- **Un modèle différent relit** — conditions et traçabilité dans la [Charte Claude §3](charte-claude-gsinformatique.md#3-review--responsabilité).
- **Le fichier de story est le dossier de la story** : AC cochés, Completion Notes, File List, Change Log. Tout arbitrage pris en cours de route (par un humain ou un agent) y est **gravé au Change Log**.

### Suivi — `sprint-status.yaml`

- `sprint-status.yaml` est la **source de vérité de l'avancement** : toute story y a un état à jour (backlog, in-progress, review, done, parked).
- Une story **parked** porte un **motif réel et vérifiable** (dépendance précise, décision en attente, échéance) — jamais un motif générique.

### Personas — libres par projet

- Chaque projet choisit ses agents/personas (Winston, Amelia, ou d'autres) et les **déclare dans son `CLAUDE.md`**.
- Ce qui est non négociable, c'est le cycle et la séparation des rôles ci-dessus — pas les noms.

---

## 4. Circuit court — `quick-dev` pour le trivial

Tout ne mérite pas une story formelle. Le circuit court officiel est **`bmad-quick-dev`** (ou équivalent), réservé aux changements **triviaux** :

- **≤ ~50 lignes modifiées** ;
- **zéro changement de comportement métier** ;
- **zéro changement de schéma de données, d'auth ou de permissions**.

Restent obligatoires même en quick-dev : **commit conventionnel, revue LLM croisée, CI verte**. La documentation pure (typo, reformulation) est dispensée de story mais pas de PR.

**En cas de doute sur le caractère trivial : story.** Un « petit fix » qui touche au métier n'est pas trivial.

---

## 5. Runs autonomes

Les runs autonomes (`/autonomie-globale`, story-automator) sont soumis aux conditions de la [Charte Claude §4](charte-claude-gsinformatique.md#4-autonomie-des-agents) (garde-fous verts, lignes rouges), plus trois exigences propres à BMad :

1. **Dry-run avant le run** : cartographie des stories candidates (proceed / park avec motif) validée avant de lancer. On ne lance pas un run autonome sur un backlog flou.
2. **Handoff documenté en fin de run** : un document de passation (`HANDOFF-run-autonome-<date>.md`) résume ce qui a été fait, les arbitrages pris et ce qui reste — l'humain qui supervise a posteriori lit ça, pas 200 commits.
3. **Stories sensibles → session supervisée** : toute story touchant les garde-fous eux-mêmes (self-modification, lint rules de conformité, zones suspendues) est exclue des runs autonomes et se traite avec un humain dans la boucle.

---

## 6. Hygiène & gouvernance

- **Clôture d'épic = passe d'hygiène documentaire** : à chaque épic terminé, on vérifie que le North Star, les conventions et le `CLAUDE.md` reflètent l'état réel (convention `cloture-epic-hygiene-docs` du socle). Un épic n'est pas fini tant que la doc ment.
- **Mises à jour de BMad** (installeur, nouvelles versions de workflows) : par le **référent** (le même que pour la charte Claude), testées sur le socle avant généralisation.
- **Revue annuelle** de la charte, alignée sur celles des chartes GitHub et Claude. Un changement majeur de BMad déclenche une relecture sans attendre.

---

## Adoption

Cette charte n'a de valeur que si elle est appliquée par défaut. Pour démarrer :

1. Inventorier les dépôts actifs : BMad installé ? North Star complet ? Combler les manques avant toute nouvelle fonctionnalité.
2. `gsi-modeleMT` est la **référence canonique** : config, structure `_bmad-output/`, fichiers de story, sprint-status — à copier, pas à réinventer.
3. Tour d'équipe de 30 min : cycle de story (§3) et frontière du quick-dev (§4) en priorité.
4. Ajuster après 2–3 semaines de pratique — via PR sur ce fichier.
