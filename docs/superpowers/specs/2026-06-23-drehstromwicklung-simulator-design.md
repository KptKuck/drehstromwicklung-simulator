# Drehstromwicklungs-Simulator — Design

Datum: 2026-06-23
Status: Freigegeben (Brainstorming abgeschlossen)

## 1. Ziel & Zweck

Ein eigenstaendiger HTML-Simulator fuer Drehstromwicklungen aller Art. Primaerer
Anwendungsfall: **Motor-Neuwicklung / Praxis** — der Nutzer hat einen vorhandenen
Stator (Nutenzahl Q, Polzahl 2p bekannt) und will eine gueltige Wicklung
entwerfen, pruefen und eine druckbare Wickelanleitung erhalten. Daneben dient das
Werkzeug der vollstaendigen Visualisierung und dem Verstaendnis beliebiger
Drehstromwicklungen.

### Abgedeckte Wicklungsarten
- Einschicht- und Zweischichtwicklung
- Ganzlochwicklung (q ganzzahlig) und Bruchlochwicklung (q gebrochen)
- Zahnspulenwicklung (konzentriert, q < 1)
- Frei waehlbarer Wicklungsschritt (gesehnte Wicklungen; Schleifen-/Wellen-/
  konzentrische Ausfuehrung ergibt sich aus Schritt und Verschaltung)

### Vollstaendige Anzeige (alle vier Darstellungen)
1. Wickelschema (abgewickelt)
2. Nuten-/Spannungsstern
3. Zonenplan & Wicklungsfaktor
4. MMK-Treppenkurve & Drehfeld-Animation

## 2. Nicht-Ziele (YAGNI)

- Keine Maschinen-Vollauslegung (kein Wirkungsgrad-, Drehmoment- oder
  thermisches Modell). Die Drahtauslegung ist eine **Naeherung** und klar als
  solche gekennzeichnet.
- Keine Mehrphasensysteme ausser m = 3 (Drehstrom). Intern wird m als Parameter
  gefuehrt, die UI bietet aber nur m = 3 an.
- Kein Server, kein Build-Schritt, keine externen Bibliotheken.

## 3. Technische Grundsatzentscheidungen

| Entscheidung | Wahl | Begruendung |
|---|---|---|
| Form | Eine self-contained `.html` | Per Doppelklick lauffaehig, kein Build/Server |
| Rendering | SVG durchgaengig | Scharf, druck-/exportfaehig; Animation via `requestAnimationFrame`-Update der SVG-Elemente |
| Berechnung | Generische Nutenstern-Methode (Zeigersumme) | Ein Algorithmus fuer Ganzloch, Bruchloch und Zahnspulen inkl. Oberwellen |
| Abhaengigkeiten | Keine | Vanilla JS, kein Framework |

**Dateigroesse:** Eine vollstaendige Single-File-Umsetzung wird voraussichtlich
~1500–2500 Zeilen umfassen und ueberschreitet damit bewusst die uebliche
Modul-Groessengrenze. Der Self-contained-Wunsch hat hier Vorrang. Gegenmassnahme:
strikte interne Gliederung in klar getrennte, kommentierte Abschnitte (siehe 4).

## 4. Architektur

Reaktives Einweg-Datenflussmuster, alles in einer Datei:

```
state (alle Eingaben)
  -> compute(state) -> winding-Modell (Zonenbelegung, kw, Spulenliste, MMK, Drahtdaten)
    -> Renderer (4x SVG) + Tabellen + Validierungs-Hinweise
```

Jede Eingabeaenderung loest denselben Pfad aus: `recompute() -> rerenderAll()`.

Interne Abschnitte (durch Kommentar-Banner getrennt) innerhalb der einen Datei:

- **CONFIG/STATE** — Default-State, Presets, Konstanten (Phasenfarben etc.)
- **ENGINE** — reine Berechnungsfunktionen (keine DOM-Zugriffe):
  - `windingParams(Q, p, m, layers)` -> q, tau_p, alpha, t
  - `slotStar(Q, p)` -> Zeiger je Nut
  - `assignZones(star, m, zoneWidth)` -> Strang+Vorzeichen je Spulenseite
  - `windingFactor(zones, nu, pitch)` -> kd, kp, kw je Oberwelle
  - `coilList(state)` -> Spulenliste (Nut-von, Nut-nach, Strang, Schicht)
  - `mmf(state, omega_t)` -> Durchflutungs-Treppenkurve zum Zeitpunkt
  - `feasibility(state)` -> {ok, warnings[]}
  - `wireDesign(state)` -> N_strang, N_spule, d_draht, n_parallel, Nutfuellfaktor
- **RENDER** — vier reine Render-Funktionen, je `(winding) -> SVG-String/DOM`:
  `renderSchema`, `renderStar`, `renderZonePlan`, `renderMmfField`
- **UI/CONTROLLER** — Eventbindung, State-Updates, Animation-Loop, Export/Print
- **TESTS** — eingebettete Assert-Sektion (per URL-Param `?selftest=1` aktiv)

## 5. Berechnungs-Engine (Detail)

Grundgroessen:
- Polpaarzahl `p = 2p / 2`, Straenge `m = 3`
- Nuten pro Pol und Strang `q = Q / (2p * m)` (ganz = Ganzloch, gebrochen = Bruchloch)
- Polteilung in Nuten `tau_p = Q / (2p)`
- Elektrischer Nutwinkel `alpha = 360deg * p / Q`
- `t = ggT(Q, p)` (Zahl der Grundsterne / Symmetrie)

**Nutenstern:** Spulenseite/Nut k erhaelt EMK-Zeiger unter Winkel `(k-1)*alpha`
(elektrisch). Bei Zweischicht entstehen Ober- und Unterschicht-Zeiger.

**Zonenzuordnung:** Aufteilung in 2m = 6 Zonen je 60deg (60deg-Phasenband,
Standard). Optional 120deg-Zone. Strangzuordnung mit Vorzeichen in der Reihenfolge
U+, W-, V+, U-, W+, V-.

**Wicklungsfaktor (generisch, je Oberwelle nu):** `kw_nu = kd_nu * kp_nu`
- Verteilungs-/Zonungsfaktor als Betrag der Zeigersumme der Nut-Zeiger eines
  Strangs: `kd_nu = | sum( e^{j*nu*phi_i} ) | / (Anzahl Spulenseiten des Strangs)`
- Sehnungsfaktor aus Schritt: `kp_nu = sin(nu * (y / tau_p) * 90deg)`
- Aequivalente Querprobe: die Vektorsumme ueber die tatsaechlichen Spulenseiten
  (beide Seiten jeder Spule, mit Vorzeichen gemaess Schritt y) liefert kw_nu
  direkt. Beide Wege muessen uebereinstimmen (Test).
- Diese Methode liefert korrekt auch Bruchloch- und Zahnspulenwicklungen.
- Ausgegeben fuer nu = 1, 5, 7, 11, 13, ... (charakteristische Ordnungen).

**Feasibility/Symmetrie:**
- Q muss durch m (=3) teilbar sein.
- Symmetrische Zonenbelegung pruefen; sonst Warnung
  „nicht symmetrisch als Drehstromwicklung realisierbar".
- Hinweis bei sehr ungleicher Zonenbelegung (Bruchloch) auf Wickelkopf-Asymmetrie.

## 6. Eingabe & Auto-Vorschlag

Eingabefelder:
- Nutenzahl Q
- Polzahl 2p
- Schichtzahl (1 / 2)
- Typ (verteilt / Zahnspule)
- Wicklungsschritt y (Auto = Durchmesserschritt = round(tau_p); per Slider sehnbar)
- Zonenbreite (60deg / 120deg)
- Parallele Zweige a
- Schaltung (Stern Y / Dreieck D)

**Auto-Vorschlag:** Bei Eingabe von Q + 2p wird sofort eine gueltige
Standardwicklung berechnet und alle abhaengigen Felder vorbelegt. Danach ist jedes
Feld frei ueberschreibbar (Auto-Vorschlag + manuelle Korrektur).

**Presets:** Dropdown gaengiger Kombinationen, u. a. 24/4, 36/4, 48/4 (verteilt)
sowie 12/10, 12/8, 9/8 (Zahnspule).

## 7. Darstellungen

Alle vier gleichzeitig sichtbar (scrollbares Layout, zusaetzlich Tabs/Umschalter
fuer kleine Bildschirme):

1. **Wickelschema (abgewickelt):** Q Nuten horizontal; Spulen als Boegen, die
   Nutpaare im Abstand y verbinden; Phasen farbig; Ober-/Unterschicht getrennt
   dargestellt; Wickelkopf-Verschaltung; Klemmen U1/U2, V1/V2, W1/W2.
2. **Nuten-/Spannungsstern:** Kreis mit Q EMK-Zeigern, farbig nach Zone,
   Nutnummern beschriftet, Zonensektoren eingezeichnet.
3. **Zonenplan & Wicklungsfaktor:** Tabelle Nut -> Strang/Vorzeichen; Kennwerte
   q, tau_p, kd, kp, kw; Oberwellentabelle (nu, kw_nu).
4. **MMK & Drehfeld:** Durchflutungs-Treppenkurve ueber dem Umfang zum Zeitpunkt t;
   animiertes Drehfeld (3 Strangstroeme zeitlich, Grundwelle rotiert). Play/Pause,
   Geschwindigkeit.

## 8. Drahtauslegung (Naeherung)

Eigenes Panel, deutlich als Naeherung gekennzeichnet.
Eingaben: Strangspannung U, Frequenz f, Leistung P bzw. Strang-Strom I,
Stromdichte J, Luftspaltinduktion B_delta, Bohrungsdurchmesser D,
Blechpaketlaenge L, Nutquerschnitt A_n.

Berechnung:
- Polfluss `Phi = B_delta * (pi * D * L) / (2p)`
- Windungen je Strang aus EMK-Gleichung `U = 4,44 * f * N * kw * Phi` -> `N`
- Windungen je Spule aus N, paralleler Zweige a und Reihen-Spulenzahl
- Leiterquerschnitt `A_Cu = I / (a * J)`, Drahtdurchmesser daraus; ggf. parallele
  Draehte
- Nutfuellfaktor `= (Kupferquerschnitt in der Nut) / A_n`

## 9. Praxis-Output

- **Druckbare Wickelanleitung:** Tabelle Spule-Nr / Nut-von -> Nut-nach / Strang /
  Windungen / Schicht, mit Kenndaten-Kopf (Q, 2p, q, kw, Schaltung).
- **CSS-Print-Stylesheet:** Im Druck nur Wickelanleitung + Schema, Bedien-UI
  ausgeblendet.
- **Export:** Konfiguration als JSON (Datei-Download + URL-Hash zum Teilen);
  SVG/PNG-Export der Diagramme.

## 10. Validierung & Tests

Engine-Funktionen sind rein und gegen Lehrbuch-Referenzwerte testbar. Tests als
eingebettete Assert-Sektion, aktivierbar ueber `?selftest=1`, Ausgabe in einer
Test-Box.

Referenz-Testfaelle (Auswahl):
- 36 Nuten / 4 Pole, Zweischicht, q = 3 -> kw1 ~ 0,960
- 24 Nuten / 4 Pole, Zweischicht, Durchmesserschritt -> bekannte kd/kp
- 12 Nuten / 10 Pole, Zahnspule -> kw1 ~ 0,933
- Symmetrie-Check: Q nicht durch 3 teilbar -> Warnung

## 11. Projektstruktur & Speicherort

```
C:\Work\Drehstromwicklung\
  drehstromwicklung-simulator.html      # das Werkzeug (Deliverable)
  docs\superpowers\specs\               # dieses Design-Dokument + Plan
  README.md
```

Git-Repo neu initialisiert. Commit-Messages auf Englisch.

## 12. Offene Punkte / spaetere Erweiterungen

- Schraegung (Skewing) als zusaetzlicher Faktor ks (spaeter)
- Wellenwicklung als explizite Verschaltungsoption (zunaechst implizit ueber Schritt)
- Mehr Presets / Bibliothek gaengiger Industriemotoren
