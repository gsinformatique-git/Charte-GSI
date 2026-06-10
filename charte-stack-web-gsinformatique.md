# Charte de la stack web — GSInformatique SA

> **Version 1.0** · Document vivant. Toute proposition d'amendement passe par une PR sur ce fichier.
> Complète les chartes [GitHub](charte-github-gsinformatique.md), [Claude](charte-claude-gsinformatique.md) et [BMad](charte-bmad-gsinformatique.md) — les quatre s'appliquent ensemble.
> **Référence vivante : le socle `gsi-modeleMT`.** En cas de contradiction entre cette charte et un document plus ancien (dont le skill `gsi-webapp`), le socle et cette charte font foi.

## Objectif

Garantir que toutes les applications web GSI partagent la même stack, les mêmes patterns et les mêmes conventions — pour qu'un développeur (humain ou agent) qui change de projet retrouve immédiatement ses repères, et que les acquis d'un projet profitent à tous. La charte nomme les briques et les règles ; le détail vivant (versions exactes, patterns, exemples canoniques) habite le socle `gsi-modeleMT`.

**Trois principes directeurs :**

1. **Une seule stack, un seul exemple canonique.** On imite le socle, on ne réinvente pas.
2. **Les technos sont dans la charte, les versions dans le socle.** La charte ne se périme pas à chaque upgrade.
3. **Toute dérogation est un ADR validé** — jamais une exception silencieuse.

---

## 1. La stack de référence

| Couche | Brique | Rôle |
|---|---|---|
| Framework | **Next.js** (App Router, RSC, Server Actions) + **React** | Server Components par défaut, Client Components justifiés |
| Langage | **TypeScript strict** sur **Node.js** | Pas de `as any` |
| Styling | **Tailwind CSS** (config CSS-first) | Design tokens partagés via `@gsi/ui` |
| ORM | **Prisma** (double schéma `common` + `tenant`) | Isolation tenant, accès uniquement via `@gsi/db` |
| BDD | **PostgreSQL** | `timestamptz`, `snake_case` via `@map` |
| Stockage objets | **MinIO** | Auto-hébergé |
| Auth | **Auth.js** Credentials + **TOTP** | 4 rôles, matrice de permissions ; pas de provider OAuth externe |
| i18n | **next-intl** (fr/en/de/it, défaut `fr-CH`) | Aucun libellé en dur |
| Validation | **Zod** à toute frontière HTTP | Erreurs métier via `DomainError` → `handleDomainError` |
| Tests | **Vitest** (unit/intégration) + **Playwright** (E2E) | Preuve navigateur exigée (charte Claude §3) |
| Lint | **ESLint** + règles custom `gsi/*` | Les conventions deviennent des lint rules dès que possible |
| Monorepo | **Turborepo + pnpm** | Socle et packages `@gsi/*` |
| Logs | **Pino** via `@gsi/logging` | Jamais `console.*` en prod |
| PDF / Excel / E-mail | **@react-pdf/renderer** · **write-excel-file** · **Nodemailer** | Génération côté serveur |
| Hash | **bcryptjs** | Pas de dépendance native |

**Versions : le socle fait foi.** Les versions exactes (Next 16, Prisma 6, PostgreSQL 16-17…) vivent dans le `package.json` et le `CLAUDE.md` du socle — pas ici. **Règle d'upgrade** : toute montée de version majeure se valide d'abord sur le socle, puis se propage aux apps ; jamais l'inverse.

---

## 2. Architecture — le socle et les apps

- **`gsi-modeleMT` est le socle** : `apps/admin` + packages `@gsi/*` + outillage + conventions. C'est la **référence canonique** de toute nouvelle app.
- **Chaque app cliente** (ERP, Conservatoire, GED, MyAccessWeb…) vit **dans son propre dépôt** et consomme les `@gsi/*` publiés (registre privé).
- **Promotion par la Rule of Three** : un module naît local à son app ; il est promu en `@gsi/modules-*` dès le **2ᵉ consommateur**, avec mini-ADR. On ne crée jamais un module directement dans le socle par spéculation.
- **L'exemple à imiter** : le module Contact (`@gsi/modules-contact`), le plus mature. Réflexe avant de coder : « où est l'exemple canonique ? »
- **Le pattern standalone historique** (skill `gsi-webapp`, POC) est **déprécié pour toute nouvelle app**. Ses règles contraires au socle — soft delete `estActif` sur données personnelles, libellés UI en dur, BDD Neon/Supabase — sont **caduques** ; le skill sera réaligné (voir Adoption).

---

## 3. Conventions cardinales

Les invariants que toute app respecte — le détail exhaustif vit dans le `project-context.md` du socle, qui **fait foi** ; la charte ne le duplique pas.

- **Langue** : français métier / anglais techno dans le code (`civilites`, `findContactById`) ; doc, commits et messages utilisateur **100 % français**.
- **IDs** : `BigInt @id` pour les entités métier (sérialisées en `string` côté API) ; `String cuid` pour `User`.
- **Financier** : `Decimal` Prisma — jamais `number` JS ; devises ISO 4217, locales BCP 47.
- **Cycle de vie des entités — 3 mécanismes strictement séparés** :
  - référentiels → flag `estSelectionnable` ;
  - transactionnel → champ `statut` + énum métier ;
  - données personnelles → **hard delete réel + `audit_log`** (conformité **LPD**, ADR-001). Le soft-delete déguisé sur des données personnelles est une non-conformité, pas un choix technique.
- **Interdits** (lint rules `gsi/*` existantes ou planifiées) : `console.*` en prod, `if (user.role === …)` éparpillé, import direct de `@prisma/client`, libellé en dur dans le JSX, composant Client par défaut, `as any`, `<img>` HTML.
- **Taille de fichier** : cible ≤ 250 LOC ; au-delà de 400 → `// EXEMPTION:` + ADR.

---

## 4. Hébergement & services — souveraineté

- **Hébergement : Infomaniak ou le data center GSI.** Jamais Vercel, AWS, Azure, Cloudflare, Netlify, Fly.io ou équivalent.
- **Tout service de la stack doit être auto-hébergeable** (PostgreSQL, MinIO, SMTP, monitoring…). Un SaaS externe (Neon, Supabase, Sentry cloud, Stripe SaaS…) est interdit sauf dérogation §5.
- Le **pourquoi** : maîtrise des données (LPD, clientèle suisse), indépendance vis-à-vis des hyperscalers, prévisibilité des coûts. Ce n'est pas une préférence, c'est un engagement vis-à-vis des clients.

---

## 5. Dérogations — ADR + validation référent

- Toute brique hors stack (lib, BDD, service, hébergeur imposé par un client…) exige un **ADR motivé** — besoin, alternatives standard écartées, plan de sortie — **validé par le référent avant usage**, pas après.
- Une dérogation adoptée par un **2ᵉ puis 3ᵉ projet** devient candidate au standard : on amende la charte plutôt que de multiplier les exceptions (même Rule of Three que pour les modules).
- Une brique hors stack découverte **sans ADR** est une dette : on la régularise (ADR rétroactif ou retrait) dès sa découverte.

---

## 6. Gouvernance

- **Référent** (le même que pour les chartes Claude et BMad) : garde la cohérence socle ↔ apps, valide les ADRs de dérogation, pilote les upgrades majeurs et la publication des `@gsi/*`.
- **Revue annuelle** de la charte, alignée sur les trois autres. Un changement majeur d'une brique structurante (Next, Prisma, Auth.js) déclenche une relecture sans attendre.
- Les conventions ont vocation à devenir des **lint rules** : une règle qui reste « de la discipline » est une règle qui finira par être violée.

---

## Adoption

Cette charte n'a de valeur que si elle est appliquée par défaut. Pour démarrer :

1. **Réaligner le skill `gsi-webapp`** sur le socle : corriger soft delete → 3 mécanismes LPD, Neon/Supabase → Infomaniak/data center, libellés en dur → i18n.
2. **Inventorier les apps existantes** : écarts vs cette charte → liste de dette technique par app, versionnée dans le dépôt de l'app.
3. Tour d'équipe de 30 min : cycle de vie des entités / LPD (§3) et règle de dérogation (§5) en priorité.
4. Ajuster après 2–3 semaines de pratique — via PR sur ce fichier.
