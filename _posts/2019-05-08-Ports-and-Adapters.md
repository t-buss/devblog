---
layout:         [post, post-xml]              
title:          "Ports and Adapters revisited"
date:           2019-05-08 09:00
author:         t-buss
categories:     [Architektur]
tags:           [architektur, cloud, microservices]
---
Die Ports and Adapters-Architektur, auch bekannt als Hexagonale Architektur, ist seit 2005 durch Alistair Cockburn vielen Software-Entwicklern bekannt.
In diesem Blogartikel wollen wir das Architekturmuster von verschiedenen Blickwinkeln betrachten.
Wir betrachten die Terminologie, das Verhältnis zu Domain-Driven-Design und Microservices und die Möglichkeiten für Cloud-Anwendungen.

Kleiner Disclaimer am Anfang: Vieles in diesem Artikel spiegelt meine persönliche Meinung wieder.
Wenn jemand einer anderen Meinung ist: Ich bin offen für einen Dialog!

![Open Architecture Hex-Sys](/assets/images/posts/ports-and-adapters/hex-sys.jpg)
Bildquelle: [Open Architecture](http://www.openarch.com/task/69)

Fangen wir mit den Basics an und erklären das Architekturmuster in einigen kurzen Sätzen (Wer eine detailliertere Beschreibung haben möchte, dem kann ich diesen ausführlichen [Blogartikel](https://softwarecampament.wordpress.com/portsadapters/) empfehlen).

Die Kernidee ist, Abhängigkeiten des Systems nach außen auf Distanz zu halten, um die Kernlogik isoliert betrachten und testen zu können.
Dazu wird die Anwendung in drei Teile geteilt:
1. Driving Adapters (linke Seite)
2. Core Domain (Mitte)
3. Driven Adapter (rechte Seite)

![Ports and Adapters](/assets/images/posts/ports-and-adapters/portsandadapters.png)
Abbildung 1: Ports and Adapters

Beginnen wir mit der Core Domain.
Hier wird die Anwendung aus Geschäftssicht programmiert.
So finden sich hier für einen Online-Shop beispielsweise Konzepte wie "Produkt" oder "Warenkorb".
Auch Benutzer und Use Cases werden hier eingeordnet.
Darüber hinaus enthält die Core Domain sogenannte "Ports".
Ports sind Interfaces, die die verschiedenen Interaktionen zwischen der Core Domain und äußeren Komponenten, den sogenannten Adapters, abstrahieren.

Ports gibt es in zwei Arten: bereitgestellt für oder bereitgestellt von der Core Domain.
Für die Core Domain könnte beispielsweise ein Port existieren, über den man Produkte laden kann.
Der Use Case, der eine Produktsuche abbildet, ist von diesem Interface abhängig, nicht von der konkreten Implementierung des Interfaces (ein Adapter auf der Driven-Seite).
Auf ähnliche Weise können Adapter auf der Driving-Seite die Ports nutzen, die von der Core Domain bereitgestellt werden.
Ein Web-Controller mit einem GET-Endpunkt `/products/?nameContains=bier` ist von dem Port "Produktsuche" abhängig, nicht von der konkreten Implementierung der Core Domain.
Die Abhängigkeiten der Adapter fließen in beiden Fällen in Richtung der Core Domain.

# Vor- und Nachteile
Alistair Cockburn nennt in einem seiner Blogartikel den größten Vorteil dieser Architektur:

> The ultimate benefit of a ports and adapters implementation is the ability to run the application in a fully isolated mode.

Indem die Komponenten innerhalb und außerhalb der Core Domain nur von Interfaces abhängig sind, kann eine Komponente mit Dummy-Komponenten konfiguriert werden, um die Funktion zu testen.
Statt einer "echten" Datenbank kann man so also für einen Unit-Test eine einfachere, nicht-persistente Datenstruktur verwenden.
Statt die Oberfläche mit der realen Anwendung im Hintergrund zu testen, können für einen Test Antworten vorab definiert werden, um wirklich nur die Oberflächen-Logik und nichts anderes testen zu können.

Der Nachteil der Architektur ist die erhöhte Komplexität des Codes, die bei kleineren Anwendungen nicht immer im Verhältnis zum Aufwand steht.

# Terminologie
Schauen wir uns die Begriffe in diesem Muster genauer an.

Das Architekturmuster wird oft auch "Hexagonale Architektur" genannt.
Das bezieht sich auf den Arbeitstitel des Musters und stammt von der sechseckigen Form, die der Core Domain auf Skizzen und Diagrammen oft gegeben wird.
Der Name ist jedoch irreführend, da die Architektur nichts mit der Zahl Sechs zu tun hat.
Auch Alistair Cockburn spricht sich daher dafür aus, konsequent nur noch "Ports and Adapter" zu verwenden.

Ein Begriff, der mich persönlich immer etwas gestört hat, ist "Port".
Während "Ports and Adapters" fabelhafte Begriffe sind, um zu beschreiben, wie einfach die Komponenten "zusammengesteckt" werden können, fehlt bei dem Begriff der fachliche Aspekt.
Wer nicht gerade Software für eine Hafenbehörde schreibt, der wird den Begriff "Port" wohl kaum in seiner Fachsprache finden.
Wir sollten die Core Domain von technischen Details und Jargon frei halten, und stattdessen einen Begriff wählen, der auch fachlich Sinn ergibt.
Ich schlage den Begriff "Capability" vor.
Meiner Meinung nach drückt er besser aus, warum die Komponenten von dieser abhängig sind.

Ein Beispiel: In einer Java-Anwendung wird ein Produktkatalog umgesetzt.
Die Klassendeklarationen:
```java
class ProductCatalog implements ProductSearchPort { ... }
vs.
class ProductCatalog implements ProductSearchCapability { ... }
```
Mit "Capability" anstatt "Port" ergibt diese Zeile auch für Domänenexperten ohne Programmierkenntnisse Sinn.

Der Begriff "Adapters" ist ebenso wenig fachlich korrekt, allerdings befinden sich die Adapter nicht in der Core Domain.
Auf der Driving- und Driven-Seite tummeln sich allerhand technische Details und Begriffe, sodass man sich zwar einen besseren Begriff überlegen könnte, dadurch allerdings keinen wirklichen Mehrwert erhält.

Stellt sich die Frage, wie das Architekturmuster genannt werden sollte, wenn man "Port" zu "Capability" ändert.
Der Begriff "Capabilities and Adapters"-Architektur ist zu klobig und zu unhandlich für den alltäglichen Gebrauch.
Daher wird sich der geläufige Begriff weiterhin halten (nicht, dass ich einzelner Entwickler überhaupt etwas daran ändern könnte).

# Domain-Driven-Design und Microservice
Domain-Driven-Design (kurz DDD) ist eine beliebte Methode, Software zu modellieren.
In der jüngeren Vergangenheit hat sie wieder an Bedeutung gewonnen, da es gerne für die Modellierung von Microservices verwendet wird.
Wie verhält sich DDD zu Ports and Adapters?
Und wie zu Microservices?

Ein zentrales Konzept in DDD ist der Bounded Context, der abgeschlossene Bereich einer Domäne.
Innerhalb dieses Contexts haben Wörter der Fachsprache eine ganz bestimme Bedeutung (die innerhalb eines anderen Contexts eine andere sein kann).
Eine Anwendung hat üblicherweise mehrere dieser Bounded Contexts.
Wenn ein Context mit einem andern Context interagiert, dann besteht eine Verbindung.
Zusammengesetzt entsteht eine Context-Map, ein Graph, indem die Knoten Bounded Contexts sind und die Kanten die Verbindungen.

Jeder Bounded Context kann eigenständig mit Ports and Adapters implementiert werden.
Damit erhält jeder Context eine Core Domain und kann selbst definieren, wie Abläufe funktionieren und 
Begriffe zu verstehen sind.

![Mehrere Bounded Contexts, jeder in sich mit Ports and Adapters](/assets/images/posts/ports-and-adapters/multiple_contexts.png)
Abbildung 2: Mehrere Bounded Contexts, jeder in sich mit Ports and Adapters

# Ports and Adapter für Cloud-Landschaften

# Verhältnis zu anderen Mustern
