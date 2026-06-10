# Charte d'utilisation de Claude — GSInformatique SA

> **Version 1.0** · Document vivant. Toute proposition d'amendement passe par une PR sur ce fichier.
> Complète la [Charte GitHub](charte-github-gsinformatique.md) — les deux s'appliquent ensemble.
> **Périmètre : le développement.** L'usage bureautique de Claude (offres, documents, e-mails) sera traité dans un document séparé.

## Objectif

Donner à toute l'équipe (Suisse / France / Algérie, travail asynchrone) un cadre commun et explicite pour développer avec Claude — du prompt interactif dans Claude Code jusqu'aux agents autonomes qui déroulent des stories de bout en bout. La charte officialise la pratique éprouvée sur le dépôt pilote `gsi-modeleMT` (revue LLM croisée, hooks de garde, preuve navigateur, ADR-022) et la rend applicable à tous les dépôts.

**Trois principes directeurs :**

1. **L'humain reste responsable de ce qui est livré.** Il ne relit pas chaque ligne — il répond des garde-fous qui les ont validées, et il assume le résultat.
2. **Aucune donnée sensible dans un prompt** : ni secret, ni donnée personnelle réelle. L'anonymisation est obligatoire, pas optionnelle.
3. **L'autonomie se mérite, dépôt par dépôt.** Pas de garde-fous installés → pas de merge autonome.

---

## 1. Outils & comptes

### Outils

- **Claude Code** (CLI, app desktop, extension IDE) est l'outil de développement IA **standard et supporté** de GSI : c'est lui que les skills, hooks, `CLAUDE.md` et conventions outillent.
- **Autres outils IA** (Copilot, Cursor, ChatGPT…) : tolérés en expérimentation personnelle. Les mêmes règles s'appliquent intégralement (données §2, review §3), et **rien n'entre dans un dépôt GSI sans passer par le circuit de validation de cette charte**. Un outil tiers qui fait ses preuves peut être proposé au standard — via PR sur cette charte.

### Comptes & licences

- Chaque développeur dispose d'un **abonnement Claude individuel payé par GSI** (Pro/Max selon le besoin), rattaché à son **adresse e-mail GSI**. Pas de compte personnel pour du travail GSI ou client.
- Les **clés API** Anthropic (agents, automatisations, produits) sont des secrets : gérées par le référent, jamais versionnées, jamais partagées en clair (voir [Charte GitHub §3](charte-github-gsinformatique.md#3-sécurité--secrets)).
- Au départ d'un collaborateur : résiliation de l'abonnement et rotation des clés API auxquelles il avait accès, le jour même — même logique que les accès GitHub.

---

## 2. Données & confidentialité

> Pilier prioritaire. Ce qu'on envoie à Claude part chez un tiers (Anthropic) : on applique les mêmes réflexes que pour tout sous-traitant.

### Règles fermes

- **Aucun secret** dans un prompt, un `CLAUDE.md`, un skill, un hook ou un contexte : mots de passe, clés API, chaînes de connexion, tokens. Mêmes règles que pour Git.
- **Anonymisation obligatoire** : aucune donnée personnelle réelle (noms, adresses, données RH, médicales, financières…) dans un prompt ou un fichier lu par Claude. Pour déboguer une migration de données réelles : **pseudonymiser d'abord** (noms fictifs, identifiants tronqués) ou utiliser des jeux de test fictifs. C'est la même rigueur **LPD** que celle déjà ancrée dans le code (hard delete + audit_log, ADR-001) — appliquée aux prompts.
- **Projets sous NDA client** : vérifier que le contrat n'interdit pas le recours à un outil IA tiers avant d'utiliser Claude sur le code du client. En cas de doute, on demande au référent avant, pas après.

### Autorisé sans question

- Notre propre code source, nos schémas de données, notre documentation technique.
- Des données de test fictives ou correctement pseudonymisées.
- Tout contenu déjà public.

---

## 3. Review & responsabilité

### La règle d'or

**Celui qui livre en est responsable, que Claude ait écrit le code ou non.** « C'est Claude qui l'a généré » n'est jamais une excuse — ni pour un bug, ni pour une faille, ni pour une non-conformité LPD.

### Revue LLM croisée — la règle officielle

- Toute PR est relue par un **LLM différent du modèle qui a écrit le code** (ex. le code écrit par Opus/Fable est relu par Sonnet). Le relecteur travaille en adversaire : conformité aux conventions, cas limites, sécurité, duplication de l'existant.
- **Une revue LLM croisée approuvée vaut approbation de PR.** L'humain n'intervient que lorsque la revue signale un doute, un désaccord entre modèles, ou un sujet listé comme sensible dans le `CLAUDE.md` du dépôt.
- **Traçabilité** : le verdict de revue est consigné dans l'historique Git par un **commit docs dédié** — pattern éprouvé sur gsi-modeleMT :
  ```
  docs(story): revue Sonnet approuve 10-1 présence numérique et seuils
  ```
  Le détail (modèle relecteur, findings, patches appliqués) vit dans le fichier de suivi de la story. Une PR sans trace de revue ne se merge pas.

> Cette règle **amende la [Charte GitHub §2](charte-github-gsinformatique.md#2-pull-requests--reviews)** : la review reste obligatoire, mais elle peut être portée par un LLM croisé. La review humaine garde tout son sens comme outil de partage de connaissance — elle n'est simplement plus le goulot d'étranglement systématique.

### Definition of Done — preuve, pas affirmation

- **Claude peut affirmer qu'un code fonctionne sans l'avoir exécuté.** La parade est systématique : tout changement touchant l'auth, l'UI ou le comportement runtime exige un **smoke test dans un vrai navigateur** (Playwright ou skill `verify`). **curl ne compte pas** comme preuve.
- Vigilance spécifique au code généré, pour le relecteur (LLM ou humain) : dépendances inventées ou obsolètes, gestion d'erreurs trop optimiste, code plausible mais faux sur les cas limites, réinvention de logique existant déjà dans le projet.
- Les commits générés respectent la charte Git : **1 commit = 1 chose qui fonctionne**, Conventional Commits en français.

---

## 4. Autonomie des agents

### Le principe — autonomie complète encadrée

Les agents Claude (story-automator, autonomie-globale, agents parallèles en worktrees) peuvent dérouler une story **de bout en bout, merge compris**, à condition que **tous** les garde-fous soient verts :

1. **CI verte** (lint, type-check, tests) ;
2. **Hooks de garde actifs** (`pretooluse-worktree-guard`, `stop-ui-smoke-guard`) ;
3. **Preuve navigateur** quand la story touche auth/UI/runtime (§3) ;
4. **Revue LLM croisée approuvée et tracée** (§3).

L'humain supervise **a posteriori** : il lit les verdicts de revue et l'historique, il n'est pas un point de passage bloquant. C'est le modèle de l'ADR-022 (orchestration autonome), généralisé.

### Le kit garde-fous — obligatoire avant tout travail Claude

**Tout dépôt actif doit installer le kit minimal avant qu'on y développe avec Claude :**

- un **`CLAUDE.md`** à la racine (stack, conventions, commandes, pièges, sujets sensibles nécessitant un humain) ;
- la **CI** (lint, type-check, tests) ;
- les **hooks de garde** du socle, adaptés au contexte du dépôt.

Pas de kit → pas de travail Claude sur ce dépôt, et a fortiori pas d'autonomie. Le kit s'installe en premier, comme le `.gitignore` robuste de la charte Git. Le socle `gsi-modeleMT` fournit la référence à copier.

### Lignes rouges — toujours un humain

Quelles que soient les couleurs des garde-fous, restent des actes humains :

- toute action **destructive ou irréversible** hors du dépôt : suppression de données, migration destructive en production, rotation de secrets ;
- tout **déploiement en production** ;
- toute communication **sortante vers un client** ;
- la modification des **garde-fous eux-mêmes** (hooks, CI, cette charte) — un agent ne désactive jamais ce qui le contrôle.

---

## 5. Capitalisation

Ce qu'on apprend avec Claude se partage — pas dans la tête de chacun.

- Le **`CLAUDE.md`** de chaque dépôt est versionné et amendé par PR : ce qu'un collègue apprend à Claude (piège, convention, commande) profite à toute l'équipe. Garder le fichier **lean** : il renvoie aux sources détaillées (ADRs, conventions, project-context), il ne les duplique pas.
- Un pattern qui se répète devient un **skill partagé** (modèle : `gsi-webapp`, pipeline `wl-recup`…), pas un prompt recopié de mémoire. Avant d'en créer un, vérifier qu'il n'existe pas déjà.
- Une décision technique prise avec ou pour Claude (nouveau garde-fou, exception, règle d'autonomie spécifique) se documente en **ADR**, comme toute décision d'architecture. Règle manquante → ADR plutôt qu'improvisation.

---

## 6. Gouvernance

- **Référent Claude** (direction technique) : centralise les abonnements, les clés API, les arbitrages NDA (§2), la liste des dépôts ayant le kit garde-fous, et l'évolution de cette charte.
- **Revue annuelle** de la charte, alignée sur la revue des accès GitHub. Un changement majeur (nouveau modèle, nouvelle capacité d'autonomie, changement des conditions Anthropic) déclenche une relecture sans attendre l'échéance.
- **En cas d'incident** (donnée réelle partie dans un prompt, merge autonome défaillant, garde-fou contourné) : corriger immédiatement, puis **documenter l'incident** et ajuster la charte ou les hooks. Même réflexe que pour un secret commité.

---

## Adoption

Cette charte n'a de valeur que si elle est appliquée par défaut. Pour démarrer :

1. Amender la charte GitHub §2 (revue LLM croisée) — fait dans la même PR que cette charte.
2. Vérifier que chaque dev a son abonnement GSI et aucun compte personnel en usage pro.
3. Établir la liste des dépôts actifs et leur état vis-à-vis du kit garde-fous ; planifier l'installation là où il manque.
4. Tour d'équipe de 30 min : anonymisation (§2) et conditions d'autonomie (§4) en priorité.
5. Ajuster après 2–3 semaines de pratique — via PR sur ce fichier.
