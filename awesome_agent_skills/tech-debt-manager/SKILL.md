---
name: tech-debt-manager
description: >
  Expert en gestion de dette technique pour Pradio (React 18 + TypeScript 5.8 + Vite + Supabase + shadcn/ui). 
  Utilise ce skill pour tout ce qui concerne la dette technique : audits complets du codebase, détection de code smells, 
  dépendances obsolètes ou vulnérables, problèmes architecturaux, absence de tests, dette de sécurité, et génération 
  de backlogs d'actions exécutables par l'agent Claude Code. Déclenche dès que l'utilisateur mentionne : "dette technique", 
  "refactoring", "audit code", "code smell", "dépendances obsolètes", "sécurité", "performance", "couverture de tests", 
  "nettoyage", "amélioration architecture", "dead code", "duplication", ou demande un état des lieux du codebase. 
  Déclenche aussi sur des formulations implicites : "le code est devenu un peu lourd", "on a accumulé des trucs moches", 
  "faut qu'on nettoie ça", "c'est du legacy", "ça part dans tous les sens".
---

# Tech Debt Manager — Pradio

Skill orienté **action** pour Claude Code. L'objectif n'est pas de produire un rapport — c'est de produire un **backlog exécutable** que l'agent traite directement dans le repo.

## Stack de référence

- **Frontend** : React 18, TypeScript 5.8, Vite 5 (SWC), Tailwind CSS 3, shadcn/ui, Radix UI
- **State/Data** : TanStack React Query 5, React Context, React Hook Form, Zod
- **Routing** : React Router v6 | **Charts** : Recharts | **PDF** : @react-pdf/renderer | **i18n** : i18next
- **Backend** : Supabase (PostgreSQL, Auth, Storage, Edge Functions Deno)
- **Paiements** : Stripe | **Email** : Resend
- **Tests** : Vitest (unit), k6 (charge)
- **CI/CD** : GitHub Actions (workflow_dispatch only) → Lovable

---

## Workflow principal

### Phase 1 — Audit du repo

Avant de proposer quoi que ce soit, **lire le repo** pour constituer une image fidèle de l'état actuel.

#### 1.1 Audit des dépendances

```bash
# Dépendances obsolètes
npm outdated --json

# Vulnérabilités connues
npm audit --json

# Dépendances inutilisées
npx depcheck --json

# Taille des bundles (Vite)
npx vite-bundle-visualizer 2>/dev/null || echo "bundle-visualizer non installé"
```

#### 1.2 Analyse statique TypeScript

```bash
# Erreurs TypeScript non bloquantes ignorées
npx tsc --noEmit --strict 2>&1 | head -100

# Fichiers avec "any" explicites
grep -r ": any" src/ --include="*.ts" --include="*.tsx" -l

# Assertions de type forcées (as X)
grep -r " as " src/ --include="*.ts" --include="*.tsx" | grep -v "// " | wc -l

# TODO / FIXME / HACK dans le code
grep -rn "TODO\|FIXME\|HACK\|XXX\|@ts-ignore\|@ts-expect-error" src/ --include="*.ts" --include="*.tsx"
```

#### 1.3 Détection de code smells

```bash
# Fichiers trop longs (>300 lignes = candidat à la décomposition)
find src/ -name "*.tsx" -o -name "*.ts" | xargs wc -l | sort -rn | head -20

# Composants React avec trop de props (proxy: lignes dans l'interface Props)
grep -r "interface.*Props" src/ -A 30 --include="*.tsx" | grep -c "  [a-z]"

# Fonctions trop longues dans lib/ (calculs purs)
find src/lib/ -name "*.ts" | xargs wc -l | sort -rn | head -10

# Duplication : imports identiques dans >5 fichiers (candidats à un hook/util partagé)
grep -r "^import" src/ --include="*.tsx" | sed 's/.*from //' | sort | uniq -c | sort -rn | head -20

# Dead code potentiel : exports non utilisés
grep -r "^export " src/ --include="*.ts" --include="*.tsx" -l | head -20
```

#### 1.4 Couverture de tests

```bash
# Couverture globale
npx vitest run --coverage 2>&1 | tail -30

# Fichiers dans src/lib/ sans test associé (calculs purs = priorité haute)
for f in src/lib/*.ts; do
  base=$(basename "$f" .ts)
  if ! find src/ -name "*${base}*test*" -o -name "*${base}*spec*" 2>/dev/null | grep -q .; then
    echo "SANS TEST: $f"
  fi
done

# Hooks sans test
for f in src/hooks/*.ts src/hooks/*.tsx; do
  base=$(basename "$f" | sed 's/\.[^.]*$//')
  if ! find src/ -name "*${base}*test*" -o -name "*${base}*spec*" 2>/dev/null | grep -q .; then
    echo "SANS TEST: $f"
  fi
done
```

#### 1.5 Audit Supabase / Edge Functions

```bash
# Migrations sans rollback
find supabase/migrations/ -name "*.sql" | xargs grep -L "DROP\|ALTER.*DROP" | head -10

# RLS manquant sur les tables (heuristique : migrations sans ENABLE ROW LEVEL SECURITY)
grep -L "ROW LEVEL SECURITY" supabase/migrations/*.sql | head -10

# Edge Functions sans gestion d'erreur explicite
grep -L "try\|catch\|error" supabase/functions/*/index.ts 2>/dev/null

# Types Supabase générés désynchronisés (date du fichier vs dernière migration)
ls -la src/integrations/supabase/types.ts 2>/dev/null
ls -la supabase/migrations/ | tail -3
```

#### 1.6 Audit sécurité

```bash
# Clés/secrets dans le code source
grep -rn "sk_live\|sk_test\|service_role\|SUPABASE_SERVICE\|password.*=.*['\"]" src/ supabase/functions/ --include="*.ts" --include="*.tsx" 2>/dev/null | grep -v ".env"

# Variables d'environnement non typées (accès direct sans validation)
grep -rn "process\.env\.\|import\.meta\.env\." src/ --include="*.ts" --include="*.tsx" | grep -v "VITE_" | head -20

# Inputs Zod manquants dans les Edge Functions
grep -L "z\.\|zod" supabase/functions/*/index.ts 2>/dev/null
```

---

### Phase 2 — Scoring et priorisation

Pour chaque item détecté, applique cette grille de scoring :

| Critère | Faible (1) | Moyen (2) | Élevé (3) |
|---|---|---|---|
| **Impact sécurité** | Cosmétique | Exposition de données | Faille exploitable |
| **Impact utilisateur** | Invisible | UX dégradée | Erreur/crash |
| **Effort de correction** | < 30 min | 30 min – 2h | > 2h |
| **Propagation** | 1 fichier | Module | Cross-domain |
| **Fréquence d'occurrence** | Isolé | Quelques occurrences | Pattern systémique |

**Priorité = (Impact sécurité × 2) + Impact utilisateur + (4 - Effort) + Propagation + Fréquence**

Tiers de priorité :
- **P0** (score ≥ 14) : Blocant — traiter immédiatement
- **P1** (score 10–13) : Haute priorité — sprint courant
- **P2** (score 6–9) : Normale — backlog planifié
- **P3** (score < 6) : Nice-to-have — si opportunité

---

### Phase 3 — Génération du backlog

Produis un fichier `TECH_DEBT_BACKLOG.md` à la racine du repo.

#### Structure d'un item de backlog

```markdown
## [TD-XXX] Titre court et actionnable

**Priorité** : P0 / P1 / P2 / P3  
**Catégorie** : Sécurité | TypeScript | Tests | Architecture | Dépendances | Performance | Supabase  
**Effort estimé** : XS (< 30min) / S (30min–2h) / M (2h–4h) / L (4h–8h) / XL (> 8h)  
**Fichiers concernés** : `src/...`, `supabase/...`

### Problème
Description factuelle de ce qui a été détecté. Inclure les métriques (nb de fichiers, lignes, occurrences).

### Impact
Ce qui se passe si on ne corrige pas (risque sécurité, dette accumulée, bug potentiel).

### Actions de correction

- [ ] Étape 1 — description précise (fichier, fonction, transformation)
- [ ] Étape 2 — ...
- [ ] Étape 3 — ...

### Critère d'acceptation
Comment vérifier que c'est fait (test qui passe, commande qui retourne 0, métrique améliorée).

---
```

#### En-tête du backlog

```markdown
# Tech Debt Backlog — Pradio

> Généré le : {DATE}  
> Commit analysé : {GIT_HASH}  
> Score dette global : {X}/100 (100 = aucune dette détectée)

## Résumé exécutif

| Catégorie | P0 | P1 | P2 | P3 | Total |
|---|---|---|---|---|---|
| Sécurité | | | | | |
| TypeScript | | | | | |
| Tests | | | | | |
| Architecture | | | | | |
| Dépendances | | | | | |
| Supabase | | | | | |
| **Total** | | | | | |

## Items P0 — Traiter immédiatement
...

## Items P1 — Sprint courant
...

## Items P2 — Backlog planifié
...

## Items P3 — Nice-to-have
...
```

---

### Phase 4 — Exécution par l'agent

Après validation du backlog par l'utilisateur, l'agent traite les items **dans cet ordre strict** :

1. **P0 Sécurité** — jamais d'exécution automatique, toujours confirmation explicite
2. **P0 autres** — exécution directe si non-destructif (ajout de types, suppression de `any`, etc.)
3. **P1** — exécution directe pour les corrections XS/S
4. **P2/P3** — exécution si l'utilisateur demande explicitement

#### Règles d'exécution

**Exécution directe autorisée (sans demander) :**
- Mise à jour de dépendances patch/minor non-breaking
- Suppression de `@ts-ignore` avec remplacement par le bon type
- Extraction d'une constante/fonction dans `src/lib/`
- Ajout d'un test unitaire Vitest sur une fonction pure
- Ajout de validation Zod dans une Edge Function

**Toujours demander confirmation avant :**
- Mise à jour major d'une dépendance (React, Vite, TanStack Query...)
- Refactoring d'un composant qui touche à plusieurs domaines fonctionnels
- Toute modification dans `supabase/migrations/` (irréversible)
- Changement de structure de dossiers
- Modification de la logique de calcul fiscal (`src/lib/profitabilityCalculations.ts`, etc.)

**Jamais toucher sans pair review explicite :**
- `src/lib/*Calculations.ts` — calculs fiscaux critiques
- `supabase/migrations/` — migrations de base
- Stripe webhooks et Edge Functions de paiement

---

## Référence : patterns de dette fréquents sur cette stack

Lire `references/debt-patterns.md` pour les patterns spécifiques à cette stack (React Query, Zod, Supabase RLS, Vitest, i18next).

---

## Output attendu

À la fin de chaque run, l'agent produit :

1. **`TECH_DEBT_BACKLOG.md`** — backlog complet avec tous les items scorés
2. **Un résumé en prose** dans le chat (max 15 lignes) : score global, nb d'items par priorité, les 3 items P0 si existants, recommandation sur quoi traiter en premier
3. **Si des corrections ont été appliquées** : liste des fichiers modifiés avec une ligne de description par fichier
