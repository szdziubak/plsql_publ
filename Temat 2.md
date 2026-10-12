# Temat 2. Instrukcje sterowania i pętle. IF, LOOP, WHILE, FOR, Loop Labeling oraz Continue i GOTO

Instrukcje sterowania przepływem pozwalają na kontrolowanie kolejności wykonywania poleceń w bloku PL/SQL. Umożliwiają tworzenie logiki warunkowej oraz powtarzanie operacji w pętlach.

### Instrukcje warunkowe

Instrukcje warunkowe wykonują różne sekwencje poleceń w zależności od tego, czy określone warunki są spełnione.

**Instrukcja `IF`**
Jest to podstawowa instrukcja warunkowa w PL/SQL. Sprawdza prawdziwość warunku i w zależności od wyniku wykonuje odpowiedni fragment kodu.

**Składnia:**

```sql
IF warunek1 THEN
   -- Instrukcje do wykonania, jeśli warunek1 jest prawdziwy
ELSIF warunek2 THEN
   -- Instrukcje do wykonania, jeśli warunek2 jest prawdziwy
ELSE
   -- Instrukcje do wykonania, jeśli żaden z powyższych warunków nie jest prawdziwy
END IF;
```

* `ELSIF` i `ELSE` są opcjonalne. Można użyć wielu klauzul `ELSIF`, ale tylko jedną `ELSE`.
* Instrukcje `IF` mogą być zagnieżdżane.

**Przykład użycia `IF-ELSIF-ELSE`:**

```sql
DECLARE
    v_number NUMBER := 5;
    v_name   VARCHAR2(50) := 'Krzysztof';
BEGIN
    IF v_number < 10 AND v_name = 'Krzysztof' THEN
        DBMS_OUTPUT.PUT_LINE('Liczba jest mniejsza niż 10, a imię to Krzysztof.');
    ELSIF v_number < 20 THEN
        DBMS_OUTPUT.PUT_LINE('Liczba jest mniejsza niż 20.');
    ELSIF v_number < 30 THEN
        DBMS_OUTPUT.PUT_LINE('Liczba jest mniejsza niż 30.');
    ELSE
        -- Przykład zagnieżdżonego IF
        IF v_number IS NULL THEN 
            DBMS_OUTPUT.PUT_LINE('Liczba ma wartość NULL.');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Liczba jest większa lub równa 30.');
        END IF;
    END IF;
END;
/
```

### Instrukcje `CASE`

Instrukcje `CASE` w PL/SQL pozwalają na wybór jednej z wielu sekwencji instrukcji do wykonania w oparciu o selektor (wyrażenie) lub serię warunków logicznych[^2][^3][^4]. Stanowią alternatywę dla rozbudowanych konstrukcji `IF-ELSIF-ELSE` i często poprawiają czytelność kodu[^5]. W PL/SQL istnieją dwa typy instrukcji `CASE`: `CASE Expression` i `CASE Statement`[^1][^6].

#### `CASE Expression` (Wyrażenie CASE)

`CASE Expression` jest używane do zwrócenia pojedynczej wartości na podstawie serii warunków[^7][^5]. Działa podobnie do funkcji `DECODE`, ale jest bardziej elastyczne i czytelne[^5].

**Typy `CASE Expression`:**

1. **Simple `CASE Expression` (Proste wyrażenie CASE)**: Ocenia pojedyncze wyrażenie (selektor) i porównuje je z szeregiem potencjalnych wartości[^1][^7][^5]. Jeśli znajdzie dopasowanie, zwraca przypisaną wartość[^7].
**Składnia:**

```sql
CASE selektor
    WHEN wartość1 THEN wynik1
    WHEN wartość2 THEN wynik2
    ...
    [ELSE wynik_domyślny]
END;
```

    * `selektor`: Wyrażenie, które jest oceniane tylko raz[^3].
    * `WHEN wartość THEN wynik`: Jeśli `selektor` jest równy `wartość`, zwracany jest `wynik`[^3].
    * `ELSE`: Opcjonalna klauzula. Jeśli żadna z wartości w `WHEN` nie pasuje do selektora, zwracany jest `wynik_domyślny`[^2][^7]. Jeśli `ELSE` zostanie pominięte i nie znajdzie się dopasowania, wyrażenie zwróci `NULL`[^2][^7].
2. **Searched `CASE Expression` (Wyszukiwane wyrażenie CASE)**: Ocenia wiele wyrażeń logicznych (`BOOLEAN expressions`) i zwraca wartość dla pierwszego warunku, który jest prawdziwy[^1][^7]. Nie ma selektora.
**Składnia:**

```sql
CASE
    WHEN warunek1 THEN wynik1
    WHEN warunek2 THEN wynik2
    ...
    [ELSE wynik_domyślny]
END;
```

    * `WHEN warunek THEN wynik`: Jeśli `warunek` jest prawdziwy, zwracany jest `wynik`[^7].
    * `ELSE`: Opcjonalna klauzula. Jeśli żaden z warunków nie jest prawdziwy, zwracany jest `wynik_domyślny`[^2]. Jeśli `ELSE` zostanie pominięte i żaden warunek nie jest prawdziwy, wyrażenie zwróci `NULL`[^2][^7].

**Kluczowe cechy `CASE Expression`:**

* **Krótkie spięcie (Short-circuit evaluation)**: Oracle ocenia warunki `WHEN` od lewej do prawej i zatrzymuje się, gdy tylko znajdzie pierwszy prawdziwy warunek lub dopasowanie. Pozostałe warunki nie są oceniane[^7].
* **Wartości `NULL`**: Jeśli `ELSE` jest pominięte i żaden warunek `WHEN` nie jest spełniony, zwracane jest `NULL`[^2][^7].
* Może być używane w zapytaniach SQL (w `SELECT`, `WHERE`, `ORDER BY` itp.)[^8].

**Przykład `CASE Expression` (Simple):**

```sql
SET SERVEROUTPUT ON;
DECLARE
    v_job_code VARCHAR2(30) := 'SA_MAN';
    v_salary_increase NUMBER;
BEGIN
    v_salary_increase := CASE v_job_code
        WHEN 'SA_MAN' THEN 0.2
        WHEN 'SA_REP' THEN 0.3
        ELSE 0
    END;
    DBMS_OUTPUT.PUT_LINE('Twoja podwyżka to: ' || v_salary_increase);
END;
/
```

**Przykład `CASE Expression` (Searched):**

```sql
DECLARE
    v_score NUMBER := 85;
    v_grade CHAR(1);
BEGIN
    v_grade := CASE
        WHEN v_score >= 90 THEN 'A'
        WHEN v_score >= 80 THEN 'B'
        WHEN v_score >= 70 THEN 'C'
        WHEN v_score >= 60 THEN 'D'
        ELSE 'F'
    END;
    DBMS_OUTPUT.PUT_LINE('Ocena studenta to: ' || v_grade);
END;
/
```


#### `CASE Statement` (Instrukcja CASE)

`CASE Statement` jest instrukcją sterującą przepływem, podobną do `IF-THEN-ELSIF`, która wykonuje sekwencję instrukcji (akcji) w zależności od spełnionych warunków[^1][^5][^8]. Nie zwraca wartości, a jedynie kontroluje wykonanie kodu[^1][^5].

**Typy `CASE Statement`:**

1. **Simple `CASE Statement` (Prosta instrukcja CASE)**: Ocenia pojedyncze wyrażenie (selektor) i porównuje je z wartościami z klauzul `WHEN`.
**Składnia:**

```sql
CASE selektor
    WHEN wartość1 THEN instrukcje1;
    WHEN wartość2 THEN instrukcje2;
    ...
    [ELSE instrukcje_domyślne;]
END CASE; -- Zauważ END CASE zamiast samego END
```

    * `selektor`: Wyrażenie oceniane tylko raz[^3].
    * `WHEN wartość THEN instrukcje`: Jeśli `selektor` jest równy `wartość`, wykonywane są `instrukcje`[^3][^9].
    * `ELSE`: Opcjonalna klauzula. Jeśli żadna z wartości w `WHEN` nie pasuje do selektora, wykonywane są `instrukcje_domyślne`[^3]. Jeśli `ELSE` zostanie pominięte i nie znajdzie się dopasowania, PL/SQL zgłosi wyjątek `CASE_NOT_FOUND`[^2][^3].
2. **Searched `CASE Statement` (Wyszukiwana instrukcja CASE)**: Ocenia wiele wyrażeń logicznych (`BOOLEAN expressions`) i wykonuje instrukcje dla pierwszego warunku, który jest prawdziwy[^1][^9]. Nie ma selektora.
**Składnia:**

```sql
CASE
    WHEN warunek1 THEN instrukcje1;
    WHEN warunek2 THEN instrukcje2;
    ...
    [ELSE instrukcje_domyślne;]
END CASE;
```

    * `WHEN warunek THEN instrukcje`: Jeśli `warunek` jest prawdziwy, wykonywane są `instrukcje`[^9].
    * `ELSE`: Opcjonalna klauzula. Jeśli żaden z warunków nie jest prawdziwy, wykonywane są `instrukcje_domyślne`[^9]. Jeśli `ELSE` zostanie pominięte i żaden warunek nie jest prawdziwy, PL/SQL zgłosi wyjątek `CASE_NOT_FOUND`[^2].

**Kluczowe cechy `CASE Statement`:**

* **Brak wartości zwrotnej**: Nie zwraca wartości, a jedynie wykonuje instrukcje[^1].
* **Wymaga `END CASE`**: Instrukcja kończy się słowami kluczowymi `END CASE`, a nie tylko `END`[^1][^5].
* **Błąd `CASE_NOT_FOUND`**: Jeśli żadna klauzula `WHEN` nie zostanie spełniona i brakuje klauzuli `ELSE`, zostanie zgłoszony wyjątek `CASE_NOT_FOUND`[^2][^3].
* Wykonuje się tylko pierwsza pasująca klauzula `WHEN`; reszta jest ignorowana[^2].

**Przykład `CASE Statement` (Simple):**

```sql
DECLARE
    v_day_of_week VARCHAR2(10) := 'Niedziela';
BEGIN
    CASE v_day_of_week
        WHEN 'Poniedziałek' THEN DBMS_OUTPUT.PUT_LINE('Początek tygodnia roboczego.');
        WHEN 'Piątek' THEN DBMS_OUTPUT.PUT_LINE('Prawie weekend!');
        WHEN 'Sobota' THEN DBMS_OUTPUT.PUT_LINE('Weekend!');
        WHEN 'Niedziela' THEN DBMS_OUTPUT.PUT_LINE('Weekend!');
        ELSE DBMS_OUTPUT.PUT_LINE('Nieznany dzień tygodnia.');
    END CASE;
END;
/
```

**Przykład `CASE Statement` (Searched):**

```sql
DECLARE
    v_job_code VARCHAR2(10) := 'IT_PROG';
    v_salary_increase NUMBER;
    v_department VARCHAR2(10) := 'IT';
BEGIN
    CASE 
        WHEN v_job_code = 'SA_MAN' THEN
            v_salary_increase := 0.2;
            DBMS_OUTPUT.PUT_LINE('Podwyżka dla menadżera sprzedaży: ' || v_salary_increase);
        WHEN v_department = 'IT' AND v_job_code = 'IT_PROG' THEN
            v_salary_increase := 0.3;
            DBMS_OUTPUT.PUT_LINE('Podwyżka dla programisty IT: ' || v_salary_increase);
        ELSE
            v_salary_increase := 0;
            DBMS_OUTPUT.PUT_LINE('Brak podwyżki: ' || v_salary_increase);
    END CASE;
END;
/
```

### Pętle

Pętle służą do wielokrotnego wykonywania bloku kodu. W PL/SQL dostępne są trzy główne rodzaje pętli.

**1. Pętla `LOOP` (pętla prosta)**
Jest to najbardziej podstawowa pętla, która wykonuje się w nieskończoność, dopóki nie zostanie jawnie przerwana za pomocą instrukcji `EXIT` lub `EXIT WHEN`.

**Składnia:**

```sql
LOOP
   -- Sekwencja instrukcji
   EXIT [WHEN warunek];
END LOOP;
```

Aby uniknąć pętli nieskończonej, konieczne jest umieszczenie warunku wyjścia[^1].

**Przykład z `EXIT WHEN`:**

```sql
DECLARE
    v_counter NUMBER := 1;
BEGIN
    LOOP
        DBMS_OUTPUT.PUT_LINE('Licznik: ' || v_counter);
        v_counter := v_counter + 1;
        EXIT WHEN v_counter > 5; -- Wyjście z pętli, gdy licznik przekroczy 5
    END LOOP;
    DBMS_OUTPUT.PUT_LINE('Koniec pętli.');
END;
/
```

**2. Pętla `WHILE`**
Pętla `WHILE` wykonuje blok kodu tak długo, jak długo podany warunek jest prawdziwy. Warunek jest sprawdzany przed każdą iteracją[^2]. Jeśli na początku warunek jest fałszywy, pętla nie wykona się ani razu.

**Składnia:**

```sql
WHILE warunek LOOP
   -- Sekwencja instrukcji
END LOOP;
```

**Przykład:**

```sql
DECLARE
    v_counter NUMBER := 1;
BEGIN
    WHILE v_counter <= 5 LOOP
        DBMS_OUTPUT.PUT_LINE('Iteracja WHILE nr: ' || v_counter);
        v_counter := v_counter + 1; -- Należy pamiętać o inkrementacji licznika!
    END LOOP;
    DBMS_OUTPUT.PUT_LINE('Koniec pętli WHILE.');
END;
/
```

**3. Pętla `FOR`**
Pętla `FOR` jest używana do iteracji przez określony zakres liczb całkowitych. Automatycznie zarządza licznikiem – nie trzeba go deklarować ani inkrementować[^3].

**Składnia:**

```sql
FOR licznik IN [REVERSE] dolna_granica..gorna_granica LOOP
   -- Sekwencja instrukcji
END LOOP;
```

* `licznik` jest niejawnie deklarowany jako `PLS_INTEGER` i jego wartość nie może być zmieniana wewnątrz pętli.
* Słowo kluczowe `REVERSE` powoduje iterację od górnej do dolnej granicy[^3].

**Przykład:**

```sql
BEGIN
    -- Iteracja od 1 do 3
    DBMS_OUTPUT.PUT_LINE('Pętla FOR w przód:');
    FOR i IN 1..3 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;

    -- Iteracja od 3 do 1
    DBMS_OUTPUT.PUT_LINE('Pętla FOR w tył:');
    FOR i IN REVERSE 1..3 LOOP
        DBMS_OUTPUT.PUT_LINE(i);
    END LOOP;
END;
/
```


### Zaawansowane sterowanie pętlami

**Etykiety pętli (Loop Labeling)**
Pętlom można nadawać etykiety, co jest szczególnie przydatne w przypadku pętli zagnieżdżonych. Etykieta pozwala jednoznacznie określić, którą pętlę ma opuścić instrukcja `EXIT` lub którą ma kontynuować instrukcja `CONTINUE`[^4][^5].

**Składnia:**

```sql
<<nazwa_etykiety>>
LOOP
   -- ...
END LOOP nazwa_etykiety;
```

**Przykład zagnieżdżonej pętli z etykietą:**

```sql
BEGIN
    <<outer_loop>> -- Etykieta pętli zewnętrznej
    FOR i IN 1..3 LOOP
        <<inner_loop>> -- Etykieta pętli wewnętrznej
        FOR j IN 1..3 LOOP
            DBMS_OUTPUT.PUT_LINE('i: ' || i || ', j: ' || j);
            -- Wyjście z pętli zewnętrznej, gdy i*j >= 6
            EXIT outer_loop WHEN i * j >= 6; 
        END LOOP inner_loop;
    END LOOP outer_loop;
    DBMS_OUTPUT.PUT_LINE('Koniec pętli.');
END;
/
```

**Instrukcja `CONTINUE`**
Instrukcja `CONTINUE` przerywa bieżącą iterację pętli i przechodzi do następnej, pomijając resztę instrukcji w ciele pętli[^6].

**Składnia:**

```sql
CONTINUE [nazwa_etykiety] [WHEN warunek];
```

**Przykład:** Poniższy kod wypisuje tylko liczby nieparzyste.

```sql
BEGIN
    FOR i IN 1..10 LOOP
        -- Jeśli liczba jest parzysta, pomiń resztę iteracji
        CONTINUE WHEN MOD(i, 2) = 0; 
        DBMS_OUTPUT.PUT_LINE('Liczba nieparzysta: ' || i);
    END LOOP;
END;
/
```


### Instrukcja `GOTO`

Instrukcja `GOTO` pozwala na bezwarunkowe przeniesienie sterowania do określonej etykiety w tym samym bloku PL/SQL[^7]. Jej użycie jest często odradzane, ponieważ może prowadzić do powstania trudnego do analizy i utrzymania kodu (tzw. "spaghetti code").

**Składnia:**

```sql
GOTO nazwa_etykiety;
...
<<nazwa_etykiety>>
-- Instrukcja do wykonania
```

**Ograniczenia `GOTO`**[^7]:

* Nie można przenieść sterowania do wnętrza pętli, instrukcji `IF` lub bloku zagnieżdżonego.
* Nie można przenieść sterowania z bloku zewnętrznego do wewnętrznego.
* Nie można przenieść sterowania z sekcji obsługi wyjątków do sekcji wykonawczej.

**Przykład (sprawdzanie, czy liczba jest pierwsza):**

```sql
DECLARE
    v_szukana_liczba NUMBER := 17;
    v_czy_pierwsza   BOOLEAN := TRUE;
BEGIN
    FOR i IN 2..v_szukana_liczba-1 LOOP
        IF MOD(v_szukana_liczba, i) = 0 THEN
            v_czy_pierwsza := FALSE;
            GOTO koniec_sprawdzania; -- Przeskok do etykiety
        END IF;
    END LOOP;

    IF v_czy_pierwsza THEN
        DBMS_OUTPUT.PUT_LINE(v_szukana_liczba || ' jest liczbą pierwszą.');
    ELSE
        DBMS_OUTPUT.PUT_LINE(v_szukana_liczba || ' nie jest liczbą pierwszą.');
    END IF;

    <<koniec_sprawdzania>>
    DBMS_OUTPUT.PUT_LINE('Sprawdzanie zakończone.');
END;
/
```

[^1]: https://docs.oracle.com/cd/B19306_01/appdev.102/b14261/controlstructures.htm

[^2]: https://tts.inso.tuwien.ac.at/i/doc/XEDEV/xedev_plsql012.htm

[^3]: https://docs.oracle.com/database/121/LNPLS/controlstatements.htm

[^4]: https://www.w3schools.blog/loop-label-plsql

[^5]: https://www.tutorialspoint.com/plsql/plsql_loops.htm

[^6]: https://www.ibm.com/docs/en/db2-warehouse?topic=plsql-continue-statement

[^7]: https://www.oracletutorial.com/plsql-tutorial/plsql-goto/

[^8]: skrypt.txt

[^9]: https://www.tutorialspoint.com/plsql/plsql_for_loop.htm

[^10]: https://stackoverflow.com/questions/46661416/while-loop-and-if-else-in-pl-sql

[^11]: https://stackoverflow.com/questions/65237405/a-for-loop-inside-a-while-loop-in-plsql

