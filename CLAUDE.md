# Unternehmensbewertung – CLAUDE.md

## Überblick

Szenariobasierte Unternehmensbewertung als Offline-PWA. 3 Schritte: Stammdaten → Risikofaktoren → Ergebnis. Keine Backend-Abhängigkeiten, keine Auth.

## Dateien

| Datei | Zweck |
|-------|-------|
| `index.html` | App-Shell (HTML, CSS, JS alles in einer Datei) |
| `manifest.json` | PWA-Metadaten |
| `sw.js` | Service Worker (Cache-Name: `bewertung-v2`) |
| `icon.svg` | Quell-SVG für Icon-Generierung |
| `icons/icon-192.png` | 192×192 px |
| `icons/icon-512.png` | 512×512 px |
| `apple-touch-icon.png` | 180×180 px |
| `bewertung-pwa.zip` | Backup der Originalversion |

## Hosting

**Live:** https://seppofaz.github.io/Unternehmensbewertung/
**GitHub:** https://github.com/sEppofaz/Unternehmensbewertung
**Deployment:** `git push` → GitHub Pages (automatisch)

## Icon neu generieren

```bash
cd ~/Dropbox/Apps/Claude/Unternehmensbewertung
qlmanage -t -s 512 -o . icon.svg
sips -z 512 512 icon.svg.png --out icons/icon-512.png
sips -z 192 192 icon.svg.png --out icons/icon-192.png
sips -z 180 180 icon.svg.png --out apple-touch-icon.png
rm icon.svg.png
```

Nach Icon-Änderung: Cache-Name in `sw.js` hochzählen (`bewertung-v3` etc.).

## SW Cache-Name

Aktuell: `bewertung-v2`. Bei Änderungen an Icons, `manifest.json` oder `sw.js` selbst hochzählen.

## Bewertungslogik

- Ergebnis ist primär der **Enterprise Value (EV)** aus drei Methoden: Umsatz-Multiple, EBITDA-Multiple, Substanzwert
- **EBITDA-Feld leer**: EBITDA aus Umsatz × Branchenmarge (Damodaran) geschätzt → im Football Field als „(Schätzung)" gekennzeichnet
- **EBITDA-Feld gefüllt**: Tatsächlicher EBITDA für EBITDA-Multiple-Methode
- **Net Debt leer**: Nur EV ausgewiesen
- **Net Debt gefüllt**: Equity Value = EV − Net Debt wird zusätzlich in grünem Block gezeigt

## Risikofaktoren (RISKS-Array)

| ID | Beschreibung | Abschlag |
|----|-------------|---------|
| verluste | Mehrjährige Verluste (≥ 3 Jahre) | –25% |
| fallend | Fallender Umsatz | –15% |
| konzern_neg | Konzerninterne Verflechtung / Intransparenz | –10% |
| illiquid | GmbH-Illiquiditätsabschlag | –15% |
| kunden | Abhängigkeit von wenigen Kunden | –10% |
| wachstum | Starkes Wachstum / positive Trendwende | +15% |
| konzern_pos | Globaler Konzernrückhalt | +10% |

Gesamt-Abschlag: max. 70%.

## Pitfalls

- `manifest.json`: `"purpose": "any"` – NIEMALS `"any maskable"` (iOS schneidet Icon ab, Bug bestätigt 2026-05-22)
- `apple-touch-icon` muss `apple-touch-icon.png` heißen (nicht `icon-180.png`)
- SW fetch: HTML → network-first, Assets → cache-first
- Kein `alert()` verwenden (blockiert Browser-Extension)

## Datenquellen

- Branchenspezifische Multiples: Damodaran (Januar 2025)
- Methodik: IDW S1 / Multiplikatorverfahren
