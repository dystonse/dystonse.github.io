---
layout: post
title:  "Hürden und Herausforderungen"
authors: "Lena Schimmel, Kirstin Rohwer"
date:   2020-08-28 17:45:00 +0200
categories: prototypefund
excerpt_separator: <!--more-->
image: /assets/demoweek/header.jpg
---

Während der regelmäßigen Gespräche mit den anderen Projekten und den Betreuer\*innen vom [Prototype Fund](https://prototypefund.de/) wurden wir immer wieder gefragt, was denn in letzter Zeit die größten Schwierigkeiten waren, und wie wir damit umgegangen sind. Auch in unseren [Montagsupdates](https://github.com/dystonse/dystonse/blob/master/project-status/Montagsupdates.md) sollten wir jede Woche angeben, was uns gerade bremst.

Heute haben wir die letzten sechs Monate mal Revue passieren lassen, um zu überlegen, was die größten Herausforderungen waren. Denn ganz ohne Schwierigkeiten kommt so ein Open-Source-Softwareprojekt in keinem Fall zustande. Und es ist uns wichtig, zu zeigen, dass es ok ist, wenn nicht immer alles glatt läuft - für Projekte in zukünftigen Prototype Fund-Runden und anderswo: lasst euch nicht entmutigen, das Ergebnis kann am Ende trotzdem gut werden!

<!--more-->

In den Montagsupdates finden sich hauptsächlich äußere Einflüsse, die uns gebremst haben - schlechte Stimmung wegen der Pandemie, heißes Wetter im Dachgeschoss-Homeoffice, privater Stress...

Das Programmieren hat, wenn wir trotz der Umstände genug Zeit und Konzentration dafür hatten, eigentlich immer ganz gut geklappt. Sowohl mit unserer Zusammenarbeit im gemeinsamen Homeoffice (ein Luxus, den viele andere Projekte diesmal nicht hatten), als auch mit der Wahl der [Programmiersprache Rust](https://www.rust-lang.org/), waren wir sehr zufrieden und würden das wieder genauso machen. Natürlich gibt es auch damit hier und da kleinere Probleme, die sich aber mit einem Blick in die Doku schnell lösen lassen. Größere _Blocker_, wie wir sie aus anderen Projekten kennen, gab es tatsächlich keine. 

Größere Hürden sind vor allem am Anfang aufgetreten, als wir mehr Zeit als vermutet in Recherche stecken mussten, um erstmal einen Verkehrsverbund zu finden, der zusammenpassende Fahrplan- und Echtzeitdaten als echte *Open Data* bereitstellt - zum Glück haben wir mit dem VBN eine gute Datenquelle gefunden, aber da ist bei den anderen Verkehrsverbünden in Deutschland auf jeden Fall noch viel Luft nach oben...

Seit wir größere Datenmengen gesammelt haben, bereiten diese auch Schwierigkeiten auf mehreren Ebenen. Sowohl in der Datenbank als auch im eigenen Code konnten wir Performance-Optimierung nicht aufschieben, bis unser Produkt großflächig genutzt wird. Wenn z.B. eine SQL-Abfrage erst nach mehreren Stunden ein Ergebnis liefert, bremst das die Entwicklung und das Debuggen komplett aus. Und gerade beim Debuggen ist es nach wie vor kompliziert, sich _manuell_ durch Millionen Datensätze zu graben. 

Der knappe Zeitrahmen, der immer zu so einer Förderzeit gehört, ist natürlich auch eine gewisse Hürde - da neigt man schon mal dazu, den Code etwas "mit der heißen Nadel zu stricken", was dann letztendlich doch mehr Zeit kostet, wenn später eine Umstrukturierung nötig wird.

Die zeitliche Begrenzung wirkt sich nicht nur auf die Qualität des Codes aus - auch andere wichtige Aufgaben wie User Testing sind bei uns etwas auf der Strecke geblieben. Das hatte allerdings nicht nur mit dem Zeitrahmen zu tun, sondern auch mit dem Pandemie-bedingten Rückgang der ÖPNV-Nutzung im Allgemeinen. Wir selbst haben während der Förderzeit auch nur wenige Male einen Zug oder Bus betreten.

Alles in allem lief das ganze Projekt aber bisher ziemlich reibungslos. Es geht hoffentlich auch genau so weiter! Im Moment versuchen wir, einen neuen Arbeitsmodus zu finden, der weniger von Termindruck geprägt ist, und die größeren Aufgaben in den Fokus nimmt, die wir bisher aufschieben mussten.
