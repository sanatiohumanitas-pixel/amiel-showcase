# Governance-Modell — Amiel

## Zweck

Das Governance-Layer ist ein orthogonales Sicherheitsmodul, das alle Aktionen des Systems in Echtzeit überwacht — unabhängig davon, welche Komponente die Aktion auslöst. Es stellt sicher, dass das System auch bei autonomem Betrieb innerhalb definierter Grenzen bleibt.

---

## Eskalationsstufen

Das Governance-System kennt sechs Modi, die bei zunehmender Bedrohungslage eskaliert werden:

| Modus | Verhalten |
|---|---|
| `OFF` | Keine Überwachung — nur in kontrollierten Entwicklungsumgebungen |
| `OBSERVE` | Alle Aktionen werden protokolliert, keine Eingriffe |
| `WARN` | Verdächtige Aktionen werden gemeldet, aber nicht blockiert |
| `SOFT_BLOCK` | Bestimmte Aktionen werden pausiert und zur Bestätigung vorgelegt |
| `HARD_BLOCK` | Blockierte Aktionen werden verweigert und protokolliert |
| `FREEZE` | Vollständiger Systemstopp — alle autonomen Aktionen werden abgelehnt |

Im `FREEZE`-Modus wirft das System eine Exception auf jeden Versuch, eine Aktion auszuführen. Nur manuelle Eingriffe durch den Betreiber können den Freeze aufheben.

---

## Actor-basierte Write-Ownership

Das Governance-Modell definiert für jeden Actor (Orchestrator, Subagent, externes Script, manueller Eingriff), welche Datenbankfelder er schreiben darf — und welche nicht.

**Prinzip:** Jedes Feld in der Datenbank hat genau einen autorisierten Schreiber. Unautorisierte Schreibversuche werden blockiert und als Governance-Event protokolliert.

Diese Matrix ist die architektonische Umsetzung des Prinzips "Menschliche Souveränität": Das System kann keine eigenen Ziele überschreiben, keine Prioritäten eigenmächtig neu definieren, und keinen Governance-Modus selbst ändern.

---

## Subprocess-Scoping

Jeder Subagent darf nur innerhalb seines definierten Scopes Subprocess-Aufrufe ausführen. Zugriffe außerhalb des Scopes werden durch das Governance-Layer abgefangen, unabhängig davon, ob der Code technisch die Möglichkeit hätte, den Aufruf zu machen.

---

## Health-API

Das Governance-Layer stellt eine öffentliche Health-API bereit, die von externen Monitoring-Scripts abgefragt werden kann:

- Aktueller Governance-Modus
- Anzahl blockierter Aktionen im Zeitfenster
- Status des FREEZE-Zählers
- Letzte Governance-Events

Diese API gibt nur lesenden Zugriff — der Governance-Modus kann über sie nicht verändert werden.

---

## Designentscheidung: Governance als separate Schicht

Governance-Logik ist bewusst nicht in die Business-Logik des Orchestrators oder der Subagenten eingebettet. Stattdessen ist sie ein separates Modul, das von außen auf alle Aktionen aufgesetzt ist.

**Vorteil:** Neue Subagenten oder Orchestrator-Versionen erben automatisch die Governance-Regeln, ohne dass Governance-Code kopiert oder eingebettet werden muss. Die Sicherheitsregeln sind zentral und können unabhängig vom Rest des Systems aktualisiert werden.
