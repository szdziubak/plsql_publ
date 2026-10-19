### Zadania do Tematu 3: Współpraca SQL i PL/SQL

**Wymagania wstępne:**
Przed rozpoczęciem zadań, utwórz kopie robocze tabel, aby nie modyfikować oryginalnych danych. Wykonaj poniższe polecenia:

```sql
CREATE TABLE employees_copy AS SELECT * FROM employees;
CREATE TABLE departments_copy AS SELECT * FROM departments;
```


---

**Zadanie 1: Pobieranie danych do zmiennych (`SELECT ... INTO`)**

Napisz anonimowy blok PL/SQL, który pobierze imię, nazwisko oraz datę zatrudnienia pracownika o `employee_id = 102`.

1. Zadeklaruj trzy oddzielne zmienne, aby przechować pobrane dane. Użyj atrybutu `%TYPE` w celu zapewnienia zgodności typów danych z kolumnami w tabeli `employees`.
2. W sekcji wykonawczej użyj instrukcji `SELECT ... INTO`, aby pobrać wymagane dane do zadeklarowanych zmiennych.
3. Wyświetl na ekranie komunikat w formacie: `Pracownik [imię] [nazwisko] został zatrudniony dnia [data]`.

**Oczekiwany wynik:**

```
Pracownik Lex De Haan został zatrudniony dnia 13-JAN-93
```

**Zadanie 2: Pobieranie całego wiersza do rekordu (`%ROWTYPE`)**

Napisz anonimowy blok PL/SQL, który wykorzysta rekord do pobrania wszystkich informacji o departamencie o `department_id = 50`.

1. Zadeklaruj zmienną rekordową `r_department` o typie zakotwiczonym w tabeli `departments` za pomocą atrybutu `%ROWTYPE`.
2. Użyj polecenia `SELECT * INTO ...`, aby pobrać cały wiersz z tabeli `departments_copy` do zadeklarowanego rekordu.
3. Wyświetl na ekranie informacje o departamencie, odwołując się do pól rekordu: `Departament [nazwa] ma ID [id] i znajduje się w lokalizacji o ID [id_lokalizacji].`

**Oczekiwany wynik:**

```
Departament Shipping ma ID 50 i znajduje się w lokalizacji o ID 1500.
```

**Zadanie 3: Wstawianie nowego wiersza (`INSERT`)**

Napisz blok PL/SQL, który doda nowy departament o nazwie 'Human Resources' do tabeli `departments_copy`.

1. W sekcji `BEGIN` użyj instrukcji `INSERT`, aby dodać nowy wiersz. Wstaw następujące wartości:
    * `department_id`: 40
    * `department_name`: 'Human Resources'
    * `location_id`: 2400
2. Po instrukcji `INSERT` dodaj polecenie `COMMIT;`, aby trwale zapisać zmiany.
3. Wyświetl komunikat o powodzeniu operacji, np. `Dodano nowy departament.`.
4. Po wykonaniu bloku, zweryfikuj wynik za pomocą zapytania: `SELECT * FROM departments_copy WHERE department_id = 40;`.

**Zadanie 4: Aktualizacja całego wiersza za pomocą rekordu (`UPDATE SET ROW`)**

Napisz blok PL/SQL, który zaktualizuje dane pracownika o `employee_id = 110` w tabeli `employees_copy`, używając rekordu.

1. Zadeklaruj zmienną rekordową `r_emp` typu `employees_copy%ROWTYPE`.
2. Pobierz cały wiersz pracownika o ID 110 do rekordu `r_emp`.
3. Zmodyfikuj wartości w rekordzie:
    * `r_emp.salary` ustaw na `12000`.
    * `r_emp.job_id` ustaw na `'SA_MAN'`.
4. Użyj instrukcji `UPDATE ... SET ROW = r_emp`, aby zaktualizować cały wiersz w tabeli na podstawie zmodyfikowanego rekordu.
5. Zatwierdź transakcję za pomocą `COMMIT;` i wyświetl komunikat o sukcesie.

**Zadanie 5: Usuwanie wierszy (`DELETE`)**

Napisz blok PL/SQL, który usunie z tabeli `employees_copy` wszystkich pracowników z departamentu o `department_id = 20`.

1. W sekcji `BEGIN` użyj instrukcji `DELETE` z odpowiednim warunkiem `WHERE`.
2. Po wykonaniu operacji `DELETE`, użyj atrybutu `SQL%ROWCOUNT`, aby wyświetlić komunikat o liczbie usuniętych wierszy: `Usunięto [liczba] pracowników.`
3. Wycofaj zmiany za pomocą `ROLLBACK;`.

**Oczekiwany wynik:**

```
Usunięto 2 pracowników.
```

**Zadanie 6: Sekwencje - użycie `NEXTVAL` w instrukcji `INSERT`**

Napisz blok PL/SQL, który doda dwa nowe departamenty, używając sekwencji do automatycznego generowania ich ID.

1. Najpierw utwórz sekwencję:

```sql
CREATE SEQUENCE dept_copy_seq START WITH 300 INCREMENT BY 10;
```

2. W bloku PL/SQL, w pętli `FOR` wykonującej się dwa razy, wstaw do tabeli `departments_copy` nowe departamenty: 'Research' i 'Development'. Użyj `dept_copy_seq.NEXTVAL` bezpośrednio w klauzuli `VALUES` instrukcji `INSERT`, aby nadać im unikalne `department_id`.
3. Zatwierdź zmiany (`COMMIT;`) i zweryfikuj wynik, wyświetlając wszystkie departamenty o `department_id >= 300`.

**Zadanie 7: Sekwencje - użycie `NEXTVAL` i `CURRVAL`**

Napisz blok PL/SQL, który pobierze kolejny numer ID z sekwencji, użyje go do wstawienia rekordu, a następnie odwoła się do tej samej wartości za pomocą `CURRVAL` w celu utworzenia wpisu w tabeli z logami.

1. Najpierw utwórz tabelę na logi:

```sql
CREATE TABLE action_logs (log_id NUMBER, log_desc VARCHAR2(100), log_date DATE);
```

2. W bloku PL/SQL:
    * Pobierz następną wartość z sekwencji `dept_copy_seq` do zmiennej `v_new_id`.
    * Wstaw nowy departament 'Logistics' do tabeli `departments_copy`, używając wartości ze zmiennej `v_new_id` jako klucza głównego.
    * Wstaw nowy wiersz do tabeli `action_logs`, gdzie:
        * `log_id` będzie pobrane z `dept_copy_seq.CURRVAL`.
        * `log_desc` będzie tekstem 'Dodano nowy departament'.
        * `log_date` będzie aktualną datą.
3. Zatwierdź zmiany i zweryfikuj zawartość obu tabel.


[^1]: skrypt.txt

[^2]: https://www.w3resource.com/plsql-exercises/

[^3]: https://www.javainhand.com/2025/01/rowtype-in-oracle-plsql-with-example.html

[^4]: https://docs.oracle.com/en/database/other-databases/timesten/22.1/plsql-developer/example-using-insert-statement.html

[^5]: https://stackoverflow.com/questions/39221349/updating-a-table-with-record-type-object

[^6]: https://www.w3resource.com/sql-exercises/sql-data-manipulation-language-exercises_index.php

[^7]: https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-SEQUENCE.html

[^8]: https://www.red-gate.com/simple-talk/databases/oracle-databases/typical-uses-oracle-sequences/

[^9]: https://docs.oracle.com/en/database/other-databases/timesten/22.1/plsql-developer/examples-using-select-statement-pl-sql.html

[^10]: https://www.w3resource.com/plsql-exercises/cursor/plsql-cursor-exercise-7.php

[^11]: https://www.red-gate.com/simple-talk/databases/oracle-databases/a-guide-to-insert-update-and-delete-statements-in-oracle/

[^12]: https://www.w3resource.com/plsql-exercises/cursor/plsql-cursor-exercise-2.php

[^13]: https://www.oracletutorial.com/plsql-tutorial/plsql-select-into/

[^14]: https://blog.toadworld.com/oracle-create-sequence-explained-with-examples

[^15]: https://docs.oracle.com/cd/B12037_01/appdev.101/b10807/13_elems045.htm

[^16]: https://www.plsql.co/sequence.html

[^17]: https://www.tricentis.com/learn/how-to-insert-data-in-oracle-sql

[^18]: https://stackoverflow.com/questions/62640141/pl-sql-insert-from-select

[^19]: https://stackoverflow.com/questions/10210273/how-to-retrieve-the-current-value-of-an-oracle-sequence-without-increment-it

[^20]: https://www.w3schools.com/sql/sql_exercises.asp

[^21]: https://docs.oracle.com/cd/B14117_01/appdev.101/b10807/13_elems045.htm

[^22]: https://piembsystech.com/rowtype-in-pl-sql/

[^23]: https://amysimpsongrange.com/2022/12/12/oracle-pl-sql-select-into/

[^24]: https://www.w3schools.com/sql/sql_insert_into_select.asp

[^25]: https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/INSERT.html

[^26]: https://docs.tibco.com/pub/as/4.10.0/doc/html/Concepts/Modifying_Rows_in_a_Table_Using_the_SQL_UPDATE_Statement.htm?TocPath=Concepts%7CProgramming+with+ActiveSpaces%7CModifying+Data+in+a+Table%7CThe+SQL+UPDATE%C2%A0Statement%7C_____1

[^27]: https://www.ibm.com/docs/en/db2-for-zos/12.0.0?topic=data-delete-statements

[^28]: https://stackoverflow.com/questions/12964703/oracle-pl-sql-sequence

[^29]: https://www.youtube.com/watch?v=8Hoik_F3h8U

[^30]: https://database.guide/how-nextval-works-in-postgresql/

[^31]: https://www.oracle-developer.net/display.php?id=507

