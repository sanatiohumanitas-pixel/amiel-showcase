# Roadmap — Amiel

## Entwicklungsphilosophie

Amiel wird in klar abgegrenzten Phasen entwickelt. Jede Phase schließt mit verifizierten Erfolgskriterien ab — nicht mit einem Datum. "Fertig" bedeutet: stabil, getestet, in Produktion bewährt.

---

## Phase 1 — Fundament ✅

**Ziel:** Lauffähiges System mit persistenter Task-Queue und Event Log

- SQLite-Schema mit WAL-Modus
- Append-only Event Log mit Basisfeldern
- Task-Lifecycle: open → in_progress → done / failed
- systemd-Integration für Dauerbetrieb
- Erster produktiver Einsatz auf Hetzner-Server

**Lerneffekt dieser Phase:** Die Event-Tabelle zu früh als "nice to have" einzustufen war ein Fehler. Sie ist keine Ergänzung — sie ist die Grundlage für alles danach.

---

## Phase 2 — Goal-System ✅

**Ziel:** Tasks werden übergeordneten Zielen zugeordnet; Fortschritt ist messbar

- Goals-Tabelle mit quantitativen Metriken (`progress`, `target`)
- Verknüpfung Tasks ↔ Goals via `goal_id`
- Stagnationserkennung: `stagnation_days` + automatische Eskalation
- Prioritätssystem mit dynamischen Boosts

---

## Phase 3 — Memory-System ✅

**Ziel:** Das System lernt aus Erfahrungen und hält relevantes Wissen verfügbar

- Memory-Tabelle mit Tier-System (hot / warm / cold)
- Relevanz-Scoring und automatische Tier-Migration
- Bestätigungsworkflow: `candidate` → bestätigt → aktiv
- `access_count` und `last_accessed` für Relevanzberechnung

---

## Phase 4 — Feedback-Loop ✅

**Ziel:** Ergebnisse von Tasks fließen zurück in Planung und Prioritäten

- `action_results`-Tabelle für strukturierte Outcome-Protokollierung
- Outcome Evaluator bewertet Erfolg/Misserfolg
- Pattern Learner aggregiert über Goals hinweg
- Orchestrator passt Prioritäten auf Basis von Mustern an

---

## Phase 5 — Governance-Härtung 🔄

**Ziel:** Das System ist auch bei erhöhter Autonomie sicher und kontrollierbar

- Governance-Layer als separates, orthogonales Modul
- Sechs Eskalationsstufen (OFF → FREEZE)
- Actor-basierte Write-Ownership-Matrix für alle DB-Felder
- Subprocess-Scoping für alle Subagenten
- Health-API für externes Monitoring
- FREEZE-Modus: vollständiger Systemstopp auf Exception-Ebene

**Aktueller Stand:** Kern implementiert und in Produktion. Härtung läuft (Edge-Cases, Audit-Log-Vollständigkeit).

---

## Phase 6 — Wissens-Autonomie 📋

**Ziel:** Das System kann selbstständig neues Wissen akquirieren, bewerten und integrieren

- Autonome Recherche mit Qualitätsbewertung der Quellen
- Selbstständige Memory-Aktualisierung innerhalb definierter Scopes
- Wissenslücken-Erkennung: System meldet, wenn es etwas nicht weiß
- Konfidenz-Tracking für alle Memory-Einträge

**Voraussetzung:** Phase 5 vollständig abgeschlossen. Autonomie ohne stabiles Governance-Fundament ist nicht akzeptabel.

---

## Phase 7 — Orchestrator-Evolution 📋

**Ziel:** Multi-Agenten-Koordination mit echter Delegation und Parallelverarbeitung

- Mehrere Subagenten arbeiten parallel an unabhängigen Goals
- Orchestrator delegiert und koordiniert, entscheidet nicht mehr alles selbst
- Konfliktresolution: was passiert wenn zwei Subagenten dieselbe Ressource brauchen?
- Erweiterte Kausalitätsketten im Event Log für parallele Abläufe

---

## Prinzipien (phasenübergreifend)

1. **Stabilität vor Intelligenz** — Kein Feature ohne stabiles Fundament darunter
2. **Schrittweise Evolution** — Phasen bauen aufeinander auf, kein Überspringen
3. **Messen statt vermuten** — Fortschritt ist quantifizierbar oder er existiert nicht
4. **Single Source of Truth** — Jeder Zustand hat eine autoritative Quelle
5. **Menschliche Souveränität** — Richtung kommt vom Menschen, Ausführung von der Maschine
