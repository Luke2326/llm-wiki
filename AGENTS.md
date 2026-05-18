# Knowledge Base di [Il tuo dominio] — Schema

## Scopo

<!-- PERSONALIZZA: Sostituisci questo testo con una descrizione in un paragrafo del tuo dominio di conoscenza. -->
<!-- Esempi: "ricerca sul machine learning", "letteratura del XIX secolo", "panorama competitivo degli strumenti SaaS" -->
Questa è una knowledge base mantenuta da un LLM su [IL TUO ARGOMENTO]. L'LLM scrive e mantiene tutti i file sotto `wiki/`. L'essere umano cura le fonti grezze e guida le query. L'essere umano non modifica mai direttamente i file wiki.

## Struttura delle directory

- `raw/` — Documenti sorgente immutabili (trascrizioni, articoli, note). Non modificarli mai.
- `wiki/index.md` — Catalogo principale. Ogni pagina wiki deve comparire qui.
- `wiki/log.md` — Log attività append-only.
- `wiki/summaries/` — Una pagina di riepilogo per ogni documento sorgente grezzo.
- `wiki/concepts/` — Pagine di concetti, strategie e framework.
- `wiki/entities/` — Pagine di entità (persone, strumenti, organizzazioni, prodotti — qualunque "cosa" esista nel tuo dominio).
- `wiki/syntheses/` — Tabelle comparative, framework decisionali, analisi trasversali.
- `wiki/journal/` — Voci di diario di ricerca o di sessione.
- `wiki/presentations/` — Presentazioni Marp generate dal contenuto della wiki.

## Nomenclatura dei file

- Tutto in minuscolo, trattini per separare le parole: `nome-concetto.md`
- Nessuno spazio, nessun carattere speciale, nessuna maiuscola
- Il nome deve corrispondere allo slug del titolo della pagina

## Formato della pagina

Ogni pagina wiki usa questo frontmatter e questa struttura:

```yaml
---
title: "Titolo Pagina"
type: concept | entity | summary | synthesis
tags: [tag1, tag2, tag3]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: ["raw/nomefile.txt"]
confidence: high | medium | low
---
```

### Sezioni richieste per tipo di pagina

**Pagine di riepilogo** (`wiki/summaries/`):
- `## Key Points` — Elenco puntato delle principali affermazioni/idee
- `## Relevant Concepts` — Link alle pagine concetto toccate da questa fonte
- `## Source Metadata` — Tipo di fonte, autore/speaker, data, URL o identificatore

**Pagine concetto** (`wiki/concepts/`):
- `## Definition` — Definizione in linguaggio semplice, in un paragrafo
- `## How It Works` — Meccanica, processo o struttura del concetto
- `## Key Parameters` — Variabili, dimensioni o fattori importanti
- `## When To Use` — Situazioni e contesti in cui questo concetto si applica
- `## Risks & Pitfalls` — Failure mode noti, errori comuni, limitazioni
- `## Related Concepts` — Link wiki a pagine correlate
- `## Sources` — Fonti grezze che informano questa pagina

**Pagine entità** (`wiki/entities/`):
- `## Overview` — Che cos'è questa entità
- `## Characteristics` — Proprietà, attributi e struttura chiave
- `## Common Strategies` — Link a pagine concetto per strategie o metodi associati a questa entità
- `## Related Entities` — Link a pagine entità correlate

**Pagine di sintesi** (`wiki/syntheses/`):
- `## Comparison` — Tabella o confronto strutturato
- `## Analysis` — Insight trasversali
- `## Recommendations` — Quando preferire quale approccio
- `## Pages Compared` — Link a tutte le pagine coinvolte

## Convenzioni di linking

- Usa link wiki in stile Obsidian: `[[concepts/nome-concetto]]`
- Usa sempre percorsi relativi dalla root della wiki
- Ogni pagina deve linkare almeno un'altra pagina (nessuna pagina orfana)
- Quando menzioni un concetto che ha una pagina, linkalo sempre

## Tassonomia dei tag

<!-- PERSONALIZZA: Sostituisci queste categorie placeholder con tag rilevanti per il tuo dominio. -->
<!-- Ogni categoria dovrebbe avere 3-8 tag specifici. -->
<!-- Esempio per una KB di cucina: -->
<!--   Cucina: italiana, giapponese, francese, messicana -->
<!--   Tecnica: brasatura, fermentazione, sous-vide, griglia -->
<!--   Ingrediente: proteina, verdura, cereale, latticino -->

- **Categoria-A**: `tag-1`, `tag-2`, `tag-3`
- **Categoria-B**: `tag-4`, `tag-5`, `tag-6`
- **Categoria-C**: `tag-7`, `tag-8`, `tag-9`
- **Ambito**: `fondazionale`, `avanzato`, `sperimentale`
- **Stato**: `ben-consolidato`, `emergente`, `speculativo`

## Livelli di confidenza

- **high** — Idea ben consolidata, più fonti corroboranti, dimostrata con esempi concreti
- **medium** — Supportata da fonti ma con esempi limitati o da una singola fonte
- **low** — Singola menzione, aneddotica o speculativa

## Workflow

### Ingest

Quando l'utente dice "ingest [fonte]" o aggiunge un file a `raw/`:

1. Leggi completamente la fonte grezza
2. Crea `wiki/summaries/<source-slug>.md` con un riepilogo completo
3. Identifica tutti i concetti, le entità e le strategie menzionate
4. Per ogni concetto/entità: crea la pagina se non esiste, oppure aggiornala con le nuove informazioni se esiste già
5. Aggiungi cross-link in entrambe le direzioni tra tutte le pagine toccate
6. Aggiorna `wiki/index.md` — aggiungi nuove voci, aggiorna i riepiloghi delle pagine modificate
7. Aggiungi una voce a `wiki/log.md` con timestamp, nome della fonte, pagine create/aggiornate
8. Segnala eventuali contraddizioni con il contenuto wiki esistente

### Query

Quando l'utente fa una domanda:

1. Leggi `wiki/index.md` per trovare le pagine rilevanti
2. Leggi quelle pagine
3. Sintetizza una risposta citando pagine specifiche con link wiki
4. Se la risposta rivela un nuovo insight che vale la pena preservare:
   - Crea una pagina di sintesi in `wiki/syntheses/`
   - Aggiorna index e log

### Lint

Quando l'utente dice "lint" o "health check":

1. Leggi tutte le pagine wiki
2. Controlla: pagine orfane (nessun link in ingresso), affermazioni obsolete, contraddizioni tra pagine, cross-link mancanti, sezioni incomplete, pagine a bassa confidenza che potrebbero essere rafforzate
3. Correggi automaticamente ciò che può essere corretto
4. Segnala i problemi che richiedono giudizio umano
5. Suggerisci nuove fonti o argomenti da investigare
6. Aggiorna il log

## Regole

- Non modificare mai i file in `raw/`
- Aggiorna sempre `index.md` e `log.md` dopo ogni modifica alla wiki
- Preferisci aggiornare pagine esistenti invece di creare duplicati
- In caso di dubbio su un'affermazione, imposta la confidenza a "low" e annota l'incertezza
- Mantieni le pagine focalizzate — un concetto per pagina, separa la pagina se diventa troppo lunga
- Usa linguaggio semplice — definisci il gergo al primo utilizzo in ogni pagina
- Tutte le date in formato ISO 8601: YYYY-MM-DD
- Quando una fonte fornisce esempi specifici, includili con dettagli concreti
