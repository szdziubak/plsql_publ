### Zadania do Tematu 5: Zaawansowane typy danych

**Zadanie 1: Rekordy (`%ROWTYPE` i `TYPE ... RECORD`)**

Napisz anonimowy blok PL/SQL, który stworzy niestandardowy raport o pracowniku, łącząc dane z dwóch tabel (`employees` i `departments`) za pomocą zagnieżdżonych rekordów.

1. W sekcji `DECLARE` zdefiniuj dwa typy rekordów:
    * `t_department_info`, który będzie miał taką samą strukturę jak wiersz w tabeli `departments` (zdefiniuj typ ręcznie).
    * `t_employee_report`, który będzie zawierał następujące pola:
        * `employee_id` typu `employees.employee_id%TYPE`
        * `full_name` typu `VARCHAR2(100)`
        * `salary` typu `employees.salary%TYPE`
        * `department` typu `t_department_info` (rekord zagnieżdżony)
2. Zadeklaruj zmienną `r_report` typu `t_employee_report`.
3. W sekcji `BEGIN`:
    * Pobierz dane pracownika o `employee_id = 101` do odpowiednich pól rekordu `r_report` (użyj konkatenacji do stworzenia `full_name`).
    * Pobierz pełne informacje o jego departamencie do zagnieżdżonego rekordu `r_report.department`.
4. Wyświetl na ekranie raport w formacie:

```
Raport pracownika:
ID: [id_pracownika]
Imię i nazwisko: [imię nazwisko]
Pensja: [pensja]
Nazwa departamentu: [nazwa_departamentu]
```
nowa linia: ```|| CHR(13) || CHR(10) ||```

**Oczekiwany wynik:**

```
Raport pracownika:
ID: 145
Imię i nazwisko: John Russell
Pensja: 14000
Nazwa departamentu: Sales
```

**Zadanie 2: Kolekcje `VARRAY`**

Napisz blok PL/SQL, który użyje kolekcji `VARRAY` do przechowania listy trzech najważniejszych umiejętności dla nowego stanowiska pracy.

1. Zadeklaruj typ `t_skills_list` jako `VARRAY` mogący pomieścić maksymalnie 5 elementów typu `VARCHAR2(50)`.
2. Zadeklaruj zmienną `v_top_skills` tego typu.
3. W sekcji `BEGIN`:
    * Zainicjalizuj kolekcję `v_top_skills` za pomocą konstruktora, przypisując jej dwie początkowe umiejętności: `'SQL'` i `'PL/SQL'`.
    * Użyj metody `.EXTEND()`, aby dodać do kolekcji trzecią umiejętność: `'Analityczne myślenie'`.
4. Wyświetl na ekranie maksymalny rozmiar kolekcji (`.LIMIT`) oraz aktualną liczbę elementów (`.COUNT`).
5. Użyj pętli `FOR`, aby wyświetlić wszystkie umiejętności z kolekcji.

**Oczekiwany wynik:**

```
Maksymalna liczba umiejętności: 5
Aktualna liczba umiejętności: 3
Wymagane umiejętności:
1. SQL
2. PL/SQL
3. Analityczne myślenie
```

**Zadanie 3: Kolekcje `Nested Table`**

Napisz blok PL/SQL, który pobierze adresy e-mail wszystkich pracowników z departamentu o `department_id = 50` i przechowa je w tabeli zagnieżdżonej.

1. Zadeklaruj typ `t_email_list` jako `TABLE OF employees.email%TYPE`.
2. Zadeklaruj zmienną `v_emails` tego typu i **zainicjalizuj ją** jako pustą kolekcję.
3. W sekcji `BEGIN`:
    * Użyj pętli kursorowej `FOR` (z niejawnym kursorem), aby pobrać e-maile wszystkich pracowników z działu 50.
    * Wewnątrz pętli, dla każdego pobranego e-maila, rozszerzaj kolekcję `v_emails` o jeden element (`.EXTEND()`) i przypisuj do niego wartość.
4. Po zakończeniu pętli, użyj drugiej pętli `WHILE` do iteracji po kolekcji `v_emails` i wyświetl wszystkie zebrane adresy e-mail.

**Oczekiwany wynik (kolejność może się różnić):**

```
Zebrane adresy e-mail z działu 50:
TKING
CHUNG
SBELL
... (pozostałe adresy)
```

**Zadanie 4: Kolekcje `Associative Array`**

Napisz blok PL/SQL, który stworzy w pamięci "słownik" statusów pracowników, używając tablicy asocjacyjnej indeksowanej przez `PLS_INTEGER`.

1. Zadeklaruj typ `t_status_map` jako `TABLE OF VARCHAR2(50) INDEX BY PLS_INTEGER`.
2. Zadeklaruj zmienną `v_statuses` tego typu.
3. W sekcji `BEGIN`:
    * Wypełnij tablicę, przypisując wartości do kilku **niesekwencyjnych** indeksów, np.:
        * Indeks `100`: `'AKTYWNY'`
        * Indeks `105`: `'NA URLOPIE'`
        * Indeks `210`: `'ZAWIESZONY'`
4. Użyj pętli `WHILE`, metody `.FIRST()` oraz `.NEXT()` do iteracji po elementach tablicy i wyświetl każdą parę klucz-wartość w formacie `ID Statusu: [indeks] -> Opis: [wartość]`.

**Oczekiwany wynik:**

```
ID Statusu: 100 -> Opis: AKTYWNY
ID Statusu: 105 -> Opis: NA URLOPIE
ID Statusu: 210 -> Opis: ZAWIESZONY
```

[^1]: skrypt.txt

[^2]: https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/plsql-collections-and-records.html

[^3]: https://www.softwaretestinghelp.com/pl-sql-collections/

[^4]: https://www.tutorialspoint.com/plsql/plsql_arrays.htm

[^5]: https://www.oracletutorial.com/plsql-tutorial/plsql-nested-tables/

[^6]: https://www.oracletutorial.com/plsql-tutorial/pl-sql-associative-array/

[^7]: https://www.tutorialspoint.com/plsql/plsql_collections.htm

[^8]: https://dev.to/mrcaption49/plsql-collections-associative-arrays-nested-tables-varrays-3i6j

[^9]: https://docs.oracle.com/cd/A97630_01/appdev.920/a96624/05_colls.htm

[^10]: https://pages.di.unipi.it/ghelli/didattica/bdldoc/B19306_01/appdev.102/b14261/collections.htm

[^11]: https://www.w3resource.com/plsql-exercises/cursor/index.php

