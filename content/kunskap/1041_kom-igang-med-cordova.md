---
author:
  - aar
  - efo
category:
  - cordova
  - kurs webapp
revision:
    "2017-03-20": (B, efo) Gjort om för webapp-v3.
    "2017-03-08": (A, aar) Första utgåvan inför webapp-v2.
...
Kom igång med Cordova
==================================

[FIGURE src=/image/kunskap/cordova_logo.png?w=500&h=300]

Vi ska använda Apache Cordova och mithril för att skapa en "Hello World"  och testa köra den i en Android Emulator. Cordova är ett [cross-plattform tool](https://en.wikipedia.org/wiki/Cross-platform) som är gratis och open source. Apparna byggs i HTML, CSS och JavaScript och mithril.



<!--more-->



Förutsättning {#pre}
--------------------------------------

Artikeln är en del av kursen webapp och förutsätter att du har gjort motsvarande "[installera Cordova](kunskap/installera-cordova)".


Introduktion {#introduktion}
--------------------------------------

Med hjälp av Cordova ska vi skapa en _hybrid app_, alltså en webbsida som exekveras i mobilen som en native app. Detta gör att vi kan utnyttja native funktionaliteter i våran app, t.ex. kamera, GPS och kontakter. Vi kommer åt de här funktionaliteterna med hjälp av ett [JavaScript API som Cordova har utvecklat](https://cordova.apache.org/docs/en/latest/#plugin-apis).

I den här artikeln kommer vi inte testa någon native funktion utan vi fokuserar på att få ihop en "Hello World" app och testa den i en Android emulator.

[INFO]
Exemplen för Cordova i kursen innehåller bara `www`-mappen. För att testa koden behöver du ha/skapa ett Cordova projekt du kan kopiera in www-mappen i. Du behöver även lägga till plattformarna Android och Browser för att testa koden.
[/INFO]


Hello World {#hello_world}
--------------------------------------

När vi skapar ett nytt Cordova projekt får vi med kod och filstruktur för en fungerande app så vi testar skapa ett nytt projekt i terminalen.

```bash
# Stå i me/kmom05/hello
$ cordova create . se.dbwebb.helloWorld HelloWorld
```

`cordova create` kommandot tar tar emot följande argument:

* **directory**: Vad mappen som vårt prjojekt ska ligga i ska heta som. I detta fallet vill vi skapa katalogstrukturen i den befintliga katalogen `me/kmom05/hello`.

* **identifier**: Identifierare för appen, används bland annat för Android appar. Den är skriven i [reverse domain name notation](https://en.wikipedia.org/wiki/Reverse_domain_name_notation). I detta fallet "se.dbwebb.helloWorld".

* **title**: Projektets title. I vårt fall "HelloWorld".

Vi tar en titt på vad för mappar och filer som har skapats.

```bash
$ tree hello -L 2
hello
├── config.xml
├── hooks
│   └── README.md
├── package.json
├── platforms
├── plugins
├── res
│   ├── icon
│   ├── README.md
│   └── screen
└── www
    ├── css
    ├── img
    ├── index.html
    └── js
```

* **config.xml**: Configurerar appen. I den kan du ändra beteendet av appen och ändra titel, beskrivning och skapare av appen. [Dokumentation](https://cordova.apache.org/docs/en/latest/config_ref/index.html) för de som vill gräva ner sig.

* **package.json**: Är som tidigare paket filen för npm.

* **hook**: Här kan vi lägga in skript som vi vill ska ingå i Cordovas skripts. T.ex. om vi vill lägga till ett skript till `build` kommandot.

* **platforms**: Här finns källkoden för alla platformar, som vi lägger till i projektet. Vi kommer ha källkod för Android och Browser. Oftast ska/behöver man inte ändra på någon av den koden.

* **plugins**: Plugin vi lägger till i projektet kommer kopieras hit.

* **www**: Här vi kommer jobba och ändra saker. Innehåller vår källkod för webb delen, HTML, CSS och JavaScript.



###Inspektera koden {#inspektera}

Öppna `index.html` så kollar vi vad som finns där. Du kan börja med att läsa igenom kommentarerna och granska koden för att få lite förståelse.

Jag har tagit bort kommentarerna.

```html
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *; img-src 'self' data: content:;">
        <meta name="format-detection" content="telephone=no">
        <meta name="msapplication-tap-highlight" content="no">
        <meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1, width=device-width">
        <link rel="stylesheet" type="text/css" href="css/index.css">
        <title>Hello World</title>
    </head>
    <body>
        <div class="app">
            <h1>Apache Cordova</h1>
            <div id="deviceready" class="blink">
                <p class="event listening">Connecting to Device</p>
                <p class="event received">Device is Ready</p>
            </div>
        </div>
        <script type="text/javascript" src="cordova.js"></script>
        <script type="text/javascript" src="js/index.js"></script>
    </body>
</html>
```

Vi börjar uppifrån och jobbar neråt.

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self' data: gap: https://ssl.gstatic.com 'unsafe-eval'; style-src 'self' 'unsafe-inline'; media-src *; img-src 'self' data: content:;">
```
Första meta taggen är [_content security policy_](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP). Den skyddar vår app mot cross-site scripting (XSS) attacker. Om du vill använda dig av t.ex. en CDN för CSS behöver du editera taggen och tillåta appen att hämta kod från domänen du vill hämta från.

```html
<meta name="format-detection" content="telephone=no">
```
`format-detection` taggen konverterar telefonnummer till länkar. Användare kan klicka på länken så startas ett samtal till det numret.

```html
<meta name="msapplication-tap-highlight" content="no">
```
Den här kan vi ta bort då vi inte ska använda vår app på en Windows Phone.

```html
<meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1, width=device-width">
```
Sen har vi `viewport` meta tag som ni har introducerats till tidigare. Den ser till så att hela skärmytan på våra mobila enheter används för att visa innehåll.

Längst ner i `body` inkluderar vi två JavaScript filer. `index.js` filen kommer vara vår start punkt. `cordova.js` ger oss tillgång till Cordovas API för att komma åt native funktionaliteter.

Vi går vidare och inspekterar `index.js`. Jag har tagit bort kommentarerna.

```js
var app = {
    initialize: function() {
        document.addEventListener('deviceready', this.onDeviceReady.bind(this), false);
    },
    onDeviceReady: function() {
        this.receivedEvent('deviceready');
    },
    receivedEvent: function(id) {
        var parentElement = document.getElementById(id);
        var listeningElement = parentElement.querySelector('.listening');
        var receivedElement = parentElement.querySelector('.received');

        listeningElement.setAttribute('style', 'display:none;');
        receivedElement.setAttribute('style', 'display:block;');

        console.log('Received Event: ' + id);
    }
};
app.initialize();
```

Vanlig JavaScript kod, ett objekt med tre funktioner. Det intressanta här är event lyssnaren som lyssnar efter `deviceready`. `deviceready` är ett event från Cordova som meddelar när appen är färdig laddad och det går att använda Cordovas plugins. Vi kommer utgå från funktionen `onDeviceReady` som startpunkt för vår kod.
Koden som finns där nu döljer `<p>` taggen med innehållet `Connecting to Device` och gör `<p>` taggen som innehåller `Device is ready` synlig.

Cordova har fler events vi kan utnyttja, här kan du läsa om [de olika eventen](https://cordova.apache.org/docs/en/latest/cordova/events/events.html). Det finns bl.a. `pause` som aktiveras när appen läggs i bakgrunden och `resume` som aktiveras när appen plockas fram från bakgrunden.

Då kan det vara dags att kicka på hur appen ser ut. Vi måste lägga till vilka plattformar appen ska fungera på och sen starta appen. Vi börjar med att lägga till browser och köra appen i den inbyggda webbservern.

```bash
$ cordova platform add --save browser
$ cordova run browser
```

<!-- [FIGURE src=/image/kunskap/android-emulator-cordova-app.png?w=200&h=400]

`cordova emulate android` startar `cordova build` som bygger en [_apk_-fil](https://sv.wikipedia.org/wiki/APK_(filformat)), som du kan hitta här, `/platforms/android/build/outputs/apk/android-debug.apk`.

För att felsöka appen i webbläsaren behöver vi lägga till `browser` som en platform och starta en emulator för browsern.

```bash
$ cordova platform add browser --save
$ cordova emulate browser
``` -->

Om du öppnar `index.html` filen i webbläsaren istället för att köra `cordova run browser` kommer `cordova.js` att saknas. `cordova.js` läggs till när du exekverar `cordova run browser`. Då flyttas även din kod till `/platforms/browser/www/` och det är här webbservern utgår ifrån.

<!-- [INFO]
Att köra emulatorer använder mycket resurser från datorn. Om du inte lyckas stänga ner emulator processerna helt kan det sluta med att du har flera liggandes i bakgrunden vilket får din dator att prestera sämre. Kolla vilka processer du har igång med `ps` kommandot i samma terminal du har startat emulatorerna. Om det finns gamla processer kan du döda dem med `kill -9 <PID>`.
[/INFO] -->


Utveckla appen {#utveckla}
--------------------------------------

När vi utvecklar appar med Cordova är det viktigt att använda [SPA design](https://en.wikipedia.org/wiki/Single-page_application), Single Page Application, då vi behöver vänta på `deviceready` eventet för att kunna använda plugins. Om vi aldrig byter sida behöver vi bara vänta på `deviceready` när applikationen startar annars behöver vi vänta på det eventet varje gång vi laddar om en sida.

Vi sätter igång och ändrar i koden i `index.html`.

```html
...
<body>
    <div class="app">
        <p>App is loading. Hold on!</p>
        </div>

...
```
Vi ersätter innehållet i `app`-diven med en placeholder text som endast ska visas innan `deviceready` har aktiverats.

I `index.js` rensar vi också. Lägg till `"use strict";` överst i filen så valideringen blir rätt. Behåll `initialize` och `onDeviceReady` funktionerna, ta bort `receivedEvent` funktionen och ersätt den med en `homePage` funktion.

```js
"use strict";
...

    onDeviceReady: function() {
        this.homePage();
    },

    homePage: function() {
    }

...
```

Då ska vi ändra innehållet i vår app. Med SPA tänket väljer vi att ändra innehållet i `<div class="app">` med hjälp av JavaScript för att inte behöva ladda om sidan.

```js
homePage: function() {
    var content = document.getElementsByClassName("app")[0];

    var html = "<h1 class='title'>Hello World</h1>";
    html += "<div class='main'>Hej och välkommen till min första Cordova app.</div>";
    content.innerHTML = html;
}
```
Många DOM förändringar har stor inverkan på prestanda, därför väljer vi att skapa en sträng som innehåller all html kod och tilldelar `content.innerHTML` med strängen. Därmed påverkar vi bara DOM en gång istället för två när vi ändrar innehållet.

Testa appen och kolla hur det ser ut.

```bash
$ cordova emulate android
```

Jag har så klart ändrat i CSS filen. Testa du också att leka med stylen.

[FIGURE src=/image/kunskap/cordova/cordova-intro-v1.png?w=500&h=300]


##Touch event {#touch}

För att ändra innehållet i appen lägger vi till en `button` och skapa ett `touchend` event.

```js
homePage: function() {
    var content = document.getElementsByClassName("app")[0];

    var html = "<h1 class='title'>Hello World</h1>";
    html += "<div class='main'>Hej och välkommen till min första Cordova app.";
    html += "<button class='otherPage'>Nästa sida</button></div>";
    content.innerHTML = html;

    var button = document.getElementsByClassName("otherPage")[0];
    button.addEventListener("touchend", this.otherPage);
},

otherPage: function() {
    var title = document.getElementsByClassName("title")[0];
    var content = document.getElementsByClassName("main")[0];

    title.innerHTML = "Other page";

    var html = "Try an alert!<br>";
    html += "<button class='alertButton'>Alert</button>";
    content.innerHTML = html;

    var button = document.getElementsByClassName("alertButton")[0];
    button.addEventListener("touchend", function() {
        window.alert("Hej");
    });
}
```
Vi skapar en ny funktion `otherPage` där vi uppdaterar innehållet i appen. På den nya sidan lägger vi till en knapp som skapar ett `alert`.
Det är viktigt att använda något `touch-event` istället för `click`. `click` har en 300ms fördröjning innan den reagerar på att du har klickat på knappen.

Testa appen igen.

```bash
$ cordova emulate android
```
[FIGURE src=/image/kunskap/cordova/cordova-intro-alert.png?w=500&h=300]

Nu har vi gjort vår första app i Cordova och testat den i Android. Om vi istället testar appen i webbläsaren som en vanlig webbsida kommer inte knapparna funka. Testa appen med `cordova emulate browser` och aktivera inte responsive mode i developer tools. Webbläsare aktiverar bara 'touch' event om man är i responsive mode. Om vi också vill att knapparna ska funka som en vanlig webbsida behöver vi lägga till ett `click` event som gör samma sak som vårt `touch` event.

Jag gör det smidigast genom att skapa en ny funktion som tar emot ett element och en callback funktion. I funktionen skapar vi både ett 'click' och ett 'touchend' event för elementet.

```js
    homePage: function() {
        ...
        var button = document.getElementsByClassName("otherPage")[0];
        app.addEventListeners(button, this.otherPage);
    },

    otherPage: function() {
        ...
        var button = document.getElementsByClassName("alertButton")[0];
        app.addEventListeners(button, function() {
            window.alert("Hej");
        });
    },

    addEventListeners: function(element, callback) {
        element.addEventListener("touchend", callback);
        element.addEventListener("click", callback);
    }
```

Nu funkar appen både i responsive mode och i en vanlig webbläsare. Om du testade klicka på `alertButton` i responsive mode så märkte du kanske att `alert` visades två gånger, det är för att webbläsaren aktiverar både click och touchend när den är i responsive mode (även emulatorn gör det). När vi använder appen i en mobil/responsive mode i webbläsaren vill vi inte aktivera click utan bara touchend. I och med att touchend aktiveras före click (click har 300ms delay) kan vi lägga till [event.preventDefault](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault) i touchend, det kommer stoppa click från att aktiveras.

```js
addEventListeners: function(element, callback) {
    element.addEventListener("touchend", function(event) {
        event.preventDefault();
        callback();
    });
    element.addEventListener("click", callback);
}
```

Nu ska appen fungera som det är tänkt. Testa den i webbläsaren, både med och utan responsive mode. Lek med eventen, preventDefault och console.log om du inte hängde med på hur det fungerar.

Felsöka appar {#felsoka}
--------------------------------------

Det finns olika sätt att felsöka en Cordova app.
Vi börjar med det lättaste, felsöka den i webbläsaren och som en vanlig webbsida.
När det fungerar kan du gå vidare och kolla om det fungerar i emulatorn. När vi felsöker i emulatorn är det bra om vi kan se utskrifter vi gör med `console.log()`.
För att se `console.log` kan vi starta `adb logcat` i terminalen medans vi kör appen. Jag lägger in en `console.log()` i eventet för alert, startar emulatorn och kör sen kommandot `adb logcat`.
```js
button.addEventListener("touchend", function() {
    console.log("alert sent");
    window.alert("Hej");
});
```
[FIGURE src=/image/kunskap/cordova/cordova-intro-logcat.png]


Det är även möjligt att felsöka emulatorn via chrome. Starta emulatorn, öppna en chrome webbläsare och gå till `chrome://inspect/#devices`.
[FIGURE src=/image/kunskap/cordova/cordova-intro-chrome-dbg.png?w=500&h=300]
Klicka på inspect så får du upp ett nytt fönster där du kan styra din emulator och felsöka appen som om det vore en webbsida.
[FIGURE src=/image/kunskap/cordova/cordova-intro-inspect.png?w=500&h=300]



Avslutningsvis {#avslutning}
--------------------------------------

Nu har vi skapat en mobil webbapp i Cordova och lärt oss felsöka den.
Tänk på att påverka DOM så lite som möjligt och använd `touch` event  istället för `click`.

Har du [tips, förslag eller frågor om artikeln](t/6312) så finns det en specifik forumtråd för det.
