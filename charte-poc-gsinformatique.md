# Charte des POC — GSInformatique SA

> **Version 1.0** · Document vivant. Toute proposition d'amendement passe par une PR sur ce fichier.
> Complète les chartes [GitHub](charte-github-gsinformatique.md), [Claude](charte-claude-gsinformatique.md), [BMad](charte-bmad-gsinformatique.md) et [stack web](charte-stack-web-gsinformatique.md). En cas de tension, cette charte précise **comment ces règles s'appliquent au cas particulier d'un POC** — elle ne les remplace pas.

## Objectif

Cadrer la façon dont GSI crée, configure, **déploie** et clôt ses **proofs of concept** : des dépôts dédiés à **prouver une faisabilité ou une valeur**, vite et à moindre coût, avant tout engagement. Un POC n'est pas une petite app de production — c'est un brouillon assumé, montrable à un client ou à l'équipe. La charte garantit que cette liberté reste **sûre** (secrets, données personnelles) et **réversible** (jetable sans regret, ou consolidable sans douleur dans le socle).

Elle s'appuie sur les deux POC de référence de l'organisation : **`poc-dessange`** (web, déployé sur `poc.gsinfo.ch`) et **`poc-brand`** (outil MCP, local).

**Trois principes directeurs :**

1. **Un POC prouve une seule chose, vite, et se jette sans regret.** Périmètre serré, durée courte.
2. **Un POC ne va jamais en production.** Le serveur `poc.gsinfo.ch` est un serveur de **démonstration**, pas de prod. Ce qui doit vivre se **réécrit** dans le socle via un ADR — jamais un copier-coller de POC en prod.
3. **La liberté du POC porte sur la stack, pas sur la sécurité.** Mêmes règles fermes que partout : aucun secret versionné, aucune donnée personnelle réelle (LPD).

---

## 1. Deux familles de POC

| Famille | Exemple | Forme | Exécution |
| ------- | ------- | ----- | --------- |
| **POC web** (démo client / équipe) | `poc-dessange` | app conteneurisée (Next.js, FastAPI…) | **déployé sur `poc.gsinfo.ch`** (voir §5) |
| **POC outil / MCP** (capacité pour une IA ou un script) | `poc-brand` | serveur MCP, CLI, script | **local** (`localhost`), branché à Claude via `claude mcp add` |

La famille détermine le mode de déploiement : un POC web va sur le serveur de démonstration ; un POC outil reste local tant qu'il n'a pas besoin d'être exposé. Un POC répond à **une question unique** ; si elle se dédouble, on fait deux POC.

---

## 2. Nommage & emplacement

- **Nommage** : `poc-<sujet>` en minuscules — `<sujet>` = le client, la techno ou la question (ex. `poc-dessange`, `poc-brand`). Cohérent avec la [charte GitHub §4](charte-github-gsinformatique.md).
- **Emplacement** : dans l'organisation `gsinformatique-git`, en **privé**. Pas de POC sur un compte personnel — un POC dispersé est un POC perdu.
- **Accès** : via les équipes (`dev` / `lead` / `admin`), jamais en droits individuels.

---

## 3. Configuration minimale du dépôt

Tout POC contient, dès le premier commit :

- **`README.md`** : le but en une phrase, le **statut** (voir §7), **comment lancer en local** (commandes, prérequis), et — pour un POC web — une section **Déploiement** pointant vers `poc.gsinfo.ch`.
- **`.gitignore` robuste** dès la création : `.env`, `.env.deploy`, sources de référence non versionnées, environnements jetables (`.venv/`, `node_modules/`), artefacts.
- **`.env.example`** (et **`.env.deploy.example`** pour un POC web) documentant les variables attendues, **sans valeurs**.
- **Fiche d'identité** : objectif, contexte, périmètre *Inclus / Exclus* et statut — dans le README ou un `PDR.md` dédié. Tout ce qui n'est pas indispensable à la preuve est **Exclus** par défaut.

**Structure de README recommandée** (modèle `poc-dessange`) : (1) titre + **statut** + mention des données ; (2) ce que fait l'app ; (3) **stack technique** ; (4) structure du projet ; (5) **données & sources** ; (6) tests ; (7) **déploiement** (POC web → §5) ; (8) limites connues / pistes. Un lecteur doit pouvoir lancer le POC et comprendre ses limites en lisant ce seul fichier.

---

## 4. Stack — liberté encadrée

- Un POC **peut sortir de la stack standard** : c'est souvent son rôle (explorer une techno, un driver, un framework). **Aucun ADR n'est exigé** pour le choix de techno d'un POC — c'est l'exception légitime prévue par la [charte stack web §4](charte-stack-web-gsinformatique.md).
- En contrepartie, **un POC reste un POC** : démonstration seulement, pas de production.
- Si une consolidation dans le socle est envisagée, **prévoir le couplage minimal dès le départ** : isoler l'adhérence aux données / au tenant dans **un seul fichier** clairement identifié (cf. `poc-brand`, `src/tenants.ts`), pour que la reprise se fasse sans démêlage.

---

## 5. Le serveur de démonstration `poc.gsinfo.ch`

Serveur unique qui expose les **POC web** en HTTPS, sous un domaine commun, pour les montrer à un client ou à l'équipe. **Référence d'implémentation : `poc-dessange`.**

### 5.1 Architecture

- **Un domaine, plusieurs POC par préfixe d'URL.** Chaque POC est servi sous `https://poc.gsinfo.ch/<préfixe>` (ex. `/dessange`). Convention : `<préfixe>` = le nom du dépôt sans `poc-`.
- **Caddy en façade** (`caddy:2-alpine`) : ports **80/443**, **HTTPS automatique via Let's Encrypt** (compte ACME : `daniel.vallon@gsinformatique.ch`). Caddy vit dans `caddy/` avec son **propre `docker-compose.yml`** ; les certificats persistent dans le volume **`caddy_data` — à ne jamais supprimer**.
- **Réseau Docker externe partagé `proxy`** : créé **une seule fois** (`docker network create proxy`). Caddy et **toutes** les apps y sont connectés.
- **Les apps ne publient aucun port** : elles ne sont joignables **que par Caddy** via le réseau `proxy`. C'est l'isolation réseau — rien n'est exposé en direct sur Internet sauf Caddy.
- **basePath obligatoire** : l'app doit être servie sous son préfixe (ex. `NEXT_PUBLIC_BASE_PATH=/dessange`), sinon les liens et les assets cassent.
- **Persistance par volumes Docker nommés** (base SQLite, uploads…), jamais dans l'image. Données = volumes.
- **Administration via Tailscale.** Les coordonnées d'accès (hôtes, identifiants, nom Tailscale) vivent dans **`.env.deploy`** — **jamais versionné** ; un `.env.deploy.example` documente les variables.

### 5.2 Ajouter un POC au serveur

1. **Conteneuriser** l'app (`Dockerfile`), avec `basePath = /<préfixe>`, **aucun port publié**, et rattachement au réseau externe `proxy` dans son `docker-compose.yml`.
2. **Router dans Caddy** : ajouter un bloc `handle /<préfixe>* { reverse_proxy <conteneur>:<port> }` dans le `Caddyfile`, **avant le catch-all** final, puis recharger Caddy.
3. **Lancer** : `docker compose up -d` du POC (le réseau `proxy` et Caddy doivent déjà tourner).
4. **Vérifier** : `https://poc.gsinfo.ch/<préfixe>` répond en HTTPS.

### 5.3 Mise en route du serveur (une fois)

```bash
docker network create proxy          # réseau partagé Caddy ↔ apps (une seule fois)
(cd caddy && docker compose up -d)    # le reverse proxy (ports 80/443)
```

### 5.4 Règles du serveur

- Le serveur ne sert **que des POC** : pas de production, **aucune donnée personnelle réelle**, données de démonstration uniquement.
- **Conserver `caddy_data`** (certificats Let's Encrypt) lors de toute manipulation, pour éviter de réémettre les certificats.
- **Retirer un POC** : supprimer son bloc dans le `Caddyfile`, `docker compose down` (et ses volumes si la donnée est jetable).
- L'accès d'administration reste **derrière Tailscale** ; seul le couple 80/443 de Caddy est public.

### 5.5 Exploitation & mise à jour

Sur le serveur (Debian, accès derrière Tailscale) :

```bash
docker compose ps              # état des conteneurs
docker compose logs -f app     # logs en direct
docker compose restart app     # redémarrer l'app
```

**Mettre à jour** : renvoyer le code, puis `docker compose up -d --build`. Le conteneur **applique lui-même les migrations et le seed initial** au démarrage (via son entrypoint) — aucune étape de base de données manuelle.

### 5.6 Sécurité de l'exposition publique

Un POC servi sur `poc.gsinfo.ch` est **public** et, par défaut, **sans authentification** : n'importe qui peut agir dessus. En conséquence :

- **Aucune donnée réelle ni personnelle** (rappel §6) — non négociable sur un serveur public.
- **Protéger au minimum les fonctions sensibles** (back-office, édition) si le POC les expose.
- Le POC ne se durcit pas pour devenir une prod par glissement : pour un usage réel, **réécriture sur le socle** `gsi-modeleMT` (Auth.js, PostgreSQL, i18n).

---

## 6. Données & conformité (LPD)

- **Aucune donnée personnelle réelle** dans un POC, jamais — ni en base, ni en fixture, ni dans un prompt. On utilise des **données fictives** (ex. le client fictif « Favre ») ou **anonymisées**.
- **Données externes** (scraping, catalogue tiers — ex. catalogue Dessange) : mention claire de la **source** et de l'usage **« démonstration uniquement »** dans le README ; pas de rediffusion ni d'usage commercial.
- En cas de doute sur la nature personnelle d'une donnée, on l'exclut ou on l'anonymise — la charge de la preuve est sur le POC.

---

## 7. Statut & cycle de vie

Le **statut** est visible dans le README, et tenu à jour :

| Statut | Sens |
| ------ | ---- |
| 🚧 **En cours** | preuve en construction |
| ✅ **Validé** | la question a une réponse (positive) |
| ❌ **Abandonné** | preuve négative ou piste écartée — on documente *pourquoi* |
| ⬆️ **Promu** | consolidé dans le socle ou une app ; le POC passe en lecture seule |

**Trois sorties possibles** — un POC ne reste jamais indéfiniment « en cours » :

1. **Jeté** : archivé (dépôt en lecture seule) ou supprimé, et **retiré du serveur** (§5.4). Une preuve négative documentée a de la valeur — on archive plutôt que d'effacer.
2. **Consolidé** : réécrit aux normes dans le socle / une app, via un **ADR** qui acte la décision. Le POC est marqué ⬆️ et figé.
3. **Gardé comme référence** : explicitement, avec un statut clair, sinon il devient un zombie.

**Anti-zombie** : un POC sans activité depuis **6 mois** est revu — archivé, clôturé et retiré du serveur. On n'accumule ni dépôts ni conteneurs morts.

---

## 8. Process Git — régime allégé

Un POC est typiquement **mono-développeur et jetable** : le rituel PR complet de la [charte GitHub §2](charte-github-gsinformatique.md) y est disproportionné.

- **Travail direct sur `main` autorisé** sur un POC solo — dérogation **explicite** à la règle « tout passe par une PR », justifiée par la nature jetable.
- **Non négociable** : la **push protection anti-secrets** active, le `.gitignore` robuste, le `.env.example` / `.env.deploy.example`.
- **Bascule en régime standard** dès qu'un POC devient **collaboratif** (≥ 2 contributeurs) **ou** candidat à consolidation : protection de `main`, PR + revue redeviennent obligatoires.

---

## 9. Gouvernance

- **Référent** (le même que pour les autres chartes) : valide le passage d'un POC ⬆️ vers le socle (ADR), administre `poc.gsinfo.ch` et arbitre l'anti-zombie.
- **Rule of three** : un même besoin prouvé par un 2ᵉ puis 3ᵉ POC est un signal — on consolide dans le socle plutôt que de multiplier les preuves.
- **Revue** : alignée sur les autres chartes ; la liste des POC, leurs statuts et les POC encore servis sur `poc.gsinfo.ch` sont passés en revue au moins une fois par an.

---

## Adoption

1. **Régulariser les POC existants** : statut à jour et fiche d'identité pour `poc-dessange` et `poc-brand` ; rapatrier dans l'org tout POC resté sur un compte personnel.
2. **Documenter l'ajout d'un POC au serveur** (§5.2) dans le README de chaque POC web, en s'alignant sur `poc-dessange`.
3. **Trancher le régime Git** des POC (§8) et l'appliquer à la protection de branche des dépôts `poc-*`.
4. Ajuster après quelques POC de pratique — via PR sur ce fichier.
