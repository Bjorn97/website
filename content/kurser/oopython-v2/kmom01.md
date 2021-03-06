---
author: lew
revision:
    "2017-11-10": (PB1, mos) Utkast till v2.
    "2017-01-06": (A, mos) Lade till CGI på studentservern.
...
Kmom01: Kom igång med objekt och Flask
====================================

Vi ska börjar kursen med grunderna om objekt och klasser. Vid sidan av ska vi arbeta med ett mini-ramverk, "Flask", där vi skriver redovisningstexten och en kort presentation om oss själva. Vi kommer jobba mer med Flask-appen genom kursens gång, samt titta på mer tekniker och strukturer angående objektorienterad programmering.

<!--more-->

[FIGURE src=/image/oopython/kmom01/flask1.png?w=w2 caption="En enkel me-sida med Flask."]



<small><i>(Detta är instruktionen för kursmomentet och omfattar det som skall göras inom ramen för kursmomentet. Momentet omfattar cirka **20 studietimmar** inklusive läsning, arbete med övningar och uppgifter, felsökning, problemlösning, redovisning och eftertanke. Läs igenom hela kursmomentet innan du börjar jobba. Om möjligt -- planera och prioritera var du vill lägga tiden.)</i></small>



Labbmiljön  {#labbmiljo}
---------------------------------

*(ca: 2-4 studietimmar)*

Det finns en [längre beskrivning om kursens labbmiljö](./../installera-labbmiljo). Läs den om du är osäker på vad som skall göras, eller om detta är din första dbwebb-kurs.

Den korta varianten är att du behöver [installera labbmiljön](./../labbmiljo), uppdatera [dbwebb-cli](dbwebb-cli) samt klona och initiera kursrepot.

```text
# Gå till din katalog för dbwebb-kurser
dbwebb selfupdate
dbwebb clone oopython
cd oopython
dbwebb init
```



Läsanvisningar  {#lasanvisningar}
---------------------------------

*(ca: 8-10 studietimmar)*



###Kurslitteratur  {#kurslitteratur}

Läs följande:

1. [Python 3 Object-oriented Programming](kunskap/boken-python3-object-oriented-programming)  
    * Ch 1 - Object-oriented Design  
        - Introducing object-oriented --- Specifying attributes and behaviors.
    * Ch 2 - Objects in Python  
        - Creating Python classes



###Artiklar {#artiklar}

Läs följande:

1. Fräscha upp minnet med [dokumentationen som finns på Pythons webbplats](https://www.python.org/doc/). Kika runt lite och orientera dig.

1. Se översikten av dokumentationen för [Python 3.x Doc](https://docs.python.org/3/).

1. Titta på [Flasks hemsida](http://flask.pocoo.org/). Där hittar du även installationsanvisningar och exempel.

Följande artiklar är korta och bra att läsa:  
1. [Motivation for obejct-oriented](https://atomicobject.com/resources/oo-programming/introduction-motivation-for-oo).

1. [The object-oriented paradigm](https://atomicobject.com/resources/oo-programming/the-oo-paradigm).

1. [Naming conventions](https://atomicobject.com/resources/oo-programming/naming-conventions).

1. [Messaging](https://atomicobject.com/resources/oo-programming/messaging).

1. [Abstraction and identity](https://atomicobject.com/resources/oo-programming/abstraction-and-identity).



###Video  {#video}

Titta på följande:  

1. Videoserien [Lär dig objektorienterad Python](https://www.youtube.com/playlist?list=PLKtP9l5q3ce8cmKXE9Gw1Ra0GaYufGbN7) är tätt kopplat till kursmaterialet. Kika på de videor som börjar med 1.

1. Titta på följande videos om Flask av "TheNewBoston":  

    * [Basic app](https://www.youtube.com/watch?v=ZVGwqnjOKjk)  
    * [Routing](https://www.youtube.com/watch?v=27Fjrlx4s-o)

1. Titta på de första 7-minuterna av [Object-oriented programming](https://www.youtube.com/watch?v=lbXsrHGhBAU).  



###Lästips {#lastips}

Om du känner att du har tid och lust.

1. [Python 3 Object-oriented Programming](kunskap/boken-python3-object-oriented-programming)  
    * Ch 4 - Expecting the Unexpected. 

1. Bekanta dig med debugger-verktyget [PDB](https://docs.python.org/3.2/library/pdb.html).

1. Kika på [Jinja2's dokumentation](http://jinja.pocoo.org/).



Övningar & Uppgifter  {#ovningar_uppgifter}
-------------------------------------------

*(ca: 8-10 studietimmar)*



###Labbmiljö {#labbmiljo}

Installera labbmiljön för kursen.

1. [Installera labbmiljön](oopython/labbmiljo) som behövs för kursen.

1. [Installera kommandot `dbwebb`  samt kursrepot för kursen](dbwebb-cli/clone).

Om detta är din första dbwebb-kurs så läser du också igenom hela [manual-sidan för kommandot `dbwebb`](dbwebb-cli).



###Övningar {#ovningar}

Genomför följande övning för att träna dig.

1. Kom igång och gör din första webbapplikation i Python tillsammans med övningen "[Flask och Jinja2](kunskap/flask-med-jinja2)".

1. Gör din [Flask applikation som ett CGI-skript](coachen/flask-som-cgi-script) och publicera på studentservern med `dbwebb publish`. Detta behöver bara fungera på studentservern, alltså inte lokalt.

1. Läs igenom artikeln som handlar om objekt i Python "[Kom igång med objekt](kunskap/kom-igang-med-objekt)".  



###Uppgifter {#uppgifter}

Dessa uppgifter skall utföras och redovisas.

1. Gör uppgiften "[Python med objekt och klasser](uppgift/python-med-objekt-och-klasser2)" (lab 1)

2. Gör uppgiften "[En me-sida med Python och Flask](uppgift/en-me-sida-med-flask)".



###Extra {#extra}

<!-- 1. Gör uppgiften "[Återställ trasigt objekt](uppgift/aterstall-trasigt-objekt)" -->
Det finns inga extrauppgifter.


Resultat & Redovisning  {#resultat_redovisning}
-----------------------------------------------

*(ca: 1-2 studietimmar)*

Läs [instruktionen om hur du skall redovisa](./../redovisa).

Se till att följande frågor besvaras i redovisningstexten.

* Vilken utvecklingsmiljö använder du?
* Var det några problem med att installera Flask?
* Är du bekant med objekt och klasser sedan tidigare?
* Har du någon erfarenhet av Bootstrap sedan tidigare?
* Gick det bra att komma i gång med kursmomentet, var det lagom, för litet, för stort?
* Vilken del av kursmaterialet (böcker, artiklar, videor, etc) uppskattade du mest?
