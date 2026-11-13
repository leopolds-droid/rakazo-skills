---
name: filehub-handoff
description: Tauscht KI-Aufträge und Ergebnisse über einen vorhandenen FileHub als Dateien aus; Chat-Nachrichten enthalten nur kurze Statussignale.
---

# FileHub Handoff

## Ziel
Verwende den **bereits vorhandenen FileHub** als gemeinsame Dateiablage. Sende keine langen Aufträge oder Antworten im Chat. Nutze einen bestehenden, explizit benannten Signalkanal ausschließlich für Auftrags-IDs und Dateipfade.

## Konfiguration
Bevor du beginnst, ermittle aus der **privaten** Betriebsdokumentation oder aus der bereits freigegebenen Laufzeitkonfiguration:
- FileHub-Basis-URL und tatsächlich verfügbare Endpunkte
- Projektname und zulässiger Dateibereich
- sicher hinterlegte FileHub-Zugangsdaten
- Zielchat bzw. vereinbarter Benachrichtigungskanal

**Keine** Zugangsdaten, LAN-Adressen oder privaten Gesprächslinks in diesen öffentlichen Skill, Git-Repositories, Chat-Nachrichten oder Logs übernehmen. Ist ein Zugriff nicht konfiguriert, melde den fehlenden Zugang; umgehe die Authentifizierung nicht.

## Dateiprotokoll
- Auftrag: `handoff-<ID>-task.md`
- Ergebnis: `handoff-<ID>-result.md`
- `ID`: eindeutige Zeichenfolge nur mit ASCII-Buchstaben, Ziffern, `_` oder `-`.
- Verwende einen konfigurierten privaten Projektpfad. Veröffentliche Auftragsdaten niemals im öffentlichen Dateibereich.
- Erzeuge für jede Ausführung eine neue ID; niemals vorhandene Dateien durch erneuten Upload überschreiben.

## Auftrag übernehmen
1. Bei einem Signal `JOB <ID> READY | <pfad>` den Dateipfad über die vorhandene FileHub-Schnittstelle abrufen.
2. ID, Existenz und plausiblen Inhalt prüfen. Kein Erfolg melden, solange der Abruf nicht bestätigt wurde.
3. Den Auftrag mit vorhandenen, freigegebenen Werkzeugen ausführen. Für produktive Eingriffe gelten gesonderte Freigaben.
4. Das Ergebnis als neue Datei unter `handoff-<ID>-result.md` im FileHub ablegen. Erfolgreichen Upload und möglichst den Rückabruf/Hash verifizieren.
5. Erst danach im vereinbarten Chat nur `RESULT <ID> READY | <pfad>` senden.
6. Falls die Benachrichtigung scheitert: Ergebnisdatei behalten und den Fehler melden; keine Endlosschleife.

## Auftrag abgeben
1. Auftragsdatei mit neuer ID erzeugen und im FileHub ablegen.
2. Upload und Lesbarkeit prüfen.
3. Danach über einen bereits unterstützten Benachrichtigungsweg nur `JOB <ID> READY | <pfad>` senden.
4. Nach `RESULT <ID> READY` die tatsächliche Ergebnisdatei abrufen und prüfen.

## Verwendung der Werkzeuge
- Nutze bevorzugt eine vorhandene FileHub-API-/MCP-Integration; alternativ die reguläre Shell-/HTTP-Funktion des Agent Computers, soweit freigegeben.
- Das FileHub-Protokoll unterscheidet sich je nach installierter Version. **Prüfe die live vorhandenen Endpunkte**, statt dokumentierte zukünftige Endpunkte vorauszusetzen.
- Für den ChatGPT-Eingang gibt es in diesem Skill **keinen garantierten API-Webhook**. Eine zuvor regulär angemeldete Browser-Sitzung kann kurze Signale übermitteln, wenn deren Nutzung freigegeben und technisch verfügbar ist.
- Ein Rakazo-Skill ist eine **Anweisung**, kein Hintergrunddienst. Er startet weder sich selbst noch beiliegende Skripte. Ein automatischer Start von Rakazo kann separat über eine native Routine mit Webhook erfolgen.

## Abnahme
- Mit einer harmlosen Textdatei beginnen.
- Mindestens nachweisen: Auftrag hochgeladen, Dateiliste/Download erfolgreich, Ergebnis hochgeladen, Ergebnisinhalt korrekt, Signal angekommen.
- In Berichten strikt zwischen einem manuell getesteten Einzelablauf und dauerhaft automatischem Betrieb unterscheiden.
