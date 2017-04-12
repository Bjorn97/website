---
author:
    - mos
category:
    - kurs oophp
    - php
    - sql
    - mysql
    - php pdo
revision:
    "2017-04-10": "(D, mos) Uppdaterad till nya version inför oophp-v3."
    "2014-10-20": "(C, mos) Bytte koden till doFilter(), innehöll fel."
    "2013-12-03": "(B, mos) Smärre justeringar inför campus-kursen."
    "2013-09-30": "(A, mos) Första utgåvan i samband oophp-kursen."
...
Lagra innehåll i databas för webbsidor och bloggposter (v2)
==================================

[FIGURE src=image/?w=c5 class="right" caption=""]

Ibland vill vi hantera webbplatsens innehåll genom att lagra det i databasen. Denna artikel visar hur du bygger upp en tabell som lagringsstruktur för innehåll som kan användas till webbsidor och bloggposter. Via ett formulär kan du redigera sidans titel och innehåll som sparas undan i databasen. Det blir enkelt att uppdatera din webbsidas innehåll och visa upp den på valfri länk, utan att ha direkt tillgång till databasen eller webbservern. Det räcker med en webbläsare.

När du är klar har du byggt både webbsidor och en blogg från innehåll som du lagrat och redigerat i databasen.

<!--more-->

[WARNING]
**Arbete pågår**
[/WARNING]

Så här kan det se ut när man redigerar innehåll i databasen via ett formulär.

[FIGURE src=/image/snapshot/Uppdatera_innehall___Innehall_i_databasen.jpg?w=w2 caption="Formulär för att editera innehåll i databasen."]



Förutsättningar {#pre}
-----------------------------------------

Du är bekant med PHP PDO och MySQL och/eller du har jobbat igenom artikeln "[Kom igång med PHP PDO och MySQL (v2)](kunskap/kom-igang-med-php-pdo-och-mysql-v2)".

Exempelkoden finns i kursrepot (oophp) under [`example/content`](https://github.com/dbwebb-se/oophp/tree/master/example/content).



Tankar om *innehåll* i databasen {#om}
-------------------------------

Låt oss fundera, varje webbsida har en `<title>` och bör ha en rubrik i form av en `<h1>`. Det skall finnas _content_, själva innehållet i webbsidan.

En sida kan ha en unik länk, en _path_ till där sidan kan nås. Man skall kunna nå innehållet via en slug, en url som bygger på innehållets titel. Det är olika sätt att hitta matchande innehåll i databasen.

En sida kan ha olika status, den kan vara publicerad eller inte publicerad. Att vara _inte publicerad_ kan innebära en arbetskopia som man inte vill visa upp förrän den är klar.

Det är bra att ha datumstämplar för när sidan skapades, uppdaterades och raderades. 

En sida kan vara av en speciell typ, en typ som bestämmer hur och var sidan kan visas. Det kan vara bra att vara lite flexibel och visa sidor på olika sätt, baserade på dess typ. Till exempel så visar man innehåll av typen 'post' i bloggen och innehåll av typen 'page' som egna sidor.

Innehållet i sidan skulle kunna filtreras på olika sätt, vill man skriva PHP-kod i sidan, eller HTML, BBCode eller Markdown? Det kan behövas olika filter som bearbetar innehållet innan det presenteras. 

Oj, det blev en del. Bäst att börja enkelt.



En databastabell {#table}
-------------------------------

Det första vi behöver är en lagringsstruktur för innehållet. Vi behöver fundera igenom vad vi vill och behöver lagra samt hur lagringen skall ske. Det går ju naturligtvis att justera strukturen efter hand men det är rätt bra att tänka igenom strukturen från början. Det är alltså en databastabell vi behöver, en grundstruktur för vad vi vill lagra.

Vi behöver alltså skapa en tabell och fylla på med lite innehåll. Sedan kan vi börja jobba.



###Tabellstruktur {#tabellstrukt}

Jag lägger in all SQL-kod i en fil som jag sedan kan köra i Workbench eller direkt i terminalen. Det gör det enkelt att jobba och uppdatera SQL-koden efter hand.

```sql
-- Ensure UTF8 as chacrter encoding within connection.
SET NAMES utf8;

--
-- Create table for Content
--
DROP TABLE IF EXISTS `content`;
CREATE TABLE `content`
(
  `id` INT AUTO_INCREMENT PRIMARY KEY NOT NULL,
  `path` CHAR(120) UNIQUE,
  `slug` CHAR(120) UNIQUE,

  `title` VARCHAR(120),
  `data` TEXT,
  `type` CHAR(20),
  `filter` VARCHAR(80) DEFAULT NULL,

  `published` DATETIME DEFAULT CURRENT_TIMESTAMP,
  `created` DATETIME DEFAULT CURRENT_TIMESTAMP,
  `updated` DATETIME DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
  `deleted` DATETIME DEFAULT NULL

) ENGINE INNODB CHARACTER SET utf8 COLLATE utf8_swedish_ci;
```

Tabellens struktur matchar de grundkrav som jag tänkt mig. Att ha både `path` och `slug` är två skilda saker, men båda skall unikt kunna peka ut ett visst innehåll.

Jag använder default värden på tidsstämplarna `published` och `created` för att förenkla för mig.

Jag väljer att använda en inbyggd konstruktion för `updated` som heter `ON UPDATE CURRENT_TIMESTAMP` vilket gör att den tidsstämpeln ändras varje gång jag gör en update på tabellen. Man kan fundera på om det är bra eller ej, jag väljer här att bygga in mig i MySQL och släppa på kompabiliteten med andra databaser. Det fungerar i min _domän_ som är detta exemplet och här tänker jag inte vara kompatibel mellan databaser.



###Tabellens innehåll {#tabellcont}

För att komma i gång så lägger jag in lite rader i tabellen. Radernas innehåll visar lite hur jag har tänkt att det skall fungera. Det är inte komplett, men en start.

```sql
INSERT INTO `content` (`path`, `type`, `title`, `data`, `filter`) VALUES
    ("hem", "page", "Hem", "Detta är min hemsida. Den är skriven i [url=http://en.wikipedia.org/wiki/BBCode]bbcode[/url] vilket innebär att man kan formattera texten till [b]bold[/b] och [i]kursiv stil[/i] samt hantera länkar.\n\nDessutom finns ett filter 'nl2br' som lägger in <br>-element istället för \\n, det är smidigt, man kan skriva texten precis som man tänker sig att den skall visas, med radbrytningar.", "bbcode,nl2br"),
    ("om", "page", "Om", "Detta är en sida om mig och min webbplats. Den är skriven i [Markdown](http://en.wikipedia.org/wiki/Markdown). Markdown innebär att du får bra kontroll över innehållet i din sida, du kan formattera och sätta rubriker, men du behöver inte bry dig om HTML.\n\nRubrik nivå 2\n-------------\n\nDu skriver enkla styrtecken för att formattera texten som **fetstil** och *kursiv*. Det finns ett speciellt sätt att länka, skapa tabeller och så vidare.\n\n###Rubrik nivå 3\n\nNär man skriver i markdown så blir det läsbart även som textfil och det är lite av tanken med markdown.", "markdown"),
    ("blogpost-1", "post", "Välkommen till min blogg!", "Detta är en bloggpost.\n\nNär det finns länkar till andra webbplatser så kommer de länkarna att bli klickbara.\n\nhttp://dbwebb.se är ett exempel på en länk som blir klickbar.", "link,nl2br"),
    ("blogpost-2", "post", "Nu har sommaren kommit", "Detta är en bloggpost som berättar att sommaren har kommit, ett budskap som kräver en bloggpost.", "nl2br"),
    ("blogpost-3", "post", "Nu har hösten kommit", "Detta är en bloggpost som berättar att sommaren har kommit, ett budskap som kräver en bloggpost", "nl2br");

SELECT `id`, `path`, `slug`, `title`, `created` FROM `content`;
```

Jag lägger bara in värden i ett par av kolumnerna. Du kan se att jag än så länge utelämnar `slug` som blir `null` för alla rader. Jag nöjer mig med att kunna nå allt innehåll via dess `path` som jag tänker använda som en del i länken till innehållet.



###Läs in innehållet till databasen {#contentread}

Så här gör jag för att läsa in innehållet till min databas.

```bash
$ mysql -uuser -ppass oophp < sql/setup.sql
Warning: Using a password on the command line interface can be insecure.
id   path          type    title                      created
1    hem           page    Hem                        2017-04-10 12:35:27
2    om            page    Om                         2017-04-10 12:35:27
3    blogpost-1    post    Välkommen till min blogg!  2017-04-10 12:35:27
4    blogpost-2    post    Nu har sommaren kommit     2017-04-10 12:35:27
5    blogpost-3    post    Nu har hösten kommit       2017-04-10 12:35:27
```

Men det fungerar lika bra med Workbench eller någon annan klient.



Kommentarer till lagringsstrukturen {#tankt}
-------------------------------

När man skapar sina tabeller så har man också en grundtanke om hur man skall lösa sin applikation. Man ser till att lagra det som behövs för att applikationen skall fungera.

Låt oss titta på hur jag har tänkt.



###Två typer `page` och `post` {#tva-typer}

Jag tänker mig två typer av innehåll, `page` och `post`. Jag tänker mig rena webbsidor som bygger på innehållet i databasen, dessa har typen `page`. Sedan tänker jag mig en blogg och det innehållet ger jag typen `post`. Två typer och två tänkta hanteringar av innehållet.



###Innehållet {#innehall}

Själva innehållet är kolumnerna `title` och `data`, det är det som ger sidans innehåll. Det blir bra som en `<h1>` följt av text. Dessutom kan `title` bli en del av sidans `<title>` element.

Jag tänker också använda mig av `title` för att skapa innehållets `slug`.



###Att komma åt innehållet {#id}

Det finns (minst) tre olika sätt att komma åt ett specifikt innehåll, dels via `id` som kan användas när användaren skall editera eller radera en sida.

Delen med `path` är en tänkt del av länken till innehållet. Det ger möjligheten att matcha en länk/route direkt till ett innehåll.

Varianten med `slug` är en översättning av innehållets titel som lämpar sig att använda i länkningen till sidan. En länk med en slug kan se ut så här:

> `https://dbwebb.se/coachen/gor-en-lasbar-url-med-slugify`

Texten *gor-en-lasbar-url-med-slugify* är en slug till en sida som har titeln "Gör en läsbar url med slugify()". Läs mer om en [funktion som gör en slug av en sträng](coachen/gor-en-lasbar-url-med-slugify).

Det är alltså tre olika sätt att peka ut det innehåll som skall hanteras. Det ger viss flexibilitet och det vill vi ha.



###Filter {#filter1}

Tanken är att innehållet skall processas via olika filter innan det visas upp. Detta ger möjligheten att använda olika strategier när man skriver sitt innehåll. Exempel på olika filter kan vara BBCode eller Markdown.

Här kan man till och med tillåta PHP-kod i artikeln, men det är kanske att gå till överdrift, eller iallafall en möjlighet som inte skall erbjudas var och en.

Här får man även tänka till om man skall tillåta HTML-kod i innehållet, eller om det skall filtreras bort. Att tillåta HTML-kod innebär att man även tillåter JavaScript. Vill man ge den öppningen till webbplatsens användare?

Vill du få ett smakprov på ett filter som översätter BBCode till HTML så kikar du via artikeln "[Reguljära uttryck i PHP ger BBCode formattering](coachen/reguljara-uttryck-i-php-ger-bbcode-formattering)". Det leder till en funktion som översätter BBCode likt `[b]Bold[/b]` till `<b>Bold</b>` och erbjuder vissa möjligheter för en användare att formattera sina inlägg.



###Tidsstämplar {#tid}

Jag sätter fyra olika tidsstämplar. När innehållet skapas så sätts `created`, varje gång som innehållet uppdateras så uppdateras `updated`. När innehållet raderas så sätts `deleted`. 

Tanken med `deleted` är att markera innehållet som raderat, utan att fysiskt ta bort det. Kanske vill man återskapa det, eller införa en hantering som liknar en *papperskorg*. Ett dokument som slängs i papperskorgen kan man ta upp (eller kasta bort på riktigt).

Den fjärde tidsstämpeln är `published` som sätter en tidpunkt för när innehållet är publicerat. Detta ger möjligheten att sätta ett datum i framtiden för att publicera innehållet vid en bestämd tidpunkt. Vill man inte att innehållet skall visas så kan man nollställa `published` till `null` alternativt sätta ett datum långt fram i tiden.



Visa allt innehåll {#visa-allt}
-------------------------------

Nu finns det en lagringsstruktur och några exempel på innehåll i tabellen. Låt oss nu att visa upp tabellens innehåll i en webbsida.

Det kan se ut så här.

[FIGURE src=image/snapvt17/content-all-take1.png?w=w2 caption="En första ansats att visa vilket innehåll som finns i databasen."]

Det som behövs är en route motsvarande den som vi använda i film-databasen. Jag kunde nästan återanvända all min kod från det exemplet.

```php
switch ($route) {
    case "":
        $title = "Show all content";
        $view[] = "view/show-all.php";
        $sql = "SELECT * FROM content;";
        $resultset = $db->executeFetchAll($sql);
        break;
```

Vyn innehåller koden för att översätta innehållet i `$resultset` till en HTML-tabell.

```php
<?php
if (!$resultset) {
    return;
}
?>

<table>
    <tr class="first">
        <th>Rad</th>
        <th>Id</th>
        <th>Title</th>
        <th>Type</th>
        <th>Published</th>
        <th>Created</th>
        <th>Updated</th>
        <th>Deleted</th>
    </tr>
<?php $id = -1; foreach ($resultset as $row) :
    $id++;
?>
    <tr>
        <td><?= $id ?></td>
        <td><?= $row->id ?></td>
        <td><?= $row->title ?></td>
        <td><?= $row->type ?></td>
        <td><?= $row->published ?></td>
        <td><?= $row->created ?></td>
        <td><?= $row->updated ?></td>
        <td><?= $row->deleted ?></td>
    </tr>
<?php endforeach; ?>
</table>
```

Jag bygger alltså vidare på samma kodstruktur som jag använde i exemplet med film-databasen. Det känns som en enkel och ren struktur för att bygga ett exempelprogram och ändå jobbar med routes, vyer och hålla koden hyggligt separerad.



<!--
Nu behövs ett gränssnitt till användaren så innehållet kan uppdateras via webben. En bra start är att visa allt innehåll i databasen. Så här kan det se ut.

[FIGURE src=/image/snapshot/Visa_allt_innehall___Innehall_i_databasen.jpg?w=w2 caption="Visa allt innehåll från databasen."]

**SQL-sats för att hämta allt innehåll.**

```sql
SELECT *, (published <= NOW()) AS available
FROM Content;
```

Så länge som datumet på `published` är passerat så anses innehållet vara publicerat.

Det finns två olika typer av innehåll. Jag vill att innehållet av typen `page` skall hanteras av sidkontrollern `page.php` och jag vill att innehåll av typen `post` skall hanteras av sidkontrollern `blog.php`. För att lyckas med det så väljer jag att göra en funktion som kan länka till innehållet genom att titta på dess typ.

**En funktion för att koppla innehållets typ till en webbsida.**

```php
/**
 * Create a link to the content, based on its type.
 *
 * @param object $content to link to.
 * @return string with url to display content.
 */
function getUrlToContent($content) {
  switch($content->type) {
    case 'page': return "page.php?url={$content->url}"; break;
    case 'post': return "blog.php?slug={$content->slug}"; break;
    default: return null; break;
  }
}
```

Man kan se det som att man skapar en permalänk till varje innehåll, men beroende på innehållets typ så skall olika sidkontroller hantera innehållet.
-->



Uppdatera innehållet {#crud-update}
-------------------------------

Nästa steg känns som jag vill göra CRUD-relaterade operationer på mitt innehåll.



###Route och formulär {#editroute}

Jag börjar med en enklare variant av vyn `show-all` som visar mindre saker tillsammans med en edit-knapp från FontAwesome.

[FIGURE src=image/snapvt17/content-edit.png?w=w2 caption="Ett admin-gränssnitt för att hantera innehållet."]

Routen till edit-formuläret blir `?route=edit&id=1`.

Nu behöver jag ett edit-formulär. Det kan se ut så här.

content-do-edit.png
[FIGURE src=image/snapvt17/content-do-edit.png?w=w2 caption="Innehållet kan nu redigeras i ett formulär."]

Vi kan kika kort på routen.

```php
case "edit":
    $title = "Edit content";
    $view[] = "view/edit.php";
    $contentId = getGet("id");

    $sql = "SELECT * FROM content WHERE id = ?;";
    $content = $db->executeFetch($sql, [$contentId]);
    break;
```

Vi hämtar innehållets id från querysträngen och använder det för att hämta ut rätt innehåll från databasen. Här vet vi att det bara finns en rad att hämta ut så vi använder metoden `fetch()` för att bara hämta första raden.

I vyn byggs formuläret upp med dess innehåll. Här är det klokt att använda `htmlentities()` och jag gör det via min egna wrapper `esc()`.

Här kan du se vissa delar av formuläret.

```php
<p>
    <label>Slug:<br> 
    <input type="text" name="contentSlug" value="<?= esc($content->slug) ?>"/>
</p>

<p>
    <label>Text:<br> 
    <textarea name="contentData"><?= esc($content->data) ?></textarea>
 </p>

 <p>
     <label>Publish:<br> 
     <input type="datetime" name="contentPublish" value="<?= esc($content->published) ?>"/>
 </p>
```

Det kan också var en idé att styla till sitt formulär så att det ser lite vänligare ut. Jag gör lite enklare styling.

```php
input[type=text],
input[type=datetime],
textarea {
    width: 400px;
}

textarea {
    height: 200px;
}

label {
    font-size: small;
}

input,
textarea,
legend {
    font-size: large;
}
```

Då ska vi se vad som bör hända när man klickar på Spara-knappen.



###Spara uppdaterat innehåll {#edit-save}

Här fångar vi in alla delar från formuläret och bygger upp en array med parametrar som skickas med SQL UPDATE uttrycket.

Det vi vill uppnå ser ut så här.

```php
if (hasKeyPost("doSave")) {
    //$params gets all values from getPost()
    $sql = "UPDATE content SET title=?, path=?, slug=?, data=?, type=?, filter=?, publish=? WHERE id = ?;";
    $db->execute($sql, array_values($params));
    header("Location: ?route=edit&id=$movieId");
    exit;
}
```

Kolumnen `update` bör nu uppdateras automatiskt av MySQL.

Finns det ett smart, och någorlunda kontrollerar sätt, att fylla på arrayen `$params` utifrån POST?

Ja, vi skulle kunna uppdatera funktionen `getPost()` till att stödja följande usecase och hämta många värden på en gång och spara i en array.

```php
//$params gets all values from getPost()
$params = getPost([
    "contentTitle",
    "contentPath",
    "contentSlug",
    "contentData",
    "contentType",
    "contentFilter",
    "contentPublish",
    "contentId"
]);
```

Ovan sparar vi ett antal anrop till `getPost()`, vi hade behövt en för varje värde vi vill hämta.

Nu löser vi det med manipulering av arrayer.

```php
/**
 * Get value from POST variable or return default value.
 *
 * @param mixed $key     to look for, or value array
 * @param mixed $default value to set if key does not exists
 *
 * @return mixed value from POST or the default value
 */
function getPost($key, $default = null)
{
    if (is_array($key)) {
        $key = array_flip($key);
        return array_replace($key, array_intersect_key($_POST, $key));
    }

    return isset($_POST[$key])
        ? $_POST[$key]
        : $default;
}
```

I koden ovan så bygger vi ut funktionen `getPost()` till att tolka inkommande argument till funktionen och utföra till viss del olika arbeten beroende av det är en sträng eller en array som skickas via `$key`.

Som du ser kan det vara bekvämt att ha koll på de inbyggda array-funktionerna i PHP. Där kan finnas viss tid och kodrader att spara. Ovan konstruktion hämtar hem värdena från POST som matcher de nycklar som angavs i `$key`.

Man kunde löst samma sak med en enkel loop också.

```php
if (is_array($key)) {
    // $key = array_flip($key);
    // return array_replace($key, array_intersect_key($_POST, $key));
    foreach ($key as $val) {
        $post[$val] = getPost($val);
    }
    return $post;
```

Ibland vinner man på de inbyggda funktionerna men ibland går det likabra att lösa själv med en enkel loop.



###Generera en slug {#generera-slug}

Nu kan vi även lösa vår slug. Den skall skapas utifrån innehållets titel och vi kan använda en funktion liknande den vi finner i "[Gör en läsbar url med slugify()](coachen/gor-en-lasbar-url-med-slugify)".

Jag väljer att generera en slug om fältet är tomt. Då kan användaren själv skapa sin slug och i annat fall så sköts det automatiskt.

Förutsatt att funktionen `slugify()` finns på plats så kan jag lösa det så här.

```php
if (!$params["contentSlug"]) {
    $params["contentSlug"] = slugify($params["contentTitle"]);
}
```

Det blir både automatik och det ger användaren möjlighet att sätta slugen.

Men vad händer om det blir två likadana slugs?

[FIGURE src=image/snapvt17/content-duplicate-slug.png?w=w2 caption="Det kan inte finnas två likadana slugs."]

Här får man bygga in någon form av felhantering om man vill lösa detta bättre. Det lämnar vi som en övning till läsaren.

Man kan också fundera på om slugen skall vara unik, eller om den egentligen är unik i kombination med sin typ och kanske pathen. Det är något som man får bestämma sig hur, hur det skall fungera i just denna applikationen.



###Att lagra användarredigerat innehåll {#lagrastrat}

En sak som kan vara på sin plats att nämna är strategin för att lagra informationen i databasen. Skall man **sanitera den innan den lagras i databasen** eller **innan den används i webbsidan**? Jag väljer främst strategin att lagra innehållet precis som användaren skriver in det och sedan filtrera och sanitera när informationen skall visas i webbsidan. Jag föredrar att användaren kan spara ned precis vad som helst, men det ställer krav på att jag verkligen saniterar informationen och rensar bort skadlig kod, innan informationen skrivs ut. 

Att låta användare skriva in vad som helst, utan att sanitera, har fördelen att användaren alltid känner igen texten som skrivs och det finns ingen _magisk_ hantering som säger vilken text som är rätt eller fel. I mitt fall är det helt okey att skriva in både HTML-kod och PHP-kod, det som är viktigt är vilka filter man väljer att sätta på innehållet. Det är kombinationen av filter (och `htmlentities()`) som gör att texten saniteras och skadlig kod rensas bort.

Skillnaden mellan strategierna är _när_ den skadliga koden rensas bort. 

Det är också bra att kontrollera att inkommande variabler är definierade enligt den värdemängd som gäller.

```php
$contentId = getPost("contentId") ?: getGet("id");
if (!is_numeric($contentId)) {
    die("Not valid for content id.");
}
```


Lägga till nytt innehåll {#content-add}
-----------------------------------------

Då skapar vi funktionen för att lägga till nytt innehåll. Här får man tänkta till då alternativen är flera. Skall man lägga till en knapp eller en länk och var skall den finnas?



###Formulär och route för skapa {#content-create}

Jag väljer att lägga till en länk "Create" i menyn som leder till ett enkelt formulär där användaren får skriva in innehållets titel. När det formuläret postas så görs INSERT in i databasen och därefter en redirect till sidan där innehållet kan redigeras.

Tittar man på användargränssnittet så kan det se ut så här.

[FIGURE src=image/snapvt17/content-create.png?w=w2 caption="Ett minimalt formulär för att skapa en ny artikel."]

När användaren klickar "Create" så skapas innehållet i databasen. Routen tar hand om det.

```php
case "create":
    $title = "Create content";
    $view[] = "view/create.php";

    if (hasKeyPost("doCreate")) {
        $title = getPost("contentTitle");

        $sql = "INSERT INTO content (title) VALUES (?);";
        $db->execute($sql, [$title]);
        $id = $db->lastInsertId();
        header("Location: ?route=edit&id=$id");
        exit;
    }
    break;
```

Varken routen eller vyn innehåller särskilt mycket kod och jag är nöjd att kunna återanvända routen "edit" för att vidare redigera innehållet.

[FIGURE src=image/snapvt17/content-create-edit.png?w=w2 caption="Det nyskapade innehållet finns nu i databasen och jag kan fylla på och spara dess värden."]

Som du ser så är en nyskapad artikel redan publicerad. I mitt exempel är det bra eftersom jag slipper skriva in ett publiceringsdatum i formuläret. Men egentligen vill jag troligen inte att helt nyskapade artiklar skall vara publicerade. Det handlar om hur man väljer att sätta, eller inte sätta, defaultvärde på `published`.



###Tom sträng är inte samma sak som null {#tom-null}

Var uppmärksam på att en tom sträng och NULL inte är samma sak i en databaskolumn. Att ha en path som är NULL är okey i databastabellen, även om det finns flera och trots att vi har satt UNIQUE på kolumnen. Men, det är inte okey att ha flera rader som har en path som är tomma strängen. Då får du fel, ungefär så här.

> *Duplicate entry '' for key 'path'.*

När du läser upp innehållet från databasen, lägger det i formuläret och åter sparar det, så är det lätt att ditt ursprungliga NULL görs om till en tom sträng. Detta leder i sin tur till att du får problem med databastabellens restriktioner om att path skall vara UNIQUE. Ett enkelt sätt att gå runt problemet är att kontrollera om fältet är tomt (tom sträng) och då sätta den till NULL istället.

I sådana här fall är det bra att ha bra felhantering och det försöker vi har i vår klass `Database` som kastar Exception så fort något går fel.

När jag byggde mitt exempel så fick jag exakt detta problemet och jag löste det genom att sätta en tom path till null i min edit-sida.

```php
if (!$params["contentPath"]) {
    $params["contentPath"] = null;
}
```



Radera innehåll {#content-del}
-----------------------------------------

Då har vi bara delete delen kvar i CRUD. Till att börja med så gör jag det enkelt och lägger till en knapp "Delete" i Edit-formuläret.

Sen tänker jag att det vore trevligt att kunna radera ett innehåll direkt från översikten i Admin via en delete-ikon.

Jag vill också har en säkerhetsfråga om "är du verkligen säker", så att användaren inte av snabbhet eller misstag råkar radera ett innehåll.

Då ska vi se vilket användargränssnitt detta flöde kan få.

Först en ny knapp, nu med ikoner, i edit-formuläret.

[FIGURE src=image/snapvt17/content-delete-edit.png?w=w2 caption="Nu kan man radera ett innehåll från edit-formuläret."]

Sedan en radera-ikon på översikten.

[FIGURE src=image/snapvt17/content-delete-admin.png?w=w2 caption="Det går även radera ett innehåll direkt från admin-vyn."]

I båda fallen leder länken till en vy där man verkligen kan utföra raderingen. Allt för att undvika misstag.

[FIGURE src=image/snapvt17/content-delete-true.png?w=w2 caption="Nu kan vi verkligen radera innehållet."]

Vyerna och formulärens förändringar är enkla. Det är kan vara lite klurigt är hur man vill att flödet i applikationen skall fungera. Var kan kan radera och hur skall det gå till.

Vi kan ta en titt på routen som utför själva raderingen av innehållet.

```php
case "delete":
    $title = "Delete content";
    $view[] = "view/delete.php";
    $contentId = getPost("contentId") ?: getGet("id");
    if (!is_numeric($contentId)) {
        die("Not valid for content id.");
    }

    if (hasKeyPost("doDelete")) {
        $contentId = getPost("contentId");
        $sql = "UPDATE content SET deleted=NOW() WHERE id=?;";
        $db->execute($sql, [$contentId]);
        header("Location: ?route=admin");
        exit;
    }

    $sql = "SELECT id, title FROM content WHERE id = ?;";
    $content = $db->executeFetch($sql, [$contentId]);
    break;
```

Notera att vi inte gör en riktig DELETE.

```php
$sql = "DELETE FROM content WHERE id=?;";
```

Istället markerar vi innehållet som raderat, via en tidsstämpel som säger när innehållet raderades.

I övrigt så ser det mer eller mindre likadant ut som i de andra routerna. Liknande struktur och hantering. Det känns som vi börjar få koll på det här nu.

Vi kan avsluta med en kort titt på delete-vyn.

```php
<form method="post">
    <fieldset>
    <legend>Delete</legend>

    <input type="hidden" name="contentId" value="<?= esc($content->id) ?>"/>

    <p>
        <label>Title:<br> 
            <input type="text" name="contentTitle" value="<?= esc($content->title) ?>" readonly/>
        </label>
    </p>

    <p>
        <button type="submit" name="doDelete"><i class="fa fa-trash-o" aria-hidden="true"></i> Delete</button>
    </p>
    </fieldset>
</form>
```

Det man kan nämna är att innehållets id ligger i formuläret som postas för att raderas. Sen skadar det inte att påpeka att göra `htmlentites()` på de värden som skrivs ut.

När ett innehåll är raderat så markeras det som raderat och det syns i admin-vyn.

[FIGURE src=image/snapvt17/content-delete-bydate.png?w=w2 caption="Artikeln som raderades är märkt som raderad, men inte fysiskt borttagen."]

Nu hade vi kunnat lägga till en funktion som tar bort tidsstämpeln som anger när innehållet raderades. Då hade innehållet _kommit tillbaka_ igen, som inget hade hänt.

Nu är vi nöjda med CRUD-delen, det administrativa gränssnittet för att jobba med innehållet.



Visa innehållet i webbplatsen {#page}
-------------------------------

Då tar vi och visar upp innehållet från databasen i egna webbsidor. Tanken är att sidor av typen `page` skall visas upp som en egen webbsida och sidor av typen `post` visas som en blogg. 



###Länkstrukturen {#links}

Här behöver vi en plan för hur länkningen skall ske. Min plan är så här.

Typen `page` kan nås via `?route=path` där path mappas mot databaskolumnen `path`. Om man skapar en sida som har sin path satt till `om` så blir länken till sidan `?route=om`.

Typen `blog` skall visa en blogglista på routen `?route=blog` där blogginläggen kommer i ordning med senaste inlägget först. Vill man nå ett inlägg direkt och se hela inlägget som en egen sida, så är länken dit `?route=blog/slug` där slug mappas mot databaskolumnen `slug`.

Om vi hade haft en riktig router så hade detta varit smidigt att sätta upp. I vår exempelkod med den hemmasnickrade varianten av router kan vi lösa det med en default hantering i switch-satsen. Grovt kan det se ut så här.

```php
default:
    if (substr($route, 0, 5) === "blog/") {
        //  Matches blog/slug, display content by slug and type post
    } elseif (substr($route, 0, 4) === "blog") {
        // Matches blog, show bloglist of type post
    } else {
        // Try matching content for type page and its path
    }
```

Det blev tre olika varianter.



###Visa innehåll av typ page via dess path {#pagepath}

Här behöver vi söka ut det innehåll som är av typen `page` och där `path` matchar `$route` samt att innehållet är publicerat och inte raderat.

Fundera lite hur du hade löst det. Nu får vi använda våra SQL-kunskaper.

Snart kommer min variant, men först valde jag att bygga en översikt över de sidor som fanns. Det blir enklare att se och testa konstruktionerna samt enklare att klicka direkt till de sidor som kan visas. Om en sidan inte kan visas så får det bli en variant av 404.

Så här blev min översikt.

[FIGURE src=image/snapvt17/content-view-pages.png?w=w2 caption="Här är en översikt av de webbsidor som är av typen `page`."]

Du kan se att det visas vilken status sidan har, om den är publicerad eller inte och om den är raderad.

Det är lite spännande att titta på den route som bygger upp resultatet.

```php
case "pages":
    $title = "View pages";
    $view[] = "view/pages.php";

    $sql = <<<EOD
SELECT
    *,
    CASE 
        WHEN (deleted <= NOW()) THEN "isDeleted"
        WHEN (published <= NOW()) THEN "isPublished"
        ELSE "notPublished"
    END AS status
FROM content
WHERE type=?
;
EOD;
    $resultset = $db->executeFetchAll($sql, ["page"]);
    break;
```

Själva routen är liten och enkel men SQL-frågan hade ett inslag av en if-sats som kollar status på kolumnerna `deleted` och `published` och därefter bestämmer den vilken status innehållet har och lämnar svaret i den nyskapade kolumnen `status`.

Om jag nu klickar på länken till `home` så kan det se ut så här.

[FIGURE src=image/snapvt17/content-home.png?w=w2 caption="Sidan för `home` visas."]

Ännu finns ingen textfiltrering, men innehållet visas som det ska.

Vi kan titta på vyn som är ansvarig för att visa upp resultatet.

```php
<article>
    <header>
        <h1><?= esc($content->title) ?></h1>
        <p><i>Latest update: <time datetime="<?= esc($content->modified_iso8601) ?>" pubdate><?= esc($content->modified) ?></time></i></p>
    </header>
    <?= esc($content->data) ?>
</article>
```

Inte så mycket kod men notera hur publikationsdatumet skrivs ut i två varianter, en läsbar för människor `modified` och en läsbar för maskiner `modified_iso8601`.

Att skapa de två datumen väljer jag att göra med SQL-koden. Vi kan titta på routen och dess SQL.

```php
} else {
    // Try matching content for type page and its path
    $sql = <<<EOD
SELECT
    *,
    DATE_FORMAT(COALESCE(updated, published), '%Y-%m-%dT%TZ') AS modified_iso8601,
    DATE_FORMAT(COALESCE(updated, published), '%Y-%m-%d') AS modified
FROM content
WHERE
    path = ?
    AND (deleted IS NULL OR deleted > NOW())
    AND published <= NOW()
;
EOD;
    $content = $db->executeFetch($sql, [$route]);
    if (!$content) {
        header("HTTP/1.0 404 Not Found");
        $title = "404";
        $view[] = "view/404.php";
        break;
    }
    $title = $content->title;
    $view[] = "view/page.php";
}
```

Själva SQL-frågan löser dels de två datumformaten där den väljer det första datumet av `updated` och `published` som inte är null som bas för `modified`.

I WHERE-delen så säkerställs att sidans path matchar och den är publicerad och inte raderad, annars väljs den inte.

Om SQL-frågan ger ett tomt svar tillbaka så visas en 404-sida.

[FIGURE src=image/snapvt17/content-404.png?w=w2 caption="404 för att sidan kunde inte visas."]

Här var en sida som inte kunde visas för att den antingen inte fanns, den var raderad, eller var den ännu inte publicerad.



Visa innehållet som blogg- och bloggposter {#blog}
-------------------------------

Då ger vi oss på att visa innehåll för bloggen vilket är allt innehåll av typen `post`.

När vi visar innehåll av typen `post` blir det lite mer hantering i sidkontrollern `blog.php`. Tanken är att den skall både kunna visa en enskild bloggpost samt en översikt av alla bloggposter.

Så här kan det se ut när alla bloggposter visas.

> `blog.php`

[FIGURE src=/image/snapshot/Bloggen___Innehall_i_databasen.jpg?w=w2 caption="Alla bloggposter visas efter varandra."]

Vill man bara visa en bloggpost så anropar man sidkontrollern med innehållets slug som parameter, så här.

> `blog.php?slug=blogpost-1`

[FIGURE src=/image/snapshot/Valkommen_till_min_blogg____Bloggen___Innehall_i_databasen.jpg?w=w2 caption="En bloggpost visas ensam på en sida."]

I min kod väljer jag att bygga upp en SQL-sats beroende på om sluggen är närvarande eller ej. 

**En SQL-sats för båda fallen.**

```php
// Get content
$slugSql = $slug ? 'slug = ?' : '1';
$sql = "
SELECT *
FROM Content
WHERE
  type = 'post' AND
  $slugSql AND
  published <= NOW()
ORDER BY updated DESC
;
";
$res = $db->ExecuteSelectQueryAndFetchAll($sql, array($slug));
```

Det gör att jag kan hantera resultatet på samma sätt oavsett om det är en eller flera bloggposter som skall visas. Så här ser den delen ut.

**Samma kodsekvens visar upp blogglistan och enkla bloggposter.**

```php
// Prepare content and store it all in variables in the Anax container.
$anax['title'] = "Bloggen";
$anax['main'] = null;
if(isset($res[0])) {
  foreach($res as $c) {
    $title  = htmlentities($c->title, null, 'UTF-8');
    $data   = doFilter(htmlentities($c->data, null, 'UTF-8'), $c->filter);

    $anax['main'] .= <<<EOD
<section>
  <article>
  <header>
  <h1><a href='blog.php?slug={$c->slug}'>{$title}</a></h1>
  </header>

  {$data}

  <footer>
  </footer
  </article>
</section>
EOD;
  }
}
else if($slug) {
  $anax['main'] = "Det fanns inte en sådan bloggpost.";
}
else {
  $anax['main'] = "Det fanns inga bloggposter.";
}
```

Låt oss nu kika lite på hur filtreringen fungerar.


Filtrering av texten {#filter}
-------------------------------

I mitt exempel har jag fyra olika textfilter.

| Filter     | Funktion         |
|------------|------------------|
| nl2br      | Ersätt varje nyrad med en `<br>` för att ge enkel formattering av texten. PHP funktionen `nl2br()` utför jobbet. |
| bbcode     | Formattering enligt bbcode är vanligt på forum och i kommentarsfält där användaren inte skall kunna skriva HTML-kod. En funktion med reguljära uttryck löser detta. |
| link       | Gör länkar som skrivs i texten klickbara. Ett reguljärt uttryck gör jobbet. |
| markdown   | Formattering enligt Markdown. Ett externt libb hjälper oss med Markdown. |

Coachen har skrivit en del artiklar om filtrering av text, bland annat ["Reguljära uttryck i PHP ger BBCode formattering"](coachen/reguljara-uttryck-i-php-ger-bbcode-formattering), ["Låt PHP-funktion make_clickable() automatiskt skapa klickbara länkar"](coachen/lat-php-funktion-make-clickable-automatiskt-skapa-klickbara-lankar) samt ["Skriv för webben med Markdown och formattera till HTML med PHP"](coachen/skriv-for-webben-med-markdown-och-formattera-till-html-med-php). Dessa hjälper mig att göra en filterhantering till mitt innehåll.

Varje innehåll listar de filter som skall användas för att processa sidan, det är en komma-separerad lista av filter. Till exempel så här.

> `bbcode,nl2br`

Ovanstående innebär att texten först filtreras enligt filtret för *bbcode* och därefter med filtret för *nl2br*.

Varje filter finns i en funktion och det finns en funktion `doFilter()` som har koll på vilka filter som finns och mappar dem mot en funktion som gör själva "filterjobbet". 

**Funktionen `doFilter()`.**

```php
/**
 * Call each filter.
 *
 * @param string $text the text to filter.
 * @param string $filter as comma separated list of filter.
 * @return string the formatted text.
 */
function doFilter($text, $filter) {
  // Define all valid filters with their callback function.
  $valid = array(
    'bbcode'   => 'bbcode2html',
    'link'     => 'make_clickable',
    'markdown' => 'markdown',
    'nl2br'    => 'nl2br',  
  );

  // Make an array of the comma separated string $filter
  $filters = preg_replace('/\s/', '', explode(',', $filter));

  // For each filter, call its function with the $text as parameter.
  foreach($filters as $func) {
    if(isset($valid[$func])) {
      $text = $valid[$func]($text);
    } 
    else {
      throw new Exception("The filter '$filter' is not a valid filter string.");
    }
  }

  return $text;
}
```

Källkoden för samtliga filter finns i [`filter.php`](kod-exempel/lagra-webbsidans-innehall-i-databasen/source.php?path=filter.php#file).



Avslutningsvis {#avslutning}
------------------------------

All [källkod](kod-exempel/lagra-webbsidans-innehall-i-databasen/source.php) och [ett körbart exempel](kod-exempel/lagra-webbsidans-innehall-i-databasen/view.php) finns så att du kan testa hur det fungerar i verkligheten.

Om innehållet verkar trasigt så kan du [återställa det](kod-exempel/lagra-webbsidans-innehall-i-databasen/reset.php) innan du börjar att testa.

Detta är grunderna och visar hur du på ett enkelt sätt kan uppnå en flexibel hantering av innehåll i databasen.