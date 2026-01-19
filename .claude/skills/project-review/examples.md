# Esempi di Utilizzo

## Esempio 1: Review Completa di un Progetto Node.js

**Richiesta utente:**
> "Fai una review completa del progetto nella cartella src/"

**Esecuzione Claude Code:**

```
🔍 FASE 1: Mappatura

[Glob] **/*.ts → 45 file trovati
[Glob] **/*.tsx → 12 file trovati
[Bash] find ./src -name "*.ts" -exec wc -l {} + | sort -rn | head -10

Risultato:
  892 ./src/services/payment.service.ts    ⚠️ File grande
  654 ./src/controllers/user.controller.ts ⚠️ File grande
  423 ./src/models/order.model.ts
  ...

[Write] REVIEW_MAP.md
```

```
🔍 FASE 2: Quick Scan

[Grep] "TODO|FIXME" → 23 match
[Grep] "console.log" → 45 match ⚠️
[Grep] "password|secret" → 8 match ⚠️⚠️
[Grep] "any" (TypeScript) → 67 match ⚠️

[Write] REVIEW_INDEX.md
```

```
🔍 FASE 3: Analisi Prioritaria

# Prima i file con "password|secret"
[Read] src/config/database.ts limit=50
→ Trovato: DB_PASSWORD hardcoded! CRITICAL

[Read] src/services/auth.service.ts limit=100
→ JWT_SECRET in codice! CRITICAL

[Write] Aggiungi findings a REVIEW_FINDINGS.md
```

```
🔍 FASE 4: Report

[Read] REVIEW_FINDINGS.md
[Write] REVIEW_REPORT.md

# Output finale
## CRITICAL (2)
- Hardcoded DB password in config/database.ts:12
- Hardcoded JWT secret in services/auth.service.ts:8

## HIGH (5)
- 45 console.log in production code
- 67 uses of 'any' type
- No input validation in user.controller.ts
...
```

---

## Esempio 2: Debug di un Bug Specifico

**Richiesta utente:**
> "L'app crasha quando un utente fa login con email vuota"

**Esecuzione Claude Code:**

```
🔍 FASE 1: Localizza codice login

[Grep] "login|Login" files_with_matches →
  src/controllers/auth.controller.ts
  src/services/auth.service.ts
  src/routes/auth.routes.ts

[Grep] "email" in auth files →
  auth.controller.ts:23, 45, 67
  auth.service.ts:12, 34
```

```
🔍 FASE 2: Analizza flusso

[Read] src/controllers/auth.controller.ts limit=30 offset=20
→ Vedo: const { email, password } = req.body
→ Nessuna validazione!

[Read] src/services/auth.service.ts limit=20 offset=10
→ Vedo: user = await User.findOne({ email })
→ Se email è vuota/undefined → query invalida
```

```
🔍 FASE 3: Root Cause

ROOT CAUSE: Nessuna validazione dell'email prima della query.
Quando email è stringa vuota "", la query MongoDB cerca { email: "" }
che non trova nulla, poi il codice fa user.id che crasha.

[Write] BUG_ANALYSIS.md
```

```
🔍 FASE 4: Fix

[Edit] src/controllers/auth.controller.ts
+ if (!email || !password) {
+   return res.status(400).json({ error: 'Email and password required' });
+ }

[Bash] npm test → All tests pass
```

---

## Esempio 3: Refactoring di File Troppo Grande

**Richiesta utente:**
> "Il file payment.service.ts è troppo grande, aiutami a splittarlo"

**Esecuzione Claude Code:**

```
🔍 FASE 1: Analizza struttura

[Bash] wc -l src/services/payment.service.ts → 892 linee

[Read] src/services/payment.service.ts limit=50
→ Vedo le funzioni esportate

[Grep] "export.*function|export.*class|export.*const" →
  - processPayment (linea 45)
  - refundPayment (linea 156)
  - validateCard (linea 234)
  - calculateTax (linea 312)
  - generateInvoice (linea 445)
  - sendReceipt (linea 567)
  - ... altre 8 funzioni
```

```
🔍 FASE 2: Identifica responsabilità

[Read] payment.service.ts offset=45 limit=100
→ processPayment, refundPayment = core payment logic

[Read] payment.service.ts offset=234 limit=70
→ validateCard = validation logic

[Read] payment.service.ts offset=445 limit=120
→ generateInvoice, sendReceipt = document generation

Proposta split:
├── payment.service.ts (core: process, refund)
├── payment-validation.service.ts (validate)
├── invoice.service.ts (generate, send)
└── tax.service.ts (calculate)
```

```
🔍 FASE 3: Esegui refactoring

[Write] src/services/payment-validation.service.ts
[Write] src/services/invoice.service.ts
[Write] src/services/tax.service.ts
[Edit] src/services/payment.service.ts (rimuovi codice spostato)
[Edit] Aggiorna import nei file che usano queste funzioni

[Bash] npm test → Verifica che tutto funzioni
```

---

## Esempio 4: Analisi Progetto Grande (Lazy)

**Richiesta utente:**
> "Ho un monorepo con 500+ file, trova i problemi di security"

**Esecuzione Claude Code:**

```
🔍 Approccio: NON leggere tutto, usa grep mirato

[Glob] **/*.{ts,js,py} → 523 file (NON leggerli!)

# Security scan automatico
[Grep] "eval\(" → 3 file
[Grep] "exec\(" → 2 file
[Grep] "innerHTML" → 12 file
[Grep] "dangerouslySetInnerHTML" → 4 file
[Grep] "sql.*\+|query.*\+" → 8 file (possibile SQL injection)
[Grep] "password.*=.*['\"]" → 5 file (hardcoded passwords)
[Grep] "BEGIN RSA|BEGIN PRIVATE" → 2 file (private keys!)

[Write] SECURITY_SCAN.md con tutti i match
```

```
🔍 Analisi mirata (solo file problematici)

# Leggo SOLO i 5 file con password hardcoded
[Read] config/prod.ts limit=30 offset=X
→ Confermato: PASSWORD = "admin123"

[Read] ... altri 4 file ...

[Write] SECURITY_FINDINGS.md
```

```
📊 Report finale

CRITICAL:
- 2 file con chiavi private committate
- 5 file con password hardcoded
- 3 file con eval() su input utente

HIGH:
- 8 file con possibile SQL injection
- 12 file con innerHTML non sanitizzato

Token usati: ~3000 (invece di ~50000 leggendo tutto)
```

---

## Come Invocare la Skill

Dopo aver installato la skill, puoi usarla dicendo:

```
"Usa la skill project-review per analizzare src/"
```

Oppure semplicemente:

```
"Fai review del progetto src/ usando il metodo lazy loading"
"Debug: l'app crasha su login - trova il problema senza leggere tutto"
"Trova vulnerabilità security nel progetto (scan veloce)"
```

Claude Code caricherà automaticamente le istruzioni della skill e seguirà il workflow ottimizzato.
