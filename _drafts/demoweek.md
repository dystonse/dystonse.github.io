---
layout: post
title:  "Dystonse - Projektvorstellung zur Demo Week"
categories: demoweek
excerpt_separator: <!--more-->
---

# Dystonse - Wir steigern das Vertrauen in den ÖPNV

## Wer kennt es nicht?

_Du stehst an der Haltestelle und fragst dich, wann die Straßenbahn endlich kommt. Laut Fahrplan sollte sie schon längst da sein, aber sie ist noch nicht zu sehen. So langsam wirst du nervös - falls sie jetzt noch mehr als drei Minuten braucht, dann würdest du lieber den Bus nehmen, den du am Bussteig nebenan schon einfahren siehst. Wobei... der braucht halt eigentlich eh immer länger, du müsstest unterwegs umsteigen, und der Umstieg ist dann auch oft ziemlich knapp. Also doch lieber weiter auf die Straßenbahn warten?_

Eine nervige Situation, die im Moment noch ziemlich oft vorkommt. Und wenn du mal in einer anderen Stadt unterwegs bist, dann hast dort du noch nichtmal mehr dieses Gefühl dafür, welche Linien meistens pünktlich sind und welche nicht. Mit etwas Glück zeigt dir dort ein Abfahrtsmonitor die aktuellen Verspätungen, aber das sagt dir auch nichts darüber, wie diese sich während der Fahrt wohl entwickeln werden, und wie wahrscheinlich es ist, dass du deinen Umstieg schaffst.

Da wäre es doch gut, eine Software-Lösung zu haben, die dir diese Zweifel abnimmt!

<!--more-->

So ging es uns auch selbst oft - und deswegen haben wir Dystonse entwickelt. Im Gegensatz zu klassischen ÖPNV-Routensuchen und -Abfahrtsmonitoren ist die Abfahrtszeit bei uns nicht nur **eine** Zeit – sondern eine Verteilung von Wahrscheinlichkeiten. 

Diese Verteilung kannst du jetzt schon in unserem **erweiterten Abfahrtsmonitor** sehen. Er zeigt dir nicht nur die Abfahrten an deiner Start-Haltestelle an, sondern mit einem Klick auch den weiteren Verlauf der Linie und wie sich ihre Pünktlichkeit voraussichtlich entwickeln wird. Damit kannst du selbst deine Route zusammenstellen und dabei sehen, wie wahrscheinlich diese Umstiege sind.

<a href="/assets/demoweek/abfahrten.png"><img title="Screenshot 1" src="/assets/demoweek/abfahrten.png" width="45%"></a>
<a href="/assets/demoweek/halte.png"><img title="Screenshot 2" src="/assets/demoweek/halte.png" width="45%"></a>

Letztendlich soll die Routensuche natürlich auch automatisch passieren, so dass du dich im Liniennetz nicht auskennen musst, um eine Route zu finden. In [unserem ersten Prototypen von 2019](https://dystonse.org/route) kannst du schon sehen, wie das mal ungefähr aussehen soll. Jenen Prototyp haben wir damals innerhalb weniger Tage entwickelt, und entsprechend unvollständig und fehlerhaft ist er auch. Außerdem sind die Daten, auf denen das Verspätungsmodell dort basiert, eine sehr starke Vereinfachung, die in der Realität nicht sehr nützlich ist.

Das [ursprüngliche Ziel unserer Förderung](https://prototypefund.de/project/dystonse-wahrscheinlichkeitsbasierte-oepnv-routensuche/) ist, diese Routensuche komplett neu zu schreiben. Damit sind wir noch nicht fertig. Die wichtigste Neuheit von Dystonse - das Rechnen mit Wahrscheinlichkeitsverteilungen anstatt fester Zeiten, kommt aber auch im erweiterten Abfahrtsmonitor schon zum Einsatz. Und auch unser neues, umfassendes Verspätungsmodell wird bereits für den Monitor verwendet, sowie die gesamte Infrastruktur, die unsichtbar dahinter steht (siehe unten).

Hier kannst du den erweiterten Abfahrtsmonitor für den **Verkehrsverbund Bremen/Niedersachsen** gleich ausprobieren:

{% raw %}
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width="100%" height="100" src="http://[2a02:8108:8240:7329:ca2a:14ff:fe2a:e7e]:3000/"></iframe>
{% endraw %}

## Erweiterter Abfahrtsmonitor vs automatische Routensuche

Unser Ziel ist es, alle wichtigen Infos in einer Anwendung zusammen zu bringen: Die Umsteigemöglichkeiten im **Liniennetz**, die Abfahrtszeiten laut **Fahrplan**, die **aktuellen Echtzeit-Daten** von einzelnen Fahrzeugen, und eben diese Intuition, die man als Mensch nur durch viel Erfahrung entwickeln kann - wir nennen das **Verspätungsmodell**. Also das Wissen darüber, welche Linien dazu neigen, besonders pünktlich oder unpünktlich zu sein, wo sie ihre Verspätung wieder aufholen, etc...

Klassische Routensuchen und Abfahrtsmonitore berücksichtigen diese Infos teilweise gar nicht oder nur eingeschränkt. Unser erweiterter Abfahrtsmonitor verwendet alles davon - nur die Auswahl, welcher Umstieg nun der bessere ist, musst du hier im Gegensatz zur Routensuche noch selbst treffen. Mit der Visualierung der Wahrscheinlichkeiten geben wir dir dazu möglichst viel Info an die Hand, um diese Entscheidung besser fundiert zu treffen.

Hier nochmal eine kleine Übersicht, welche Software-Lösungen welche dieser Daten nutzen:

|                            | Linien&shy;netz | Fahr&shy;plan&shy;zeiten | aktu&shy;elle Echt&shy;zeit&shy;daten | Ver&shy;spät&shy;ungs&shy;modell |
|----------------------------|--------------|----------------|------------------------|-------------------|
| **Klassischer Monitor**    | ❌ Nein      | ✅Ja           | ✅Ja                   | 🔸Teilweise      |
| **Dystonse Monitor**       | 🔸 Teilweise | ✅Ja           | ✅Ja                   | ✅Ja             |
| **Klassische Routensuche** | ✅ Ja        | ✅Ja           | 🔸Teilweise            | 🔸Teilweise      |
| **Dystonse Routensuche**   | ✅Ja         | ✅Ja           | ✅Ja                   | ✅Ja             |

## Was alles dahinter steckt - der Dystonse-Software-Stack

Der Abfahrtsmonitor (und später auch die Routensuche) ist zwar der einzige Teil, der für alle Nutzer:innen sichtbar ist - aber der größte Teil der Software steckt "hinter den Kulissen": Hier werden die Daten gesammelt, die wir für das Verspätungsmodell brauchen, und das Modell wird berechnet.

### Unser zentrales Datenformat: Wahrscheinlichkeits-Kurven

Wie so ein Verspätungsmodell aussieht, haben wir schonmal ausführlich in unserem Blogpost ["Um die Kurve gedacht"](https://blog.dystonse.org/analysis/2020/06/10/kurven.html) erklärt. Kurz gesagt: die Verteilung der Abfahrts- und Ankunftszeiten wird in Form von Kurven zusammengefasst, in denen jeder Zeit eine Wahrscheinlichkeit zugeordnet ist, wie häufig diese Zeit tatsächlich zutrifft.

Bei den Linien, von denen wir viele Echtzeitdaten aufgezeichnet haben, können wir nicht nur eine Kurve für jede Haltestelle berechnen, sondern auch die Abhängigkeit zwischen verschiedenen Haltestellen und den Einfluss vorheriger Verzögerungen. Also z.B. "wenn dieser Bus an Haltestelle Nr. 3 mit X Sekunden Verspätung abfährt, wie verteilt sich dann die Ankunftszeit an Haltestelle Nr. 7". Im Idealfall ist diese Kurve sogar noch nach der Tageszeit aufgeschlüsselt.

Bei den Linien, für die wir weniger Echtzeitdaten gesammelt haben, sind die Kurven dann etwas weniger genau, damit die Daten-Grundlage nicht zu klein ist. Dann wird zum Beispiel die Tageszeit nicht berücksichtigt, oder es werden nicht nur Daten einer bestimmten Straßenbahnlinie genutzt, sondern von allen Straßenbahnen zusammengefasst.

### Datensammlung

Um diese Kurven berechnen zu können, brauchen wir natürlich erstmal eine Menge Daten. Davon gibt es zwei Sorten: Fahrpläne und Echtzeit-Updates. 

Fahrpläne sind prinzipiell leicht verfügbar, und zwar seit Anfang 2020 für ganz Deutschland als Open Data. Über die Schwierigkeit, dazu passende Echtzeitdaten zu bekommen, und diese dann auch zu verwenden, haben wir in den ersten Wochen unserer Förderzeit bereits [einen eigenen Blogpost](https://blog.dystonse.org/opendata/2020/03/13/datensammlung.html) geschrieben. 

Zum Glück haben wir mit dem VBN einen Anbieter gefunden, der selbst nicht nur Echtzeitupdates mit einer echten OpenData-Lizenz, sondern auch die Fahrpläne mit den passenden IDs [zur Verfügung stellt](https://www.vbn.de/service/entwicklerinfos/). 

Der erste Teil unseres Software-Stacks sind also zwei `collect`-Skripte, die regelmäßig die Fahrplan- und Echtzeitdaten aus den VBN-Quellen herunterladen. Danach werden die für unsere Anwendung relevanten Daten von der `import`-Komponente zusammengefügt und in einer Datenbank gespeichert - die ist dann deutlich schneller durchsuchbar als die Rohdaten.

### Analysen

Sobald wir genügend Echtzeit-Updates gesammelt und mit dem richtigen Fahrplan verknüpft haben, können wir mit den Analysen beginnen. Dieser `analyse`-Schritt erzeugt das Verspätungsmodell (also die Kurven, die oben schon beschrieben wurden) je nach vorhandener Datenmenge so spezifisch wie möglich, so allgemein wie nötig.

### Prognosen

Um diese Analysen dann auch nutzen zu können, ordnen wir jeder zukünftigen Fahrt (Im aktuellen Testbetrieb reicht unser Monitor bis ca. eine Woche in die Zukunft) die passenden Kurven zu. So lange im Voraus können wir natürlich nur sehr allgemeine Vorhersagen treffen. Sobald Echtzeitdaten zu einem Fahrzeug eintreffen, aktualisieren wir damit laufend all seine Prognosen.

Diese werden dann z.B. vom Abfahrtsmonitor (und später von der Routensuche) direkt für einen bestimmten Zeitraum aus der Datenbank abgerufen.

Im Prinzip ist die Berechnung dieser Prognosen ein eigener `predict`-Schritt in der Datenverarbeitung, wie er im nachfolgenden Diagramm auch zu sehen ist. In Wirklichkeit finden diese Berechnungen bei uns schon im `import`-Schritt statt, so dass jede Echtzeit-Datei nur einmal verarbeitet wird.

<a href="/assets/demoweek/stack.svg"><img title="Überisicht über unseren Software-Stack" src="/assets/demoweek/stack.svg"></a>

Bei `collect` handelt es sich nur um ein paar Shell-Skripte. Die Komponenten `import`, `analyse`, `predict` und auch `monitor` sind in der Sprache Rust geschrieben und sind Module innerhalb unseres Universalwerkzeugs `dystonse-gtfs-data` ([zum Code](https://github.com/dystonse/dystonse-gtfs-data)). Sie haben also alle die selbe Codebasis, aber laufen als eigenständige Prozesse. Außerdem muss eine MySQL-Datenbank laufen, auf welche die anderen Komponenten zugreifen, und ein gemeinsam genutztes Verzeichnis für die Dateiablage vorhanden sein. Zusammen mit PhpMyAdmin, das wir zur Pflege der Datenbank verwenden, erreicht der Software-Stack also schon einiges an Komplexität.

### Automatisierung und einfache Installation

Um die Installation des ganzen Software-Stacks möglichst einfach zu gestalten, haben wir die Abläufe mit Hilfe von [Docker](https://docs.docker.com/get-started/overview/) und [Docker Compose](https://docs.docker.com/compose/) automatisiert.

Dabei definieren [Konfigurationsdateien](https://github.com/dystonse/dystonse-docker), welche Komponenten gebaut und gestartet werden müssen und wie diese miteinander interagieren, und andere Dateien beschreiben jeweils die Konfiguration einer einzelnen Komponente. Mit wenigen Änderungen in einer Konfigurationsdatei könnten wir so z.B. weitere Datenquellen hinzufügen, und die Installation auf einem neuen Server braucht nur noch wenig Handarbeit. Außerdem können wir dadurch die einzenen Komponenten einfach getrennt voneinander updaten, debuggen und überwachen.

## Erfahrungen und Ausblick

Wir haben nun schon fast ein halbes Jahr Förderzeit hinter uns, in der wir sowohl ungeplante Schwierigkeiten als auch überraschende Erfolge erlebt und vor allem viel gelernt und eine große Community gefunden haben.

So haben z.B. die Vorbereitung der Datensammlung, das Einrichten der Datenbank, und die ersten Analysen viel länger gebraucht, als wir vermutet hatten.

Dazu kam dann noch, dass während der ersten Phase des "Lockdowns" wegen der Covid19-Pandemie plötzlich viele Verkehrsmittel ausgefallen sind und auch etwas später die Menschen aus Sicherheitsgründen kaum noch öffentliche Verkehrsmittel benutzt haben. Für eine neue ÖPNV-Software also ein ziemlich ungünstiger Zeitpunkt - daher haben wir nicht wie vorher gelant schon früh User-Befragungen o.ä. durchgeführt, sondern uns zuerst auf den Backend-Stack konzentriert. Und uns dabei manchmal zu lange mit Details aufgehalten.

Als dann dank der Pandemie auch noch das Konzept für die abschließende Präsentation von einer Veranstaltung vor Ort in ein Online-Event umgewandelt werden musste, wofür alles ein paar Wochen früher fertig sein muss, war die Zeit plötzlich noch knapper.
Mit Hilfe eines geförderten Coachings haben wir dann ein neues Konzept für die Präsentation erarbeitet - deshalb zeigen wir heute unsere Arbeit und die wichtigen Konzepte von Dystonse anhand des erweiterten Abfahrtsmonitors, und können die automatische Routensuche unabhängig von dieser Demo-Anwendung weiter entwickeln.

Aber die Pandemie geht ja hoffentlich bald vorbei, und eine möglichst angenehme Nutzung des ÖPNV wird noch lange gebraucht, um Mobilität in Zeiten des Klimawandels zu ermöglichen. Also bleiben wir dran und entwickeln die geplante Routensuche ggf. auch nach dem Ende der Förderung noch weiter.

Dabei unterstützt uns eine Community verschiedener Menschen und Organisationen, die sich ebenfalls dafür einsetzen, dass öffentliche Infrastruktur auch mit offenen Daten betrieben wird. Beim VBN ist unser Projekt schon sehr positiv empfangen worden, und wir hoffen, dass sich auch andere Verkehrsverbünde daran ein Vorbild nehmen, so dass wir irgendwann Abfahrtsmonitore und Routensuche für ganz Deutschland anbieten können.

Wenn ihr wissen wollt, wie es mit Dystonse weiter geht, schaut gerne auf [unserem Blog](https://blog.dystonse.org) vorbei - dort wird es demnächst noch viele spannende Artikel geben.
