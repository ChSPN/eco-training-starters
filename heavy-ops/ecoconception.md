# Diagnostic écoconception — Projet heavy-ops

**Référentiel appliqué** : 115 bonnes pratiques GreenIT (INR)  
**Branches** : `bp18-limiter-requetes-http`, `bp29-supprimer-code-mort`, `bp40-limiter-donnees-json`  
**Date** : 12/06/2026  

---

## 1. Méthodologie de mesure

Toutes les mesures ont été réalisées avec Chrome DevTools sur `http://localhost:5173` (mode dev) sauf indication contraire.

### Outils utilisés

| Outil | Ce qu'il mesure |
|-------|----------------|
| **Lighthouse** | Score performance, accessibilité, bonnes pratiques |
| **Network** | Nombre de requêtes, poids transféré, temps de chargement |
| **Performance** | Timeline de chargement, répartition CPU |
| **Memory** | Consommation mémoire JS (heap snapshot) |
| **Coverage** | % de code CSS/JS inutilisé |
| **Scripts projet** | `node scripts/analyze.mjs` et `node scripts/lighthouse.mjs` |

### Procédure de mesure

#### Lighthouse
1. DevTools → onglet **Lighthouse**
2. Cocher : Performance, Accessibility, Best Practices
3. Device : **Desktop**
4. Cliquer **Analyze page load**
5. Capturer le score circulaire + métriques

#### Network
1. Onglet **Network** → cocher **Preserve log**
2. **Ctrl+Shift+R** (rechargement forcé)
3. Attendre la fin du chargement
4. Capturer : liste complète, résumé en bas, filtres JS/CSS/Img, Headers d'une requête

#### Performance
1. Onglet **Performance** → bouton **⟳ Record+Reload**
2. Attendre l'arrêt automatique
3. Capturer : timeline + Summary (camembert)

#### Memory
1. Onglet **Memory** → **Heap snapshot** → **Take snapshot**
2. Capturer la taille totale

#### Coverage
1. DevTools `⋮` → More tools → **Coverage**
2. Cliquer **⟳** → **Ctrl+Shift+R**
3. Capturer les barres rouge/vert par fichier

---

## 2. Mesures initiales (baseline — branche `main`)

### Scripts automatiques

```
Projet: heavy-ops
Assets: 3 fichiers / 30.4 KB
Data: 2 fichiers / 202.5 KB
Frontend attendu sur http://localhost:5173
Backend attendu sur http://localhost:4100
```

> ⚠️ Les données JSON (202.5 kB) pèsent **6x plus lourd** que les assets (30.4 kB) — ratio inversé.

### Lighthouse Desktop

| Indicateur | Valeur |
|-----------|--------|
| Performance | **87/100** |
| Accessibility | **100/100** |
| Best Practices | **100/100** |
| First Contentful Paint | 1.1 s |
| Largest Contentful Paint | 2.1 s |
| Total Blocking Time | 0 ms |
| Speed Index | 1.1 s |
| Cumulative Layout Shift | 0 |

### Network (chargement initial)

| Indicateur | Valeur |
|-----------|--------|
| Requêtes initiales | **31** |
| Poids transféré | **631 kB** |
| Resources totales | 2.3 MB |
| Finish time | 10.12 s |
| **Requêtes après 30s (polling)** | **+171 requêtes** |

### Performance

| Indicateur | Valeur |
|-----------|--------|
| Scripting | 80 ms |
| System | 60 ms |
| Rendering | 17 ms |
| Painting | 4 ms |
| Total | 5 103 ms |

### Memory

| Indicateur | Valeur |
|-----------|--------|
| Total JS heap | **~12-15 MB** (croissance progressive) |

### Coverage

| Indicateur | Valeur |
|-----------|--------|
| Code inutilisé total | **~48%** (~960 kB / 1.7 MB) |
| chunk-WGEYUP5.js | 46.2% inutilisé (928 kB) |
| react-router-dom.js | 79% inutilisé (209 kB) |
| OpsApp.tsx | 46.2% inutilisé (154 kB) |
| chunk-4MVSNZ2N.js | 53.7% inutilisé (79 kB) |

---

## 3. Bonnes pratiques identifiées et implémentées

Trois BP issues des **115 bonnes pratiques GreenIT (INR)** ont été sélectionnées :

| BP | Titre | Problème ciblé |
|----|-------|---------------|
| **BP 18** | Limiter le nombre de requêtes HTTP | 171+ requêtes de polling |
| **BP 29** | Supprimer le code mort | 48% de code jamais exécuté |
| **BP 40** | Limiter les données transmises | 202.5 kB de JSON pour 30.4 kB d'assets |

---

## 4. BP 18 — Limiter les requêtes HTTP

**Branche** : `bp18-limiter-requetes-http`  
**Fichier modifié** : `frontend/src/OpsApp.tsx`

### Problème

Un `setInterval` dans le `useEffect` principal relançait `loadAll()` toutes les **5 000 ms** en permanence, rechargeant simultanément 4 endpoints (`/api/dashboard`, `/api/records`, `/api/settings`, `/api/analytics`) sans condition, même si les données n'avaient pas changé.

### Modification — 2 lignes supprimées

```typescript
// AVANT
loadAll();
const timer = window.setInterval(loadAll, 5000);
return () => window.clearInterval(timer);

// APRÈS
loadAll();
```

### Mesures avant / après

#### Network (30 secondes d'observation)

| Indicateur | AVANT | APRÈS | Évolution |
|-----------|-------|-------|-----------|
| Nombre de requêtes | **53** | **18** | **-66%** ✅ |
| Données transférées | **1.4 MB** | — | — |
| Requêtes répétitives | **Oui** (toutes les 5s) | **Non** | ✅ Supprimé |

#### Lighthouse Desktop

| Indicateur | AVANT | APRÈS | Évolution |
|-----------|-------|-------|-----------|
| Performance | **87/100** | **87/100** | Stable ✅ |
| Accessibility | **100/100** | **100/100** | Stable ✅ |
| Best Practices | **100/100** | **100/100** | Stable ✅ |

#### Coverage

| Fichier | AVANT | APRÈS | Évolution |
|---------|-------|-------|-----------|
| OpsApp.tsx | **46.2%** inutilisé | **30.8%** inutilisé | **-15.4 pts** ✅ |

### Critères de réussite

| Critère | Objectif | Résultat | Statut |
|---------|----------|----------|--------|
| Réduction des requêtes après 30s | ≥ 50% | **-66%** | ✅ Atteint |
| Suppression du polling répétitif | 0 appel toutes les 5s | **0 setInterval** | ✅ Atteint |
| Score Lighthouse stable ou amélioré | ≥ 87/100 | **87/100** | ✅ Atteint |
| Réduction du code inutilisé | Amélioration visible | **-15.4 pts** | ✅ Atteint |

### Référentiels

| Référentiel | Critère | Description |
|-------------|---------|-------------|
| **115 BP GreenIT** | BP 18 | Limiter le nombre de requêtes HTTP |
| **RGESN** | 4.8 | Limiter la fréquence de rafraîchissement |
| **RGESN** | 4.9 | Éviter les transferts de données inutiles |

---

## 5. BP 29 — Supprimer le code mort

**Branche** : `bp29-supprimer-code-mort`  
**Fichier modifié** : `frontend/vite.config.ts`

### Problème

D'après la Coverage, les fichiers les plus problématiques étaient :
- `chunk-WGEYUP5.js` → 46.2% inutilisé (928 kB)
- `react-router-dom.js` → **79%** inutilisé (209 kB)
- `OpsApp.tsx` → 46.2% inutilisé (154 kB)
- `chunk-4MVSNZ2N.js` → 53.7% inutilisé (79 kB)

Le code mort venait principalement de `react-router-dom` chargé en entier alors que seuls `NavLink`, `Route` et `Routes` étaient utilisés.

### Modification — configuration build Vite optimisée

```typescript
// AVANT
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    strictPort: true,
    proxy: {
      '/api': 'http://localhost:4100',
      '/assets': 'http://localhost:4100'  // ← inutile
    }
  }
});

// APRÈS
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    strictPort: true,
    proxy: {
      '/api': 'http://localhost:4100'
    }
  },
  preview: {
    port: 4173,
    proxy: {
      '/api': 'http://localhost:4100'
    }
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          react: ['react', 'react-dom'],
          router: ['react-router-dom']
        }
      }
    },
    minify: 'esbuild',
    target: 'esnext'
  }
});
```

**Ce que ça fait :**
- `manualChunks` → sépare React et react-router-dom en chunks distincts
- `minify: 'esbuild'` → minification agressive
- `target: 'esnext'` → code moderne sans polyfills inutiles
- Suppression du proxy `/assets` inutile

> ⚠️ Ces optimisations s'appliquent au **build de production**. Mesures effectuées sur `http://localhost:4173` (mode preview) après `npm run build`.

### Mesures avant / après

#### Network (mode preview - localhost:4173)

| Indicateur | AVANT | APRÈS | Évolution |
|-----------|-------|-------|-----------|
| Nb fichiers JS/CSS | **13 fichiers** | **3 fichiers** | **-77%** ✅ |
| Données transférées | **1.4 MB** | **130 kB** | **-90%** ✅ |
| Nb requêtes total | **53** | **35** | **-34%** ✅ |

#### Lighthouse Desktop (mode preview - localhost:4173)

| Indicateur | AVANT | APRÈS | Évolution |
|-----------|-------|-------|-----------|
| Performance | **87/100** | **100/100** | **+13 pts** ✅ |
| Accessibility | **100/100** | **100/100** | Stable ✅ |
| Best Practices | **100/100** | **100/100** | Stable ✅ |

#### Coverage (mode preview - localhost:4173)

| Fichier | AVANT | APRÈS | Évolution |
|---------|-------|-------|-----------|
| Nb fichiers analysés | **13** | **3** | **-77%** ✅ |
| react-router-dom | 209 kB / 79% | 163 kB / 48.4% | **-30.6 pts** ✅ |
| index-CbDSCtln.js | 154 kB / 46.2% | 18 kB / 80.4% | **-136 kB** ✅ |
| CSS | 10 kB | 7 kB / 68.7% | **-30%** ✅ |

### Critères de réussite

| Critère | Objectif | Résultat | Statut |
|---------|----------|----------|--------|
| Réduction poids transféré | ≥ 30% | **-90%** | ✅ Atteint |
| Réduction nb fichiers JS/CSS | Visible | **-77%** | ✅ Atteint |
| Score Lighthouse | ≥ 87/100 | **100/100** | ✅ Atteint |
| Code inutilisé router | Réduction visible | **-30.6 pts** | ✅ Atteint |
| Minification active | Build optimisé | **esbuild actif** | ✅ Atteint |

### Référentiels

| Référentiel | Critère | Description |
|-------------|---------|-------------|
| **115 BP GreenIT** | BP 29 | Supprimer le code mort |
| **RGESN** | 3.5 | Minifier les fichiers CSS et JavaScript |
| **RGESN** | 3.6 | Compresser les fichiers CSS, JavaScript et HTML |

---

## 6. BP 40 — Limiter les données transmises

**Branche** : `bp40-limiter-donnees-json`  
**Fichier modifié** : `backend/src/index.ts`

### Problème

`records.json` contient **180 entrées** (192 kB) mais le frontend n'en affiche que **18** (`records.slice(0, 18)` dans `TablePage`). 162 entrées étaient chargées inutilement, soit **90% des données jamais affichées**.

### Modification — 1 ligne ajoutée

```typescript
// AVANT — retourne les 180 entrées en entier
app.get('/api/records', (_req, res) => {
  res.json(readJson('data/records.json'));
});

// APRÈS — retourne uniquement les 18 entrées affichées
app.get('/api/records', (_req, res) => {
  const records = readJson('data/records.json');
  res.json(records.slice(0, 18));
});
```

### Mesures avant / après

#### Network

| Indicateur | AVANT | APRÈS | Évolution |
|-----------|-------|-------|-----------|
| Poids réponse `/api/records` | **~140 kB** | **14.7 kB** | **-90%** ✅ |
| Entrées chargées | **180** | **18** | **-90%** ✅ |
| Entrées affichées | 18 | 18 | Identique ✅ |

#### Lighthouse Desktop (mode dev - localhost:5173)

| Indicateur | AVANT | APRÈS | Évolution |
|-----------|-------|-------|-----------|
| Performance | **87/100** | **89/100** | **+2 pts** ✅ |
| Accessibility | **100/100** | **100/100** | Stable ✅ |
| Best Practices | **100/100** | **100/100** | Stable ✅ |

#### Coverage

| Fichier | AVANT | APRÈS | Évolution |
|---------|-------|-------|-----------|
| OpsApp.tsx | **46.2%** inutilisé | **30.7%** inutilisé | **-15.5 pts** ✅ |

### Critères de réussite

| Critère | Objectif | Résultat | Statut |
|---------|----------|----------|--------|
| Réduction poids `/api/records` | ≥ 50% | **-90%** | ✅ Atteint |
| Entrées chargées = entrées affichées | 18 = 18 | **18 = 18** | ✅ Atteint |
| Score Lighthouse stable ou amélioré | ≥ 87/100 | **89/100** | ✅ Atteint |
| Zéro donnée superflue transmise | 0 entrée inutile | **162 supprimées** | ✅ Atteint |

### Référentiels

| Référentiel | Critère | Description |
|-------------|---------|-------------|
| **115 BP GreenIT** | BP 40 | Limiter les données transmises |
| **RGESN** | 4.1 | N'échanger que les données nécessaires |
| **RGESN** | 4.9 | Éviter les transferts de données inutiles |

---

## 7. Synthèse des résultats

| BP | Problème | Action | Résultat clé |
|----|----------|--------|-------------|
| **BP 18** | 171+ requêtes polling | Suppression `setInterval` | **-66% requêtes** |
| **BP 29** | 48% code mort | Optimisation build Vite | **-90% poids transféré · Lighthouse 87→100** |
| **BP 40** | 180 entrées pour 18 affichées | Pagination backend | **-90% poids `/api/records`** |

### Impact global estimé

- **Réseau** : réduction massive du trafic HTTP (polling supprimé + données paginées + bundle optimisé)
- **CPU** : moins de cycles serveur (48 requêtes/min → 0 polling)
- **Batterie client** : moins de wake-ups réseau
- **Lighthouse Performance** : 87 → 100 en mode production

---

*Document généré à partir des mesures DevTools réalisées le 12/06/2026.*  
*Référentiel : 115 bonnes pratiques GreenIT (INR) · RGESN v1 — DINUM 2022*