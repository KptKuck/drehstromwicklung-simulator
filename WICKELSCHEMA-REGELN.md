# Wickelschema – verbindliche Regeln

Diese Regeln sind mit dem Auftraggeber festgelegt. **Vor jeder Aenderung am
Wickelschema (drawPhasePath / Verschaltung) zuerst hier lesen.** Nicht neu
interpretieren – nur ergaenzen, wenn der Auftraggeber eine Regel aendert.

## Begriffe (grafische Elemente einer Spule)
- **Scheitel** – hoechster/tiefster Punkt eines Wickelkopfs (Apex).
- **oberer / unterer Wickelkopf** – Verbindung go<->ret oben bzw. unten ("Dach").
- **Schenkel** – die einzelnen schraegen Geraden eines Wickelkopfs (linker/rechter).
- **Nutleiter / Spulenseite** – Balken in der Nut. **go = Hinleiter (links, Anfang)**,
  **ret = Rueckleiter (rechts, Ende)**.
- **Stromrichtungspfeil** – Polung bei wt=0.
- **Schaltverbindung / Uebergang** – Verbindung einer Spule zur naechsten (Reihe).
- **Ausfuehrung / Klemme** – Strang-Endpunkt (U1, U2, ...).
- **Polgruppe** – Gruppe benachbarter Spulen eines Strangs.

## Verschaltung (Reihenschaltung eines Strangs)
1. **U1 (Anfang) startet IMMER an der inneren Spule** der Polgruppe; verdrahtet
   wird von **innen nach aussen**. **U2 (Ende) an der aeusseren Spule.**
2. Jede Spule wird **go -> ret** (links -> rechts) durchlaufen.
3. Jede **Schaltverbindung startet IMMER im rechten Teil (ret-Seite)** der Spule
   und laeuft zur go-Seite der naechsten Spule. Kein abwechselndes Links/Rechts.
4. **Schaltverbindungen zwischen Spulen: gerade.** Nur kurze
   **Nut-zu-Nut-Uebergaenge (benachbarte Nuten): gebogen**, und nur rechts.
5. Schaltverbindungen/Uebergaenge liegen auf **halber Hoehe der unteren Schenkel**.
6. **Ausfuehrungen U1/U2 ganz unten**: senkrecht von der Nut bis zur Klemme.
   Nur U1 (Anfang) und U2 (Ende) gehen ganz unten; dazwischen nur Schaltverbindungen.

## Darstellung
- Spule bleibt **geschlossene Raute** (oberer + unterer Wickelkopf).
- Spulenform umschaltbar: **gleiche Weite (Schleife)** / **konzentrisch** (verschachtelt,
  unterschiedliche Weiten, gleiche Nutbelegung und gleicher k_w).
- Schema-Hoehe gross (dominiert die Seite); komplette Wicklung **zoombar** (Fit / +/- / Strg+Rad).

## Engine-Invarianten (nicht brechen)
- Feste Nutenzahlen: 12, 18, 24, 36, 48. Drehzahlen nur synchron (3000/1500/1000/750/...).
- Zonenzuordnung ganzzahlig-exakt (symmetrisch auch bei Bruchloch).
- Selbsttest muss 14/14 bleiben; Render-Scan 0 fehlerhafte Attribute ueber alle Presets.
