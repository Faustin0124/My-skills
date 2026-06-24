# Patterns de dette technique — Stack Pradio

Ce fichier recense les patterns de dette les plus fréquents sur la stack Pradio. À lire pendant la Phase 1 pour enrichir la détection.

---

## React 18 + TypeScript 5.8

### Patterns à détecter

**`any` implicites dans les props**
```tsx
// ❌ Dette
const MyComponent = ({ data }: { data: any }) => ...

// ✅ Correction
const MyComponent = ({ data }: { data: SimulationResult }) => ...
```

**`useEffect` avec dépendances manquantes ou vides**
```tsx
// ❌ Dette courante — masque des bugs de stale closure
useEffect(() => {
  fetchData(userId); // userId utilisé mais absent des deps
}, []); 

// Détecter avec :
grep -rn "useEffect" src/ --include="*.tsx" | grep "\[\]"
```

**Composants > 200 lignes dans src/components/**
- Signal : mélange de logique métier + UI dans le même composant
- Solution : extraire la logique dans un hook `src/hooks/use{Domain}.ts`

**Context utilisé pour du state serveur**
- Anti-pattern : utiliser React Context pour cacher des données Supabase
- Solution : migrer vers TanStack React Query (déjà dans la stack)

**Détection :**
```bash
grep -rn "createContext\|useContext" src/ --include="*.tsx" --include="*.ts" | grep -v "hooks\|provider" | head -20
```

---

## TanStack React Query 5

### Patterns à détecter

**`staleTime` non configuré (= 0 par défaut)**
```bash
# Requêtes sans staleTime explicite = re-fetch inutile à chaque focus
grep -rn "useQuery\|useMutation" src/ --include="*.tsx" --include="*.ts" | grep -v "staleTime" | wc -l
```

**Invalidation trop large (`queryClient.invalidateQueries()` sans filtre)**
```bash
grep -rn "invalidateQueries()" src/ --include="*.tsx" --include="*.ts"
```

**Gestion d'erreur absente sur les mutations critiques (Stripe, Supabase)**
```bash
grep -rn "useMutation" src/ --include="*.tsx" -A 10 | grep -L "onError"
```

---

## Zod + React Hook Form

### Patterns à détecter

**Schémas Zod dupliqués entre frontend et Edge Functions**
```bash
# Comparer les schémas dans src/ vs supabase/functions/
grep -rn "z\.object" src/ --include="*.ts" --include="*.tsx" -l
grep -rn "z\.object" supabase/functions/ --include="*.ts" -l
```

**Validation Zod absente dans les Edge Functions**
```bash
# Toute Edge Function qui lit req.json() sans parse Zod
grep -rL "\.parse\|\.safeParse" supabase/functions/*/index.ts
```

**Schémas de validation fiscale non partagés**
- Les 7 régimes fiscaux ont probablement des schémas redondants
- Candidat à une extraction dans `src/lib/schemas/fiscal.ts`

---

## Supabase

### Patterns critiques (P0/P1)

**RLS absent ou incomplet**
```sql
-- Détecter les tables sans RLS dans les migrations
-- Chercher CREATE TABLE sans ENABLE ROW LEVEL SECURITY dans la même migration
```
```bash
grep -l "CREATE TABLE" supabase/migrations/*.sql | while read f; do
  if ! grep -q "ENABLE ROW LEVEL SECURITY" "$f"; then
    echo "RLS manquant : $f"
  fi
done
```

**Types générés désynchronisés**
```bash
# Comparer la date du type généré vs la dernière migration
echo "Types générés :" && stat src/integrations/supabase/types.ts | grep Modify
echo "Dernière migration :" && ls -la supabase/migrations/ | tail -2
```
- Si la migration est plus récente → régénérer les types : `supabase gen types typescript --local > src/integrations/supabase/types.ts`

**Appels Supabase directs dans les composants (hors hooks)**
```bash
# Anti-pattern : supabase.from() dans un composant = pas de cache, pas de retry
grep -rn "supabase\.from\|supabase\.auth" src/components/ --include="*.tsx" -l
```

**Edge Functions sans CORS correct**
```bash
# Vérifier que corsHeaders est importé depuis _shared/
grep -rL "_shared/cors\|corsHeaders" supabase/functions/*/index.ts 2>/dev/null
```

**Migrations irréversibles sans commentaire**
```bash
# Migrations avec DROP sans explication
grep -rn "DROP TABLE\|DROP COLUMN" supabase/migrations/ | head -10
```

---

## Vitest — Couverture de tests

### Priorités de couverture

**P0 — Calculs fiscaux (zéro tolérance)**
- `src/lib/profitabilityCalculations.ts`
- `src/lib/creditCalculations.ts`
- `src/lib/validation.ts`
- Tout fichier `src/lib/*Calculations*.ts` ou `src/lib/*fiscal*.ts`

**P1 — Hooks métier**
- `src/hooks/` — tout hook qui appelle Supabase ou fait un calcul

**P2 — Composants critiques**
- Formulaires fiscaux (7 régimes)
- Composants de paiement Stripe

**Template de test Vitest pour calcul pur :**
```typescript
import { describe, it, expect } from 'vitest';
import { calculerMicroFoncier } from '../profitabilityCalculations';

describe('calculerMicroFoncier', () => {
  it('applique l\'abattement de 30% sur les revenus bruts', () => {
    const result = calculerMicroFoncier({ revenusAnnuels: 10000, chargesReelles: 2000 });
    expect(result.revenuNet).toBe(7000); // 10000 * 0.7
  });

  it('bascule vers régime réel si charges > 30%', () => {
    const result = calculerMicroFoncier({ revenusAnnuels: 10000, chargesReelles: 4000 });
    expect(result.recommandation).toBe('regime-reel');
  });
});
```

---

## i18next

### Patterns à détecter

**Clés de traduction manquantes (string hardcodées en FR)**
```bash
# Textes français hardcodés dans le JSX (heuristique : chaînes > 15 chars en français)
grep -rn '"[A-ZÀ-Ù][a-zà-ù ]\{15,\}"' src/components/ --include="*.tsx" | head -20
grep -rn "'[A-ZÀ-Ù][a-zà-ù ]\{15,\}'" src/components/ --include="*.tsx" | head -20
```

**Clés définies dans les fichiers de traduction mais non utilisées**
```bash
# Lister les clés dans les fichiers i18n
find src/ -name "*.json" -path "*/locales/*" | head -5
```

---

## Performance

### Vite / Bundle

**Imports non tree-shakés**
```bash
# Import de tout un module au lieu d'un export nommé
grep -rn "import \* as" src/ --include="*.ts" --include="*.tsx" | grep -v "// "
```

**Recharts importé entièrement**
```bash
# Préférer les imports nommés
grep -rn "from 'recharts'" src/ --include="*.tsx" | grep "import \*"
```

**Composants lourds sans lazy loading**
```bash
# Pages/sections lourdes sans React.lazy
grep -rn "React.lazy\|lazy(" src/ --include="*.tsx" | wc -l
# Comparer avec le nb de routes
grep -rn "<Route" src/ --include="*.tsx" | wc -l
```

---

## GitHub Actions

### Patterns à détecter

**Actions sans version épinglée (risque supply chain)**
```bash
grep -rn "uses:" .github/workflows/ | grep -v "@v[0-9]\|@[a-f0-9]\{40\}" | head -10
```

**Secrets non utilisés ou redondants**
```bash
grep -rn "\${{ secrets\." .github/workflows/ | sed 's/.*secrets\.\([A-Z_]*\).*/\1/' | sort -u
```
