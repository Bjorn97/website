---
author:
    - mos
category:
    - databas
    - sql
    - kurs dbjs
    - kurs oophp
    - kurs databas
revision:
    "2018-01-11": "(D, mos) Nytt stycke SHOW WARNINGS."
    "2018-01-09": "(C, mos) Genomgången inför kursen databas."
    "2017-04-25": "(B, mos) Nu även i kursen oophp, la till stycke om parametrar och variabler."
    "2017-03-06": "(A, mos) Första utgåvan inför kursen dbjs."
...
Lagrade procedurer i databas
==================================

[FIGURE src=/image/snapvt17/lagrad-procedur.png?w=c5&a=0,20,70,0 class="right"]

Ibland räcker det inte till med SQL utan man behöver någon form av ytterligare programmeringsmässig hantering av informationen. Säg att man vill flytta pengar från ett konto till ett annat, men bara om det finns tillräckligt mycket pengar på kontot.

Man kan naturligtvis koda detta i godtyckligt externt programmeringsspråk. Men kan det finnas en möjlighet att koda sådant direkt i databasen?

Lagrade procedurer kommer till vår hjälp.

<!--more-->



Förutsättning {#pre}
--------------------------------------

Artiklen bygger löst vidare på det exemplet som beskrevs i artikeln "[Transaktioner i databas](kunskap/transaktioner-i-databas)".

Exemplet visar hur du jobbar med lagrade procedurer i MySQL.

SQLite stödjer inte lagrade procedurer.

[SQL-koden som visas i exemplet](https://github.com/dbwebb-se/databas/blob/master/example/sql/stored_procedure.sql) finner du på GitHub eller i ditt kursrepo (databas, dbjs, oophp) under `example/sql/stored_procedure.sql`.



Att skriva små program i databasen {#prog}
--------------------------------------

Databasen MySQL stödjer något de kallar [compound statements i MySQL](https://dev.mysql.com/doc/refman/5.7/en/begin-end.html) som är det "programmeringsspråk" som används för att skriva lagrade procedurer, funktioner och triggers. Det är icke SQL-kod som kan användas tillsammans med ren SQL-kod.

Det ger oss en möjlighet att skriva små program i databasen.

Låt oss kika på hur en lagrad procedur kan se ut, skriven med både SQL och med compound statements.



Exampel {#exempel}
--------------------------------------

Vi tar samma exempel vi använde i "[Transaktioner i databas](kunskap/transaktioner-i-databas)". Adam och Eva skall flytta pengar mellan varandra. SQL-koden för exemplet ser ut så här.

```sql
--
-- Example transactions
-- 
DROP TABLE IF EXISTS account;
CREATE TABLE account
(
	`id` CHAR(4) PRIMARY KEY,
    `name` VARCHAR(8),
    `balance` DECIMAL(4, 2)
);

INSERT INTO account
VALUES
	("1111", "Adam", 10.0),
    ("2222", "Eva", 7.0)
;

SELECT * FROM account;
```

Sen är det själva flytten av pengarna, från ett konto till ett annat, som är omslutet av en transaktion.

Eva skall få 1.5 pengar av Adam.

```sql
--
-- Move the money, within a transaction
--
START TRANSACTION;

UPDATE account 
SET
	balance = balance + 1.5
WHERE
	id = "2222";

UPDATE account 
SET
	balance = balance - 1.5
WHERE
	id = "1111";
    
COMMIT;

SELECT * FROM account;
```

Vad kan en lagrad procedur göra för oss här?



En lagrad procedur för att flytta pengar {#sp}
--------------------------------------

Vi kan bara flytta pengar om det finns några pengar. Vi behöver alltså kontrollera om Adam har så mycket pengar på kontot som han nu är benägen att flytta till Eva.

Detta är inget vi direkt kan skriva i SQL, iallafall inte utan att skriva en mer komplex SQL-sats.

Istället gör vi en lagrad procedur som flyttar pengarna, förutsatt att de finns.



###CREATE PROCEDURE {#create}

För att skapa en lagrad procedur så omsluter vi dess kod på följande sätt, med en [`CREATE PROCEDURE`](https://dev.mysql.com/doc/refman/5.7/en/create-procedure.html).

```sql
--
-- Procedure moveMoney()
--
DROP PROCEDURE IF EXISTS moveMoney;

DELIMITER //

CREATE PROCEDURE moveMoney(
	fromaccount CHAR(4),
    toaccount CHAR(4),
    amount NUMERIC(4, 2)
)
    -- Here comes SQL and compund statements
//

DELIMITER ;
```

Proceduren tar tre parametrar som berättar från konto, till konto och summan som skall flyttas.

Koden ändrar *delimiter* för att koden inuti den lagrade proceduren inte skall krocka med det semikolon som avslutar själva proceduren. Det finns även [beskrivet i manualen](https://dev.mysql.com/doc/refman/5.7/en/stored-programs-defining.html) om varför man gör så här.

Låt oss göra en minimal procedur för att anropa den, som ett litet test.

```sql
CREATE PROCEDURE moveMoney(
	fromaccount CHAR(4),
    toaccount CHAR(4),
    amount NUMERIC(4, 2)
)
BEGIN
    SELECT fromaccount, toaccount, amount;
END
//
```

Då anropar vi proceduren.

```sql
CALL moveMoney("1111", "2222", 1.5);
```

Resultatet blir att parametrarna skrivs ut i SQL-satsen, som en form av resultat från proceduren. En enkel procedur kan alltså vara att samla en eller flera SELECT-satser och skriva ut dem.

Bra, då kan vi skapa och anropa en lagrad procedur, och även skicka parametrar till den.



###Procedur för moveMoney {#move}

Då plockar vi in koden som flyttar pengarna, in i proceduren. Det kan se ut så här.

```sql
CREATE PROCEDURE moveMoney(
	fromaccount CHAR(4),
    toaccount CHAR(4),
    amount NUMERIC(4, 2)
)
BEGIN
    START TRANSACTION;

    UPDATE account 
    SET
    	balance = balance + amount
    WHERE
    	id = toaccount;

    UPDATE account 
    SET
    	balance = balance - amount
    WHERE
    	id = fromaccount;
        
    COMMIT;

    SELECT * FROM account;
END
//
```

Nu kan jag anropa proceduren, om och om igen. Om jag kör den tillräckligt många gånger så kommer Eva att bli riktigt rik och Adam något fattig.

```sql
CALL moveMoney("1111", "2222", 1.5);
```

Att jag väljer att skriva ut behållningen i slutet med SELECT-satsen är bara för att det skall vara enklare att utveckla, det blir lite som en `console.log()` eller `echo`.

Så här långt har vi åstakommit en lagrad procedur som omsluter en större kodsekvens som jag troligen vill utföra många gånger. Det blir som ett API mot min databas. Om man vill flytta pengar mellan konton så är det rätta sättet att göra det via den lagrade proceduren, inte att skriva egen SQL-kod. Lagrade procedurer kan alltså vara ett sätt att bygga API mot databasen.

Men kom ihåg att lagrade procedurer och koden som skrivs inte är kompatibel mellan olika databasmotorer. Det kan vara en nackdel, eller inte.



Kolla om pengar finns {#kolla}
--------------------------------------

Då skall vi se om vi kan uppdatera den lagrade proceduren för att kontrollera att det verkligen finns pengar på kontot, innan flyttan av pengar utförs.

Det första jag vill ha är en lokal variabel som jag tänker fylla med nuvarande balans på kontot. Om balansen inte är tillräcklig så kommer jag att avbryta transaktionen med en ROLLBACK.



###Lokal variabel {#lokal}

Låt oss börja kika på den lokala variabeln `currentBalance` och hur den får sitt värde.

```sql
BEGIN
	DECLARE currentBalance NUMERIC(4, 2);
    
    START TRANSACTION;

	SET currentBalance = (SELECT balance FROM account WHERE id = fromaccount);
    SELECT currentBalance;

    -- Some code omitted
```

Notera att den får sitt värde inuti transaktionen, för att även den delen skall kunna dra nytta av transaktionens atomära princip.



###IF-sats {#if}

Då kan vi skapa en if-sats som kontrollerar om nuvarande balansen är tillräcklig för att flytta pengarna.

```sql
IF currentBalance - amount < 0 THEN
    ROLLBACK;
    SELECT "Amount on the account is not enough to make the transaction.";

ELSE

    UPDATE account 
    SET
        balance = balance + amount
    WHERE
        id = toaccount;

    UPDATE account 
    SET
        balance = balance - amount
    WHERE
        id = fromaccount;
        
    COMMIT;

END IF;
```

Jag valde att omsluta koden i IF-satsen, det finns nämligen ingen `RETURN` i en lagrad procedur, vilket hade varit ett alternativ när man väl förstod att transaktionen inte kunde utföras.



IN och UT parametrar {#inout}
--------------------------------------

En lagrad procedur kan ta IN, OUT och INOUT parametrar. Här är ett exempel på en lagrad procedur som sätter ett värde och som anroparen kan lagra i en variabel utanför den lagrade proceduren.

```sql
--
-- Try OUT variables from SP
--
DROP PROCEDURE IF EXISTS getMoney;

DELIMITER //

CREATE PROCEDURE getMoney(
	IN account CHAR(4),
    OUT total NUMERIC(4, 2)
)
BEGIN
	SELECT balance INTO total FROM account WHERE id = account;
END
//

DELIMITER ;

CALL getMoney("1111", @sum);
SELECT @sum;
```

Proceduren tar två argument, det ena är IN och det andra är OUT.

I SELECT-satsen hämtas ett värde från databasen och lagras i variabeln.

När anropet sker med CALL så bifogas en variabel som efter anropet kan läsas av och användas vidare.

En lagrad procedur kan även ta ett argument som är INOUT.

Vill du skriva en SELECT-sats som sätter flera variabler på en gång så skriver du så här.

```sql
SELECT id, balance INTO idx, total FROM account WHERE id = account;
```

Ovan förutsätts att `idx` och `total` är argument, eller lokala variabler i den lagrade proceduren.

Samma princip kan man använda även utanför en lagrad procedur, om man vill hämta värden och för tillfället spara undan i lokala variabler.

```sql
--
-- Select mutiple into variables
--
SELECT 1, 2 INTO @a, @b;
SELECT @a, @b;
```

Med sådan taktik kan man göra "lite mer" i en vanlig SQL-fil, vilket kan vara behändigt när databasen växer.



SHOW WARNINGS {#warnings}
--------------------------------------

Ibland kan man se att man får en varning av ett anrop till en lagrad procedur. I följande exempel anger jag ett värde på en variabeln som är utanför variabelens range.

```sql
mysql> CALL editAccount("1337", "Mega", 4200000);
Query OK, 1 row affected, 1 warning (0.00 sec)
```

För att se vad varningen säger så kan jag läsa av den.

```sql
mysql> SHOW WARNINGS;
+---------+------+---------------------------------------------------+
| Level   | Code | Message                                           |
+---------+------+---------------------------------------------------+
| Warning | 1264 | Out of range value for column 'aBalance' at row 1 |
+---------+------+---------------------------------------------------+
1 row in set (0.00 sec)
```

Det kan vara bra att ha koll på detta, det är en felkälla och ett bra verktyg för felsökning och utveckling.



SHOW PROCEDURE {#show}
--------------------------------------

När man vill se vilka procedurer som finns i databasen så kan man visa dem.

```sql
SHOW PROCEDURE STATUS;
```

Svaret blir en lista med alla de procedurer som finns och till vilken databas de är kopplade.

Vill du sedan titta på koden som ligger bakom den lagrade proceduren så frågar du efter den.

```sql
SHOW CREATE PROCEDURE moveMoney;
```

Fram kommer källkoden för proceduren.



Avslutningsvis {#avslutning}
--------------------------------------

Detta var grunderna i hur du kan jobba med lagrade procedurer som ett sätt att programmera i en databas. Kanske kan detta även vara ett sätt att bygga ett API mot en databas.

Vill du se fler sätt att skapa API:er för databasen, och utföra programmering inuti databasen, så läser du om "[Triggers i databas](kunskap/triggers-i-databas)" som är en fristående fortsättning på denna artikeln.

Har du [tips, förslag eller frågor om artikeln](t/6292) så finns det en specifik forumtråd för det.
