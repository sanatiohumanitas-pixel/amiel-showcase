# Systemarchitektur — Amiel

## Überblick

Amiel folgt einer event-getriebenen Architektur mit drei Wahrheitsschichten und einem orthogonalen Governance-Layer. Alle Zustandsänderungen werden als unveränderliche Events protokolliert; der aktuelle Systemzustand ergibt sich aus der Projektion dieses Logs auf die State-Datenbank.

---

## Schichtenmodell

```
┌─────────────────────────────────────────────────────────┐
│                    Governance Layer                     │
│         (überwacht alle Aktionen in Echtzeit)           │
└────────────────────────┬────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────┐
│                      Orchestrator                       │
│              Koordination · Planung · Routing           │
└──────┬──────────────────┬───────────────────┬───────────┘
       │                  │                   │
  Subagent A         Subagent B          Subagent C
  (Recherche)    (Wissensdatenbank)    (spezialisiert)
       │                  │                   │
└─────────────────────────▼───────────────────────────────┐
│                       Event Log                         │
│                  append-only · WAL                      │
└─────────────────────────┬───────────────────────────────┘
                          │
        ┌─────────────────┴──────────────────┐
        │                                    │
   State DB                            Memory Store
  (Tasks · Goals)              (hot · warm · cold Tier)
        │                                    │
        └─────────────────┬──────────────────┘
                          │
                   Outcome Evaluator
                          │
                   Pattern Learner
                          │
                   (zurück zum Orchestrator)
```

---

## Komponenten

### Event Log

- **Typ:** append-only Tabelle in SQLite (WAL-Modus)
- **Felder:** `timestamp`, `event_type`, `source`, `payload`, `version`, `causation_id`, `correlation_id`, `actor`
- **Zweck:** Vollständige, unveränderliche Aufzeichnung jeder Systemhandlung
- **Prinzip:** Kein Zustandsübergang ohne Event — der Log ist die Wahrheit

Die Felder `causation_id` und `correlation_id` ermöglichen vollständige Kausalitätsketten: welches Event hat welches ausgelöst, welche Events gehören zu einem Vorgang.

### Task-System

Tasks sind die atomaren Arbeitseinheiten des Systems. Jeder Task hat:
- Einen Zyklus: `open` → `in_progress` → `done` / `failed`
- Einen `retry_count` mit `cooldown_until` — automatische Wiederholung mit Backoff
- Eine `goal_id` — Tasks sind immer einem Ziel zugeordnet
- Ein `outcome` — das Ergebnis wird strukturiert gespeichert, nicht nur als Freitext

### Goal-System

Goals sind übergeordnete Ziele mit messbaren Fortschrittsmetriken:
- `progress` und `target` definieren den Fortschritt quantitativ
- `priority` und `priority_boost` steuern die Bearbeitungsreihenfolge
- `stagnation_days` und `stagnation_notified` erkennen blockierte Ziele automatisch
- `progress_owner` und `progress_last_writer` implementieren Write-Ownership auf Goal-Ebene

### Memory-System

Das Memory-System speichert Erfahrungswissen in drei Tiers:

| Tier | Beschreibung | Zugriff |
|---|---|---|
| `hot` | Aktuell relevantes Wissen, häufig abgerufen | bevorzugt |
| `warm` | Gelegentlich nützlich, moderate Relevanz | bei Bedarf |
| `cold` | Selten genutzt, niedriger Relevanz-Score | archiviert |

Felder `relevance`, `access_count`, `last_accessed` und `tier_changed_at` steuern die automatische Tier-Migration. Wissen mit `status = "candidate"` wird erst nach Bestätigung in den aktiven Bestand übernommen.

### Outcome Evaluator & Pattern Learner

Nach Abschluss jedes Tasks bewertet der Outcome Evaluator das Ergebnis (Tabelle `action_results`). Der Pattern Learner aggregiert Ergebnisse über Goals hinweg und passt Prioritäten und Strategien des Orchestrators an — ein geschlossener Feedback-Loop.

---

## Datenbank-Design

SQLite im WAL-Modus (Write-Ahead Logging) ermöglicht gleichzeitige Reads während Writes laufen — wichtig für ein System, das parallel Events schreibt und den State abfragt. Keine externe Datenbank-Infrastruktur notwendig: das gesamte System ist auf einem einzelnen Server self-contained.

---

## Deployment

- **Server:** Hetzner Cloud, Ubuntu 22.04 LTS
- **Prozessverwaltung:** systemd (Langläufer) + Cron (zeitgesteuerte Tasks)
- **Monitoring:** Health-API des Governance-Layers, Telegram-Benachrichtigungen bei kritischen Ereignissen
- **Updates:** Git-basierter Deploy-Workflow

---

Weiterführend: [GOVERNANCE.md](GOVERNANCE.md) · [EVENT-SCHEMA.md](EVENT-SCHEMA.md) · [ROADMAP.md](ROADMAP.md)
