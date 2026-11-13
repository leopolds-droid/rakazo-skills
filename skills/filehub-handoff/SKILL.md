---
name: filehub-handoff
description: Tauscht Aufträge und Ergebnisse zwischen KI-Agenten über den vorhandenen FileHub mit curl aus; Chats enthalten nur kurze Signale.
---

# FileHub Handoff – Minimalweg

**Ein Skill, ein FileHub, ein Werkzeug: `curl`.** Keine zusätzliche OpenAPI-, MCP- oder Upload-Adapter-Integration. Keine zweite Dateiablage.

## Vorbereitung (einmalig, außerhalb dieses öffentlichen Skills)
- Auf dem Agent Computer muss `curl` vorhanden sein.
- FileHub-URL, Projekt und geschütztes `curl`-Konfigurationsfile aus der **privaten** Betriebsumgebung verwenden.
- Das Konfigurationsfile (z. B. `~/.config/filehub/curl.conf`, Modus 0600) enthält den FileHub-Zugangsheader. **Keine Tokens oder privaten URLs in Git, Chat, Tool-Ausgaben oder ungeschützte Logs schreiben.** Kein `curl -v` und keine Request-Header ausgeben.
- Falls der Zugang fehlt: Einrichtung melden, keine Authentifizierung umgehen. Der Skill darf keinen Token aus anderen Systemen auslesen.
- Die installierte Legacy-API hat `/health`, `/list`, `/download`, `/upload`. `/v1/*` erst nutzen, wenn live bestätigt.

## Dateinamen
`handoff-<ID>-task.md` und `handoff-<ID>-result.md`. Neue eindeutige ID nur aus `[A-Za-z0-9_-]`. Nie vorhandene Dateien überschreiben; bei Wiederholung erst Dateiexistenz prüfen. Auftrag/Ergebnis nur in einem freigegebenen privaten FileHub-Projektbereich ablegen.

## Kommandos (auf Chiefs Agent Computer)
Die folgenden Variablen werden aus der **privaten** Laufzeitkonfiguration gesetzt, nicht aus untrusted Aufträgen:

```sh
# Beispiele: FILEHUB_URL=https://.../api/filehub
# FILEHUB_PROJECT=allgemein
# FILEHUB_CURL_CONFIG=$HOME/.config/filehub/curl.conf

# Dateiliste; kind=all ist wichtig, da der Legacy-Default 'pieces' nur Musikdateien zeigt.
curl -fsS --config "$FILEHUB_CURL_CONFIG" --get \
  --data-urlencode "project=$FILEHUB_PROJECT" \
  --data-urlencode "kind=all" \
  "$FILEHUB_URL/list"

# Gewünschten Pfad exakt aus der Liste verwenden:
curl -fsS --config "$FILEHUB_CURL_CONFIG" --get \
  --data-urlencode "path=$FILEHUB_PATH" \
  --output "$LOCAL_FILE" "$FILEHUB_URL/download"

# Neue lokale Ergebnisdatei hochladen; Namen vorher auf ASCII-ID-Muster prüfen:
curl -fsS --config "$FILEHUB_CURL_CONFIG" \
  --request PUT --data-binary "@$LOCAL_FILE" \
  --write-out '\nHTTP=%{http_code}\n' \
  "$FILEHUB_URL/upload?project=$FILEHUB_PROJECT&area=projects&name=$FILEHUB_NAME"
```

Beim Upload **HTTP 201** und `ok=true` kontrollieren. Anschließend über `/download` den exakten Ergebnis-Pfad zurücklesen und SHA-256 vergleichen. Keine Löschoperationen. Rohdatei-Upload nur mit `--data-binary` (kein JSON).

## Ablauf
1. Signal empfangen: `JOB <ID> READY | <pfad>`.
2. Auftragsdatei aus FileHub abrufen und ID prüfen.
3. Aufgabe bearbeiten; produktive Eingriffe nur mit ausdrücklich bestehender Freigabe.
4. Ergebnis unter eindeutigem neuen Dateinamen per `curl` hochladen und zurückprüfen.
5. Erst dann im fest vereinbarten Chat ausschließlich `RESULT <ID> READY | <pfad>` melden.
6. Gegenseitige Aufträge genauso als Dateien ablegen; im Chat ausschließlich `JOB <ID> READY | <pfad>` melden. Keine Endlosschleifen.

Eine vorhandene, regulär angemeldete ChatGPT-Browsersitzung darf nur für kurze Signale genutzt werden. **ChatGPT hat keinen nativen Echtzeit-Webhook für diese Unterhaltung.** Der Skill ist eine Anleitung, kein Hintergrunddienst und startet keine Skripte selbst. Für automatische Starts von Rakazo kann später dessen native Webhook-Routine genutzt werden.

## Abnahme
Harmloser Test: Datei erstellen → hochladen (201) → auflisten (`kind=all`) → herunterladen (200) → Inhalt und SHA-256 prüfen → kurzes Signal senden. Einzeltest und dauerhaften Automatikbetrieb getrennt bewerten.
