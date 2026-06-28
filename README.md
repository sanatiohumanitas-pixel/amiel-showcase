# Amiel — Semi-autonomes Agentensystem

![Status](https://img.shields.io/badge/System-produktiv%20seit%20Feb%202026-green)
![Sprache](https://img.shields.io/badge/Python-3.11-blue)
![DB](https://img.shields.io/badge/SQLite-WAL--Modus-lightgrey)
![LLM](https://img.shields.io/badge/LLM-Claude%20API%20%2B%20Moonshot-orange)

> Persönlicher KI-Begleiter mit Event-Sourcing-Architektur, Ziel-Tracking und Governance-Layer.  
> Produktiv betrieben auf einem Linux-Server (Hetzner, Ubuntu 22.04).

---

## Was ist Amiel?

Amiel ist ein semi-autonomes, event-getriebenes Agentensystem, das als persönlicher Begleiter für seinen Betreiber fungiert. Es verfolgt Ziele, führt Tasks aus, lernt aus Ergebnissen und wird durch ein mehrstufiges Governance-Layer kontrolliert.

**Kernmerkmale:**

- Läuft produktiv auf einem dedizierten Linux-Server
- Verarbeitet Aufgaben asynchron über eine persistente Task-Queue
- Speichert alle Zustandsänderungen als unveränderliche Events (append-only)
- Bewertet eigene Ergebnisse und passt Prioritäten dynamisch an
- Wird durch einen Governance-Layer mit mehreren Sicherheitsstufen überwacht

Amiel ist kein Chatbot mit Gedächtnis-Plugin — es ist ein eigenständiges System mit Zustandsmaschine, Feedback-Loops und definierten Verantwortlichkeitsgrenzen zwischen menschlicher Richtung und maschineller Ausführung.

---

## Warum dieses Projekt?

Ich wollte verstehen wie autonome Systeme wirklich funktionieren —
nicht in der Theorie, sondern unter echten Produktivbedingungen.
Also habe ich eines gebaut. Amiel ist das Ergebnis: ein System das
täglich läuft, aus seinen eigenen Fehlern lernt und sich schrittweise
weiterentwickelt — ohne dass ich jeden Schritt manuell steuere.

---

## Architektur (Kurzübersicht)

### Drei Wahrheitsschichten

| Schicht | Zweck | Charakteristik |
|---|---|---|
| Event Log | Unveränderliche Protokollierung jeder Aktion | append-only, mit Kausalitätskette |
| State DB | Aktueller Systemzustand (Tasks, Goals, Memory) | WAL-Modus, konsistente Reads |
| Memory | Erfahrungswissen mit Relevanz-Scoring | Tier-System: hot / warm / cold |

### Rollenmodell

```
William (Richtung & Entscheidung)
    └── Orchestrator (Koordination & Planung)
            ├── Subagent: ki_tools_research
            ├── Subagent: wissensdatenbank
            └── Subagent: [weitere spezialisierte Agenten]
```

### Governance

Ein eigenständiges Governance-Modul überwacht alle Aktionen des Systems in Echtzeit.  
Es kennt fünf Eskalationsstufen (OBSERVE → WARN → SOFT_BLOCK → HARD_BLOCK → FREEZE)  
und eine Actor-basierte Write-Ownership-Matrix, die definiert, welche Systemkomponente  
welche Datenbankfelder schreiben darf.

### LLM-Integration

- **Claude API (Anthropic)** — Hauptmodell für Reasoning und Generierung
- **Moonshot/Kimi API** — Ergänzend für spezifische Aufgaben

---

## Entwicklungsphasen

| Phase | Titel | Status |
|---|---|---|
| 1 | Fundament: Task-Queue, Event Log, SQLite-Schema | ✅ Abgeschlossen |
| 2 | Goal-System: Ziel-Tracking mit Fortschrittsmetriken | ✅ Abgeschlossen |
| 3 | Memory-System: Tier-basiertes Gedächtnis mit Relevanz-Scoring | ✅ Abgeschlossen |
| 4 | Feedback-Loop: Outcome-Evaluation und Pattern-Learning | ✅ Abgeschlossen |
| 5 | Governance-Härtung: Actor-Matrix, FREEZE-Modus, Health-API | 🔄 In Arbeit |
| 6 | Wissens-Autonomie: Selbstständige Wissensakquise und -bewertung | 📋 Geplant |
| 7 | Orchestrator-Evolution: Multi-Agenten-Koordination mit Delegation | 📋 Geplant |

---

## Technologie-Stack

| Bereich | Technologie |
|---|---|
| Laufzeitumgebung | Python 3.11, Linux/systemd, Cron |
| Datenpersistenz | SQLite (WAL-Modus), append-only Event Log |
| LLM-Integration | Claude API (Anthropic), Moonshot/Kimi API |
| Versionierung | Git / GitHub |
| Benachrichtigungen | Telegram Bot API |
| Design-Prinzip | Event-Sourcing, Single Source of Truth |

---

## Architekturprinzipien

**1. Stabilität vor Intelligenz**  
Jede neue Funktion muss auf einem stabilen Fundament stehen. Kein Feature ohne Testbarkeit, keine Autonomie ohne Kontrollmechanismus.

**2. Schrittweise Evolution**  
Das System wächst in klar abgegrenzten Phasen. Jede Phase schließt mit verifizierten Erfolgskriterien ab, bevor die nächste beginnt.

**3. Messen statt vermuten**  
Entscheidungen werden datenbasiert getroffen. Der Event Log und das Outcome-Tracking liefern die Grundlage — keine Annahmen ohne Evidenz.

**4. Single Source of Truth**  
Jeder Systemzustand hat genau eine autoritative Quelle. Redundante Wahrheiten führen zu Drift — das wird durch die Drei-Schichten-Architektur verhindert.

**5. Menschliche Souveränität**  
William gibt die Richtung. Das System führt aus. Diese Grenze ist architektonisch verankert — nicht nur konventionell vereinbart.

---

## Was ich dabei gelernt habe

- **LLM-Output ist kein Beweis** — Text der "Ja, erledigt" sagt bedeutet
  nichts ohne DB-Event als Nachweis. Dieses Anti-Pattern ("Phantom-Done")
  hat mich Wochen gekostet bis ich es systematisch eliminiert hatte.
- **Governance muss im Code leben, nicht in Dokumenten** — Eine
  Constitution als Datei ändert nichts. Erst als ich sie als laufenden
  Code implementiert habe, hat sie gewirkt.
- **Messen statt vermuten** — Jede Verbesserung braucht eine Baseline
  davor und eine Messung danach. Sonst ist es nur Hoffnung.
- **Stabilität vor Intelligenz** — Ein stabiles einfaches System ist
  wertvoller als ein cleveres instabiles. Das klingt offensichtlich,
  ist es in der Praxis nicht.

---

## Projektstatus

Aktuell in **Phase 5** (Governance-Härtung).

Siehe [ROADMAP.md](ROADMAP.md) für Details zu allen Phasen.  
Siehe [ARCHITEKTUR.md](ARCHITEKTUR.md) für das vollständige Systemdesign.  
Siehe [GOVERNANCE.md](GOVERNANCE.md) für das Sicherheitsmodell.

---

## Hinweis

Dieses Repository dokumentiert Architektur und Designentscheidungen eines produktiv laufenden Systems.  
Kein produktiver Code, keine Konfigurationsdaten, keine Credentials enthalten.

---

## Roadmap

Das System wird aktiv weiterentwickelt:

| Phase | Thema | Status |
|-------|-------|--------|
| Phase 5e | Governance-Härtung (WARN/FREEZE-Modi) | 🔄 in Arbeit |
| Phase 6 | Orchestrator — zentrale Aufgabenkoordination | 🔴 Juli 2026 |
| Phase 7 | Lokales Modell via Ollama auf Mac Mini | 🔴 geplant |

Prinzip: Keine neuen Features ohne stabile Grundlage.
Jede Phase wird beobachtet, gemessen und erst dann erweitert.
