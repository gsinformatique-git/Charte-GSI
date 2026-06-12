# Charte d'utilisation GitHub — GSInformatique SA

> **Version 1.2** · Document vivant. Toute proposition d'amendement passe par une PR sur ce fichier.
> Voir aussi la [Charte Claude](charte-claude-gsinformatique.md) pour le développement assisté par IA.

## Objectif

Donner à toute l'équipe (Suisse / France / Algérie, travail asynchrone) des réflexes communs et explicites pour collaborer sur GitHub sans friction et sans surprise. La charte privilégie la simplicité : on optimise pour une équipe distribuée et hétérogène, pas pour un process maximal.

**Trois principes directeurs :**

1. `main` est toujours déployable.
2. Tout passe par une Pull Request — jamais de push direct sur `main`.
3. Un secret ne rentre jamais dans un dépôt.

---

## 1. Branches & commits

### Modèle de branches — GitHub Flow adapté

- **`main`** : branche de référence, toujours déployable, protégée (voir §4 et §5). On ne pousse jamais directement dessus.
- **Branches de travail** : courtes (idéalement mergées en quelques jours), créées depuis `main`, supprimées après merge.
- **Branches d'intégration** : pour un gros chantier qui s'étale dans le temps (ex. une réécriture), on crée une branche longue (ex. `multi-tenant-rewrite`) sur laquelle on greffe des sous-branches courtes. Elle est mergée dans `main` une fois stabilisée.

### Convention de nommage des branches

```
type/description-courte
```

Types : `feat`, `fix`, `refactor`, `chore`, `docs`, `test`.

Exemples : `feat/datalist-pagination`, `fix/contact-civilite-abreviation`, `refactor/auth-common`.

### Messages de commit — Conventional Commits

```
type(scope): description à l'impératif
```

- **Types** : `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `perf`, `style`.
- **Scope** : optionnel, le module concerné (ex. `auth`, `datalist`, `contact`).
- **Langue** : une seule par dépôt, choisie une fois pour toutes. Recommandé : anglais pour les commits techniques, français pour la doc métier.

Exemples :
```
feat(datalist): add server-side pagination
fix(contact): correct civilité abbreviation mapping
docs(dette-technique): close BACKFILL-DUAL-MUTATIONS
```

### Règle d'or — granularité

**1 commit = 1 chose qui fonctionne.** Smoke test avant et après. Si un changement ne tient pas dans un commit cohérent et testable, c'est qu'il doit être découpé. Cette règle s'applique aussi aux commits générés via Claude Code.

---

## 2. Pull Requests & reviews

### Ouverture

- Toute intégration dans `main` (ou dans une branche d'intégration) passe par une PR.
- **Taille** : viser des PR courtes (< ~400 lignes modifiées). Une grosse PR est difficile à relire en async et retarde tout le monde. On découpe.
- **Description obligatoire**, en trois points :
  - **Quoi** : ce que fait la PR.
  - **Pourquoi** : le besoin ou le bug derrière.
  - **Comment tester** : les étapes pour valider, surtout utile pour une review en décalé horaire.

### Review

- **Au moins 1 review approuvée** avant merge. C'est non négociable, y compris pour les profils seniors.
- *(Amendement v1.1)* La review peut être portée par une **revue LLM croisée** (un modèle différent de celui qui a écrit le code relit et approuve), tracée dans l'historique Git — conditions et traçabilité dans la [Charte Claude §3](charte-claude-gsinformatique.md#3-review--responsabilité). La review humaine reste précieuse comme outil de partage de connaissance dans une équipe hétérogène, et reste requise sur les sujets sensibles listés dans le `CLAUDE.md` du dépôt.
- Le reviewer relit le **quoi** et le **comment**, pas seulement la syntaxe.
- En cas de désaccord bloquant, on tranche en synchrone (visio courte) plutôt qu'en fil de commentaires interminable.

### Merge

- **CI verte obligatoire** avant merge (lint, build, tests).
- **Squash merge** recommandé : un historique `main` propre, une PR = une entrée lisible.
- L'auteur supprime sa branche après merge.

---

## 3. Sécurité & secrets

> Pilier prioritaire. Une fuite de credentials est coûteuse et difficile à corriger après coup.

### Règles fermes

- **Aucun secret dans un dépôt** : mots de passe, clés API, chaînes de connexion DB, tokens, fichiers `.env`. Jamais, même temporairement, même dans une branche, même dans un fichier de config destiné à Claude (`CLAUDE.md`, `architect-context.md`, etc.).
- Les secrets vivent dans des **variables d'environnement** ou un gestionnaire de secrets, jamais versionnés.
- Chaque dépôt a un **`.gitignore` robuste** dès sa création : `.env`, `*.local`, `*.key`, dumps DB, etc.
- Fournir un **`.env.example`** documentant les variables attendues, sans les valeurs.

### Détection

- Activer **GitHub Secret Scanning** + **Push Protection** sur l'organisation : GitHub bloque le push si un secret est détecté.

### En cas de fuite

Si un secret a été commité par erreur :

1. **Révoquer / régénérer immédiatement** le secret concerné (clé API, mot de passe DB…). Le supprimer du code ne suffit pas : il reste dans l'historique Git et a pu être cloné.
2. Purger l'historique si nécessaire (`git filter-repo`) — en coordination, car ça réécrit l'historique partagé.
3. Documenter l'incident.

---

## 4. Gouvernance & accès

### Organisation

- Un compte **Organisation GitHub** unique pour GSInformatique (pas de dépôts dispersés sur des comptes perso).
- **Équipes GitHub par rôle** (ex. `dev`, `lead`, `admin`) plutôt que des droits attribués individuellement, dépôt par dépôt.

### Droits — principe du moindre privilège

- Chacun reçoit le niveau d'accès minimal nécessaire (`write` pour les devs, `maintain`/`admin` réservés).
- Le rôle **Owner** est limité à 2 personnes maximum (toi + un suppléant), pour éviter le point de défaillance unique.
- Revue des accès au moins une fois par an, et au départ/arrivée de chaque collaborateur.

### Dépôts

- **Convention de nommage** cohérente, en minuscules *(Amendement v1.2)* : `gsi-<projet>` pour les apps et socles internes (ex. `gsi-modeleMT`, `gsi-ressources`), `poc-<sujet>` pour les preuves de concept (ex. `poc-brand`, `poc-odbc-hfsql`). Le préfixe situe le dépôt d'un coup d'œil. La casse interne reste libre mais cohérente au sein d'un dépôt.
- Chaque dépôt contient au minimum un **`README.md`** (quoi, comment lancer en local) et une copie/lien vers cette charte (`CONTRIBUTING.md`).

### Protection de `main`

Branch protection à activer sur `main` de chaque dépôt actif :

- Push direct interdit.
- PR requise avec ≥ 1 approbation.
- Statuts CI requis avant merge.
- Historique linéaire (cohérent avec le squash merge).

---

## Adoption

Cette charte n'a de valeur que si elle est appliquée par défaut. Pour démarrer :

1. Activer la branch protection et le secret scanning sur `gsi-modeleMT` (dépôt pilote).
2. Faire un tour d'équipe de 30 min pour présenter les conventions de commits et de PR.
3. Ajuster après 2–3 semaines de pratique — via PR sur ce fichier.
