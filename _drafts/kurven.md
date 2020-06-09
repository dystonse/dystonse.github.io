---
layout: post
title:  "Verspätungskurven"
categories: analysis
date:   2020-06-09 15:26:34 +0100
excerpt_separator: <!--more-->
---

Seit Monaten sammeln wir Verspätungsdaten: aktuell knapp 7 Millionen Datenpunkte, vor allem aus dem Verkehrsverbund Bremen Niedersachsen. Das ist natürlich kein Selbstzweck.

Aus den vergangenen Verspätungsdaten möchten wir Statistiken erstellen, und daraus Prognosen für die nahe Zukunft. Und damit letztlich unserer Routensuche die bestmöglichen Ergebnisse liefern.

Aber Moment - bereits in unserem [vorletzen Post](/opendata/2020/03/13/datensammlung.html) schrieben wir doch, dass wir von den Verkehrsbetrieben bereits Prognosen bekommen. Warum denn so viel Aufwand für eigene Prognosen?

<!--more-->

## Klassische Prognosen
Zunächst muss man mal betrachten, was die Verkehrsbetriebe unter _Prognosen_ verstehen. Für ein bestimmtes Fahrzeug wird vorhergesagt, wann es an einer bestimmten Haltestelle ankommen oder abfahren wird. Dabei wird die planmäßige Zeit um eine Anzahl von Sekunden vor- oder zurückverlegt. Das Ergebnis ist also ein Zeitpunkt.

Dieser Zeitpunkt wird meist mit "sekundengenau" angegeben - wobei der Sekundenanteil bei unseren gesammelten Daten überproportional oft durch 6 oder gar durch 12 teilbar ist, was auf irgendeine Form von Rundung hinweist und bei uns zunächst für einige Verwirrung gesorgt hat... wer uns auf Twitter folgt, hat das vielleicht schon mitbekommen:

[![Screenshot eines Tweets, in dem wir uns über die Häufung der Zahl 12 wundern.](/assets/kurven/tweet_1254806815981920256.png)](https://twitter.com/dystonse/status/1254806815981920256)

[![Screenshot eines Tweets, in dem wir die Vermutung beschreiben, dass die Zahlen schon gerundet sind.](/assets/kurven/tweet_1268606241892425728.png)](https://twitter.com/dystonse/status/1268606241892425728)

Egal, wie stark oder schwach gerundet diese Zahlen sind, es handelt sich um jeweils eine einzige, quasi _alternativlose_ Prognose - so als könnte man mit Sicherheit sagen, wie viel Verspätung ein Verkehrsmittel in Zukunft haben wird. Dabei ist die Existenz von Verspätungen doch selbst schon Beweis dafür, dass ÖPNV nicht deterministisch einem Plan folgt. Realistisch betrachtet müsste jede Prognose natürlich mit irgendeiner Form von Unsicherheit behaftet sein, aber in den Daten ist diese nicht zu finden.

Im quasi-Standard GTFS-Realtime (den in Deutschland längst nicht alle Verkehrsbetriebe unterstützen) ist sogar ein Datenfeld vorgesehen, um Unsicherheiten von Prognosen auszudrücken - [dessen statistische Bedeutung aber explizit undefiniert ist](https://developers.google.com/transit/gtfs-realtime/guides/trip-updates#uncertainty) und das weltweit [kaum ein Datenanbieter nutzt](https://github.com/google/transit/pull/111#issuecomment-464783871). Also haben wir unsere eigene Defition für Unsicherheit geschaffen.

In diesem Blogpost soll es also darum gehen, was wir definiert haben, und wie bzw. warum wir das getan haben. Und letztlich auch um die Erkenntnisse, die dabei aufgetreten sind.

## Zweck unserer Analysen und Prognosen
Eine zentrale Fragestellung innerhalb unserer Routenberechnungen ist, ob ein Anschluss erreicht wird, oder genauer gesagt: mit welcher Wahrscheinlichkeit er erreicht wird, und wenn ja, wann das wohl sein wird. Ein Anschluss kommt dann zustande, wenn das erste Fahrzeug ankommt, bevor das zweite abfährt, und der Zeitabstand größer ist als die Zeit, die zum Umstieg gebraucht wird.

Dazu brauchen wir für beide Fahrzeuge eine Prognose für die Ankuft bzw. Abfahrt an der Umsteigehaltestelle. Erfahrungen mit unserem ersten Prototyp aus dem August 2019 haben gezeigt, dass wir für eine einzelne Routenabfrage oft viele Tausende solcher Prognosen brauchen. Damals hatten wir eine mittelgroße Datensammlung aus hunderttausenden Verspätungen auf wenige hunderte Bytes herunter gebrochen, die in Form von [Arrays und Maps direkt im Quelltext](https://github.com/dystonse/dystonse-search-node/blob/master/test.js#L29) standen. Damit konnten wir Prognosen innerhalb von Mikrosekunden berechnen, ohne auf Datenbanken, Netzwerkdienste oder externe Dateien zurück zu greifen. Entsprechend grob, realitätsfern und frei von aktuellem Kontext waren die Prognosen aber auch.

Diesmal gilt es, einen guten Kompromiss zu finden: Wie können wir in unsere Prognosen möglichst viele Details unserer 1,7 GB gesammelten Daten einfließen lassen, ohne dafür ebensoviele Daten im RAM vorzuhalten und sie immer wieder aufs neue durchzuarbeiten? Können wir die Essenz der Daten mathematisch fassen, verlustarm komprimieren und so ablegen, dass wir effizient Abfragen darauf durchführen können?

Natürlich haben wir etwas recherchiert, ob es bereits bewährte Methoden dazu gibt, jedoch ohne nennenswerten Erfolg. Vielleicht fehlen uns die Fachkenntnisse in multivariater Statistik, um das richtige Verfahren aus den vorhandenen zu erkennen, dafür sind wir in genug anderen Bereichen der Informatik bewandert, um eine eigene Lösung zu entwickeln und dabei nicht bei Null zu beginnen.

## Darstellung von (Wahrscheinlichkeits-)Verteilungen
[Histogramme](https://de.wikipedia.org/wiki/Histogramm) sind ein bewährtes Werkzeug zur Darstellung von Verteilungen - die meisten kennen es vermutlich aus der Bildbearbeitung, wo sie die Verteilung der Helligkeitswerte von Pixeln anzeigen.

Wir verwenden für Dystonse eine artverwandte Darstellung, nämlich die [Summenhäufigkeit](https://de.wikipedia.org/wiki/Empirische_Verteilungsfunktion). Dabei speichern wir für die einzelnen Verspätungswerte nicht, wie oft _genau_ dieser Wert auftritt, sondern welcher Anteil der Verspätungen _höchstens_ diesen Wert hat. Das hat vielfältige Vorteile:
 * Für die oben genannte zentrale Fragestellung - kommt Fahrzeug A an, bevor Fahrzeug B abfährt? - können wir Werte direkt als Summenhäufigkeit ablesen, anstatt immer wieder Einzelhäufigkeiten aufzuaddieren.
 * Die Darstellung als Summenhäufigkeit ist unabhängig von der Auflösung. Da Verspätung eine kontinuierliche Größe ist, tritt praktisch nie zweimal die selbe Verspätung auf. Eine histogrammartige Darstellung entsteht nur, wenn die Verspätungen in Klassen eingeteilt werden, wie z.B. "zwischen 2 und 3 Minuten". Die Höhe der _Peaks_ ist dann von der Breite dieser Klassen abhängig und ihr Wert ist ohne Angabe der Breite nicht aussagekräftig.
 * Histogramme sind anfälliger für Artefakte durch Messungenauigkeiten und Rundungen - somit aber auch besser geeignet, um diese aufzudecken.
 * Die Kurve der Summenhäufigkeit lässt sich effizienter komprimieren, bzw. zwangsläufig auftretende Fehler durch die Datenreduktion wirken sich kaum auf das effektive Ergebnis aus.
 * Die Darstellung der Summenhäufigkeiten ist übersichtlicher, da die Kurven sich weniger oft kreuzen.

Hier ist eine Gegenüberstellung der beiden Darstellungsweisen für die selben Daten:

![Verspätung der Straßenbahn Linie 4 in Bremen, Darstellung als Histogramm](/assets/kurven/curve_18_to_38_na.svg)

Im Histogramm ist auf die Schelle kaum etwas zu erkennen, da die Kurven sich oft kreuzen. Als Summenhäufigkeit dargestellt, sind die gleichen Daten deutlich leichter zu überblicken:

![Verspätung der Straßenbahn Linie 4 in Bremen, Darstellung als Summenhäufigkeit](/assets/kurven/curve_18_to_38.svg)

Im weiteren Verlauf dieses Textes, sowie auch in später folgenden Blogposts, verwenden wir nur noch aufsummierte Häufigkeiten.

## Verarbeitung, Vereinfachung und Speicherung
Um diese Kurven effizient und vielseitig einsetzen zu können, haben wir das Rust-Paket [dystonse-curves](https://github.com/dystonse/dystonse-curves) angelegt. Damit können wir Summenhäufigkeiten in verschiedene Datenstrukturen verpacken, die sich über ein einheitliches Interface ansprechen lassen, das die wichtigsten Operationen darauf bereitstellt. Das Herzstück für die Speicherung ist die `IrregularDynamicCurve`, die eine Kurve anhand von geordneten XY-Paaren darstellt. Da alle Kurven monoton steigend sind, können Anfragen sowohl nach X- als auch nach Y-Werten effizient per binärer Suche erfolgen. Und da die Punkte ungleichmäßig verteilt sein können, und veränderbar sind, können wir diese Kurven reduzieren, indem wir Punkte entfernen, die sich auf die Gesamtform nur unwesentlich auswirken.

Um diese Reduktion zu erreichen, verwenden wir den [Ramer-Douglas-Peucker-Algorithmus](https://en.wikipedia.org/wiki/Ramer%E2%80%93Douglas%E2%80%93Peucker_algorithm), der vor allem in der Computergrafik und Geoinformatik populär ist. In der Praxis können wir damit unsere Kurven, die anfangs aus 25 bis 1200 Punkten bestehen, meist auf 5 bis 12 Punkte reduzieren. Außerdem genügt uns theoretisch jeweils ein Byte zur Speicherung einer Koordinate, denn unsere Daten sind niemals so genau, dass sie unbrauchbar würden, wenn sie sich um 1/256 verändern.

**HIER EVTL. GRAFIK EINFÜGEN**

Auf unserem Server herrscht noch keine Speicherknappheit, doch werden wir später einmal derartige Verteilungskurven an Mobilgeräte senden müssen, die sich außerdem oft in Bereichen mit schlechtem Mobilfunkempfang befinden. Eine Reduktion der Daten ist also besonders wichtig.

## Anwendung auf unsere Problemstellung
Im ersten Schritt unserer Analyse betrachten wir für einen Linienverlauf jedes Paar aus Start- und Zielhaltestelle, und untersuchen den Zusammenhang zwischen der Verspätung am Start und am Ziel. Dazu suchen wir in unseren Aufzeichnungen Datensätze für die beiden Haltestellen und darin Paare von Datensätzen, die sich auf die selbe Fahrt beziehen.

Daraus können wir bereits zwei Verteilungen der Verspätungen erstellen, nämlich für Start und Ziel. In unseren Aufzeichnungen sind beide Werte bekannt, Wenn wir später Prognosen erstellen, ist die Verspätung um Start bekannt, und die (Verteilung der) Verspätung am Ziel wird gesucht. Um den Zusammenhang der beiden Größen zu bestimmen, teilen wir unsere Datenpunkte in Klassen verschiedener Start-Verspätungen ein.

Die Aufteilung der Klassen ist aber nicht trivial:
 * Wenn wir gleich "breite" Klassen wählen (z.B. je 10 Sekunden, also von -300s bis -290s, von -290s bis -280s, usw.) so sind die Klassen sehr unterschiedlich stark gefüllt (oft eine einstellige Anzahl von Datenpunkten oder sogar gar keine Punkte in den Extremen, dafür hunderte Punkte in der Klasse um 0s herum).
 * Wenn wir gleich "mächtige" Klassen wählen (z.B. 100 Datenpunkte je Klasse), so erhalten wir sehr breite Klassen in den Extremen (z.B. werden alle Verspätungen von 20s bis 300s zusammen geworfen) während wir bei den fast-pünktlichen Fahrten überspezifisch trennen (so als würden sich Fahren mit 2s Anfangsverspätung signifikant anders entwickeln als solche mit 3s).

Wir haben daher ein rekursives Verfahren entwickelt, das zunächst alle Daten als eine Klasse ansieht und prüft, ob und wie diese in zwei Unterklassen eingeteilt werden kann. Wenn dies geschieht, wird für jede der Unterklassen die gleiche Prüfung und ggf. Teilung wiederholt. Dazu ermitteln wird einseits anhand der Breite, andererseits anhand der Mächtigkeit Ober- und Untergrenzen für eine mögliche Teilung, und nur wenn ein Teilungspunkt existiert, der alle vier Grenzen einhält, wird dieser verwendet.

**HIER EVTL. GRAFIK EINFÜGEN**

Eine weitere Verfeinerung unseres Ansatzes besteht darin, dass wir nicht mehr alle Datenpunkte einer Klasse stumpf zusammen werfen (also etwa alle Fahrten mit einer Anfangsverspätung zwischen 20s und 300s gleich behandeln), sondern gewichtet aufsummieren. Im gegebenen Beispiel würde eine Fahrt mit 40s Anfsangsverspätung sowohl mit sehr hoher Gewichtung in die Kurve für 20s eingehen, und nur mit einer sehr geringen Gewichtung für jede mit 300s. (Konkret geht jede Fahrt in 1 oder 2 Ergebniskurven ein, wobei die beiden Gewichte sich stets zu 1 aufaddieren, d.h. jede Fahrt geht gleichermaßen in die Gesamtanalyse ein).

**HIER EVTL. GRAFIK EINFÜGEN**

Daraus ergibt sich auch, dass für jedes Haltestellenpaar eine andere Anzahl an Klassen, und somit eine andere Anzahl an Kurven entstehen kann, stets so, wie es den vorhandenen Daten angemessen ist.

## Erkenntnisse
Die allererste Feststellung: die Kurven aus unseren gesammelten Daten sehen tatsächlich etwa so aus wie das, was wir uns in den Monaten zuvor vorgestellt haben, und was sich in unseren Bleistiftskizzen vielfach wieder findet.

Und es bestätigen sich weitere Vermutungen und offensichtliche Zusammenhänge:
 * Fahrten, die zu Beginn schon mehr Verspätung haben, haben tendentiell auch am Ende noch mehr Verspätung. 
 * Negative Verspätungen - also eigentlich Verfrühungen - kommen zwar vor, aber seltener und in kleinerem Ausmaß als positive Verspätungen. Überhaupt betreffen sie viel öfter die Ankunft an einem Halt, als die Abfahrt, d.h. das Fahrzeug bleibt i.d.R. länger stehen, wenn es "zu früh" angekommen ist.
 * Es gibt eine klare Häufung um 0s herum, d.h. dass ein Fahrzeug ungefähr pünktlich ist, ist der häufigste Fall
 * Straßenbahnen sind pünktlicher als Busse (dabei haben wir aber bisher nur 2 Bus- und 2 Tramlinien genauer betrachtet)
 * Manche Linienabschnitte fügen tendenziell mehr Verspätung hinzu, andere erlauben eher, Verspätungen wieder auszugleichen

Weniger erwartet hatten wir die folgenden Erkenntnisse:
 * Ein Großteil der Daten wurde vom Anbieter auf ganze Vielfache von 6 oder gar 12 Sekunden gerundet, aber es gibt stets auch Daten, die "krumme" Werte enthalten. Durch 12 Teilbare Werte kamen dabei überproportional oft vor, also viel öfter, als sie bei einer Rundung auf den nächsten 6er sowieso auftreten würden. Vielfache von 24 sind hingegen nicht auffällig oft vorgekommen. Die Rundung hat zu sichtbaren Verzerrungen unserer Grafiken geführt, die oft die eigentliche Essenz der Information überdeckt haben. Da wir nicht davon ausgehen, dass Prognosen genauer als 12s aufgelöst sein müssen, bzw. dass eine höhere Auflösung praxisrelevant wäre, runden wir nun _alle_ Daten vor der Verarbeitung auf 12 Sekunden.
 * Je nach Haltestelle schwankt die Verfügbarkeit und Qualität von Echtzeitdaten enorm, selbst wenn diese eigentlich von gleich vielen Fahrzeugen passiert werden. Es scheint, als würden manche Haltestellen bzw. Streckenzüge über 2-4 Haltestellen hinweg in einer Art "Funkloch" liegen.
 * Es gibt einige Ausreißer mit Verspätungen in der Größenordnung von -3600 oder +3600 Sekuden, also einer ganzen Stunde. Für unsere ersten Auswertungen haben wir nicht nur diese ausgeschlossen, sondern alle jenseits von 300 Sekunden, also 5 Minuten, in beide Richtungen. Natürlich sind Verspätungen im Rahmen von 10 bis 20 Minuten alltäglich und praxisrelevant und werden später wieder in unsere Analysen einfließen. Bei der Betrachtung des Kernbereichs waren diese aber schon viel störender, als wir zunächst angenommen haben.
 * Manchmal finden sich in den Kurven interessante Muster und Unregelmäßigkeiten, die dennoch plausibel sein können. Im Folgenden Beispiel finden sich einige "Stufen" der Breite 90s in den Kurven. Da liegt die Vermutung nahe, dass sich kurz vor der Haltestelle eine Ampel befindet, an der ein Bus warten muss, bevor er den Halt erreicht. Wir haben das auf der Straßenkarte geprüft, und tatsächlich muss der Bus dort an einer Ampel warten, direkt bevor er an der Haltestelle hält. Ob die Ampeln dort wirklich einen 90-Sekunden-Zyklus haben, konnten wir aus der Ferne jedoch nicht heraus finden.

**HIER GRAFIK EINFÜGEN: Bremer Bus 21 von Haltestelle 0 nach Horn (Horner Kirche)**

Bus Linie 21 in Bremen, Verspätung an der Haltestelle *Horn*: Hier sind einige auffällige Stufen erkennbar, die jeweils ca. 90 Sekunden breit sind.

<iframe width="425" height="350" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" src="https://www.openstreetmap.org/export/embed.html?bbox=8.867897987365724%2C53.09648992973711%2C8.87175500392914%2C53.09798452567355&amp;layer=mapnik&amp;marker=53.09723723419548%2C8.86982649564743" style="border: 1px solid black"></iframe><br/><small><a href="https://www.openstreetmap.org/?mlat=53.09724&amp;mlon=8.86983#map=19/53.09724/8.86983">View Larger Map</a></small>

Im Kartenausschnitt ist die Ampel markiert, die vermutlich für die Stufen im obigen Diagramm verantwortlich ist.

## Nächste Schritte
Derzeit können wir diese Kurven nur als Grafiken ausgeben. (Wir nutzen hier SVG, aber PNG, PDF, etc. funktionieren ebenso). Natürlich brauchen wir für die Praxis eine kompakte, effizient maschinenlesbare Form und eine Weise, die Kurven für sämtliche Haltestellenpaare aller Linien abzulegen und wiederzufinden.

Später möchten wir noch weitere Einflussgrößen betrachten und danach getrennte Kurven ermitteln, wie z.B. Wochentag und Uhrzeit.

Und wir brauchen einen Algorithmus (sowie eine Schnittstelle, Infrastruktur, etc.) um daraus tatsächlich Prognosen zu erstellen. Dazu werden wir die aktuelle Verspätung jedes Fahrzeugs nutzen, um die passende(n) Kurve(n) aus unserer Analyse zu laden und mittels Sampling und Interpolation Verteilungen für den weiteren Fahrtverlauf zu prognostizieren.

Diese Prognosen werden wir für unseren eigenen Suchalgorithmus einsetzen, aber wir möchten sie auch für andere Projekte verfügbar machen. Das naheliegende Format *GTFS-Realtime* ist dafür, wie oben schon angedeutet, noch nicht ausreichend detailiert. Derzeit beteiligen wir uns an einer [Diskussion mit weiteren Entwickler_innen](https://github.com/google/transit/pull/111#issuecomment-640787656), die sich ebenfalls eine solche Erweiterung des Standards wünschen.
