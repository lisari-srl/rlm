---
name: project-review
description: Review interi progetti con contesto infinito e consumo minimo di token. Usa lazy loading, indicizzazione e summarizzazione progressiva.
allowed-tools: Read, Grep, Glob, Bash, Write, Edit, Task
user-invocable: true
---

# Project Review - Contesto Infinito

Questa skill ti permette di fare review, refactoring e debug di progetti di qualsiasi dimensione consumando pochi token.

## Principi Fondamentali

1. **MAI caricare tutto il codice** - Leggi solo quello che serve
2. **Mappa prima, leggi dopo** - Crea un indice della struttura
3. **Summarizza e dimentica** - Dopo aver analizzato, salva i finding e libera il contesto
4. **Procedi per moduli** - Un componente alla volta

## Workflow di Review

### Fase 1: Mappatura (5 minuti, pochi token)

```
1. Glob per trovare tutti i file sorgente
2. Conta linee per file (wc -l)
3. Identifica entry points (main, index, app)
4. Mappa dipendenze da package.json/requirements.txt/Cargo.toml
5. Crea REVIEW_MAP.md con la struttura
```

### Fase 2: Indicizzazione (in background)

```
1. Grep per pattern critici:
   - TODO, FIXME, HACK
   - console.log, print, debugger
   - password, secret, key, token
   - try/catch vuoti
   - any, unknown (TypeScript)
2. Salva risultati in REVIEW_INDEX.md
```

### Fase 3: Review Incrementale

Per ogni modulo/componente:
```
1. Leggi SOLO i file del modulo
2. Analizza per: bug, security, performance, code smell
3. Scrivi findings in REVIEW_FINDINGS.md
4. NON tenere il codice in memoria - summarizza e vai avanti
```

### Fase 4: Report Finale

```
1. Leggi tutti i REVIEW_*.md
2. Prioritizza: Critical > High > Medium > Low
3. Genera REVIEW_REPORT.md con azioni concrete
```

## Comandi Utili

### Mappatura veloce
```bash
# Struttura progetto
find . -type f -name "*.py" | head -50

# Conteggio linee per cartella
find ./src -name "*.ts" -exec wc -l {} + | sort -n

# File più grandi (probabili problemi)
find . -name "*.js" -exec wc -l {} + | sort -rn | head -10
```

### Pattern da cercare
```bash
# Security issues
grep -r "eval\|exec\|innerHTML" --include="*.js"

# Hardcoded secrets
grep -rn "password\|api_key\|secret" --include="*.py"

# Debug code dimenticato
grep -rn "console.log\|debugger\|print(" --include="*.ts"
```

## Template Output

### REVIEW_MAP.md
```markdown
# Project Map

## Struttura
- src/ (15 file, 2400 LOC)
  - components/ (8 file, 1200 LOC)
  - services/ (4 file, 800 LOC)
  - utils/ (3 file, 400 LOC)

## Entry Points
- src/index.ts - Main entry
- src/app.ts - App setup

## Dipendenze Critiche
- express: 4.18.0
- prisma: 5.0.0
```

### REVIEW_FINDINGS.md
```markdown
# Findings

## [CRITICAL] SQL Injection in user.service.ts:45
- Query costruita con string concatenation
- FIX: Usare prepared statements

## [HIGH] No rate limiting in auth.controller.ts
- Endpoint /login vulnerabile a brute force
- FIX: Aggiungere rate limiter

## [MEDIUM] Console.log in production
- 12 occorrenze in src/services/
- FIX: Rimuovere o usare logger
```

## Regole per Consumare Meno Token

1. **Usa Glob invece di Read per esplorare** - Glob costa ~100 token, Read di un file grande costa migliaia

2. **Usa Grep con output_mode="files_with_matches"** - Trova dove cercare senza leggere tutto

3. **Leggi file con limit** - `Read file_path limit=100` per vedere solo l'inizio

4. **Delega a Task agent** - Per analisi parallele usa subagent che non consumano il tuo contesto

5. **Scrivi findings immediatamente** - Non tenere in memoria, scrivi su file

6. **Usa head_limit in Grep** - `grep pattern --head_limit 20` per limitare risultati

## Esempio Completo

```
Utente: "Fai review del progetto src/"

1. [Glob] *.ts, *.tsx -> 45 file trovati
2. [Bash] wc -l -> 8500 LOC totali
3. [Write] REVIEW_MAP.md con struttura
4. [Grep] TODO|FIXME -> 23 match in 8 file
5. [Grep] console.log -> 45 match in 12 file
6. [Write] REVIEW_INDEX.md con findings iniziali
7. [Read] src/auth/ (3 file, 400 LOC) - modulo critico
8. [Write] Aggiungi findings auth a REVIEW_FINDINGS.md
9. [Read] src/api/ (5 file, 600 LOC)
10. [Write] Aggiungi findings api
... continua per ogni modulo ...
N. [Read] Tutti i REVIEW_*.md
N+1. [Write] REVIEW_REPORT.md finale
```

## Quando Usare Task Agent

Usa `Task` con `subagent_type=Explore` per:
- Cercare pattern complessi in codebase grandi
- Analisi parallele di moduli indipendenti
- Domande specifiche ("dove viene usata questa funzione?")

```
Il Task agent ha il suo contesto separato, quindi:
- Non consuma il tuo contesto
- Può esplorare liberamente
- Ti ritorna solo il summary
```
