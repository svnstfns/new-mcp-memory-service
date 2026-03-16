# CLAUDE.md — Memory Service (pvnkn3t Fork)

> Fork von [doobidoo/mcp-memory-service](https://github.com/doobidoo/mcp-memory-service) (v8.44.0).
> Ziel: Lokaler Docker-Service als zentrales Wissens-Backend für die pvnkn3t-Infrastruktur.

---

## 1. Projektziel

Dieses Projekt wird als **lokaler Docker-Service** betrieben und soll als zentrales Wissens-Backend dienen für:

- **Themenspeicher** — Off-Topic-Ideen aus Sessions parken und wieder aufgreifen
- **Unified Backlog** — Aufgaben projektübergreifend verwalten
- **Session-State** — Was wurde in welcher Session bearbeitet, was ist offen?
- **Knowledge Base** — Best Practices, Patterns, Architekturentscheidungen
- **Topic-Erkennung** — Erkennen, wenn ein Gespräch off-topic driftet
- **Dokument-Ingestion** — Bestehende Dokumentation einlesen und durchsuchbar machen
- **GitHub-Automation** — Repos anlegen, Docs aktuell halten, Commits/Push automatisieren

Aktuelle Nutzer-Projekte: `pvnkn3t-ecosystem`, `mediaserver`, `gen-ai-suite`.

---

## 2. Bekannte Probleme

| Problem | Status | Details |
|---------|--------|---------|
| Upstream-Deployment-Anleitung funktioniert nicht | Bekannt | Docker-Build/Run weicht von Doku ab |
| pgvector Installation | Gefixt (im Fork) | Code-Fix war nötig, genaue Änderung rekonstruieren |
| Cloudflare-Backend | Nicht benötigt | Nur lokaler SQLite-Vec-Backend relevant |
| Upstream CLAUDE.md | Ersetzt | 840 Zeilen, zugeschnitten auf Upstream-Maintainer |

---

## 3. Ziel-Setup

```
NAS (TrueNAS Scale, SRV-001)
└── Docker (Svcs-Netz 10.121.125.0/24)
    └── memory-service
        ├── Port 8000 (HTTP API + Web Dashboard)
        ├── SQLite-Vec Backend (lokal, kein Cloudflare)
        ├── Consolidation/Dreaming aktiv
        └── Volume: persistent data (/app/data)
```

**Kommunikation:** HTTP API. Claude Code Hooks sprechen den Service über HTTP an.

**Zugriff:** Nur lokales Netz (Mgmt + Svcs Subnet). Kein Public Access.

---

## 4. Vollständiger Feature-Katalog

> Alle Features des Upstream-Projekts. Bewertung (Relevanz für pvnkn3t) wird in `/brainstorm` Session evaluiert.

### 4.1 Core — Speicher & Suche

| Feature | Beschreibung | Wo im Code |
|---------|-------------|------------|
| **Semantic Search** | Vektorbasierte Ähnlichkeitssuche über alle Memories | `src/.../storage/sqlite_vec.py` |
| **Tag-basierte Suche** | Filter nach Tags (z.B. `pvnkn3t`, `architecture`) | `/api/search/by-tag` |
| **Zeitbasierte Suche** | Filter nach Zeitraum ("letzte Woche", "gestern") | `/api/search/by-time`, `utils/time_parser.py` |
| **24 Memory-Typen** | note, reference, session, fix, feature, architecture, ... | `scripts/maintenance/memory-types.md` |
| **Content Hashing** | Automatische Deduplizierung | `utils/hashing.py` |
| **ONNX Embeddings** | Lokale Vektoren ohne Cloud-API | `embeddings/onnx_embeddings.py` |

### 4.2 Consolidation — "Dreaming"

| Feature | Beschreibung | Wo im Code |
|---------|-------------|------------|
| **Exponential Decay** | Priorisiert häufig genutzte/aktuelle Memories | `consolidation/decay.py` |
| **Creative Associations** | Findet semantische Verbindungen (0.3-0.7 Similarity) | `consolidation/associations.py` |
| **Semantic Clustering** | Gruppiert verwandte Memories (DBSCAN) | `consolidation/clustering.py` |
| **Compression** | Fasst redundante Infos zusammen (behält Originale) | `consolidation/compression.py` |
| **Controlled Forgetting** | Archiviert irrelevante Memories (90+ Tage inaktiv) | `consolidation/forgetting.py` |
| **Scheduled Consolidation** | Automatisch: daily 02:00, weekly Sun 03:00, monthly 1st 04:00 | `consolidation/scheduler.py` |
| **Health Monitoring** | Gesundheitsstatus des Memory-Systems | `consolidation/health.py` |

### 4.3 Claude Code Hooks

| Feature | Beschreibung | Wo im Code |
|---------|-------------|------------|
| **Session-Start Hook** | Lädt relevante Memories bei Session-Beginn | `claude-hooks/core/session-start.js` |
| **Session-End Hook** | Speichert Session-Erkenntnisse und Entscheidungen | `claude-hooks/core/session-end.js` |
| **Topic-Change Detection** | Erkennt Themenwechsel und lädt passenden Kontext | `claude-hooks/core/topic-change.js` |
| **Natural Memory Triggers** | KI-basierte Erkennung wann Kontext geladen werden soll (85%+ Accuracy) | `claude-hooks/core/mid-conversation.js` |
| **Project Detection** | Erkennt aktuelles Projekt und lädt spezifisches Wissen | `claude-hooks/utilities/project-detector.js` |
| **Session Tracking** | Verfolgt Session-übergreifenden Kontext | `claude-hooks/utilities/session-tracker.js` |
| **Memory Scoring** | Qualitäts-basierte Relevanz-Bewertung | `claude-hooks/utilities/memory-scorer.js` |
| **Git-Aware Context** | Analysiert Git-History für kontextbezogene Memory-Suche | `claude-hooks/utilities/git-analyzer.js` |
| **Adaptive Pattern Detection** | Lernt aus Nutzerverhalten | `claude-hooks/utilities/adaptive-pattern-detector.js` |
| **Context Shift Detection** | Erkennt wann sich der Gesprächskontext ändert | `claude-hooks/utilities/context-shift-detector.js` |
| **Performance Profiles** | speed_focused (<100ms), balanced (<200ms), memory_aware (<500ms) | `claude-hooks/memory-mode-controller.js` |

### 4.4 Web Dashboard & API

| Feature | Beschreibung | Wo im Code |
|---------|-------------|------------|
| **Web Dashboard** | CRUD, Suche, Echtzeit-Updates (SSE) | `web/static/index.html` |
| **REST API** | Vollständige HTTP API für alle Operationen | `web/api/` |
| **Analytics** | Statistiken über Memory-Nutzung | `web/api/analytics.py` |
| **Backup API** | Backup-Trigger und -Status über HTTP | `web/api/backup.py` |
| **Consolidation API** | Dreaming per API triggern/überwachen | `web/api/consolidation.py` |
| **SSE Events** | Real-time Updates für Dashboard | `web/sse.py` |
| **i18n** | 7 Sprachen (de, en, es, fr, ja, ko, zh) | `web/static/i18n/` |

### 4.5 Document Ingestion

| Feature | Beschreibung | Wo im Code |
|---------|-------------|------------|
| **PDF Parsing** | PyPDF2, optional LlamaParse | `ingestion/pdf_loader.py` |
| **Text/Markdown** | Native Unterstützung | `ingestion/text_loader.py` |
| **CSV** | Tabellarische Daten einlesen | `ingestion/csv_loader.py` |
| **JSON** | Strukturierte Daten einlesen | `ingestion/json_loader.py` |
| **Intelligent Chunking** | Respektiert Absatz-/Satzgrenzen | `ingestion/chunker.py` |
| **Semtools Integration** | Optional: DOCX, PPTX via LlamaParse | `ingestion/semtools_loader.py` |

### 4.6 Storage Backends

| Backend | Performance | Relevant? | Wo im Code |
|---------|-------------|-----------|------------|
| **SQLite-Vec** | 5ms reads | **Ja — primär** | `storage/sqlite_vec.py` |
| **Cloudflare** | Network-dependent | Nein | `storage/cloudflare.py` |
| **Hybrid** | SQLite + Cloud Sync | Nein (kein Multi-Device) | `storage/hybrid.py` |
| **HTTP Client** | Remote-Service | Eventuell (Service-zu-Service) | `storage/http_client.py` |

### 4.7 Infrastructure & Operations

| Feature | Beschreibung | Wo im Code |
|---------|-------------|------------|
| **Docker Support** | Dockerfile + Compose (MCP + HTTP Mode) | `tools/docker/` |
| **mDNS Discovery** | Automatische Service-Erkennung im LAN | `discovery/mdns_service.py` |
| **Backup Scheduler** | Automatische Backups | `backup/scheduler.py` |
| **GPU Detection** | CUDA/MPS/DirectML/ROCm für Embeddings | `utils/gpu_detection.py` |
| **Health Checks** | Docker + API Health Endpoints | `web/api/health.py` |
| **OAuth** | Optional für Web Dashboard | `web/oauth/` |
| **CLI** | Kommandozeilen-Tool | `cli/main.py` |

---

## 5. Learnings aus pvnkn3t-ecosystem (in diese Session übertragen)

Die folgenden Patterns wurden in `pvnkn3t-ecosystem` entwickelt und sollen in den Memory Service integriert werden:

### 5.1 GitHub-Automation (Session-Hooks)

```
Session-Start:
  ├── Repo auf GitHub vorhanden? → git pull --rebase
  └── Nicht vorhanden? → gh repo create --private + push

Session-Ende:
  ├── Validierung (frontmatter, patterns, cross-refs)
  ├── README auto-generieren
  ├── Conventional Commit
  └── git push
```

**Referenz-Implementierung:** `pvnkn3t-ecosystem/_scripts/repo-sync.sh` + `session-finalize.sh`

### 5.2 Dokumente automatisch aktuell halten

- **Schema-getriebene Validierung:** JSON-Schema definiert Pflichtfelder, erlaubte Werte, verbotene Patterns
- **Pre-commit Hooks:** Automatische Checks bei jedem Commit
- **README Auto-Generation:** Tabellen aus Frontmatter generieren
- **CI (GitHub Actions):** Validierung bei Push/PR

**Referenz-Implementierung:** `pvnkn3t-ecosystem/_schema/doc-types.json` + `_scripts/validate.py`

### 5.3 Unified Backlog + Themenspeicher

- **Eine Aufgabenliste** statt verstreute TODO-Listen — mit Kategorie, Prio, Referenz
- **Themenspeicher** für Off-Topic-Ideen (erkennen → parken → bei Gelegenheit aufgreifen)
- **Session-Start:** Backlog lesen, Top-3 Aufgaben vorschlagen
- **Session-Ende:** Erledigtes markieren, Neues ergänzen

**Referenz-Implementierung:** `pvnkn3t-ecosystem/00-index.md` (Backlog) + `_backlog/parking.md`

**Ziel:** Diese file-basierten Lösungen durch den Memory Service ersetzen/ergänzen — zentral, durchsuchbar, projektübergreifend.

---

## 6. Entwicklung

```bash
# Docker bauen und starten (HTTP-Modus)
cd tools/docker
docker compose -f docker-compose.http.yml up -d

# Health Check
curl http://localhost:8000/api/health

# Dashboard
open http://localhost:8000/

# Tests
uv run pytest tests/ -x

# Logs
docker compose -f docker-compose.http.yml logs -f
```

### Tech Stack

- Python 3.12, FastAPI, SQLite-Vec, ONNX Embeddings
- Docker (`tools/docker/Dockerfile` + `docker-compose.http.yml`)
- Claude Hooks (Node.js, `claude-hooks/`)

### Code-Konventionen

- Python: Type Hints, async/await, PEP 8
- Tests: pytest
- Commits: Conventional Commits (`feat`, `fix`, `chore`)

---

## 7. Nächste Schritte (Brainstorm-Agenda)

Folgende Fragen in der ersten Session klären (`/brainstorm`):

1. **Docker-Build fixen** — Baut das Image aktuell? Was ist kaputt? pgvector-Fix verifizieren.
2. **Feature-Bewertung** — Feature-Katalog (§4) durchgehen: Was ist relevant, was nicht, was fehlt?
3. **Integration mit pvnkn3t-ecosystem** — Ersetzt der Service `parking.md` und den Backlog in `00-index.md`, oder ergänzt er sie? Wie sprechen Claude Hooks den Service an?
4. **Datenmodell** — Welche Memory-Typen brauchen wir? Reichen die 24 Standard-Typen oder brauchen wir eigene (Backlog-Item, Topic, Decision, Pattern)?
5. **Session-Hook-Integration** — Bestehende Hooks (`~/.claude/hooks/core/`) nutzen den Service bereits rudimentär. Wie erweitern? GitHub-Automation (repo-sync, session-finalize) integrieren?
6. **Deployment** — Docker Compose auf dem NAS, Port 8000, Svcs-Netz, Volume-Strategie, Auto-Start.
7. **Scope-Abgrenzung** — Was entfernen wir aus dem Fork? (Cloudflare, Release Manager, PR Automator, ...)

---

## 8. Verzeichnisstruktur (Kurzform)

```
src/mcp_memory_service/
├── server.py              # MCP-Protokoll Server
├── config.py              # Konfiguration
├── storage/
│   ├── sqlite_vec.py      # ← Unser Backend
│   ├── base.py            # Abstract Base Class
│   ├── cloudflare.py      # (evtl. entfernen)
│   └── hybrid.py          # (evtl. entfernen)
├── consolidation/         # "Dreaming" — Wissen verknüpfen
│   ├── associations.py    # Kreative Verbindungen
│   ├── clustering.py      # Semantisches Clustering
│   ├── compression.py     # Redundanz-Reduktion
│   ├── decay.py           # Zeitbasiertes Scoring
│   ├── forgetting.py      # Kontrolliertes Vergessen
│   └── scheduler.py       # Automatische Zeitpläne
├── ingestion/             # Dokument-Import
│   ├── pdf_loader.py
│   ├── text_loader.py
│   └── chunker.py
├── web/                   # FastAPI Dashboard + API
│   ├── app.py
│   ├── api/               # REST Endpoints
│   └── static/            # Frontend (HTML/JS/CSS)
├── models/memory.py       # Memory Datenmodell
├── cli/main.py            # CLI Tool
└── utils/

claude-hooks/              # Claude Code Integration
├── core/
│   ├── session-start.js   # Memory-Injection bei Start
│   ├── session-end.js     # Erkenntnisse speichern
│   ├── topic-change.js    # Themenwechsel erkennen
│   └── mid-conversation.js # Natural Triggers
├── utilities/
│   ├── project-detector.js
│   ├── session-tracker.js
│   ├── memory-scorer.js
│   ├── git-analyzer.js
│   ├── context-shift-detector.js
│   └── adaptive-pattern-detector.js
├── memory-mode-controller.js
└── config.json

tools/docker/              # Docker Setup
├── Dockerfile
├── docker-compose.yml     # MCP-Modus
├── docker-compose.http.yml # ← HTTP-Modus (unser Ziel)
└── docker-entrypoint-unified.sh
```
