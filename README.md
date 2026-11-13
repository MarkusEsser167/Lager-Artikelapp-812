# Lagermeldungen 812

Progressive Web App für Lagertablets: Meldung von Lagerplatzänderungen und Verschrottungen.

Standort-Kopie von [Lager-Artikelapp](https://github.com/MarkusEsser167/Lager-Artikelapp) (Standort 106)
für Standort **812** – gleicher Code, eigene Lagerplatz-/Mitarbeiterliste und eigene Empfänger.

Der Startseiten-Header zeigt das WEGO-VTI-Logo (`icons/logo-wego-vti.png`) und die aktuelle
App-Version (`js/version.js`, `APP_VERSION`). Bei jeder inhaltlichen Änderung `APP_VERSION` sowie
`CACHE_NAME` in `sw.js` hochzählen, damit auf einen Blick erkennbar ist, welcher Stand auf einem
Tablet installiert ist.

## Funktionen

- **Lagerplatzänderung melden** – Artikelnummer, Art (**Hauptlagerplatz** oder **Referenzlagerplatz**,
  Pflichtauswahl), neuer Lagerplatz, Bemerkung. Wird beim Absenden sofort als Excel-Datei an
  `frankfurt@wego-vti.de` gemailt.
- **Verschrottung melden** – Artikelnummer, Menge, Grund, mindestens 1 Pflichtfoto (bis zu 4 Fotos), Bemerkung. Wird beim Absenden
  sofort als PDF (mit allen Fotos eingebettet) an `David.Hellwig@wego-vti.de` und
  `anthony.maier@wego-vti.de` gemailt.
- **Massen-Lagerplatzkorrektur** – zur Inventurvorbereitung: Artikel scannen, aktuellen Hauptlagerplatz
  laut System (`data/artikel-lagerplatz.xlsx`) prüfen und ggf. korrigieren. Damit lässt sich **nur der
  Hauptlagerplatz** ändern (fester SAP-Massenupload); Referenzlagerplätze müssen einzeln über
  „Lagerplatzänderung melden“ erfasst werden – die Seite weist deutlich darauf hin. Mehrere Korrekturen
  werden auf der Seite gesammelt und gemeinsam als eine Excel-Datei an `frankfurt@wego-vti.de` gesendet –
  im festen SAP-Massenupload-Format `MATNR` (Artikelnummer), `WERKS`/`LGORT` (fest „812“), `LGPBE` (neuer
  Hauptlagerplatz), ohne weitere Spalten.
- **Lagerplatz-Erfassung (Kamera)** – Massenerfassung pro Lagerplatz: Lagerplatz wählen, dann die Kamera über die
  Ware halten; alle gleichzeitig sichtbaren Barcodes (EAN oder Artikelnummer) werden fortlaufend erfasst
  (`js/massScan.js`, `BarcodeDetector`). Ein Code zählt erst nach zwei Erkennungen. Ergebnis ist eine Liste
  der Artikel auf dem Platz – **keine Mengen**, jeder Artikel nur einmal – mit Abgleich gegen
  `artikel-lagerplatz.xlsx` (passt / abweichend / ohne Referenz / unbekannt). Versand als Excel an die
  Standort-Mailadresse.
  Zusätzlich wird – nur für Artikel, die laut System nicht auf diesem Lagerplatz stehen – eine zweite Excel-Datei im
  Format der Massen-Lagerplatzkorrektur (`MATNR`, `WERKS`, `LGORT`, `LGPBE`) mit dem erfassten Lagerplatz als neuem
  Hauptlagerplatz angehängt.
  Ist der erfasste Lagerplatz für einen Artikel bereits als **Referenzlagerplatz** eingetragen, warnt die App
  (rot im Kamerabild + Vibration, Hinweis in Liste und Mail); solche Artikel kommen nicht in die SAP-Datei,
  da die Massenänderung nur Hauptlagerplätze setzen kann.
  **Sammelversand (ab v1.4.0):** Mehrere Lagerplätze werden nacheinander erfasst („Lagerplatz abschließen –
  nächsten erfassen") und erst mit „Alles senden" in **einer** Mail verschickt (ein Bericht mit Spalte
  Lagerplatz, eine gemeinsame SAP-Datei). Die laufende Erfassung wird als Entwurf in der IndexedDB
  gespeichert und beim nächsten Öffnen fortgesetzt. Artikel, die in einer Runde auf mehreren Plätzen
  erfasst wurden, kommen nicht in die SAP-Datei, sondern werden in der Mail zum Klären aufgeführt.
- **Fehlbestand melden** – nur Artikelnummer (+ optionale Bemerkung), ohne Lagerplatz oder Menge. Wird
  beim Absenden sofort als einfache Text-Mail (kein Anhang) an `frankfurt@wego-vti.de` gemailt.
- Lagerplatzänderung, Verschrottung und Fehlbestand werden einzeln und sofort versendet, kein
  Sammel-Export nötig
- Nach Auswahl einer Artikelnummer (auch in Lagerplatzänderung, Verschrottung und Fehlbestand, nicht
  nur in der Massen-Lagerplatzkorrektur) wird – sofern in `data/artikel-lagerplatz.xlsx` hinterlegt –
  der aktuelle Lagerplatz laut System eingeblendet
- Artikelnummer & Lagerplatz per Barcode-/QR-Scan über die Tablet-Kamera erfassbar, mit
  Suche/Vorschlagsliste aus hinterlegten Listen (Fallback: manuelle Eingabe). Die Suche ist eine
  Volltextsuche über Nummer + Bezeichnung: mehrere durch Leerzeichen getrennte Suchbegriffe (z.B.
  „rigips feuchtraum") müssen alle irgendwo vorkommen, unabhängig von der Reihenfolge
- „Gemeldet von“ ist überall eine Pflicht-Auswahl aus der Mitarbeiterliste (`data/mitarbeiter.xlsx`) –
  keine Freitext-Eingabe, damit es keine anonymen Meldungen gibt
- Meldungen werden lokal auf dem Tablet gespeichert (IndexedDB, als Verlauf) und sind offline nutzbar –
  bei fehlgeschlagenem automatischem Versand gibt es auf der Startseite einen „Erneut senden“-Button
- Automatischer Mailversand über ein Google-Apps-Script-Webhook (dasselbe Script wie bei Standort 106,
  siehe unten); Fallback: Download + E-Mail-Entwurf, falls der automatische Versand nicht erreichbar ist
  (Anhang muss dann manuell hinzugefügt werden)

## Artikel- und Lagerplatzliste aktualisieren

Die Vorschlagslisten für die Formulare liegen als Excel-Dateien im Repo:

- `data/artikelliste.xlsx` – SAP-Exportformat, Spalten `Material` (Artikelnummer) + beliebig viele
  Text-Spalten (`Materialkurztext`, `Artikelkurztext 2`, `Materialkurztext 3`, …), die automatisch zu
  einer Bezeichnung zusammengefügt werden. Optionale Spalte `EAN` (als Text, nicht als Zahl formatieren): wird in der Artikelsuche (ab 6 Ziffern) und beim Scannen berücksichtigt, ein gescannter EAN wird direkt dem Artikel zugeordnet. Aktuell ca. 98.700 Artikel mit EAN. Aktuell ca. 153.000 Artikel (unternehmensweiter Artikelstamm,
  identisch mit Standort 106).
- `data/lagerplatzliste.xlsx` – Spalte mit „Lagerplatz“ im Namen (z.B. auch „FIS/wms® Lagerplatz“ –
  der Parser erkennt Spaltennamen, die einen bekannten Begriff enthalten, nicht nur exakte Treffer).
  Aktuell 62 Plätze für Standort 812.
- `data/mitarbeiter.xlsx` – Spalte `Name`, füllt die „Gemeldet von“-Auswahl in allen Formularen (kein
  Freitext möglich). Aktuell 14 Personen für Standort 812. Bei Personalwechsel hier einfach Namen
  ergänzen/entfernen.
- `data/artikel-lagerplatz.xlsx` – nur für die Massen-Lagerplatzkorrektur: welcher Artikel steht laut
  System aktuell auf welchem Lagerplatz (Spalten wie `artikelliste.xlsx` + `Lagerplatz` + optional
  `KZ Hlp`; zusätzliche Spalten wie `Lieferantenkennzeichen` werden einfach ignoriert). Ein Artikel darf
  mehrfach vorkommen (mehrere Lagerplätze) – die Zeile mit `KZ Hlp` = „X“ gilt als Hauptlagerplatz und
  wird in der App zuerst und mit „(Haupt)“ markiert angezeigt, weitere Lagerplätze folgen
  kommagetrennt (`js/refData.js`: `groupArtikelLagerplatz`). Aktuell ca. 1.450 Artikel für Standort 812.

Erkannt werden außerdem die gängigen deutschen Varianten (`Artikelnummer`, `Bezeichnung`, `Beschreibung`
usw.) – siehe `js/refData.js`. Nur die erste Spalte (Nummer/Platz) muss einen dieser Begriffe im
Spaltennamen enthalten, weitere Spalten sind optional.

Um eine Liste zu aktualisieren: auf GitHub in den Ordner `data/` gehen, die jeweilige Datei öffnen und über
„Upload file“ durch eine neue Version ersetzen (Commit direkt im Browser). Es ist **keine Code-Änderung**
nötig – die App liest beim nächsten Laden automatisch die aktuelle Version. Tablets, die offline sind,
nutzen bis zur nächsten Online-Verbindung weiterhin die zuletzt geladene Version.

## Automatischer Mailversand

Jede Meldung wird beim Absenden einzeln per E-Mail über ein Google-Apps-Script-Webhook
(`apps-script/Code.gs`) verschickt – **dasselbe Script/dieselbe Bereitstellung wie bei Standort 106**
(Google-Konto `wegounfallapp@gmail.com`), da es komplett generisch ist (Empfänger/Betreff/Anhang kommen
aus dem Request). Für diese App-Kopie war deshalb **keine neue Bereitstellung nötig** – `MAIL_SCRIPT_URL`
in `js/export.js` zeigt auf dieselbe Exec-URL. Empfänger und Format sind je nach Meldungstyp fest
hinterlegt (`js/export.js`):

| Meldungstyp | Empfänger | Format |
|---|---|---|
| Lagerplatzänderung | `frankfurt@wego-vti.de` | Excel (1 Zeile) |
| Verschrottung | `David.Hellwig@wego-vti.de`, `anthony.maier@wego-vti.de` | PDF mit bis zu 4 eingebetteten Fotos (`js/pdf.js`) |
| Massen-Lagerplatzkorrektur | `frankfurt@wego-vti.de` | Excel im SAP-Format `MATNR`/`WERKS`/`LGORT`/`LGPBE` (1 Zeile je Korrektur, gesammelt) |
| Fehlbestand | `frankfurt@wego-vti.de` | Kein Anhang – reine Text-Mail |

Solange `MAIL_SCRIPT_URL` leer ist, fällt die App automatisch auf Download + vorausgefüllten
E-Mail-Entwurf zurück (Anhang muss dann manuell hinzugefügt werden). Details zur Einrichtung/zu einem
eigenen Script stehen im README von [Lager-Artikelapp](https://github.com/MarkusEsser167/Lager-Artikelapp)
(Standort 106).

**Bekanntes Risiko:** Bei der Unfallaufnahme-App (Vorbild für dieses Skript) wurden E-Mails vom
Gmail-Absender an manche `@wego-vti.de`-Adressen per SPF abgelehnt. Bei `muenster@wego-vti.de` und
`Martin.Jochheim@wego-vti.de` (Standort 106) funktioniert es nachweislich; ob das auch für
`frankfurt@wego-vti.de`, `David.Hellwig@wego-vti.de` und `anthony.maier@wego-vti.de` gilt, ist nach dem
ersten echten Versand noch zu bestätigen. Falls Mails nicht ankommen, muss die IT den Gmail-Absender für
die jeweilige Adresse freischalten.

## Installation auf einem Tablet

1. Diese App über GitHub Pages öffnen (Chrome auf Android)
2. Menü → „Zum Startbildschirm hinzufügen“
3. Die App läuft danach wie eine installierte App, auch offline

## Entwicklung

Kein Build-Prozess nötig – reines HTML/CSS/JS. Lokal starten:

```
python -m http.server 8421
```

Danach `http://localhost:8421` öffnen.
