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
    "2018-01-05": "(C, mos) Genomgången och nu även i kursen databas."
    "2017-04-25": "(B, mos) Nu även i kursen oophp."
    "2017-03-06": "(A, mos) Första utgåvan inför kursen dbjs."
...
Transaktioner i databas
==================================

[FIGURE src=/image/snapvt17/sqlite-transaction.png?w=c5&a=0,50,50,0 class="right"]

En transaktion i en databas omsluter flera händelser och gör dem atomära, antingen utförs samtliga händelser eller ingen.

Säg till exempel att man vill flytta pengar från en ägare till en annan och informationen om detta finns i en databas. Först plockar man bort pengen från den ena och sedan lägger man till pengen till den andre. Flytten som består av två SQL-satser måste utföras i sin helhet, eller inte alls. Här kommer en transaktion till hjälp.

<!--more-->



Förutsättning {#pre}
--------------------------------------

Exemplet visar hur du jobbar med transaktioner i MySQL. Ett avslutande stycke visar hur samma kod ser ut i SQLite.

Du kan läsa om [transaktioner i manualen för MySQL](https://dev.mysql.com/doc/refman/5.7/en/sql-syntax-transactions.html).

Du kan även läsa om [transaktioner i manualen för SQLite](https://www.sqlite.org/transactional.html).

[SQL-koden som visas i exemplet](https://github.com/dbwebb-se/database/blob/master/example/sql/transaction.sql) finner du på GitHub eller i ditt kursrepo (databas, dbjs, oophp) under `example/sql/transaction.sql`.



ACID {#acid}
--------------------------------------

ACID är en samling properties för databastransaktioner. De beskriver hur en transaktion måste betee sig. 

| Property        | Beskrivning |
|-----------------|-------------|
| A - Atomicity   | En transaktion utförs som en enhet, eller inte alls, allt eller inget. Den är odelbar. |
| C - Consistency | Transaktionen flyttar databasen från ett giltigt läge till ett annat och alla regler såsom *constraints*, *cascades* och triggers skall vara giltiga. |
| I - Isolation   | Även om flera transaktioner exekveras samtidigt så skall resultatet från transaktionerna betraktas som om de exekverades sekventiellt. En transaktion kan inte se effekter av en delvis exekverad transaktion. |
| D - Durability  | När en transaktion är *committad* så skall den vara beständig, även om strömmen går och databasmotorn krashar. |

En transaktion skall alltså vara atomär (A), bevara konsistensen (C) i databasen, vara isolerad (I) från övriga transaktioner och vara beständig (D) när den är committad. Därav ACID.



En testdatabas {#dbexempel}
--------------------------------------

Om du inte har tillgång till en databas där du kan testa exempelkoden så föreslår jag att du skapar en ny databas som du kan använda för denna (och kommande) artikel.

Här är SQL-koden som hjälper dig att skapa databasen `dbwebb` med användaren `user` som har lösenordet `pass`. Koden ligger i ditt kursrepo under `example/sql/setup.sql`.

```sql
--
-- Create a sample database useful for testing.
--
CREATE DATABASE IF NOT EXISTS dbwebb;
GRANT ALL ON dbwebb.* TO user@localhos IDENTIFIED BY 'pass';

USE dbwebb;
SHOW DATABASES LIKE 'dbwebb';
SHOW TABLES;
```

Du behöver köra SQL-koden som en användare som har rättigheter att skapa databas och göra grants på användare. Kör det med din root-användare så här.

```text
mysql -uroot -p < example/sql/setup.sql
```

Fint, då har du en databas att leka med.



Transaktion, ett exempel {#exempel}
--------------------------------------

Låt oss ta en bitcoin bank där Adam och Eva skall flytta en mängd bitcoins mellan varandra.



###En tabell {#tabell}

Först skapar vi en tabell med innehåll.

```sql
--
-- Example transactions
-- 
DROP TABLE IF EXISTS Account;
CREATE TABLE Account
(
	`id` CHAR(4) PRIMARY KEY,
    `name` VARCHAR(8),
    `balance` DECIMAL(4, 2)
);

INSERT INTO Account
VALUES
	("1111", "Adam", 10.0),
    ("2222", "Eva", 7.0)
;

SELECT * FROM Account;
```

Bra, då har vi en tabell att ugå ifrån. Adam har 10 bitcoins och Eva har 7.



###Flytta pengar {#flytta}

Låt oss säga att Adam och Eva har slagit vad i Melodifestivalen och Adam förlorade och skall skicka 1.5 bitcoins till Eva.

SQL-koden för den flytten ser ut så här.

```sql
--
-- Move the money
--
UPDATE Account 
SET
	balance = balance + 1.5
WHERE
	id = "2222";

UPDATE Account 
SET
	balance = balance - 1.5
WHERE
	id = "1111";
    
SELECT * FROM Account;
```

I den första satsen lägger vi till pengen till Evas konto, i den andra satsen så tar vi bort pengen från Admas konto.

I mellanläget innehåller databasen mer pengar än det egentligen finns. Det är när pengarna fyllts på Evas konto men ännu inte dragits från Adams konto. I detta läget är databasen inte konsistent.

Det hjälper inte om vi byer ordning på satserna och skapar ett underskott istället. Databasen är lika mycket ur balans och ej konsistent i båda fallen.

Till vår hjälp kommer transaktionen.



###Flytta med transaktion {#flyttrans}

Vi lägger flytten av pengarna inom ramen för en transaktion, så att flytten av pengar blir atomär och databasen behåller sin konsistens oavsett vad som händer.

```sql
--
-- Move the money, within a transaction
--
START TRANSACTION;

UPDATE Account 
SET
	balance = balance + 1.5
WHERE
	id = "2222";

UPDATE Account 
SET
	balance = balance - 1.5
WHERE
	id = "1111";
    
COMMIT;

SELECT * FROM Account;
```

Det är alltså `START TRANSACTION` som påbörjar transaktionen och det är `COMMIT` som avslutar den. Alla skeenden som utförs inom transaktionen är nu atomära.

Om du gör en `ROLLBACK` istället för en `COMMIT` så görs en undo på hela sekvensen som utfördes inom transaktionen. Det blir alltså att COMMIT utför alla ändringar och ROLLBACK tar bort effekten av allt som utförts i transaktionen.

En transaktion avslutas med en COMMIT, eller avbryts med en ROLLBACK.



Transaktioner i SQLite {#sqlite}
--------------------------------------

Transaktioner i SQLite fungerar på samma sätt. Enda skillnaden är SQL-koden där transaktionens centrala delar ser ut så här.

```sql
BEGIN TRANSACTION;
--
COMMIT;
ROLLBACK;
```

Du kan se syntaxen för [transaktioner i SQLite manualen](https://www.sqlite.org/lang_transaction.html).



Avslutningsvis {#avslutning}
--------------------------------------

Detta var grunderna i databastransaktioner. Så fort du gör flera satser som behöver samtliga, eller inte alls, så kan transaktioner vara verktyget.

Vill du se hur du kan kontrollera att det finns pengar på kontot, innan du gör överföringen, så läser du vidare i artikeln "[Lagrade procedurer i databas](kunskap/lagrade-procedurer-i-databas)". Det är ju rimligt att man kontrollerar att det finns pengar att flytta, innan man gör flytten. Men det tar vi alltså i en annan artikel.

Har du [tips, förslag eller frågor om artikeln](t/6291) så finns det en specifik forumtråd för det.
