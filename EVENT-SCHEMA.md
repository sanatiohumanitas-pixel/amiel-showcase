# Event-Schema — Amiel

## Konzept

Der Event Log ist das Herzstück der Amiel-Architektur. Jede bedeutende Systemhandlung — Task-Starts, Goal-Updates, Governance-Entscheidungen, Memory-Änderungen — wird als Event protokolliert, bevor der Zustand in der State-Datenbank aktualisiert wird.

Das macht den Systemzustand zu jeder Zeit nachvollziehbar und ermöglicht vollständige Kausalitätsketten.

---

## Felder

| Feld | Typ | Beschreibung |
|---|---|---|
| `id` | INTEGER | Autoincrement, unveränderlich |
| `timestamp` | TEXT | ISO 8601, UTC |
| `event_type` | TEXT | Kategorie des Events (siehe unten) |
| `source` | TEXT | Komponente, die das Event ausgelöst hat |
| `payload` | TEXT | JSON-Objekt mit event-spezifischen Daten |
| `version` | INTEGER | Schema-Version des Payloads |
| `causation_id` | TEXT | ID des Events, das dieses ausgelöst hat |
| `correlation_id` | TEXT | Grupperungs-ID für zusammengehörige Events |
| `actor` | TEXT | Actor im Sinne der Governance-Matrix |

---

## Kausaliätskette

Die Felder `causation_id` und `correlation_id` sind das Werkzeug für vollständige Nachvollziehbarkeit:

- **`causation_id`:** Zeigt auf das direkt auslösende Event. Ermöglicht Rückverfolgung: "Warum ist dieses Event entstanden?"
- **`correlation_id`:** Verbindet alle Events, die zu einem logischen Vorgang gehören (z.B. "Task X von Start bis Abschluss")

Ein Vorgang, der fünf Events erzeugt, hat alle dieselbe `correlation_id`, aber unterschiedliche `causation_ids` — weil jeder Schritt vom vorherigen ausgelöst wurde.

---

## Event-Typen (Auswahl)

| event_type | Bedeutung |
|---|---|
| `task.created` | Neuer Task wurde in die Queue eingestellt |
| `task.started` | Task wurde zur Bearbeitung übernommen |
| `task.completed` | Task erfolgreich abgeschlossen |
| `task.failed` | Task fehlgeschlagen, Retry wird geprüft |
| `task.retrying` | Task wird nach Cooldown erneut versucht |
| `goal.updated` | Fortschritt oder Priorität eines Goals geändert |
| `goal.stagnation` | Goal zeigt keine Bewegung mehr — wird eskaliert |
| `memory.promoted` | Memory-Eintrag in höheres Tier verschoben |
| `memory.demoted` | Memory-Eintrag in niedrigeres Tier verschoben |
| `governance.warn` | Governance-Layer hat eine Warnung ausgelöst |
| `governance.block` | Aktion wurde blockiert |
| `governance.freeze` | System wurde in FREEZE-Modus versetzt |
| `system.health` | Regelmäßiger Health-Check-Event |

---

## Append-Only-Prinzip

Events werden nie verändert oder gelöscht. Korrekturen werden als neue Events vom Typ `*.corrected` oder `*.reverted` protokolliert — mit Verweis auf das ursprüngliche Event.

Das garantiert: Der Log ist immer die vollständige Wahrheit darüber, was das System wann und warum getan hat.

---

Weiterführend: [ARCHITEKTUR.md](ARCHITEKTUR.md) · [examples/event-beispiel.md](examples/event-beispiel.md)
