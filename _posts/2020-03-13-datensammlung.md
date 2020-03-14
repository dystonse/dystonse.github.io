---
layout: post
title:  "Echtzeit-Daten - Probleme und Lösungen"
date:   2020-03-13 02:02:34 +0100
categories: opendata
excerpt_separator: <!--more-->
---
Unser Projekt soll am Ende eine besonders nützliche ÖPNV-Routensuche sein. Bis dahin ist noch viel zu tun, beginnend bei der Datensammlung.

Zum einen brauchen wir ein paar statische Daten, wie z.B. Linienverläufe und die Namen der Haltepunkte. Seit Beginn des Jahres 2020 lassen sich diese zum Glück gebündelt für ganz Deutschland beziehen. Dank der [EU-Verordnung 2017/1926](https://eur-lex.europa.eu/legal-content/DE/TXT/PDF/?uri=CELEX:32017R1926&from=EN) müssen die Verkehrsunternehmen diese bereit stellen. In Deutschland hat man sich für das NeTEx-Format entschieden, doch auch es gibt bereits einen [Dienst](http://gtfs.de/), der diese ins international verwendete [GTFS-Format](https://developers.google.com/transit/gtfs/reference) umwandelt.<!--more-->

Zum anderen brauchen wir die dynamischen Echtzeit-Daten. Daraus leiten wir über längere Zeiträume Statistiken und Modelle ab, die wir später mit den neusten Echtzeitdaten kombinieren, um unsere eigenen Prognosen für einzelne Fahrzeuge herzustellen. Und auf Basis jener Prognosen erstellen wir letztlich die Routenvorschläge und die Bewertung von deren Zuverlässigkeit. Man sieht: Echtzeitdaten sind für unser Vorhaben gleich doppelt wichtig.

# Verfügbarkeit von Echtzeitdaten
Echtzeitdaten zu bekommen ist derzeit aber noch viel schwieriger, als es bei statischen Daten der Fall ist:

 * Für viele Regionen existieren diese Daten einfach nicht, so dass nicht einmal die Verkehrsbetriebe wissen, wo ihre Fahrzeuge gerade sind
 * Die meisten Verkehrsbetriebe geben ihre Echtzeitdaten nicht weiter
 * Für Echtzeitdaten hat sich noch kein eindeutiges Standardformat etabliert - neben den großen Standards GTFS realtime und SIRI bestehen unzählige Insellösungen
 * Nicht alle Echtzeitdaten, die praktisch verfügbar sind, stehen auch unter offenen Lizenzen
 * Selbst offen lizensierte Echtzeitdaten unterliegen meist Beschränkungen hinsichtlich der Anfragen, die pro Tag gestellt werden dürfen bzw. können. Und diese sind oft so niedrig, dass eine sinnvolle statistische Auswertung nicht möglich ist
 * Letztlich fehlt es an einer Übersicht über verfügbare, offene Echtzeit-Daten

Somit ist in den ersten Wochen unserer Projektförderung ein beachtlicher Teil unserer Zeit in Recherche zu Datenquellen geflossen. Die Ergebnisse haben wir fortlaufend dokumentiert, gegliedert in drei Stufen der Strukturiertheit:

 * Sehr unstrukturiert: [Liste von Notizen mit Quellenangaben](https://github.com/dystonse/dystonse/blob/master/project-status/Datenquellen.md)
 * Mäßig strukturiert: [Tabellarischer Vergleich der Datenquellen](https://github.com/dystonse/dystonse/blob/master/project-status/Datenquellen.md#vergleich-von-echtzeit-datenquellen)
 * Strukturiert: [Metadaten als JSON](https://github.com/dystonse/dystonse/blob/master/project-status/datasources.json)

Nach unserem aktuellen Wissen gibt es noch keine vergleichbare Übersicht über ÖPV-Echtzeitdaten (es gibt eine [Liste der Listen](https://git.digitaltransport4africa.org/learn/tools/awesome-transit#3rd-party-gtfs-url-directories) aber die Infos über Echtzeitdaten sind da sehr begrenzt). Vermutlich kann unsere kleine, Deutschland-zentrierte Liste also auch für andere Open Data Projekte nützlich sein.

# Uneinheitliche Daten auf allen drei Ebenen
## Formate
Ein Blick in das genannte [JSON zum jetzigen Zeitpunkt](https://github.com/dystonse/dystonse/blob/5320392ce5f2ef769a6df9e3a7f36b5046e33afa/project-status/datasources.json) zeigt: Die sieben Anbieter, die Echtzeitdaten liefern, tun dies in insgesamt sechs verschiedenen Formaten. (Was nicht ganz so schlecht ist, wie es klingt, denn einer der Anbieter stellt zwei Formate zur Wahl).

Die Vielfalt der Formate ist aber gar nicht das einzige Problem, vermutlich auch nicht das größte. Ob die Information, dass ein bestimmter Bus gerade 120 Sekunden Verspätung hat, nun als JSON oder Protocol Buffer vorliegt, macht für die Verarbeitung nur einen minimalen Unterschied aus. Die Kerninformation, und somit das Datenmodell, sind ja in beiden Fällen das gleiche.

## Referenzierungen
Viel größere Schwierigkeiten bereiten die Referenzierungen zwischen den unterschiedlichen Datenquellen. Ein Beispiel:

> Erhalten wir z.B. Echtzeitdaten für die Fahrt `125413240` auf der Route `34518_2`, deren Fahrzeug gerade pünktlich auf dem Weg zum Halt `0` ist, dann müssen wir aus den statischen Daten abfragen, welche konkrete Fahrt, Route und Haltestelle damit gemeint sind. 

Wenn diese Daten aus einer anderen Quelle kommen, ist es alles andere als selbstverständlich, dass dort die gleichen IDs genutzt werden. Die Zuordnung kann trivial sein, oder kompliziert. Vielleicht auch unzuverlässig oder schlichtweg unmöglich. Und was davon der Fall ist, wissen wir erst, nachdem wir es ausprobiert haben.

## Datenmodelle bzw. -Typen
Letztlich kann es sogar komplett verschiedene Datenmodelle geben, die grundlegend andere Arten von Aussagen über die Echtzeit-Situation machen:

 1. Wie viel Verspätung hat ein Fahrzeug in diesem Moment?
 2. Mit wie viel Verspätung wird das Fahrzeug voraussichtlich die nächste Haltestelle erreichen?
 3. In wie viel Minuten wird das Fahrzeug voraussichtlich die nächste Haltestelle erreichen?
 4. Wo befindet sich das Fahrzeug gerade?

Die ersten drei Fälle sind offensichtlich ähnlich - und falls das Fahrzeug sich gerade an einer Haltestelle befindet, quasi identisch. _Ob_ es sich gerade an einer Haltestelle befindet geht daraus aber nicht unbedingt hervor. Das ließe sich am besten aus Daten des vierten Typs ablesen. Daten von Typ 2 werden zudem oft je Fahrzeug abgefragt (oder für alle Fahrzeuge auf einmal), während Daten von Typ 3 für jede Haltestelle einzeln abgefragt werden müssen.

Natürlich lassen sich alle vier Arten von Daten ineinander umrechnen, wenn auch mit verschieden viel Aufwand und Ungenauigkeit.

Apropos Ungenauigkeit: oft werden die Daten ursprünglich in einer Form erfasst, aber vom Verkehrsbetreiber bereits umgerechnet und nur in jener Form veröffentlicht. Inklusive der systembedingten Ungenauigkeiten. Gerade im Fall von Prognosen (Typ 2 und 3 in obiger Liste) handelt es sich um Blackboxen, also intransparente Berechnungen bei denen weder das mathematische Verfahren bekannt ist, noch die weiteren Daten, die darin einfließen. Im Fall von Positionsdaten aus HAFAS-Schnittstellen haben wir sogar den Verdacht, dass hier "echte" Positionsdaten in Verspätungen oder Prognosen konvertiert wurden, aus denen dann wieder simulierte Positionsdaten entstehen.

Es wäre naheliegend für uns (wenn auch sehr aufwändig), für alle vier Datentypen Umrechnungen in alle drei anderen zu entwickeln (also 12 Konvertierungen) und deren Genauigkeit zu evaluieren. Dazu bräuchten wir idealerweise Testdaten, die bereits alle vier Datentypen enthalten. Leider ist uns kein Datensatz bekannt, der mehr als zwei davon abdeckt. Viel Aufwand für ein Vorhaben, das sich dann nichtmal evaluieren lässt? Lieber nicht.

# Weitere Anforderungen an offene Daten
In den letzten Absätzen ging es vorallem um sehr technische Problemstellungen. Auch rechtlich / organisatorisch ist die Lage komplex. Eigentlich wurde alles wichtige dazu schon vor 5 Jahren in der [EVU-Open-Data-FAQ](https://github.com/UlmApi/beyondtransit/blob/gh-pages/evu-daten-faq.md) gesagt - absolute Leseempfehlung!

# Was tun?
Irgendwo müssen wir ja anfangen, nur wo? Für die Auswahl eines ersten Testgebiets sind folgende Faktoren relevant:
 
 * Datenformat(e)
 * Datenmodell(e)
 * Lizenz
 * Anfragebegrenzungen
 * Zuverlässigkeit der Server
 * Genauigkeit der Daten
 * Anzahl Datensätze (Fahrzeuge, Linien)
 * Referenzierbarkeit zwischen Echtzeit- und Fahrplandaten
 * Unsere Ortskenntnis
 * Unsere Möglichkeit, für Testfahrten ins Testgebiet zu reisen

Kurzfristig brauchen wir erstmal eine einzelne Datenquelle, mit der wir überhaupt arbeiten können.

Mittelfristig brauchen wir aber eine Architektur, die mit allen Kombinationen aus Format und Datenmodell arbeiten kann.

Langfristig werden sich die Probleme lösen, da auch Echtzeitdaten flächendeckend und einheitlich formatiert frei verfügbar werden sollen _(vermutlich ab dem Jahr 2024, aber wo war doch gleich die Quelle für diese Zeitangabe?)_. 

# Einfach mal anfangen!
Von den uns bekannten Datenquellen haben die Daten des _Verkehrsverbund Bremen Niedersachsen_ unsere oben genannten Anforderungen am besten erfüllt.

Noch haben wir keine Pipeline fertig, die die Echtzeitdaten parst, in eine Datenbank schreibt, mit den Fahrplandaten zusammen führt, statistische Auswertungen generiert… sondern einfach nur ein Shellscript, das die Daten herunterlädt und zippt, damit unsere Mircro-SD-Karte im Raspberry Pi nicht so schnell voll ist. 

```
set -e
DIR=/var/opt/gtfsrt/vbn
DATE=`date +"%Y-%m-%dT%H:%M:%S%:z"`
FILENAMEPB=$DIR/vbn-gtfsrt-$DATE.pb
FILENAMEZIP=$DIR/vbn-gtfsrt-$DATE.zip
PINGURL=https://hc-ping.com/<zensiert>

curl http://gtfsr.vbn.de/gtfsr_connect.bin -o $FILENAMEPB \
         --silent --show-error --retry-connrefused \
         --retry 20 --retry-max-time 100
zip $FILENAMEZIP $FILENAMEPB
rm $FILENAMEPB
curl -s $PINGURL
```

Seit dem Abend des 12.03.2020 sammeln wir damit jene Echtzeitdaten im 2-Minuten-Takt, und wenn das nachfolgende Badge nichts gegenteiliges sagt, dann sammeln wir noch heute…

![badge](https://healthchecks.io/badge/5441c6f8-5c30-4c41-826d-02327f/s_7xl3wR/record-vbn-realtime.svg)

