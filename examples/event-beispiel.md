# Beispiel: Task-Lifecycle im Event Log

Dieses Beispiel zeigt, wie ein einzelner Task vom Einstellen bis zum Abschluss im Event Log abgebildet wird. Alle Werte sind anonymisiert.

---

## Szenario

Ein Subagent soll einen Recherche-Task ausführen. Der erste Versuch schlägt fehl, der zweite gelingt.

---

## Event-Sequenz

### 1. Task wird eingestellt

```json
{
  "id": 1001,
  "timestamp": "2026-06-15T08:12:00Z",
  "event_type": "task.created",
  "source": "orchestrator",
  "actor": "orchestrator",
  "correlation_id": "corr-abc123",
  "causation_id": null,
  "payload": {
    "task_id": 42,
    "goal_id": "goal-recherche-q2",
    "priority": "high"
  },
  "version": 1
}
```

### 2. Subagent übernimmt den Task

```json
{
  "id": 1002,
  "timestamp": "2026-06-15T08:12:05Z",
  "event_type": "task.started",
  "source": "subagent_research",
  "actor": "subagent",
  "correlation_id": "corr-abc123",
  "causation_id": 1001,
  "payload": {
    "task_id": 42
  },
  "version": 1
}
```

### 3. Erster Versuch schlägt fehl

```json
{
  "id": 1003,
  "timestamp": "2026-06-15T08:13:40Z",
  "event_type": "task.failed",
  "source": "subagent_research",
  "actor": "subagent",
  "correlation_id": "corr-abc123",
  "causation_id": 1002,
  "payload": {
    "task_id": 42,
    "retry_count": 1,
    "cooldown_until": "2026-06-15T08:18:40Z",
    "reason": "external_api_timeout"
  },
  "version": 1
}
```

### 4. Governance-Layer protokolliert den Fehlschlag

```json
{
  "id": 1004,
  "timestamp": "2026-06-15T08:13:40Z",
  "event_type": "governance.warn",
  "source": "governance",
  "actor": "governance",
  "correlation_id": "corr-abc123",
  "causation_id": 1003,
  "payload": {
    "task_id": 42,
    "mode": "OBSERVE",
    "trigger": "task_failed_retry_scheduled"
  },
  "version": 1
}
```

### 5. Task wird nach Cooldown erneut gestartet

```json
{
  "id": 1005,
  "timestamp": "2026-06-15T08:18:45Z",
  "event_type": "task.retrying",
  "source": "orchestrator",
  "actor": "orchestrator",
  "correlation_id": "corr-abc123",
  "causation_id": 1003,
  "payload": {
    "task_id": 42,
    "retry_count": 1
  },
  "version": 1
}
```

### 6. Task wird erfolgreich abgeschlossen

```json
{
  "id": 1006,
  "timestamp": "2026-06-15T08:19:22Z",
  "event_type": "task.completed",
  "source": "subagent_research",
  "actor": "subagent",
  "correlation_id": "corr-abc123",
  "causation_id": 1005,
  "payload": {
    "task_id": 42,
    "outcome": "success",
    "goal_id": "goal-recherche-q2"
  },
  "version": 1
}
```

---

## Was dieses Beispiel zeigt

- Alle sechs Events teilen dieselbe `correlation_id` — der gesamte Vorgang ist als Einheit abrufbar
- Die `causation_id` bildet die Kausalitätskette ab: 1001 → 1002 → 1003 → 1005 → 1006
- Das Governance-Layer protokolliert passiv (Event 1004) ohne in den Ablauf einzugreifen — im `OBSERVE`-Modus
- Der Retry-Mechanismus ist vollständig im Log sichtbar: Fehlschlag, Cooldown-Zeit, erneuter Start
- Kein Zustandsverlust möglich: selbst wenn die State-DB komplett verloren ginge, ließe sich der korrekte Endzustand aus dem Event Log rekonstruieren
