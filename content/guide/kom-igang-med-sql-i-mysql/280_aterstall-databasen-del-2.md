---
author: mos
revision:
    "2018-03-27": "(B, mos) Add function to bash reset."
    "2018-02-09": "(A, mos) Tillagd för att fokusera på hur man återställer databasen efter andra delen."
...
Återställ databasen (del 2)
==================================

Vi skall återställa databasen till det läget som gäller efter denna delen av guiden.

Spara din kod i filen `reset_part2.bash`.

Vi vill återställa databasen genom att köra ett kommando, så här.

```text
$ bash reset_part2.bash
```



Vilka filer behövs köras? {#filer}
----------------------------------

Vi utgår från [filen som återskapar databasen efter första delen av guiden](./../aterstall-databasen-del-1), `reset_part1.bash`. Jag tar en kopia av den och modifierar så att den fungerar för både del 1 del och del 2 av guiden.

Här är filerna som behövs, och ordningen de körs i. Ordningen är viktig. Man behöver ha koll på sin datamängd.

| Fil               | Vad gör den?         |
|-------------------|----------------------|
| `setup.sql`       | Kör som root för att skapa om databasen och skapa användaren user:pass. |
| `ddl.sql`         | Skapa tabellen för lärare. |
| `dml_insert.sql`  | Lägg till rader i tabellen lärare. | 
| `ddl_migrate.sql` | Alter table lärare och lägg till kompetensen. |
| `dml_update.sql`  | Förbered lönerevisionen, alla lärare har grundlön. |
| `dml_copy.sql`    | Kopiera till larare_pre innan lönerevisionen. |
| `dml_update_lonerevision.sql`  | Utför lönerevisionen. |
| `dml_view.sql`    | Skapa vyerna VNamnAlder och Vlarare. |

Därefter kan vi testa datamängden, till exempel genom att dubbelkolla lönesumman och kompetensen i tabellerna larare och larare_pre.

Låt oss samla allt i ett bash-skript.



Komplett skript för att återställa databasen del 2 {#reset}
----------------------------------

Följande bash-skript innehåller allt som återställer databasen till och med del 2 i guiden.

```text
#!/usr/bin/env bash

#
# Load a SQL file into skolan using user:pass
#
function loadSqlIntoSkolan
{
    echo ">>> $2 ($1)"
    mysql -uuser -ppass skolan < $1 > /dev/null
}

#
# Recreate and reset the database to be as after part II.
#
echo ">>> Reset skolan to after part 2"
echo ">>> Recreate the database (as root)"
mysql -uroot -p skolan < setup.sql > /dev/null

loadSqlIntoSkolan "ddl.sql"         "Create tables"
loadSqlIntoSkolan "dml_insert.sql"  "Insert into larare"
loadSqlIntoSkolan "ddl_migrate.sql" "Alter table larare"
loadSqlIntoSkolan "dml_update.sql"  "Grundlön larare"
loadSqlIntoSkolan "ddl_copy.sql"    "Kopiera till larare_pre"
loadSqlIntoSkolan "dml_update_lonerevision.sql" "Utför lönerevision"
loadSqlIntoSkolan "dml_view.sql"    "Skapa vyer VNamnAlder och Vlarare"

echo ">>> Check larare_pre: Lönesumman = 305000, Kompetens = 8."
mysql -uuser -ppass skolan -e "SELECT SUM(lon) AS 'Lönesumma', SUM(kompetens) AS Kompetens FROM larare_pre;"

echo ">>> Check larare: Lönesumman = 330242, Kompetens = 19."
mysql -uuser -ppass skolan -e "SELECT SUM(lon) AS 'Lönesumma', SUM(kompetens) AS Kompetens FROM larare;"
```

När jag kör mitt program får jag följande utskrift, du bör få motsvarande.

```text
$ bash reset_part2.bash
>>> Reset skolan to after part 2
>>> Recreate the database (as root)
Enter password:
>>> Create tables (ddl.sql)
>>> Insert into larare (dml_insert.sql)
>>> Alter table larare (ddl_migrate.sql)
>>> Förbered lönerevision, grundlön larare (dml_update.sql)
>>> Kopiera till larare_pre (ddl_copy.sql)
>>> Utför lönerevision (dml_update_lonerevision.sql)
>>> Skapa vyer VNamnAlder och Vlarare (dml_view.sql)
>>> Check larare_pre: Lönesumman = 305000, Kompetens = 8.
+------------+-----------+
| Lönesumma  | Kompetens |
+------------+-----------+
|     305000 |         8 |
+------------+-----------+
>>> Check larare: Lönesumman = 330242, Kompetens = 19.
Warning: Using a password on the command line interface can be
+------------+-----------+
| Lönesumma  | Kompetens |
+------------+-----------+
|     330242 |        19 |
+------------+-----------+
```

Du får gärna modifiera och vidarutveckla ditt bash-skript. Men det räcker om det fungerar som jag visar ovan och återställer databasen så här långt i guiden.
