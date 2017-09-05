---
layout: [post, post-xml]
title:  "NodeJS als Auskunftswebservice einer JEE-Applikation"
date:   2017-09-04 14:30
modified_date: 2017-09-04
author: AdessoMaDa
categories: [Java, Architektur]
tags: []
---

## Worum geht's hier?

Im Zuge der Digitalisierung von Gesch�ftsprozessen laufen gro�e unternehmenskritische Applikationen zunehmend nicht mehr ausschlie�lich im internen Betrieb, sondern
m�ssen auch unternehmensexterne Benutzer wie Gesch�ftspartner und Kunden ansprechen. Diesen werden ausgew�hlte Informationen zu laufenden Gesch�ftsvorg�ngen zug�nglich gemacht.
Dabei muss die Verschlossenheit der nur intern genutzten Daten zu den Gesch�ftsvorg�ngen erhalten bleiben. Nur jene Informationen, die explizit freigegeben wurden, 
d�rfen f�r externe Nutzer einsehbar sein.

Dies wird sichergestellt durch einen Prozess, der eine explizite Freigabe der zu ver�ffentlichenden Informationen vorsieht und durch separate Speicherung der ver�ffentlichten Daten 
sowie durch separate Zugriffspfade.

Die folgende Abbildung zeigt eine grobe Prozessbechreibung:
![Prozessbechreibung zu NodeJS als Auskunftswebservice einer JEE-Applikation](https://github.com/adessoAG/devblog/raw/master/assets/images/NodeJS-als-Auskunftswebservice-einer-JEE-Applikation/Aktivitaetsdiagramm.png)

Im vorliegenden Projektbeispiel haben wir uns entschieden, die Gesch�ftsapplikation als JEE-Applikation zu entwickeln und diese um einen Auskunftsservice auf Basis des MEAN-Stacks zu erweitern.

## Warum entwickeln wir JEE-Applikationen?

In vielen gr��eren Unternehmen werden f�r die Entwicklung von unternehmenskritischen Applikationen Referenzarchitekturen vorgegeben.
Diese Vorgabe dient vor allem dem Zweck, einheitliche Strukturen f�r Deployment und Rollout dieser Anwendungen einzuhalten und somit 
auch einheitliche Verfahren f�r den Betrieb dieser Applikationen durchzusetzen. Damit sollen die Total Cost of Ownership f�r den Betrieb
der Applikationen so gering wie m�glich gehalten werden. Einheitliche Architekturen unterst�tzen au�erdem die Wiederverwendung von
Querschnittsfunktionen wie Benutzerauthentifizierung, Autorisierung oder Logging durch verschiedene Applikationen.

Im vorliegenden Projektbeispiel ist durch den Kunden eine Referenzarchitektur auf Basis von Java Enterprise Edition (JEE) vorgegeben.
Auf dieser Basis haben wir eine unternehmenskritische Applikation entwickelt, die von internen Mitarbeitern des Auftraggebers im Intranet des Unternehmens genutzt wird.

## Was sind die Vorteile dieser Architektur?

Unternehmenskritische Applikationen auf Basis einer solchen Plattform aufzusetzen bringt eine Reihe von Vorteilen.
In der Datenhaltungsschicht wird eine relationales Datenbanksystem eingesetzt, das viele Anforderungen in Bezug auf eine revisionssichere Datenhaltung durch 
eingebaute Funktionen abdeckt. Konzepte und Werkzeuge zur Datensicherung und zur Wiederherstellung liegen vor und m�ssen nicht speziell entwickelt werden.
In der Gesch�ftskogikschicht steht eine Reihen von Querschnittsfunktionen und Bibliotheken zur Verf�gung, mit denen komplexe Gesch�ftsabl�ufe effizient umgesetzt werden k�nnen.
Die Verarbeitung auch von gro�en Datenmengen ist m�glich und es stehen Werkzeuge zur Verf�gung, um diese m�glichst effizient zu gestalten.

## Welche Nachteile hat diese Architektur?

Der gro�e Funktionsumfang der zugrunde liegenden Softwaresysteme  bringt aber auch Nachteile mit sich. Relationale Datenbanksysteme mit transaktionsorientierter Arbeitsweise
begrenzen die M�glichkeit von echt paralleler Verarbeitung, da beim gleichzeitigen Zugriff auf sich �berschneidende Datenbest�nde immer eine logische Serialisierung der Zugriffe stattfindet.
Dies erzwingt Wartezeiten beim gelichzeitigen Zugriff auf sich �berschneidende Datenbest�nde. Die Mechanismen, welche die dauerhafte Speicherung von ausgef�hrten Daten�nderungen
sicherstellen, f�hren zu zus�tzliche Verarbeitungsschritten und verl�ngern somit die Ausf�hrungszeit.

Die Vielzahl der eigebauten Funktionen bringt au�erdem einen hohen Bedarf an Systemressourcen wie zum Beispiel Arbeitsspeicher mit sich. Deshalb m�ssen Applikationen, die einer solchen
Architektur basieren, gezielt dimensioniert werden. Um eine solche Dimensionierung durchf�hren zu k�nnen, sollte das Ger�st an Datenmengen, die Anzahl der Nutzer, welche die Appliaktion
insgesamt nutzen sowie die Anzahl der gleichzeitigen Zugriffe auf die einzelnen Funktionen bekannt sein. Im vorliegenden Projektbeispiel wird die Mittelschicht
auf 4 unterschiedlichen Serverknoten ausgef�hrt und langlaufende Gesch�ftsprozesse werden asynchron ausgef�hrt.

## Was ist, wenn diese Randbedingungen nicht bekannt sind?

Soll unsere unternehmenskritischen Anwendung nun zus�rzlich einen �ffentlichen Auskunftsservice beinhalten, so sind viele dieser Randbedingungen nicht mehr bekannt.
Wir wissen nicht, wieviele Nutzer zu welcher Zeit in welcher H�ufigkeit Anfragen an den Auskunftsservice stellen werden. Wir ben�tigen daher eine Architektur, 
welche mit vorgegeben Systemressourcen eine hohe Skalierbarkeit erm�glicht. Wenn es sich um einenn reinen Auskunftsservice handelt, ben�tigen wir andererseits auch viele 
der in der JEE-Plattform eingebauten Funktionen nicht, da wir hier keine  Daten ver�ndern und somit auch keine besonderen Anforderungen an die Erhaltung von Datenkonsitenz 
und die dauerhafte Speicherung unserer �nderungen stellen m�ssen.

Ein leichtgewichtiger und hoch skalierbarer Auskunftsdienst l�sst sich gut auf Basis des MEAN Architekturstacks aufbauen:
* MongoDB
* Express
* AngularJS
* Node.js

Node.js ist hierbei die Serverkomponenten, welche Anfragen entgegen nimmt und beantwortet. Die Anfragen werden in eine Abarbeitungswarteschlange eingereiht und sequentiell verarbeitet.
Die Verarbeitung der einzelnen Anfragen erfolgt dabei in einer Reihe von sehr kleinen und kurz laufenden Arbeitsschritten. Dies erm�glicht es, dass ein Node.js in einem einzigen Thread 
laufen kann und dabei abwechselnd Anfragen verarbeiten und neuen Anfragen entgegennehmen kann. Werden mittels des Zusatzmoduls PM2 mehrere solche Threads gestartet, so erreicht man eine
sehr hohe Skalierbarkeit. Dabei ist Node.js nicht mehr als eine Ausf�hrunmgsumgebung f�r JavaScript-Programme und ben�tigt daher sehr wenig Systemressourcen.
F�r die Datenhaltung f�r einen mit Node.js implementierten Service bietet sich MongoDB an, da die Datenverarbeitung in den JavaScript-Programmen, die in einem Node.js-Server laufen
in der Regel aus JSON-Objekten basiert und die dokumentenorientierte Datenablage in MongoDB dieser Struktur sehr nahe kommt. 
Au�erdem untst�tzt MongoDB den direkten Austausch von bin�r kodierten JSON-Objekten (BSON). 
Der Rechenaufwand f�r Marshalling und Unmarshalling beim Datenaustausch zwischen MongoDB und Node.js ist somit minimal.

## Wie passt das alles zusammen?

Wir haben nun also einerseits eine unternehmenskritische Applikation mit vielen Querschnittsfunktionen und einer Implementierung f�r komplexe Gesch�ftsprozesse auf einer 
vergleichweise schwerf�lligen Ausf�hrungsumgebung auf Basis von JEE. Auf der anderen Seite steht ein hochskalierbarer Auskunftsservice f�r relativ einfach strukturierte Anfragen
zur Verf�gung.

Die eigentliche Datenhaltung unserer Applikation erfolgt in einem transaktionssicheren relationalen Datenbanksystemen mit allen notwendigen Funktionen zur revisionsicheren dauerhaften Speicherung unserer Gesch�ftsdaten.
Die Daten, die f�r die Beauskunftung �ber unseren Auskunftsservice freigegeben sind, �bertragen wir nun aus dieser Datenhaltung in die Datenhaltung des Auskunftsservices.
Dies erfolgt �ber ein PUSH-Verfahren von unserer Appliaktion aus. Die Anwendung des PUSH-Verfahrens hat folgende Vorteile:
* Die Gesch�ftslogik bestimmt, wann sie eine Daten�bertragung ausf�hrt. Damit k�nnen wire steuern, dass diese zus�tzliche Daten�bertragung minimalen Einfluss auf die Leistungsf�higkeit der Applikation hat.
* Die Gesch�ftslogik bestimmt, welche Daten �bertragen werden. Der Schutz vertraulicher Daten kann somit nicht von au�erhalb der Appliaktion kompromittiert werden.
* Die Gesch�ftslogik bestimmt, wohin die Daten �bertragen werden.

