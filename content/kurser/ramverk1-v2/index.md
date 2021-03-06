---
title: ramverk1-v2
author: mos
revision:
    "2018-06-08": "(prel, mos) Nytt dokument inför uppdatering av kursen."
    "2017-10-02": "(C, mos) Info om samtliga kmom."
    "2017-08-03": "(B, mos) JavaScript flyttas till ramverk2."
    "2017-06-07": "(A, mos) Första revisionen inför kursstart HT2017, bygger på phpmvc-kursen."
...
Kursen ramverk1 (v2)
==================================

Kursen **Webbaserade ramverk 1**, a.k.a. *ramverk1*, lär ut programmering och objektorienterade kodstrukturer med designmönster och modultänkande kring återanvändbara moduler i webbaserade ramverk samt tekniker för automatiserad testning, byggsystem och flöde för kontinuerlig integration av programvaran.

<!--more-->

[WARNING]

**Kursutveckling pågår till kurs ramverk v2**

Kursstart hösten 2018.

[/WARNING]


<!--
Genom att använda ramverk för webbutveckling tillsammans med tekniker HTML, CSS, PHP och databas tränas studenten i att hantera en större kodmassa som behöver organiseras i god kodstruktur med stöd av ramverk, designmönster och återanvändning av befintliga moduler för att effektivisera utvecklingsprocessen.

Kursens fokus är främst kodstruktur och ramverk på backend men fullstack-perspektivet är viktigt. Kursen introducerar utvecklingsmetoder för testdriven utveckling och continuous integration.

Via litteraturstudier, praktiska övningar och ett större avslutningsprojekt ges möjlighet att skapa sig en egen bild av vad koncepten kan innebära för ett webbutvecklingsprojekt.
-->



Förkunskaper {#forkunskaper}
------------------------

Det formella förkunskapskravet är:

> För tillträde till kursen krävs 22,5 högskolepoäng programmering, webbteknologier, databaser samt genomgången kurs Objektorienterade webbteknologier 7,5hp.



Innehåll {#innehall}
------------------------

Kursen omfattar följande områden:

* Webbaserad programmering i ramverk med designmönster implementerade främst i programmeringsspråket PHP.
* Objektorienterade teknologier.
* Utveckling av webbapplikation i ramverk.
* Återanvändbara moduler, paketering och distribution.
* Designmönster för ramverk.
* Testdriven utveckling och tekniker för enhetstestning.
* Automatiserad testning och byggning.
* Relaterade verktyg och tekniker.



Mål {#mal}
------------------------

Följande är kursens mål, indelat i undergrupper.



### Kunskap och förståelse {#kunskap}

Efter genomförd kurs skall studenten:

* ingående redogöra för webbutveckling baserat på de tekniker som omfattas genom att skriftligen beskriva och sammanfatta erfarenheter och observationer från övningar och projekt.
* kunna visa goda kunskaper i att använda valda tekniker genom att tillämpa dessa tekniker i praktiska övningar och projekt.



### Färdighet och förmåga {#fardighet}

Efter genomförd kurs skall studenten:

* självständigt utifrån en specifikation kunna utveckla, dokumentera och presentera ett projekt baserat på ramverksbaserad kodning och återanvändning av moduler.
* ha god praktisk förmåga att hantera de verktyg och utvecklingsmiljöer som används vid utveckling och felsökning vid ramverksprogrammering.



### Värderingsförmåga och förhållningssätt {#vardera}

Efter genomförd kurs skall studenten:

* via redovisningstexter kunna visa god förmåga att reflektera och argumentera över kodstruktur i ramverk med sammanhang i designmönster och återanvändbara moduler.



Kursmoment {#kursmoment}
------------------------

Kursen är uppdelad i kursmoment där varje kursmoment uppskattas till 20h studerande i form av programmering, undersökning, läsande, övningar, uppgifter, redovisning och eftertanke. Alla kursmoment skall redovisas och du samlar alla redovisningar i din me-sida.


### Kmom01: Ramverk {#kmom01}

Vi tar en mjukstart för att komma in i ramverkstänkande och läser på om bra-att-ha kunskaper inom PHP och ramverk. Det handlar om nödvändiga verktyg och att nyttja den infrastruktur som finns kring PHP och att anamma ett PHP modul-tänkande. 

[Instruktion till kursmoment 01](./kmom01).



### Kmom02: MVC {#kmom02}

Vi tittar på designmönstret Model, View, Controller (MVC) och använder det för att strukturera vår kod i ramverket.

Vi bekantar oss även med begreppet SOLID som är en akronym för designmönster som är aktuella i sammhanget kring ramverk och objektorienterad utveckling.

[Instruktion till kursmoment 02](./kmom02).



### Kmom03: DI {#kmom03}

Vi skall titta på tekniker som kan sammafattas med Dependency Injection (DI). Dessa tekniker används för att skapa en grundläggande struktur i ramverket avseende hur man lägger till "tjänster" och moduler in i ramverket. Hittills har vi använt `$app` som en kontainer för alla tjänster som finns i ramverket. Nu skall vi introducera `$di`.

Vi skall titta på begreppet Dependency Injection och några begrepp som är närliggande, begrepp såsom service locator, service container och lazy loading. Det handlar om designmönster och vanliga sätt att strukturera sin kod enligt det som betraktas som god programmeringssed.

[Instruktion till kursmoment 03](./kmom03).



### Kmom04: Databasdrivna modeller {#kmom04}

Vi skall titta på klasser i modell-lagret och utöka vår struktur med formulärhantering och databasdrivna modell-klasser.

Vi skall använda en extern modul för HTML-formulär och vi skall använda en extern modul för databashanteringen.

I arbetet skapar vi basklasser i modellagret som underlättar då vi implementerar applikationsspecifik kod. Vi kan se det som vi bygger upp modellklasser som kan scaffoldas fram, en form av återanvändning och ett försök att effektivisera vårt kodande.

[Instruktion till kursmoment 04](./kmom04).



### Kmom05: Modul {#kmom05}

Du skall skapa en fristående modul av ditt kommentarssystem och placera det i ett eget repo på GitHub. Du skall alltså lyfta bort koden från din me-sida och placera allt som modulen behöver i ett eget repo.

Du skall sedan publicera repot som en PHP modul på Packagist. När det är klart kan du åter installera modulen i din me-sida med hjälp av kommandot composer.

Du börjar införa enhetstestning på din modul.

[Instruktion till kursmoment 05](./kmom05).



### Kmom06: CI {#kmom06}

Tanken är att ge en bild av hur automatiserad testning och continuous integration (CI) fungerar mot en PHP modul som ligger publicerad på GitHub och Packagist.

Vi fortsätter jobba mot modulen vi publicerade på GitHub och Packagist i föregående kursmoment. Vi använder de tester som körs via `make install test` för att låta externa verktyg checka ut vår kod och exekvera testerna och analysera koden ur olika aspekter.

Vi bekantar oss med ett antal olika externa verktyg och försöker förstå vad de kan tillföra till en utvecklares vardag.

[Instruktion till kursmoment 06](./kmom06).



### Kmom07/10: Projekt och examination {#kmom10}

Avslutningsvis gör du ett projekt enligt en specifikation. Projektet är det sista som du gör och tillsammans med alla redovisningar som finns på din me-sida så används detta som underlag för att examinera dig från kursen.

[Instruktion till kursmoment 10](./kmom10).



Kurslitteratur {#litteratur}
----------------------------



### Kurslitteratur {#kurslitteratur}

Kurslitteraturen består av egenskrivna och länkade online-resurser såsom artiklar och guider.

Det finns läsanvisningar i samband med varje kursmoment.



### Referenslitteratur {#referenslitteratur}

Referensdokumentationen är främst olika online-resurser i form av manualer.



### Övrig litteratur {#ovriglitteratur}

Det finns närliggande litteratur som är intressant för den som vill fördjupa sig i ämnet eller den som vill skaffa sig förkunskaper innan kursen.

* **[PHP Objects, Patterns, Practice](kunskap/boken-php-objects-patterns-and-practice)** -- Matt Zandstra  
    En bok för den som kan sin PHP-programmering och vill mer med designpatterns, ramverk och mer avancerade tekniker som är nödvändiga för den professionella webbprogrammeraren.

* **[Pro PHP: Patterns, Frameworks, Testing and More](kunskap/boken-pro-php-patterns-frameworks-testing-and-more)** -- Kevin McArthur  
    En bok för den som kan sin PHP-programmering och vill lära sig mer avancerade tekniker vilka är vardagsmat för den professionella webbprogrammeraren.

* **[Code Complete](kunskap/boken-code-complete)** -- Steve McConnell  
    En bok för den som vill förädla sitt kunnande i konsten att programmera, skriva bra kod och göra rätt saker i programutvecklingsprojekt.



<!--

Kursbok saknas så inga speciella läsanvisningar syns här.

Läsanvisningar {#lasanvisning}
------------------------------

Här följer en sammanställning av de läsanvisningar till kurslitteraturen som ges i varje kursmoment.

| Kursmoment | Eloquent JavaScript: A Modern Introduction to Programming | 
|------------|-----------------------------------------------------------|
| Kmom01     | Ch 1, 12                                                  |
| Kmom02     | Ch 2                                                      |
| Kmom03     | Ch 3,                                                     | 
| Kmom04     | Ch 4, 6                                                   |
| Kmom05     | Ch 12, 13, 14                                             | 
| Kmom06     |                                                           | 
| Kmom10     |                                                           |

Dessutom har varje kursmoment läsanvisningar i artiklar och videos. 

-->



Rekommenderad studieplan {#studieplan}
---------------------------------------------

Kursen har en [rekommenderad studieplan](kurser/ramverk1/studieplan) som visar en översikt över kursens olika moment och när de i tiden bör utföras för att studenten skall ligga i fas med kursens planering.

I studieplanen visas när rättning sker av respektive inlämnat moment samt när det finns möjligheter att göra restinlämningar.

Läs mer om den [rekommenderade studieplanen](kurser/faq/rekommenderad-studieplan).



Lektionsplan {#lektionsplan}
---------------------------------------------

Det finns en [lektionsplan](kurser/ramverk1/lektionsplan) som visar en detaljplanering för undervisningen i kursen, vecka för vecka.

Läs mer om [lektionsplanen](kurser/faq/lektionsplan-och-schema).



Handledning {#handledning}
----------------------------------------

Förutom den planerade undervisningen enligt lektionsplanen så kan du få hjälp och stöd i kursens chatt och i forumet. Chatten lämpar sig för korta enkla frågor och forumet för mer utredande och längre frågor och svar. Om du inte får svar i chatten så rekommenderas att du postar i forumet.

Läs om [lärarstöd och handledning](kurser/faq/lararstod-och-handledning).



Betygsättning {#betyg}
------------------------

Det finns ett särskilt dokument som beskriver [hur bedömning och betygsättning sker](kurser/bedomning-och-betygsattning). 



Ladok {#ladok}
------------------------

Enligt kursplanen finns ett antal ladokmoment och de är kopplade till kursens kursmoment enligt följande.

| Kursens moment  | Ladok moment enligt kursplan  | Betyg |
|-----------------|-------------------------------|-------|
| Kmom01 + kmom02 | Inlämning 1 á 2.5hp           | G-U   |
| Kmom03 + kmom04 | Inlämning 2 á 2.5hp           | G-U   |
| Kmom05 - kmom10 | Inlämning 3 á 2.5hp           | A-F   |

Den sista inlämningen bestämmer kursens slutbetyg vilket utfärdas när samtliga moment godkänts.

Läs mer om [rapportering av resultat](kurser/faq/resultatrapportering).



Kursutvärdering och kursutveckling {#kursutvardering}
-----------------------------------------------------

Det finns ett särskilt dokument som beskriver hur arbetet med kursutvärderingar och kursutveckling sker. Det är oerhört viktigt att du säger till vad du tycker om kurs och kursmaterial, du kan alltid hojta till i både forum, chatt eller mail.

Läs om hur [vi jobbar med kursutvärdering och kursutveckling](kurser/kursutvardering-och-kursutveckling).



Kursplan {#kursplan}
-----------------------------------------------------

Kursplanen är kursens formella dokument som fastställts av högskolan. När kursen utvärderas görs det mot kursplanen. I kursplanen kan du läsa om kursens klassificering, syfte, innehåll, mål, generella förmågor, lärande och undervisning, bedömning och examination, litteratur, mm.

Från och med hösten 2019 är kursens namn "Webbaserade ramverk och designmönster" och kursen får en ny kursplan.

<!-- DV1610, Webbaserade ramverk och designmönster, 7,5 hp (ersätter PA1441) -->


Kursens namn är "Webbaserade ramverk 1". Du hittar [kursplanen genom att söka på kurskoden PA1441 via BTH's hemsida](http://edu.bth.se/utbildning/utb_kursplaner.asp?KKurskod=PA1441).

Kursen är en fristående vidarutveckling av kursen DV1486 "Databasdrivna webbapplikationer med PHP och MVC-ramverk", aka phpmvc.

Se [kursplan för DV486](http://edu.bth.se/utbildning/utb_kursplaner.asp?KKurskod=DV1486).



Versioner av kursen {#versioner}
-----------------------------------------------------

Om du påbörjat den äldre version av kursen så skall du också slutföra denna versionen av kursen (eller göra om den nya kursen från start). Alternativt rådgör du med den som är kursansvarig.

För tillfällen från höstterminen 2018 så [finns kursmaterialet till ramverk1 (v2) här](kurser/ramverk1-v2). 

För tillfället under höstterminen 2017, så [finns kursmaterialet till den kursen ramverk1 (v1)](kurser/ramverk1-v1).

Tidigare utgåvor av närbesläktade kurser, innan hösten 2017, finns under [kursen phpmvc](kurser/phpmvc).
