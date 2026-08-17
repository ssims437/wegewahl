# Wegewahl

**Der kürzeste Weg und was das Suchen kostet.** Ein Blatt, das nicht nur den Weg zeigt, sondern
jedes Feld, das die Suche anfassen musste, um ihn zu finden.

→ **[Blatt öffnen](https://ssims437.github.io/wegewahl/)**

Eine bemalbare Landkarte (64×36 Felder, Wand / Gras 1 / Sand 3 / Sumpf 8), fünf Vorlagen und
vier Verfahren nebeneinander:

- **Breitensuche** — zählt Schritte, nicht Preise
- **Dijkstra** — ohne Schätzung, kreisförmig
- **A\*** — mit zulässiger Schätzung (Manhattan bzw. Oktil bei Diagonalzügen)
- **Greedy-Best-First** — nur Schätzung, keine Garantie
- **ε-A\*** — Regler für die Übergewichtung der Schätzung, mit gemessenem Umweg und Ersparnis
- **Prüflauf** — 16 384 Wandmuster erschöpfend, dazu zwei Zeilen, die absichtlich brechen müssen

Der Vergleich zeigt beides: die Kosten des Weges **und** die Zahl der angefassten Felder. Auf dem
leeren Feld findet A\* denselben Weg wie Dijkstra und fasst dabei 513 statt 2266 Felder an; hinter
einem Zaun mit Lücke schrumpft der Vorsprung auf 23 %, weil die Schätzung genau in die falsche
Richtung zeigt.

## Was der Prüflauf zeigt

| Behauptung | Ergebnis |
|---|---|
| Dijkstra gegen stumpfe Aufzählung | **16 384 Wandmuster erschöpfend** (4×4-Feld), 12 556 davon ohne Weg, keine Abweichung |
| Dijkstra = Bellman-Ford = Floyd-Warshall | 30 gewichtete Karten, 47 748 Knotenpaare, Abweichung 0 |
| Dreiecksungleichung hält | 29 791 Tripel, 0 Verletzungen |
| A\* bleibt optimal | 36 Karten, Abweichung 0, im Schnitt 23 % weniger Felder |
| ε-A\* bleibt unter dem Faktor ε | 286 Läufe (ε von 1 bis 4), schlechtester Weg 1,750-fach bei ε = 4 |
| **Manhattan + Diagonalzüge ist unzulässig** | **Gegenbeispiel gefunden**: 24,657 statt 24,071 · 45 von 464 Kanten verletzen die Konsistenz |
| Oktil-Schätzung bleibt optimal | 50 Karten mit Diagonalzügen, Abweichung 3,6·10⁻¹⁵ |
| **negative Kante bricht Dijkstra** | Dijkstra sagt 1, Bellman-Ford sagt 0 — der wahre Wert ist 0 |
| zurückgegebene Wege sind gültig | 102 Wege nachgelaufen, 3040 Schritte auf Nachbarschaft, Wände und Kostensumme geprüft |

Zwei Zeilen sind Gegenbeispiel-Zeilen: sie sind **bestanden, wenn etwas schiefgeht**. Findet die
Suche kein Gegenbeispiel, ist nicht der Algorithmus gut, sondern die Erklärung im Blatt falsch.

## Was mich das gekostet hat

**Mein Gegenbeispiel für die negative Kante war keines.** Ich hatte den Standardfall aus dem Kopf
geschrieben: `S→A = 2`, `S→B = 5`, `A→B = −4`, `B→Z = 1` — und behauptet, Dijkstra fände 6 statt −1.
Der Prüflauf sagte: **beide finden −1.** Zu Recht: B wird erst nach A entnommen, und da ist die
negative Kante längst berücksichtigt. Ein Bruch entsteht nur, wenn die negative Kante auf einen
Knoten zeigt, der **schon endgültig abgehakt** ist:

| Kanten | Dijkstra | Bellman-Ford | wahr |
|---|---|---|---|
| S→A=2, S→B=5, A→B=−4, B→Z=1 | −1 | −1 | −1 |
| S→A=1, S→B=2, B→A=−2, A→Z=0 | **1** | 0 | 0 |

Die zweite Zeile funktioniert, weil A mit 1 abgehakt wird, bevor B (mit 2) den billigeren Weg nach A
eröffnet. Der eigene Prüflauf hat also nicht den Code korrigiert, sondern **meine Erklärung**.

**Das Labyrinth war unlösbar, und alle vier Verfahren stimmten überein.** Der Generator höhlt Gänge
auf ungeraden Koordinaten aus; das Ziel lag auf `(W−2, H−2)` = (62, 34) — bei geradem W und H eine
Wandzelle. Ich habe sie freigeräumt, aber alle vier Nachbarn blieben Wand. Im Vergleich stand dann
viermal „kein Weg" bei genau 1053 angefassten Feldern — also die ganze erreichbare Karte. Genau
diese Übereinstimmung war der Hinweis: wenn vier verschiedene Verfahren exakt gleich viele Felder
anfassen, hat keines gesucht, sondern alle haben abgegrast.

**Manhattan mit Diagonalzügen überschätzt — messbar, nicht nur theoretisch.** Bekannt ist, dass die
Schätzung zulässig sein muss. Interessanter ist, wie stark der Bruch tatsächlich durchschlägt: über
60 Karten fand die Suche **ein** Gegenbeispiel (Karte 7, 24,657 statt 24,071 — 2,4 % Umweg),
während gleichzeitig **45 von 464 Kanten** die Konsistenzbedingung verletzen. Viele Verletzungen,
selten Schaden: deshalb hält sich der Fehler in echtem Code so lange, bis jemand nachmisst.

**Die ε-Schranke ist locker.** Über 286 Läufe mit ε von 1 bis 4 war der schlechteste Weg
**1,750-fach**, nicht 4-fach — die Garantie ist also weit vom Beobachteten entfernt. Auf der
Vorlage „Zaun mit Lücke" liefert ε = 2,5 sogar denselben Weg wie Dijkstra bei 61 % weniger Aufwand.
Das ist die praktische Botschaft und gleichzeitig die Falle: wer daraus „ε kostet nichts" macht,
hat 285 gute Fälle gesehen und einen schlechten übersehen.

**Was das Blatt nicht kann:** Es kennt keinen Jump-Point-Search, keine Bidirektionalität, keine
Vorberechnung (Contraction Hierarchies) — die Verfahren, mit denen echte Router arbeiten. Die
Karte ist ein Gitter mit Feldkosten, kein Straßennetz mit Abbiegeverboten. Und die Zahl der
„angefassten" Felder zählt Entnahmen aus der Halde, nicht Speicherzugriffe — als Maß für Aufwand
ist das vergleichbar, als Maß für Laufzeit nur grob.

## Technik

Eine einzelne HTML-Datei. Kein Build, keine Bibliothek, nichts verlässt den Browser.
Canvas 2D, binäre Halde von Hand, typisierte Felder, hell und dunkel.

## Verwandt

- [Plotterblätter](https://github.com/ssims437/plotterblaetter) — Wave Function Collapse, Wellengleichung, Physarum, Lenia
- [Redundanz](https://github.com/ssims437/redundanz) — LZ77 und Huffman, Bitkosten je Zeichen
- [Reparatur](https://github.com/ssims437/reparatur) — Reed-Solomon über GF(256)
- [Würfel](https://github.com/ssims437/wuerfel) — Prüfstand für Zufallsgeneratoren
- [Rechenwerk](https://github.com/ssims437/rechenwerk) — ein Rechner aus NAND-Gattern
- [Nachkomma](https://github.com/ssims437/nachkomma) — IEEE 754, exakt ausgeschrieben
- [Zeitsprung](https://github.com/ssims437/zeitsprung) — Zeitzonen und Sommerzeit
- [Gradtage](https://github.com/ssims437/gradtage) — 41 Jahre Heiz- und Kühlgradtage
- [Stimmführung](https://github.com/ssims437/stimmfuehrung) — Akkorde zu MIDI mit geführten Stimmen
- [Verzerrung](https://github.com/ssims437/verzerrung) — Kartenprojektionen und Tissot-Indikatrizen
- [Handschlag](https://github.com/ssims437/handschlag) — elliptische Kurven und der Schlüsseltausch
- [Frequenzgang](https://github.com/ssims437/frequenzgang) — FFT, Fensterfunktionen und der Leckeffekt
- [Indexbaum](https://github.com/ssims437/indexbaum) — B+-Baum mit gezählten Seitenzugriffen
- [Auszählung](https://github.com/ssims437/auszaehlung) — Wahlverfahren und Sitzverteilung

## Lizenz

MIT
