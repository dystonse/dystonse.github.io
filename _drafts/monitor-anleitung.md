---
layout: post
title:  "Erweiterter Abfahrtsmonitor - Anleitung"
categories: frontend
excerpt_separator: <!--more-->
---

# Was ist der erweiterte Abfahrtsmonitor?

Die meisten Abfahrtsmonitore, denen wir bisher begegnet sind – egal ob im Internet oder an einer Bus-/Bahnhaltestelle – zeigen immer nur **genau eine Zeit** pro Fahrzeug an. Manchmal sind das einfach die Zeiten aus dem Fahrplan, manchmal ist es eine Echtzeit-basierte Prognose vom Verkehrsanbieter, der z.B. die aktuelle Position des Fahrzeugs kennt. Manchmal weiß man auch gar nicht so genau, worauf diese Zeit im Einzelfall basiert. Wir haben jedenfalls schon oft genug erlebt, dass auf die Zahlen, die auf so einem Haltestellendisplay stehen, nicht immer Verlass ist. 
Aber um zu wissen, ob so eine Zahl zuverlässig ist, muss man schon ziemlich oft mit genau diesem Bus gefahren sein - dann hat man vielleicht eine Intuition dafür, wie weit die Abfahrtszeiten in Wirklichkeit verteilt sind. Das kann sich kein Mensch für alle Fahrzeuge merken – aber zum Glück gibt es inzwischen Computer, die uns diese Arbeit abnehmen.

Leider haben wir bisher trotzdem noch kein Abfahrtsdisplay gesehen, wo aus diesen Informationen mehr gemacht wird, als nur eine einfache Uhrzeit, vielleicht noch mit einer zusätzlichen Verspätungsangabe in Minuten.

Deshalb haben wir den erweiterten Abfahrtsmonitor entwickelt. Hier kannst du die **Wahrscheinlichkeitsverteilung**, wann dein Bus abfahren wird, als **grafische Visualisierung** sehen. Weil wir davon überzeugt sind, dass diese Information dir ein umfassenderes Bild davon vermitteln kann, wie sicher dein Ein-/Umstieg in Wirklichkeit ist. 

Außerdem kannst du nicht nur die Abfahrten an einer Haltestelle sehen, sondern auch ausprobieren, wie deine Fahrt weitergeht, und deine **Reisekette mit Umstiegen** selbst zusammenstellen. Natürlich auch inklusive grafisch dargestellter Wahrscheinlichkeitsverteilung der Ankunfts- und Abfahrtszeiten.

Und weil das auch manchmal nicht übersichtlich genug ist, kannst du bei uns auch den **Prozentwert** sehen, mit welcher Chance du alle Anschlüsse auf deiner Reise bekommen wirst.

<!--more-->

## So geht's

Auf der Startseite kannst du die Haltestelle auswählen, von der deine Reise starten soll. Aktuell stehen alle Haltestellen im Gebiet des Verkehrsverbundes Bremen/Niedersachsen (VBN) zur Auswahl.

![Screenshot von der Startseite mit Haltestellen-Auswahlmenü und "Abfahrten anzeigen"-Button](/assets/monitor/haltestellenauswahl.png)

## An der ersten Haltestelle

Mit einem Klick auf "Abfahrten anzeigen" gelangst du zur Übersicht aller Verkehrsmittel, die in der nächsten halben Stunde von der gewählten Haltestelle abfahren und **mit mindestens 5% Wahrscheinlichkeit noch erreichbar** sind. 

In der Überschrift siehst du außerdem eine Angabe, von wievielen weiteren Haltestellen in der Nähe die Abfahrten ebenfalls angezeigt werden. Wenn du die Maus über die Schrift "(und … weitere)" bewegst, werden die Namen dieser Haltestellen als Tooltip eingeblendet.

![Screenshot der Abfahrtsseite für Bremen Hauptbahnhof (mit Tooltip inkl. "Bremen Hbf")](/assets/monitor/abfahrten-bremen-hauptbahnhof-mit-tooltip.png)

Mit dem Lupen-Symbol oben links (hier und auf allen weiteren Seiten) kommst du jederzeit wieder zurück zur Haltestellen-Auswahl.

Für jede Fahrt werden folgende Infos in einer Zeile angezeigt:

![Screenshot einer Zeile mit Markierungen 1 bis 10](/assets/monitor/abfahrt-zeile-mit-markierungen.png)

1. Abfahrtszeit laut Fahrplan
2. Früheste Abfahrtszeit, die in 99% der Fälle nicht unterschritten wird (in Minuten Abweichung vom Fahrplan, genaue Uhrzeit als Tooltip)
3. Mittlere Abfahrtszeit.
4. Späteste Abfahrtszeit, die in 99% der Fälle nicht überschritten wird.
5. Verkehrsmittel-Typ
6. Liniennummer
7. Richtung / Ziel
8. Bei Abfahrt von einer Haltestelle in der Nähe: Fußweg-Symbol und Entfernung (mit Fußwegdauer und Haltestellenname als Tooltip).
9. Wahrscheinlichkeit in Prozent, dass dieser Umstieg erreicht wird
10. Kürzel für die Qualität der Datenmenge, auf der diese Vorhersage basiert (siehe unten)

Unterhalb der Zeile ist die Wahrscheinlichkeitsverteilung der Abfahrtszeit durch einen farbigen Balken in blau/grün/gelb dargestellt: Je dunkler die Farbe, desto höher die Chance, dass das Fahrzeug tatsächlich zu diesem Zeitpunkt abfährt. Die Abfahrtszeit laut Fahrplan ist durch ein schwarzes Dreieck markiert.

![Screenshot einer Zeile mit blau-grün-gelbem Balken und Zeitskala](/assets/monitor/abfahrt-farbbalken-mit-dreieck.png)

## Einsteigen und Weiterfahren

Durch Klick auf eine Abfahrts-Zeile kommst du auf die nächste Seite. Dort werden alle weiteren Halte der gewählten Fahrt angezeigt - ebenfalls mit Zeitangaben und farbigen Balken (in braun/orange/gelb) zur Darstellung der Ankunftszeit-Verteilung. 

Eine Wahrscheinlichkeit wird hier nur in der obersten Zeile angezeigt. Denn wenn du den Bus erstmal erwischt hast, gehen wir davon aus, dass du genauso wahrscheinlich auch später aus diesem Bus wieder aussteigen kannst (Die Fälle, in denen sich der Bus unterwegs in Luft auflöst, sind so selten, dass sie bei unseren Datenmengen nicht ins Gewicht fallen...).

![Screenshot einer Busfahrt-Seite](/assets/monitor/busfahrt-seite.png)

Klicke dann auf die Haltestelle, an der du umsteigen möchtest. 

## Umsteigen

Du gelangst dann auf eine ähnliche Seite wie die vorherige, wo die weiteren Abfahrten an dieser Haltestelle aufgelistet sind. Für die Berechnung der Wahrscheinlichkeit, dass du den Anschluss bekommst, wird jetzt berücksichtigt, wann du wahrscheinlich mit dem Fahrzeug ankommst, was in der obersten Zeile angezeigt wird.

![Screenshot einer Abfahrtsseite mit Ankunft per Bus](/assets/monitor/abfahrten-mit-ankunft-per-bus.png)

## Reisekette

So kannst du dir deine ganze Reisekette zusammenstellen, und sehen, wie wahrscheinlich du alle Umstiege erreichst. Je nach Vorliebe kannst du dich damit schon vor der Fahrt entscheiden, ob du lieber eine Route nimmst, die nicht unbedingt klappt, aber wenn, dann eben schneller am Ziel ist – oder lieber eine sichere Route, die dafür etwas längere Umsteigezeiten enthält. Und natürlich alles dazwischen.

In beiden folgenden Beispielen beginnt die Reise um **19:00** an der Haltestelle **Braunschweig Rudolfplatz**.

Mit der schnelleren Verbindung kommst du evtl. schon um **19:42** in **Salzgitter-Lebenstedt** an - aber die Ein- und Umstiege sind insgesamt nur mit **5%** Chance überhaupt schaffbar.

Mit der langsameren Verbindung bist du zwar erst um **20:10** in **Salzgitter-Lebenstedt**, aber dafür sind die Umsteigezeiten lang genug, dass diese Verbindung zu **100%** klappt.

<a href="/assets/monitor/reisekette_schnell_aber_unwahrscheinlich.png"><img title="Schnelle, aber unsichere Verbindung" src="/assets/monitor/reisekette_schnell_aber_unwahrscheinlich.png" width="49%"></a>
<a href="/assets/monitor/reisekette_langsam_aber_wahrscheinlich.png"><img title="Langsame, aber sichere Verbindung" src="/assets/monitor/reisekette_langsam_aber_wahrscheinlich.png" width="49%"></a>

In der Navigationszeile oben links kannst du jederzeit zu früheren Teilen der Reise zurückspringen, um andere Varianten auszuprobieren.

![Screenshot Breadcrumbs-Navigation](/assets/monitor/breadcrumbs.png)

## Datenqualität

Um die Wahrscheinlichkeits-Verteilung von Abfahrt und Ankunft auszurechnen, haben wir Echtzeitdaten aus der Vergangenheit gesammelt und darüber Statistiken erstellt. Echtzeitdaten sind aber nicht für alle Linien verfügbar, daher kann die Qualität der Prognose im Einzelfall sehr unterschiedlich sein.

An den Kürzeln ganz rechts in jeder Zeile kannst du sehen, was die Grundlage für die dort angezeigte Verteilung ist.

![Screenshot: Verschiedene Daten-Kürzel mit Tooltip](/assets/monitor/symbole-fuer-datenqualitaet-mit-tooltip.png)

Die Kürzel bestehen aus zwei Teilen. Der erste Buchstabe gibt an, ob wir aktuelle Echtzeitdaten für diese Fahrt haben und sie zur Berechnung der Verteilung mit einbeziehen:

| ----- | -------------------------------------- |
| **E** | Aktuelle Echtzeitdaten werden genutzt. |
| **U** | Wir haben zwar aktuelle Echtzeitdaten, aber noch nicht genug historische Aufzeichnungen zu dieser Linienvariante, um sie damit zu verknüpfen - deswegen werden die Echtzeitdaten hier noch nicht genutzt. |
| **P** | Für diese Fahrt haben wir keine aktuellen Echtzeitdaten. |


Der Zweite Teil des Kürzels gibt an, wie genau die Statistische Auswertung auf diese Fahrt und Haltestelle zugeschnitten ist:

| ------ | ------------------------------------------------------------------------------------------------------------------------ |
| **S+** | Statistik für diese Wochen-/Tageszeit, diese Linienvariante und diese Haltestelle, verknüpft mit aktuellen Echtzeitdaten |
| **S**  | Statistik für diese Linienvariante und diese Haltestelle, verknüpft mit aktuellen Echtzeitdaten |
| **S-** | Statistik für diese Linienvariante und diese Haltestelle |
| **G+** | Statistik für diese Wochen-/Tageszeit, Verkehrsmittel-Typ und Streckenabschnitt (etwas ungenau) |
| **G**  | Statistik für diesen Verkehrsmittel-Typ (ungenau) |
| **G-** | Statistik für alle Verkehrsmittel (sehr ungenau) |

Als Tooltip wird dir außerdem eine Zahl angezeigt, die angibt, auf wievielen Aufnahmen die Berechnung basiert. Was diese Zahl genau bedeutet, erklären wir demnächst noch genauer – das ist im Moment noch nicht so wichtig.

## Mehr Info und Hintergrundwissen

Wenn du es noch genauer wissen willst, dann schau dir gerne den [Dystonse Blog](https://blog.dystonse.org) an. Dort berichten wir von der Entwicklung dieses erweiterten Abfahrtsmonitors und erklären verschiedene Details zu den Verarbeitungsschritten, die hinter den Kulissen ablaufen. 
