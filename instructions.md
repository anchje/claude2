# Wöchentliche Angebots-Recherche – moeve.de/sale

## Zweck

Jede Woche werden die Angebote auf `https://www.moeve.de/sale` vollständig erfasst,
strukturiert abgelegt und mit dem Ergebnis des unmittelbar letzten Laufs verglichen.
So entsteht über die Zeit eine Historie der Sale-Angebote inkl. aller Preis- und
Sortimentsänderungen.

## Datenquelle

- URL: https://www.moeve.de/sale
- Es müssen **alle** auf der Sale-Seite gelisteten Angebote erfasst werden – die
  tatsächliche Anzahl richtet sich nach dem aktuellen Sortiment der Website (Stand
  erster Testlauf am 2026-09-02: 143 Artikel, verteilt auf 6 Seiten à 24). Es gibt
  keine feste Mindestanzahl; Ziel ist Vollständigkeit, nicht ein fixer Schwellenwert.
- Die Seite ist ein Shopware-Shop mit serverseitig gerendertem HTML und URL-Pagination
  über den Parameter `?p=<n>` (z. B. `https://www.moeve.de/sale?p=2`). Die Gesamtzahl
  Seiten steht im HTML unter `data-listing-pagination-options` bzw. den
  `data-page="n"`-Attributen. Alle Seiten müssen abgerufen werden, bis keine weitere
  Seite mehr existiert.
- Duplikate (gleiche Artikel-ID/URL) nur einmal zählen.

## Zu erfassende Datenfelder pro Angebot

| Feld | Beschreibung |
|---|---|
| Artikelbezeichnung | Produktname/Titel wie auf der Seite angezeigt |
| Artikel-ID / SKU | falls vorhanden (z. B. aus Produkt-URL oder Datenattribut) |
| Marke | falls angegeben |
| Ursprungspreis (UVP/Streichpreis) | Preis vor Rabatt |
| Aktueller Preis | Sale-Preis |
| Ersparnis absolut | Ursprungspreis − Aktueller Preis |
| Ersparnis in % | gerundet auf ganze Prozent |
| Produkt-URL | Link zum Artikel |
| Verfügbarkeit | falls angegeben (z. B. "auf Lager", "wenige verfügbar") |

Fehlt ein Feld auf der Seite, wird die Zelle leer gelassen (nicht raten).

## Ersparnisgruppen

Die Angebote werden nach Ersparnis in % in folgende Gruppen eingeteilt:

- 0–9 %
- 10–19 %
- 20–29 %
- 30–39 %
- 40–49 %
- ab 50 %

Innerhalb jeder Gruppe werden die Angebote als Tabelle abgelegt, sortiert absteigend
nach Ersparnis in %.

## Ablauf pro Durchlauf

1. **Datum ermitteln**: aktuelles Datum als `YYYY-MM-DD`.
2. **Verzeichnis anlegen**: `runs/YYYY-MM-DD/` (relativ zu diesem Ordner `claude2/`).
3. **Seite abrufen und alle Angebote extrahieren** (siehe Datenfelder oben).
4. **Ablage der Rohdaten**: `runs/YYYY-MM-DD/offers.json` – vollständige Liste aller
   Angebote als JSON-Array (ein Objekt pro Angebot, Felder wie oben).
5. **Tabellenreport erzeugen**: `runs/YYYY-MM-DD/offers.md` – Angebote gruppiert nach
   Ersparnisgruppe, je Gruppe eine Markdown-Tabelle mit den Feldern aus Abschnitt
   "Zu erfassende Datenfelder". Vor jeder Tabelle die Gruppenüberschrift inkl. Anzahl
   der Angebote in dieser Gruppe.
6. **Vorherigen Lauf ermitteln**: Das jüngste vorhandene Verzeichnis unter `runs/`,
   dessen Datum vor dem aktuellen Lauf liegt (chronologisch, nicht zwingend genau
   7 Tage zuvor – z. B. falls ein Lauf ausgefallen ist).
   - Existiert kein vorheriger Lauf: Schritt 7 und 8 entfallen, stattdessen ein
     Hinweis "Kein Vergleich möglich – erster Lauf" im Dashboard.
7. **Vergleich mit Vorlauf** (Abgleich über Artikel-ID, ersatzweise Produkt-URL oder
   Artikelbezeichnung, falls keine ID vorhanden):
   - **Neue Angebote**: nur im aktuellen Lauf vorhanden.
   - **Weggefallene Angebote**: nur im vorherigen Lauf vorhanden.
   - **Preisänderungen**: aktueller Preis weicht vom Preis im Vorlauf ab (Richtung
     und Betrag angeben).
   - **Ersparnisänderungen**: Ersparnis-% weicht ab (auch bei gleichem Preis möglich,
     falls sich der Ursprungspreis geändert hat).
   - **Sonstige Änderungen**: z. B. Artikelbezeichnung oder Verfügbarkeit geändert.
8. **Abweichungsliste ablegen**: `runs/YYYY-MM-DD/abweichungen.md` – eine Liste je
   Kategorie (Neu / Weggefallen / Preisänderung / Ersparnisänderung / Sonstige),
   pro Eintrag: Artikelbezeichnung, alter Wert → neuer Wert.
9. **Dashboard erzeugen**: `runs/YYYY-MM-DD/dashboard.md` mit Kennzahlen:
   - Gesamtzahl Angebote (aktuell) vs. Vorlauf
   - Anzahl Angebote je Ersparnisgruppe (aktuell), mit Delta zum Vorlauf
   - Anzahl neue Angebote
   - Anzahl weggefallene Angebote
   - Anzahl Angebote mit Preisänderung
   - Anzahl Angebote mit Ersparnisänderung
   - Datum des Vorlaufs, mit dem verglichen wurde (oder Hinweis "kein Vorlauf")

## Verzeichnisstruktur

```
claude2/
  instructions.md
  runs/
    2026-09-02/
      offers.json
      offers.md
      abweichungen.md
      dashboard.md
    2026-09-09/
      ...
```

## Hinweise

- Immer echte, von der Website gelesene Werte verwenden – keine Schätzungen oder
  Platzhalter.
- Bei strukturellen Änderungen der Website (z. B. neue Feldbezeichnungen), die die
  Extraktion beeinflussen, dies im Dashboard als Hinweis vermerken.
- Ziel ist Reproduzierbarkeit: derselbe Ablauf soll jede Woche mit denselben Regeln
  wiederholt werden können, damit die Vergleiche über die Zeit konsistent bleiben.
