# Documento Tecnico - Guide Creator Flow
## Sistema Multi-Agent RAG per Generazione Guide Turistiche Contestualizzate

### Versione 1.0 (stato attuale codice)
### Data: Ottobre 2025

---

# Sommario 

[Architettura applicativa](#architettura-applicativa)  
[Business Architecture](#business-architecture)  
[Information System Architecture](#information-system-architecture)  
[Architettura Tecnica](#architettura-tecnica)  
[Application Technical Architecture](#application-technical-architecture)  
[Risorse Cloud](#risorse-cloud)  
[Release Management](#release-management)  
[Copertura Requisiti Utente](#copertura-requisiti-utente)  
[Assunzioni, vincoli e validità dell'offerta](#assunzioni-vincoli-e-validità-dellofferta)  
[Installazione e Avvio](#installazione-e-avvio)  

---

# Architettura applicativa

Il progetto **Guide Creator Flow** è un sistema di Retrieval Augmented Generation orchestrato tramite **CrewAI Flow** che genera guide o articoli turistici a partire da:

1. Contenuti interni (PDF brochure nella cartella `input_directory/`)
2. Ricerca web filtrata su domini affidabili (white list travel)

Pipeline principale:
- L'utente inserisce una domanda legata al topic turismo/viaggi
- Un agente valuta la rilevanza (gate logico: rilevante / non rilevante)
- In caso positivo parte una pipeline RAG con retrieval ibrido (semantic + keyword boost + MMR) su PDF indicizzati in **Qdrant**
- Risultati web arricchiscono il contesto se pertinenti
- Due agent generativi costruiscono risposta e formattano un report finale Markdown (`output/report.md`)
- Applicati filtri di sicurezza: redazione PII e Content Safety (Prompt Shield) prima dell'indicizzazione

Obiettivo: accelerare la creazione di contenuti turistici affidabili con trasparenza delle fonti e controllo qualitativo.

---

## Business Architecture

Il sistema supporta il processo di **content creation assistita** per il dominio travel (guide, articoli, paragrafi descrittivi) riducendo effort manuale e migliorando coerenza informativa.

### Processi impattati:

| **Processo** | **Fase** | **Componente** | **Ruolo** | **Impatto (S/N)** | **Stato** |
|--------------|----------|---------------|-----------|------------------|-----------|
| Acquisizione Contenuti | Parsing PDF | Document Loader | Estrazione testo e filtraggio | S | Implementato |
| Sicurezza Contenuti | PII & Safety | Checker (Text Analytics + Content Safety) | Redazione e blocco prompt injection | S | Implementato |
| Preparazione Dati | Chunking | Text Splitter | Suddivisione in chunk semantici | S | Implementato |
| Indicizzazione | Creazione Collection | Qdrant Vector Store | Archiviazione vettori + indici payload | S | Implementato |
| Retrieval | Ricerca Ibrida | RAGSearch Tool | Semantic + keyword + boosting + MMR | S | Implementato |
| Orchestrazione | Flow Control | CrewAI Flow | Routing stato e sequenza agent | S | Implementato |
| Enrichment | Web Search | ResearchCrew (SerperDevTool) | Arricchimento domini affidabili | S | Implementato |
| Generazione | Answer Synthesis | OutputCrew (rag_responder) | Sintesi risposta | S | Implementato |
| Generazione | Formatting | OutputCrew (markdown_author) | Strutturazione Markdown | S | Implementato |
| Osservabilità | Telemetria (opz.) | Opik Integration | Tracking flusso | N | Implementato |

**Tabella 1: Business Architecture - componenti**

## Information System Architecture

Il sistema è strutturato in una architettura modulare che comprende:

1. **Flow & Orchestration Layer**: `main.py` con `CustomState` e decorators `@start`, `@listen`, `@router`.
2. **Acquisition & Safety Layer**: Estrazione PDF (span-level) + PII redaction + Prompt Shield.
3. **Vector Index Layer**: Qdrant (HNSW + scalar quantization + payload indexing) ricreato per run.
4. **Retrieval Logic**: Ricerca ibrida (semantic normalization, keyword prefilter, boosting, MMR).
5. **Web Enrichment**: Query Serper con operatori `site:` su white list domini travel.
6. **Generation Layer**: Duo agent sequenziali (risposta + formattazione) CrewAI.
7. **Output Layer**: Markdown persistito (`output/report.md`).

### Flussi dati principali:

| **Flusso dati** | **Descrizione** |
|-----------------|-----------------|
| PDF_DISCOVERY | Scansione cartella `input_directory/` e selezione PDF |
| PDF_EXTRACTION | Estrazione testo per span filtrando font-size > soglia e colore non simile allo sfondo |
| SAFETY_PII | Redazione entità sensibili (Azure Text Analytics) |
| PROMPT_SHIELD | Verifica attacchi injection (Azure Content Safety) |
| CHUNKING | Split ricorsivo (chunk_size=700 overlap=120) |
| EMBEDDING | Calcolo vettori (Azure OpenAI Embeddings) |
| VECTOR_UPSERT | Upsert punti con payload (trusted flag, source, chunk id) |
| HYBRID_SEARCH | Semantic query + full-text scroll + boosting + MMR diversificazione |
| WEB_SEARCH | Query arricchita con domini whitelisted |
| CONTEXT_AGGREGATION | Serializzazione JSON dei doc locali e web |
| ANSWER_SYNTHESIS | Generazione risposta RAG (rag_responder) |
| MARKDOWN_OUTPUT | Formattazione articolo finale (markdown_author) |

**Tabella 2: Information Architecture - flussi**

---

# Architettura Tecnica

## Application Technical Architecture

L'architettura è modulare ad agenti con orchestrazione di stato (CrewAI Flow) e routing condizionale sulla rilevanza per ottimizzare consumo risorse.

### Stack Tecnologico:

| **Componente** | **Software/Libreria** | **Versione (pyproject)** | **Ruolo** | **Stato** |
|----------------|----------------------|--------------------------|-----------|-----------|
| Orchestrazione | crewai[tools] | 0.193.2 | Flow multi‑agent | Implementato |
| LLM Chat | AzureChatOpenAI (GPT-4o) | API 2024 | Generazione & reasoning | Implementato |
| Embeddings | Azure OpenAI Embeddings | API 2024 | Vettorializzazione chunk | Implementato |
| Retrieval Utils | langchain / community / openai | 0.3.x | Abstraction modelli e splitter | Implementato |
| Vector Store | qdrant-client | 1.15.1 | Persistenza vettori & full-text | Implementato |
| PDF Parsing | PyMuPDF | 1.26.4 | Estrazione testo span-level | Implementato |
| Web Search | SerperDevTool | (crewAI tool) | Arricchimento contenuti | Implementato |
| Safety & PII | azure-ai-textanalytics + Content Safety | >=5.3.0 | Redazione & shield | Implementato |
| Telemetria | opik | 1.8.61 | (Facoltativo, disattivato) | Disponibile |
| Configurazione | python-dotenv | - | Variabili ambiente | Implementato |
| Output | Markdown file | - | Report finale | Implementato |

**Tabella 3: Technical Architecture - componenti**

### Flussi Dati Tecnici:

| **Flusso dati** | **Protocollo/Interfaccia** | **Origine** | **Destinazione** | **Tipologia** | **Frequenza stimata** | **Picco** | **Payload medio** | **Massimo** | **Stato** |
|-----------------|---------------------------|------------|------------------|--------------|----------------------|-----------|------------------|------------|-----------|
| PDF Scan | File system | FS locale | Loader | On-demand | 1 batch/run | 10 PDF/run | 500KB | 10MB | Implementato |
| PII Redaction | HTTPS REST | App | Azure Text Analytics | Real-time | 5 req/min | 30 req/min | 4KB | 50KB | Implementato |
| Content Safety | HTTPS REST | App | Azure Content Safety | Real-time | 5 req/min | 30 req/min | 3KB | 40KB | Implementato |
| Embedding | HTTPS REST | App | Azure OpenAI | Real-time | 10 req/min | 60 req/min | 2KB | 8KB | Implementato |
| Vector Upsert | gRPC/HTTP | App | Qdrant | Batch | 1/run | 1/run | Variabile | Variabile | Implementato |
| Hybrid Query | gRPC/HTTP | App | Qdrant | Real-time | 5 req/min | 30 req/min | 1KB | 5KB | Implementato |
| Web Search | HTTPS REST | App | Serper API | Real-time | 3 req/min | 10 req/min | 4KB | 20KB | Implementato |
| Markdown Output | File I/O | App | FS locale | Batch | 1/run | 1/run | 15KB | 200KB | Implementato |

**Tabella 4: Technical Architecture - flussi**

---

# Risorse Cloud

Il sistema utilizza **Azure Cloud** come provider principale per i servizi di AI e può essere deployato sia on-premise che in cloud.

| Componente | Ambiente | Risorsa | Tipologia | Provider | Regione | Note | Dimensionamento iniziale |
|-----------|----------|---------|-----------|----------|---------|------|-------------------------|
| LLM & Embeddings | Dev/Test/Prod | Azure OpenAI | PaaS | Azure | West Europe | GPT-4o + embeddings | Throughput standard |
| PII & Safety | Dev/Test/Prod | Azure Text Analytics + Content Safety | PaaS | Azure | West Europe | Stesso endpoint region | Shared key |
| Vector Store | Dev | Qdrant (locale / Docker) | Container | Self-host | n/a | Ricreazione ad run | 2 vCPU / 4GB RAM |
| Vector Store | Prod (opz.) | Qdrant gestito / AKS | PaaS/K8s | Azure | West Europe | Persistenza dischi | 4 vCPU / 8GB RAM |
| Orchestrator | Dev | Python CLI | Runtime locale | - | n/a | Esecuzione manuale | 2 vCPU / 4GB RAM |
| Orchestrator | Prod (opz.) | Azure Container Apps / ACI | PaaS | Azure | West Europe | Scalabilità oraria | 2–4 vCPU |
| Observability | (facolt.) | Opik | SaaS/Local | - | - | Non abilitato | n/a |

**Tabella 5: Risorse Cloud**

### Stima costi indicativa (Azure):
- Dev: €120–180 / mese
- Test: €180–300 / mese
- Prod: €300–600 / mese

Include: token GPT-4o, embeddings, Text Analytics, Content Safety, compute container.

---

# Release Management

| Piattaforma | Release | Ordine | Tipologia | Contenuto |
|-------------|---------|--------|-----------|-----------|
| Core Flow | v1.0-MVP | 1 | BS | Orchestrazione + retrieval ibrido + output Markdown |
| Safety & Trust | v1.1-Safety | 2 | BS | Estensione categorie PII + audit fonti |
| Web Plus | v1.2-Web | 3 | BS | Multi-topic relevance + ranking web avanzato |
| Caching & Perf | v1.3-Cache | 4 | BS | Cache embeddings + no recreate collection |
| Observability | v1.4-Obs | 5 | BS | Attivazione Opik + metriche qualità |
| API Exposure | v1.5-API | 6 | BS | Interfaccia REST & container prod |

**Sequenza sintetica:** 1) MVP 2) Hardening sicurezza 3) Enrichment avanzato 4) Performance 5) Osservabilità 6) Industrializzazione API

---

# Copertura Requisiti Utente

| **Requisito** | **Copertura** | **Implementazione** |
|-------------|-------------|------------------|
| RF01 - Acquisizione PDF | Completa | Scansione cartella e filtro estensioni |
| RF02 - Estrazione testo | Completa | PyMuPDF per spans con filtri font/colore |
| RF03 - Rimozione PII | Completa | Azure Text Analytics redaction |
| RF04 - Prompt Safety | Completa | Azure Content Safety (shieldPrompt) |
| RF05 - Chunking parametrico | Completa | RecursiveCharacterTextSplitter (700/120) |
| RF06 - Indicizzazione vettoriale | Completa | Qdrant HNSW + indici payload |
| RF07 - Ricerca ibrida | Completa | Semantic + keyword + boosting + MMR |
| RF08 - Gate rilevanza | Completa | PoemCrew boolean relevance |
| RF09 - Arricchimento web | Completa | SerperDevTool + white list domini |
| RF10 - Guida Markdown | Completa | OutputCrew -> `output/report.md` |
| RF11 - Flag trust sorgenti | Completa | Heuristica "untrusted" nel filename |
| RF12 - Parametri tuning | Parziale | Dataclass Settings (no UI) |
| RNF01 - Performance | Parziale | Recreate collection (ottimizzabile) |
| RNF02 - Scalabilità | Parziale | Nessuna cache embeddings | 
| RNF03 - Osservabilità | Parziale | Opik disattivato |
| RNF04 - Sicurezza segreti | Parziale | .env locale (manca vault) |
| RNF05 - Portabilità | Completa | Python >=3.10, dipendenze dichiarate |

Legenda: Completa = implementato; Parziale = presente ma migliorabile.

---

# Assunzioni, vincoli e validità dell'offerta

## Assunzioni

### Volume di utilizzo stimato:
- PDF per run: 10–80 brochure
- Query per esecuzione: 1 (interattiva) o piccolo batch
- Output: 1 guida Markdown / run
- Dimensione media PDF: 0.5–2MB

### Assunzioni tecniche:
- Disponibilità servizi Azure AI (OpenAI + Text Analytics + Content Safety)
- PDF testuali (nessun OCR attivo)
- Lingue: IT / EN supportate dal modello
- Requisiti HW dev: >=2 vCPU / 4GB RAM; prod consigliato >=4 vCPU / 8GB RAM

## Vincoli

### Vincoli tecnici:
- Ricreazione collection Qdrant ad ogni esecuzione (overhead)
- Solo PDF (nessun DOCX/HTML parsing dedicato)
- Parametri retrieval modificabili solo a codice
- Assenza caching embeddings/chunks

### Vincoli operativi:
- Necessario avviare Qdrant (localhost:6333)
- Richiede variabili ambiente AZURE_* configurate
- Esecuzione CLI interattiva (input dell'utente)

### Vincoli di sicurezza:
- Segreti in `.env` (migliorabile con Key Vault)
- Heuristica trust semplice sui nomi file
- Nessuna autenticazione multi‑utente

## Validità dell'offerta

### Validità temporale (proposta):
- Orizzonte architettura: 6 mesi
- Revisione piano release: trimestrale

### Condizioni di validità:
- Disponibilità credenziali Azure
- Dataset PDF non protetti da DRM
- Manutenzione white list domini
- Possibilità containerizzazione (Qdrant + orchestrator)

---

---

# Installazione e Avvio

Questa sezione descrive i passi per installare e lanciare il progetto utilizzando **uv** (package manager Python) e la CLI di **CrewAI**.

## Prerequisiti
- Python 3.10–3.13 (consigliato 3.11+)
- Git
- Docker (per eseguire Qdrant localmente)
- Accesso a credenziali Azure (OpenAI, Text Analytics, Content Safety) e chiave Serper (per Web Search)

## 1. Clonazione repository
Sostituisci `<REPO_URL>` con l'URL reale.

```powershell
git clone <REPO_URL>
cd final_project
# La sotto-cartella del flusso è 'guide_creator_flow'
cd guide_creator_flow
```

Se nella tua organizzazione la cartella dovesse chiamarsi diversamente (ad es. `guide_content_crew`), sostituisci il nome nel comando di `cd`.

## 2. Installazione uv (se non già presente)
```powershell
pip install uv
```
Verifica:
```powershell
uv --version
```

## 3. Creazione ed attivazione ambiente virtuale
```powershell
uv venv
.\.venv\Scripts\Activate.ps1
```

## 4. Installazione dipendenze
Il file `pyproject.toml` le elenca già. Puoi:

Opzione A (più semplice, sincronizza tutto):
```powershell
uv sync
```

Opzione B (come richiesto: aggiunta manuale pacchetti):
```powershell
uv add crewai[tools] langchain langchain-community langchain-openai openai \
	azure-ai-textanalytics azure-core azure-identity opik pymupdf qdrant-client python-dotenv
```

## 5. Variabili d'ambiente (.env)
Crea un file `.env` nella cartella `guide_creator_flow` (o aggiorna quello esistente) con le chiavi:
```
AZURE_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxx
AZURE_API_BASE=https://<nome-risorsa>.openai.azure.com
AZURE_API_VERSION=2024-02-01
SERPER_API_KEY=xxxxxxxxxxxxxxxxxxxxxxxx

# (Opzionali) altri parametri custom
# QDRANT_URL=http://localhost:6333
```

## 6. Avvio Qdrant (vector store)
Se non hai un'istanza attiva:
```powershell
docker run -d --name qdrant -p 6333:6333 -p 6334:6334 qdrant/qdrant:latest
```

## 7. Avvio del flusso CrewAI
Dal path `guide_creator_flow` (dove risiede `pyproject.toml`):

Opzione 1 – usando script definito:
```powershell
uv run kickoff
```

Opzione 2 – conforme alla richiesta: usare la CLI CrewAI
```powershell
crewai run kickoff
```
(La voce `kickoff` è definita in `[project.scripts]` del `pyproject.toml`).


## 8. Interazione
Il flusso richiederà in console: `Inserisci la tua domanda ...`. Digita una query turistica (es. *"Consigli per un itinerario di 3 giorni a Londra"*). Al termine verrà generato `output/report.md`.

## 9. Rigenerare il grafo del Flow (facoltativo)
```powershell
crewai flow plot
```
Se supportato, aprirà/aggiornerà un diagramma del flusso (in repo c'è `crewai_flow.html`).

## 10. Aggiornare / aggiungere nuovi pacchetti
```powershell
uv add <nuovo-pacchetto>
```

## 11. Troubleshooting rapido
| Problema | Possibile causa | Soluzione |
|----------|-----------------|-----------|
| Errore connessione Qdrant | Container non avviato | Controlla `docker ps` |
| Nessun documento trovato | Cartella `input_directory/` vuota | Aggiungi PDF e rilancia |
| API Azure 401/403 | Chiave errata / endpoint diverso | Verifica valori in `.env` |
| Output vuoto | Query non rilevante | Riformula domanda attinente al turismo |
| Lentezza iniziale | Recreate collection | Introdurre caching (release futura) |

---

**Documento preparato da:** 
- Team tecnico: Jacopo Bonanno, Pietro Montresori, Roberto Parodo, Monica Salvati, Luca Sangiovanni

**Data:** Ottobre 2025  
**Versione:** 1.0  
**Stato:** Riflette implementazione MVP corrente