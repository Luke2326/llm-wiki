# AGENTS.md — LLM Wiki Maintainer

Questo repository implementa una knowledge base mantenuta da agenti LLM secondo il pattern **LLM Wiki**: le fonti grezze restano in `raw/`, mentre l'agente legge, sintetizza, collega e mantiene una wiki persistente in `wiki/`.

L'obiettivo non è fare semplice RAG al momento della domanda. L'obiettivo è compilare conoscenza una volta, accumularla nel tempo, collegarla, segnalare contraddizioni e renderla interrogabile tramite file Markdown versionabili.

---

## 1. Ruolo dell'agente

L'agente è il manutentore della wiki.

Deve:

1. leggere fonti grezze in `raw/`;
2. creare o aggiornare pagine strutturate in `wiki/`;
3. mantenere link bidirezionali tra pagine correlate;
4. aggiornare sempre `wiki/index.md` e `wiki/log.md` dopo modifiche alla wiki;
5. rispondere alle domande usando prima la wiki, poi le fonti grezze solo se necessario;
6. segnalare contraddizioni, incertezze e buchi conoscitivi;
7. preferire aggiornamenti incrementali a duplicazioni.

Non deve:

1. modificare, riscrivere, rinominare o cancellare file in `raw/`;
2. inventare fonti, date, autori o riferimenti;
3. creare pagine duplicate per lo stesso concetto, entità o fonte;
4. rispondere solo dalla memoria del modello quando esistono file wiki pertinenti;
5. lasciare pagine isolate senza link in ingresso o in uscita, salvo eccezione motivata nel log.

---

## 2. Struttura del repository

```text
raw/                         # fonti grezze immutabili
wiki/
  index.md                   # catalogo principale della wiki
  log.md                     # log append-only delle operazioni
  overview.md                # sintesi viva dell'intera knowledge base
  summaries/                 # una pagina di riepilogo per ogni fonte
  concepts/                  # concetti, pattern, strategie, framework
  entities/                  # persone, prodotti, strumenti, aziende, sistemi
  syntheses/                 # confronti, decisioni, analisi trasversali
  journal/                   # note di sessione o diario di ricerca
  presentations/             # presentazioni Marp generate dalla wiki
  _maintenance/              # report di lint, backlog, mapping, audit
```

Se una directory richiesta manca, crearla prima di procedere.

---

## 3. Invarianti obbligatori

Queste regole valgono sempre.

1. `raw/` è append-only e read-only per l'agente.
2. Ogni modifica sotto `wiki/` deve produrre anche una voce in `wiki/log.md`.
3. Ogni nuova pagina deve comparire in `wiki/index.md`.
4. Ogni pagina deve contenere frontmatter YAML valido.
5. Ogni pagina deve avere almeno una fonte in `sources`, salvo `index.md`, `log.md`, `overview.md` e report di manutenzione.
6. Ogni pagina deve avere almeno un link wiki verso un'altra pagina, quando esistono pagine correlate.
7. Se una pagina cita una fonte, la fonte deve esistere davvero in `raw/` oppure essere dichiarata come fonte esterna con URL verificabile.
8. Le affermazioni non supportate da fonti devono essere marcate come `confidence: low` o spostate in una sezione `## Open Questions`.
9. Non creare una nuova pagina se una pagina esistente copre già lo stesso concetto o la stessa entità: aggiorna quella esistente.
10. Prima di scrivere, controlla sempre `wiki/index.md` e cerca pagine esistenti correlate.

---

## 4. Comandi riconosciuti

Gli agenti devono interpretare sia comandi espliciti sia linguaggio naturale.

| Intento | Trigger tipici | Azione |
|---|---|---|
| Ingest | `ingest`, `importa`, `aggiungi fonte`, `processa raw/...` | Integra una o più fonti nella wiki |
| Query | domanda generica, `cosa dice la wiki su...`, `riassumi...` | Rispondi usando wiki e fonti citate |
| Lint | `lint`, `health check`, `controlla la wiki` | Verifica qualità, link, duplicati, contraddizioni |
| Synthesis | `confronta`, `fammi una sintesi`, `decision matrix` | Crea o aggiorna una pagina in `wiki/syntheses/` se utile |
| Graph | `build graph`, `mappa collegamenti`, `knowledge graph` | Genera o aggiorna report/link map in `_maintenance/` |
| Presentation | `presentazione`, `slides`, `marp` | Crea una presentazione in `wiki/presentations/` basata sulla wiki |

Se l'intento è ambiguo, scegli l'operazione più conservativa: leggere e rispondere senza modificare file. Modifica file solo quando il trigger implica chiaramente ingest, lint/fix, sintesi persistente o generazione artefatti.

---

## 5. Naming convention

Usa sempre slug stabili.

Regole:

- minuscolo;
- parole separate da trattini;
- niente spazi;
- niente caratteri speciali;
- niente maiuscole;
- rimuovi articoli non necessari;
- mantieni acronimi in minuscolo nello slug, ma leggibili nel titolo.

Esempi:

```text
wiki/concepts/retrieval-augmented-generation.md
wiki/entities/openai.md
wiki/summaries/attention-is-all-you-need.md
wiki/syntheses/rag-vs-llm-wiki.md
```

Se due fonti generano lo stesso slug, aggiungi un suffisso breve e stabile, ad esempio anno o autore: `nome-fonte-2026.md`.

---

## 6. Frontmatter standard

Ogni pagina wiki, esclusi `index.md` e `log.md`, deve iniziare così:

```yaml
---
title: "Titolo leggibile"
type: concept | entity | summary | synthesis | journal | presentation | maintenance
tags: [tag-1, tag-2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - raw/percorso-fonte.md
confidence: high | medium | low
status: draft | stable | needs-review
---
```

Regole:

- `created` non cambia mai dopo la creazione.
- `updated` cambia a ogni modifica sostanziale.
- `sources` deve contenere percorsi reali o URL verificabili.
- `status: needs-review` quando ci sono contraddizioni, buchi informativi o dubbi.
- `confidence` è una valutazione della pagina intera, non della singola frase.

---

## 7. Tipi di pagina

### 7.1 Summary — `wiki/summaries/`

Una pagina summary rappresenta una fonte grezza.

Struttura obbligatoria:

```md
## Source Metadata
- Source: raw/...
- Type: article | transcript | note | paper | ticket | email | doc | other
- Author/Speaker:
- Date:
- URL/Identifier:

## Executive Summary

## Key Points

## Extracted Concepts

## Extracted Entities

## Important Quotes Or Evidence

## Contradictions Or Tensions

## Open Questions

## Links
```

Regole:

- Non fare solo un riassunto breve: estrai conoscenza riutilizzabile.
- Collega concetti ed entità con wikilink.
- Se la fonte è lunga, conserva dettagli concreti, numeri, date, esempi e condizioni.

---

### 7.2 Concept — `wiki/concepts/`

Una pagina concept descrive una singola idea, pattern, metodo, problema o strategia.

Struttura obbligatoria:

```md
## Definition

## Why It Matters

## How It Works

## Key Parameters

## When To Use

## Risks & Pitfalls

## Examples

## Related Concepts

## Related Entities

## Sources

## Open Questions
```

Regole:

- Un solo concetto per pagina.
- Se la pagina supera circa 1.500 parole o contiene più concetti autonomi, dividi.
- Quando aggiorni una pagina concept, integra la nuova fonte senza perdere le informazioni precedenti.

---

### 7.3 Entity — `wiki/entities/`

Una pagina entity descrive una persona, azienda, prodotto, sistema, progetto, libreria, modello, repository o organizzazione.

Struttura obbligatoria:

```md
## Overview

## Characteristics

## Timeline Or Versions

## Capabilities

## Limitations

## Related Concepts

## Related Entities

## Sources

## Open Questions
```

Regole:

- Se l'entità evolve nel tempo, usa una sezione timeline/versioni.
- Distingui fatti stabili, osservazioni contestuali e opinioni provenienti dalle fonti.

---

### 7.4 Synthesis — `wiki/syntheses/`

Una synthesis combina più pagine o fonti.

Struttura obbligatoria:

```md
## Question Or Decision

## Short Answer

## Comparison

## Analysis

## Recommendation

## Trade-offs

## Evidence

## Pages Compared

## Sources

## Open Questions
```

Regole:

- Crea synthesis quando la risposta richiede confronto, decisione o insight trasversale.
- Non creare synthesis per ogni domanda banale.
- Se la sintesi nasce durante una query, chiedi conferma prima di salvarla quando l'utente non ha chiesto modifiche persistenti.

---

### 7.5 Overview — `wiki/overview.md`

`overview.md` è la sintesi viva dell'intera wiki.

Deve contenere:

```md
## Scope

## Current Mental Model

## Major Concepts

## Major Entities

## Important Syntheses

## Known Contradictions

## Open Questions

## Maintenance Notes
```

Aggiornala dopo ingest importanti o lint sostanziali.

---

## 8. Linking

Usa wikilink Obsidian-style:

```md
[[concepts/nome-concetto]]
[[entities/nome-entita]]
[[summaries/nome-fonte]]
[[syntheses/nome-sintesi]]
```

Regole:

1. Usa percorsi relativi dalla root `wiki/`, senza `.md`.
2. Quando menzioni un concetto o entità già esistente, linkalo.
3. Quando crei una nuova pagina, aggiungi link anche dalle pagine correlate verso la nuova pagina.
4. Le pagine summary devono linkare concept/entity estratti.
5. Le pagine concept/entity devono linkare almeno una summary o fonte rilevante.
6. Evita link generici irrilevanti solo per soddisfare la regola: i link devono avere valore semantico.

---

## 9. Index

`wiki/index.md` è il catalogo operativo della wiki. Deve essere utile agli agenti prima ancora che agli umani.

Struttura consigliata:

```md
# Wiki Index

## Scope

## Recently Updated

## Summaries
| Page | Source | Updated | Confidence | Notes |

## Concepts
| Page | Tags | Updated | Confidence | One-line Description |

## Entities
| Page | Tags | Updated | Confidence | One-line Description |

## Syntheses
| Page | Question/Decision | Updated | Confidence | Notes |

## Open Questions

## Maintenance Backlog
```

Regole:

- Ogni pagina nuova o modificata deve aggiornare la rispettiva riga.
- Le descrizioni devono essere brevi e concrete.
- Non lasciare placeholder generici.

---

## 10. Log

`wiki/log.md` è append-only. Non riscrivere vecchie voci salvo correzione di formattazione evidente.

Formato obbligatorio:

```md
## YYYY-MM-DD HH:mm — <operation>

- Trigger: richiesta utente o comando
- Sources read:
  - raw/...
- Pages created:
  - wiki/...
- Pages updated:
  - wiki/...
- Contradictions found:
  - ...
- Open questions:
  - ...
- Notes:
  - ...
```

Usa l'ora locale se disponibile, altrimenti solo la data ISO.

---

## 11. Workflow: Ingest

Quando devi ingerire una fonte:

### 11.1 Preflight

1. Verifica che la fonte esista.
2. Identifica tipo, nome, data, autore e URL se disponibili.
3. Leggi `wiki/index.md` se esiste.
4. Cerca pagine già esistenti correlate in `wiki/summaries/`, `wiki/concepts/`, `wiki/entities/`, `wiki/syntheses/`.
5. Prepara una lista di pagine da creare e aggiornare.

### 11.2 Lettura

1. Leggi l'intera fonte quando possibile.
2. Se la fonte è troppo grande, processala a blocchi mantenendo una lista cumulativa di fatti, concetti, entità, date, numeri, decisioni e incertezze.
3. Non ignorare tabelle, esempi, errori, edge case, decisioni operative o riferimenti a versioni.

### 11.3 Scrittura

Ordine obbligatorio:

1. crea o aggiorna la pagina summary;
2. crea o aggiorna concept;
3. crea o aggiorna entity;
4. crea o aggiorna syntheses solo se emergono insight trasversali importanti;
5. aggiorna link bidirezionali;
6. aggiorna `wiki/overview.md` se l'informazione cambia il quadro generale;
7. aggiorna `wiki/index.md`;
8. appendi una voce a `wiki/log.md`.

### 11.4 Output finale all'utente

Alla fine dell'ingest, rispondi con:

- fonti lette;
- pagine create;
- pagine aggiornate;
- contraddizioni o dubbi;
- prossime azioni consigliate.

Non incollare l'intero contenuto delle pagine create, salvo richiesta esplicita.

---

## 12. Workflow: Query ottimizzato

Quando l'utente fa una domanda, l'agente deve trattarla come una ricerca guidata dentro la wiki, non come una risposta generata dalla memoria del modello.

### 12.1 Regola principale

La prima operazione obbligatoria è sempre leggere `wiki/index.md`.

`wiki/index.md` è la mappa operativa della knowledge base. Serve a:

- capire quali pagine esistono;
- evitare letture casuali o duplicate;
- trovare rapidamente concept, entity, summary e synthesis rilevanti;
- verificare se la domanda è già coperta dalla wiki;
- capire quali fonti grezze possono essere necessarie solo in seconda battuta.

Non rispondere mai a una query dell'utente senza aver prima controllato `wiki/index.md`, salvo il caso in cui il file non esista. Se `wiki/index.md` non esiste, segnala il problema e fai una scansione minima della directory `wiki/` per ricostruire il contesto.

### 12.2 Algoritmo obbligatorio per ogni query

1. Leggi `wiki/index.md`.
2. Estrai dalla domanda:
   - argomento principale;
   - concetti citati;
   - entità citate;
   - vincoli temporali;
   - intento dell'utente: spiegazione, confronto, decisione, troubleshooting, riepilogo, ricerca puntuale.
3. Usa `wiki/index.md` per identificare le pagine candidate più rilevanti.
4. Dai priorità di lettura in questo ordine:
   1. `wiki/syntheses/` se la domanda è comparativa, decisionale o trasversale;
   2. `wiki/concepts/` se la domanda riguarda idee, pattern, strategie o problemi;
   3. `wiki/entities/` se la domanda riguarda strumenti, persone, prodotti, sistemi, repository o organizzazioni;
   4. `wiki/summaries/` se serve tornare alla fonte sintetizzata;
   5. `raw/` solo se la wiki è incompleta, ambigua o se servono dettagli non presenti nelle pagine wiki.
5. Leggi solo le pagine candidate necessarie, ma abbastanza da rispondere con precisione.
6. Se le pagine candidate rimandano ad altre pagine con wikilink rilevanti, segui quei link finché migliorano la risposta.
7. Rispondi citando le pagine wiki usate con wikilink.
8. Distingui chiaramente:
   - cosa è supportato dalla wiki;
   - cosa è inferenza ragionata;
   - cosa manca;
   - cosa è incerto o contraddittorio.
9. Non modificare file durante una query semplice, a meno che l'utente chieda esplicitamente di salvare, aggiornare o creare una sintesi persistente.

### 12.3 Strategia di ricerca dentro l'index

Quando leggi `wiki/index.md`, cerca corrispondenze in questo ordine:

1. match esatto del termine dell'utente nei titoli;
2. match di sinonimi o varianti nello stesso dominio;
3. match nei tag;
4. match nelle descrizioni one-line;
5. match nelle open questions o nel maintenance backlog;
6. pagine recentemente aggiornate, solo se semanticamente pertinenti.

Non scegliere una pagina solo perché è recente. La pertinenza semantica viene prima della recenza.

### 12.4 Query con risposta insufficiente

Se dopo aver letto index e pagine candidate la risposta non è completa:

1. dichiara cosa è stato trovato nella wiki;
2. indica cosa manca;
3. leggi le fonti `raw/` citate dalle pagine candidate, se possono colmare il gap;
4. se anche le fonti non bastano, rispondi esplicitando il limite;
5. suggerisci quali fonti aggiungere o quale ingest fare per migliorare la wiki.

Non inventare dettagli mancanti per chiudere la risposta.

### 12.5 Quando creare una synthesis da una query

Crea o aggiorna una pagina in `wiki/syntheses/` solo quando almeno una di queste condizioni è vera:

- l'utente chiede esplicitamente di salvare la sintesi;
- la domanda richiede un confronto riutilizzabile;
- la risposta produce una decision matrix o un framework ricorrente;
- emergono relazioni importanti tra più concept/entity;
- la stessa domanda potrebbe essere utile in futuro come pagina autonoma.

Se l'utente ha fatto una domanda semplice, non creare file persistenti.

### 12.6 Formato risposta consigliato

```md
## Risposta

...

## Evidenza usata
- [[syntheses/...]]
- [[concepts/...]]
- [[entities/...]]
- [[summaries/...]]

## Inferenze

...

## Mancanze o incertezze

...
```

### 12.7 Errori da evitare nelle query

- Rispondere dalla memoria del modello senza consultare `wiki/index.md`.
- Cercare direttamente in `raw/` prima di aver letto l'index.
- Leggere troppe pagine non pertinenti invece di usare l'index come filtro.
- Citare fonti grezze quando esiste già una summary più adatta.
- Creare una synthesis per ogni domanda.
- Nascondere incertezze, contraddizioni o assenza di fonti.
- Usare link wiki non letti direttamente come se fossero evidenza verificata.

---

## 13. Workflow: Lint / Health check

Quando l'utente chiede lint o health check:

1. Leggi `wiki/index.md`.
2. Scansiona tutte le pagine sotto `wiki/`, escluso `presentations/` se non richiesto.
3. Controlla:
   - frontmatter mancante o invalido;
   - pagine non presenti in index;
   - link rotti;
   - pagine orfane;
   - sezioni obbligatorie mancanti;
   - fonti inesistenti;
   - duplicati concettuali;
   - contraddizioni tra pagine;
   - pagine troppo lunghe da dividere;
   - pagine `low confidence` senza open questions;
   - summary senza concept/entity estratti.
4. Correggi automaticamente problemi meccanici:
   - index mancante;
   - link ovvi;
   - sezioni vuote obbligatorie;
   - frontmatter incompleto deducibile;
   - log entry mancante.
5. Non correggere automaticamente problemi semantici dubbi: crea un report in `wiki/_maintenance/health-check-YYYY-MM-DD.md`.
6. Aggiorna `wiki/log.md`.

Output finale:

- problemi corretti;
- problemi lasciati a revisione umana;
- file modificati;
- score sintetico della salute della wiki.

---

## 14. Gestione contraddizioni

Quando una nuova fonte contraddice una pagina esistente:

1. Non cancellare automaticamente l'informazione precedente.
2. Aggiungi o aggiorna una sezione `## Contradictions Or Tensions` nella summary e, se rilevante, nella pagina concept/entity.
3. Indica le fonti in conflitto.
4. Abbassa `confidence` se il conflitto impatta la pagina intera.
5. Imposta `status: needs-review` se serve giudizio umano.
6. Riporta la contraddizione in `wiki/log.md`.

Formato consigliato:

```md
## Contradictions Or Tensions

- `raw/fonte-a.md` afferma X, mentre `raw/fonte-b.md` afferma Y.
  Stato: non risolto.
  Impatto: ...
```

---

## 15. Politica sulle fonti

Gerarchia delle fonti:

1. fonti primarie in `raw/`;
2. pagine summary già create dalla fonte primaria;
3. documentazione ufficiale o sorgenti esterni verificabili;
4. note secondarie o interpretazioni;
5. memoria del modello solo per contesto generale, mai come fonte principale.

Regole:

- Non citare come fatto ciò che non è tracciabile.
- Se una fonte esterna viene usata per aggiornare la wiki, registra URL e data di accesso nella pagina.
- Se il contenuto è volatile, annota la data.
- Se una fonte è obsoleta ma utile storicamente, mantienila e marca il contesto temporale.

---

## 16. Confidenza

Usa questi criteri:

- `high`: più fonti indipendenti concordano, oppure una fonte primaria autorevole è chiara e recente.
- `medium`: una fonte chiara ma singola, oppure più fonti parziali.
- `low`: affermazione singola, aneddotica, incompleta, vecchia, ambigua o contraddetta.

Quando aggiorni `confidence`, spiega il motivo nella pagina o nel log.

---

## 17. Tagging

I tag devono essere utili per ritrovare e raggruppare conoscenza.

Regole:

- usa 2-6 tag per pagina;
- preferisci tag stabili e riutilizzabili;
- evita sinonimi inutili;
- se introduci un nuovo tag, usalo intenzionalmente;
- aggiorna l'index se emergono cluster importanti.

Tassonomia iniziale da personalizzare:

```md
- Domain: `development`, `architecture`, `ai`, `business`, `research`, `operations`
- Content type: `pattern`, `tool`, `system`, `decision`, `problem`, `source-summary`
- Status: `stable`, `emerging`, `speculative`, `needs-review`
- Scope: `foundational`, `advanced`, `implementation`, `troubleshooting`
```

---

## 18. Qualità del contenuto

Le pagine devono essere:

- concrete;
- sintetiche ma complete;
- orientate al riuso;
- leggibili da umani e agenti;
- collegate ad altre pagine;
- supportate da fonti.

Evita:

- frasi vaghe tipo “è importante” senza spiegare perché;
- duplicazione di lunghi blocchi della fonte;
- pagine enciclopediche generiche non collegate al dominio della wiki;
- liste di bullet senza sintesi;
- link inseriti solo per quantità.

---

## 19. Regole operative per coding agent

Quando lavori in un repository reale:

1. Prima di modificare file, ispeziona l'albero del progetto.
2. Se esiste già una struttura `raw/` o `wiki/`, rispettala.
3. Non creare directory alternative come `docs/wiki` o `knowledge/` salvo richiesta esplicita.
4. Usa modifiche atomiche: ogni operazione deve lasciare la wiki coerente.
5. Dopo modifiche massive, esegui un controllo testuale dei link e dei file referenziati se hai strumenti disponibili.
6. Non cancellare contenuto esistente se non chiaramente duplicato o errato; preferisci integrare e annotare.
7. Se il comando dell'utente è parziale, fai la migliore azione sicura e registra eventuali assunzioni nel log.

---

## 20. Definition of Done

Un ingest è completo solo se:

- la fonte è stata letta;
- esiste una summary corrispondente;
- concept/entity rilevanti sono stati creati o aggiornati;
- i link bidirezionali principali sono presenti;
- `wiki/index.md` è aggiornato;
- `wiki/log.md` contiene una nuova voce;
- eventuali contraddizioni sono segnalate;
- l'utente riceve un riepilogo operativo.

Una query è completa solo se:

- `wiki/index.md` è stato controllato per primo;
- le pagine candidate sono state scelte a partire dall'index;
- la risposta deriva prima dalla wiki e solo dopo da `raw/` se necessario;
- le pagine usate sono citate con wikilink;
- le inferenze sono separate dai fatti supportati;
- le incertezze sono esplicitate;
- non sono state fatte modifiche persistenti non richieste.

Un lint è completo solo se:

- sono stati controllati index, frontmatter, link, fonti, duplicati e sezioni;
- i fix meccanici sono stati applicati;
- i problemi semantici sono stati riportati;
- il log è aggiornato.

---

## 21. Esempi di comportamento corretto

### Esempio ingest

Utente:

```text
ingest raw/articles/rag-vs-long-context.md
```

Agente:

1. legge la fonte;
2. crea `wiki/summaries/rag-vs-long-context.md`;
3. aggiorna o crea `wiki/concepts/retrieval-augmented-generation.md`;
4. aggiorna o crea `wiki/concepts/long-context.md`;
5. crea link incrociati;
6. aggiorna index e log;
7. risponde con elenco file creati/aggiornati.

### Esempio query

Utente:

```text
Che differenza c'è tra RAG e LLM Wiki?
```

Agente:

1. legge index;
2. legge le pagine pertinenti;
3. risponde citando `[[concepts/retrieval-augmented-generation]]`, `[[concepts/llm-wiki]]` e le summary usate;
4. non crea file a meno che l'utente chieda di salvare la sintesi.

### Esempio lint

Utente:

```text
lint
```

Agente:

1. controlla tutta la wiki;
2. corregge link e index mancanti;
3. crea `wiki/_maintenance/health-check-YYYY-MM-DD.md` se ci sono problemi non banali;
4. aggiorna log;
5. restituisce report sintetico.
