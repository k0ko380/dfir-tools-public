# Time Zone Verify — Deutsche Ausgabe

Überprüfung der Zeitzone in forensischen Images und extrahierten Dateisystemen.

TZVerify liest die eingestellte Zeitzone eines Geräts aus, vergleicht sie mit der
Referenzzeitzone und liefert bei Abweichungen eine begründete Einschätzung mit
Konfidenzwert und den zugrunde liegenden Belegen.
Diese Ausgabe ist auf **Europe/Berlin** voreingestellt.

## Unterstützte Plattformen
- iOS
- Android
- Windows
- macOS

## Unterstützte Eingaben

| Eingabe | Unterstützt |
|---------|-------------|
| UFDR-Berichtspaket | Ja |
| Dateisystem-Archiv (.zip) | Ja — sehr große Archive vorher in einen Ordner entpacken |
| E01 / Ex01 | Ja |
| Rohimage (.img, .dd, .raw, .001) | Ja |
| Extrahierter Dateisystem-Ordner | Ja — am schnellsten |
| Einzelne .plist- / .db-Datei | Nein |

Bei Ordnern bitte den Ordner übergeben, der direkt `private/` (iOS) bzw.
`data/` und `system/` (Android) enthält.

## Verwendung — Windows

`TZVerify.exe` unter [Releases](../../releases) herunterladen. Keine Installation nötig.
Die Datei `tzverify.ini` (mit `home = Europe/Berlin`) in denselben Ordner legen.

# **Drag & Drop:** Image oder Ordner auf `TZVerify.exe` ziehen.

**Eingabeaufforderung (cmd):**
```
TZVerify.exe <Image_oder_Ordner>
```
**PowerShell:** mit `.\` voranstellen
```
.\TZVerify.exe <Image_oder_Ordner>
```

## Verwendung — Linux / macOS
Die aktuelle Version ist eine Windows-Anwendung. Native Versionen für Linux/macOS
sind geplant. Bis dahin bitte eine Windows-Analysestation verwenden.

## Optionen

| Option | Beschreibung |
|--------|--------------|
| `--home ZONE` | Referenzzeitzone ändern (IANA-Name, z. B. `Europe/Vienna`) |
| `--fast` | Nur Zeitzonen-Artefakte, kein Vollscan (Standard bei E01/IMG) |
| `--full` | Zusätzlich Fotos/Netzbetreiberdaten auswerten (Standard bei Ordnern/UFDR) |
| `--out DIR` | Ausgabeordner für Berichte |

Reihenfolge: `--home` > `tzverify.ini` > Umgebungsvariable `TZVERIFY_HOME` > Systemzeitzone > `Europe/Berlin`.

## Ausgabe
Es werden zwei Berichte erzeugt: `TZVerify_<Name>_<Zeit>.txt` und `.html`.
Alle Zeiten werden in deutscher Zeit angezeigt (z. B. `2026-09-27 13:04:01 CEST (UTC+02:00)`).

## Ergebnisse

| Ergebnis | Bedeutung |
|----------|-----------|
| `MATCH_HOME` | Zeitzone entspricht der Referenzzone |
| `GENUINE_FOREIGN` | Hinweise auf tatsächliche Nutzung in einer anderen Region |
| `STALE_SETTING` | Gerät vermutlich lange ausgeschaltet; alte Einstellung erhalten |
| `MANUAL_OVERRIDE` | Automatische Zeitzone aus; Wert vermutlich manuell gesetzt |
| `INCONCLUSIVE` | Nicht genug Belege — der Bericht nennt, was zu prüfen ist |

Hinweis: Die Berichtstexte selbst sind auf Englisch.

## Einschränkungen
- Im FAST-Modus ohne Erfassungsmetadaten wird der Analysezeitpunkt als Erfassungszeitpunkt verwendet.
- Standortangaben beruhen auf groben Koordinatenbereichen.

## Haftungsausschluss
Nur als Hilfsmittel zur Vorsichtung. Alle Ergebnisse anhand der Primärartefakte
prüfen. Ohne Gewähr, für die rechtmäßige Nutzung durch befugte Sachverständige.
