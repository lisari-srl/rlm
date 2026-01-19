# Workflow Dettagliati

## 1. Code Review Completa

### Obiettivo
Identificare bug, vulnerabilità, code smell e aree di miglioramento.

### Step-by-Step

```
STEP 1: Capire il progetto (2 min)
├── Leggi README.md (se esiste)
├── Leggi package.json / requirements.txt / Cargo.toml
├── Identifica: linguaggio, framework, dipendenze
└── Output: comprensione generale

STEP 2: Mappare struttura (3 min)
├── Glob **/*.{ts,js,py,go,rs}
├── Conta file per cartella
├── Identifica pattern architetturale (MVC, Clean, etc.)
└── Output: REVIEW_MAP.md

STEP 3: Quick scan automatico (5 min)
├── Grep "TODO|FIXME|HACK|XXX"
├── Grep "console.log|print\(|debugger"
├── Grep "password|secret|api_key|token"
├── Grep "eval|exec|innerHTML|dangerouslySetInnerHTML"
├── Grep "any" (TypeScript)
├── Grep "except:|catch.*pass" (empty catches)
└── Output: REVIEW_INDEX.md con tutti i match

STEP 4: Review per priorità
├── PRIMA: File con security issues
├── POI: File con più TODO/FIXME
├── POI: File più grandi (> 300 LOC)
├── INFINE: Resto del codice
└── Output: REVIEW_FINDINGS.md incrementale

STEP 5: Report finale
├── Leggi tutti i findings
├── Categorizza: Critical/High/Medium/Low
├── Suggerisci fix concreti
└── Output: REVIEW_REPORT.md
```

### Checklist Review

```markdown
## Security
- [ ] Input validation
- [ ] SQL injection
- [ ] XSS
- [ ] CSRF
- [ ] Authentication/Authorization
- [ ] Secrets hardcoded
- [ ] Dependency vulnerabilities

## Code Quality
- [ ] DRY violations
- [ ] Functions > 50 LOC
- [ ] Files > 300 LOC
- [ ] Nesting > 3 livelli
- [ ] Magic numbers/strings
- [ ] Error handling

## Performance
- [ ] N+1 queries
- [ ] Missing indexes
- [ ] Memory leaks
- [ ] Unnecessary re-renders (React)
- [ ] Missing caching

## Maintainability
- [ ] Naming chiaro
- [ ] Single responsibility
- [ ] Test coverage
- [ ] Documentation
```

---

## 2. Refactoring Guidato

### Obiettivo
Migliorare la struttura del codice senza cambiare comportamento.

### Step-by-Step

```
STEP 1: Identifica target
├── Grep per code smell specifici
├── Trova duplicazioni (funzioni simili)
├── Trova file troppo grandi
└── Output: REFACTOR_TARGETS.md

STEP 2: Prioritizza
├── Impatto: quanto codice tocca?
├── Rischio: quanto è critico?
├── Effort: quanto lavoro richiede?
└── Output: Lista ordinata per ROI

STEP 3: Per ogni refactoring
├── Leggi SOLO il codice coinvolto
├── Verifica test esistenti
├── Applica refactoring
├── Verifica che test passino
└── Commit atomico

STEP 4: Verifica finale
├── Run test suite completa
├── Check che build funzioni
└── Output: REFACTOR_SUMMARY.md
```

### Refactoring Comuni

| Pattern | Grep | Fix |
|---------|------|-----|
| Funzioni lunghe | `wc -l` per funzione | Extract method |
| Duplicazione | Confronto manuale | Extract function/class |
| God class | File > 500 LOC | Split responsibilities |
| Feature envy | Metodi che usano altri oggetti | Move method |
| Magic numbers | `grep -E "[0-9]{2,}"` | Extract constant |

---

## 3. Debug Sistematico

### Obiettivo
Trovare e fixare bug in progetti grandi.

### Step-by-Step

```
STEP 1: Riproduci il bug
├── Capire: cosa dovrebbe succedere?
├── Capire: cosa succede invece?
├── Trovare: come riprodurre?
└── Output: BUG_DESCRIPTION.md

STEP 2: Localizza
├── Grep per keyword dell'errore
├── Grep per funzioni coinvolte
├── Traccia il flusso dati
└── Output: Lista file sospetti

STEP 3: Analizza (lazy)
├── Leggi SOLO i file sospetti
├── Aggiungi log temporanei se serve
├── Identifica root cause
└── Output: ROOT_CAUSE.md

STEP 4: Fix
├── Applica fix minimale
├── Verifica che bug sia risolto
├── Verifica che non ci siano regressioni
└── Output: Fix + test

STEP 5: Cleanup
├── Rimuovi log temporanei
├── Aggiungi test per il bug
├── Documenta se necessario
└── Commit
```

### Debug Patterns

```bash
# Trova dove viene definita una funzione
grep -rn "function functionName\|def functionName\|const functionName"

# Trova dove viene chiamata
grep -rn "functionName("

# Trova dove viene modificata una variabile
grep -rn "variableName\s*="

# Traccia import/export
grep -rn "import.*moduleName\|from.*moduleName\|require.*moduleName"
```

---

## 4. Ottimizzazione Token

### Quanto costa ogni operazione

| Operazione | Token stimati | Quando usare |
|------------|---------------|--------------|
| Glob | ~100 | Sempre per esplorare |
| Grep (files_with_matches) | ~200 | Trovare dove cercare |
| Grep (content, limited) | ~500 | Vedere contesto |
| Read (file piccolo <100 LOC) | ~500 | Analisi dettagliata |
| Read (file medio 100-300 LOC) | ~1500 | Solo se necessario |
| Read (file grande >300 LOC) | ~3000+ | Evitare, usare limit |
| Task agent | ~0 (separato) | Ricerche esplorative |

### Strategie risparmio

```
INVECE DI                          USA
─────────────────────────────────────────────────
Read intero file grande     →     Read con limit=50 + offset
Grep content su tutto       →     Grep files_with_matches prima
Leggere tutti i file        →     Glob + Grep per filtrare
Tenere codice in memoria    →     Write findings su file
Analisi sequenziale         →     Task agent paralleli
```

### Esempio pratico

```
❌ COSTOSO (10000+ token):
1. Read src/big-file.ts (800 LOC)
2. Read src/another-big-file.ts (600 LOC)
3. Read src/yet-another.ts (400 LOC)

✅ ECONOMICO (2000 token):
1. Grep "functionName" → trova in big-file.ts:245
2. Read src/big-file.ts limit=30 offset=230
3. Analizza solo la parte rilevante
```
