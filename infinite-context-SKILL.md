---
name: infinite-context
description: Gestisci progetti di qualsiasi dimensione con contesto infinito e consumo minimo di token. Pattern ispirato a RLM (Recursive Language Models).
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, Task
user-invocable: true
---

# Infinite Context

Analizza, debugga e refactora progetti di qualsiasi dimensione senza limiti di contesto.

## Principio Chiave

**NON caricare tutto in memoria.** Invece:
1. Mappa la struttura (Glob)
2. Cerca pattern specifici (Grep)
3. Leggi solo ciò che serve (Read con limit)
4. Salva risultati su file (Write)
5. Libera contesto e continua

## Strategia Token

| Operazione | Token | Quando |
|------------|-------|--------|
| Glob | ~100 | Sempre per esplorare |
| Grep files_with_matches | ~200 | Trovare dove cercare |
| Read con limit=50 | ~300 | Leggere parti specifiche |
| Read file intero | ~3000+ | MAI se evitabile |
| Task agent | 0 (separato) | Ricerche esplorative |

## Workflow Universale

### 1. Mappa (non leggere!)
```bash
# Trova tutti i file
Glob **/*.{ts,py,js,go}

# Conta linee per priorità
Bash: find . -name "*.py" -exec wc -l {} + | sort -rn | head -10
```

### 2. Indicizza (grep mirato)
```bash
# Problemi comuni
Grep "TODO|FIXME|HACK"
Grep "console.log|print\(|debugger"
Grep "password|secret|api_key|token"
Grep "eval|exec|innerHTML"
```

### 3. Analizza (lazy loading)
```
Per ogni area di interesse:
1. Read file limit=50 offset=X  # Solo la parte rilevante
2. Analizza
3. Write risultati su file .md
4. Vai avanti (non tenere in memoria)
```

### 4. Sintetizza
```
1. Read tutti i file .md creati
2. Genera report finale
```

## Casi d'Uso

### Review Progetto
"Fai review di src/" → Mappa, scan security, analisi incrementale, report

### Debug
"L'app crasha su login" → Grep login, Read solo file coinvolti, trova root cause

### Refactoring
"File troppo grande" → Analizza struttura, identifica responsabilità, split

### Security Audit
"Trova vulnerabilità" → Grep pattern pericolosi, verifica solo i match

## Task Agent

Per ricerche complesse usa:
```
Task con subagent_type=Explore
```

Vantaggi:
- Contesto separato (non consuma il tuo)
- Può esplorare liberamente
- Ritorna solo il summary

## Anti-Pattern (da evitare)

- Read di tutti i file "per capire il progetto"
- Tenere codice in memoria tra analisi diverse
- Grep content su tutto il codebase
- Leggere file >200 LOC senza limit
