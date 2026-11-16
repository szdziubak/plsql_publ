# 5. Zaawansowane typy danych. Rekord. Collections: varrays, nested tables i associative arrays

Oprócz prostych, skalarnych typów danych (jak `VARCHAR2` czy `NUMBER`), PL/SQL oferuje złożone (kompozytowe) typy danych, które pozwalają na grupowanie wielu powiązanych ze sobą wartości w jednej zmiennej. Do najważniejszych należą rekordy i kolekcje[^1].

### Rekordy (Records)

Rekord to struktura danych, która grupuje logicznie powiązane elementy o różnych typach, podobnie jak wiersz w tabeli. Każdy element w rekordzie nazywany jest polem. Rekordy upraszczają kod, pozwalając na operowanie na całym wierszu danych jako jednej jednostce.

**Sposoby definiowania rekordów:**

1. **Rekord oparty na strukturze tabeli (`%ROWTYPE`)**
Jest to najczęstszy i najwygodniejszy sposób. Zmienna typu rekordowego dziedziczy strukturę kolumn bezpośrednio z tabeli lub widoku.

```plsql
DECLARE
    -- r_emp jest rekordem o takiej samej strukturze jak wiersz w tabeli employees
    r_emp employees%ROWTYPE;
BEGIN
    SELECT * INTO r_emp FROM employees WHERE employee_id = 101;
    
    DBMS_OUTPUT.PUT_LINE('Pracownik: ' || r_emp.first_name || ' ' || r_emp.last_name);
    DBMS_OUTPUT.PUT_LINE('Pensja: ' || r_emp.salary);
END;
/
```

2. **Rekord oparty na strukturze kursora**
Jeśli kursor pobiera dane z wielu tabel (np. poprzez `JOIN`), można stworzyć rekord idealnie dopasowany do wyniku tego kursora.

```plsql
DECLARE
    CURSOR c_emp_details IS
        SELECT first_name, last_name, department_name
        FROM employees e JOIN departments d ON e.department_id = d.department_id;
    
    -- r_details jest rekordem o strukturze zdefiniowanej przez kursor
    r_details c_emp_details%ROWTYPE;
BEGIN
    OPEN c_emp_details;
    FETCH c_emp_details INTO r_details;
    CLOSE c_emp_details;
    
    DBMS_OUTPUT.PUT_LINE(r_details.first_name || ' pracuje w dziale ' || r_details.department_name);
END;
/
```

3. **Rekord zdefiniowany przez użytkownika (`TYPE ... IS RECORD`)**
Pozwala na stworzenie własnej, niestandardowej struktury rekordu, która może zawierać pola o dowolnych typach, w tym inne rekordy lub kolekcje[^2].

```plsql
DECLARE
    TYPE t_adres IS RECORD (
        ulica    VARCHAR2(100),
        miasto   VARCHAR2(50)
    );
    
    TYPE t_pracownik IS RECORD (
        imie        employees.first_name%TYPE,
        pensja      employees.salary%TYPE NOT NULL DEFAULT 1000,
        adres_zam   t_adres -- Zagnieżdżony rekord
    );
    
    r_pracownik t_pracownik;
BEGIN
    r_pracownik.imie := 'Jan';
    r_pracownik.pensja := 5000;
    r_pracownik.adres_zam.ulica := 'Kwiatowa 1';
    r_pracownik.adres_zam.miasto := 'Warszawa';
    
    DBMS_OUTPUT.PUT_LINE(r_pracownik.imie || ' mieszka w ' || r_pracownik.adres_zam.miasto);
END;
/
```


**Rekordy w operacjach DML**
Rekordy można wykorzystywać w instrukcjach `INSERT` i `UPDATE`, co znacznie upraszcza kod.

```plsql
DECLARE
    r_emp employees_copy%ROWTYPE;
BEGIN
    SELECT * INTO r_emp FROM employees WHERE employee_id = 104;
    
    -- Użycie rekordu do wstawienia całego wiersza
    INSERT INTO retired_employees VALUES r_emp;

    -- Modyfikacja pól rekordu i aktualizacja całego wiersza
    r_emp.salary := r_emp.salary * 1.1;
    r_emp.commission_pct := 0;
    UPDATE retired_employees SET ROW = r_emp WHERE employee_id = 104;
END;
/
```


### Kolekcje (Collections)

Kolekcja to uporządkowana grupa elementów tego samego typu, przypominająca listę lub tablicę[^1]. PL/SQL oferuje trzy rodzaje kolekcji[^3][^4]:

1. **Tablice asocjacyjne (Associative Arrays)**
2. **Tabele zagnieżdżone (Nested Tables)**
3. **VARRAY (Variable-size Arrays)**

#### 1. Tablice asocjacyjne (Associative Arrays / Index-by Tables)

Są to kolekcje par klucz-wartość, podobne do map lub słowników w innych językach programowania[^5].

**Kluczowe cechy:**

* **Indeksowanie**: Indeks może być liczbą całkowitą (`PLS_INTEGER`) lub ciągiem znaków (`VARCHAR2`)[^4]. To ich unikalna cecha.
* **Rozmiar**: Nie mają z góry określonego limitu rozmiaru.
* **Gęstość**: Mogą być rzadkie (sparse), co oznacza, że indeksy nie muszą być po kolei i mogą występować między nimi luki.
* **Inicjalizacja**: Nie wymagają inicjalizacji za pomocą konstruktora; można od razu przypisywać wartości do poszczególnych indeksów[^4].
* **Użycie**: Idealne do tworzenia tymczasowych, małych tabel przeglądowych (lookup tables) w pamięci, np. do cachowania danych[^5].

**Składnia:**

```plsql
TYPE nazwa_typu IS TABLE OF typ_elementu INDEX BY typ_indeksu;
```

**Przykład z indeksem liczbowym:**

```plsql
DECLARE
    TYPE t_lista_pracownikow IS TABLE OF employees.first_name%TYPE INDEX BY PLS_INTEGER;
    pracownicy t_lista_pracownikow;
BEGIN
    pracownicy(101) := 'John Doe';
    pracownicy(105) := 'Jane Smith'; -- Indeksy nie muszą być po kolei
    
    DBMS_OUTPUT.PUT_LINE('Pracownik 101: ' || pracownicy(101));
END;
/
```

**Przykład z indeksem znakowym:**

```plsql
DECLARE
    TYPE t_stolice IS TABLE OF VARCHAR2(100) INDEX BY VARCHAR2(50);
    stolice t_stolice;
    idx VARCHAR2(50);
BEGIN
    stolice('Polska') := 'Warszawa';
    stolice('Niemcy') := 'Berlin';
    
    -- Iteracja po tablicy asocjacyjnej
    idx := stolice.FIRST;
    WHILE idx IS NOT NULL LOOP
        DBMS_OUTPUT.PUT_LINE('Stolicą kraju ' || idx || ' jest ' || stolice(idx));
        idx := stolice.NEXT(idx);
    END LOOP;
END;
/
```


#### 2. Tabele zagnieżdżone (Nested Tables)

Są to nieuporządkowane zbiory elementów, które można dynamicznie rozszerzać.

**Kluczowe cechy:**

* **Indeksowanie**: Zawsze indeksowane kolejnymi liczbami całkowitymi, zaczynając od 1[^1].
* **Rozmiar**: Nie mają limitu rozmiaru i mogą rosnąć dynamicznie[^3].
* **Gęstość**: Domyślnie są gęste, ale po użyciu metody `DELETE` mogą stać się rzadkie (z lukami)[^1].
* **Inicjalizacja**: Wymagają jawnej inicjalizacji za pomocą konstruktora (`nazwa_kolekcji := nazwa_typu();`)[^3].
* **Użycie**: Odpowiednie dla dużych zbiorów danych o nieznanym z góry rozmiarze. Mogą być przechowywane jako kolumny w tabelach bazodanowych[^5].

**Składnia:**

```plsql
TYPE nazwa_typu IS TABLE OF typ_elementu;
```

**Przykład:**

```plsql
DECLARE
    TYPE t_lista_pensji IS TABLE OF NUMBER;
    pensje t_lista_pensji;
BEGIN
    -- Krok 1: Inicjalizacja
    pensje := t_lista_pensji();
    
    -- Krok 2: Rozszerzenie kolekcji i przypisanie wartości
    pensje.EXTEND(3); -- Robimy miejsce na 3 elementy
    pensje(1) := 5000;
    pensje(2) := 6000;
    pensje(3) := 7000;
    
    -- Usunięcie elementu tworzy lukę
    pensje.DELETE(2);
    
    -- Iteracja po rzadkiej kolekcji
    FOR i IN pensje.FIRST..pensje.LAST LOOP
        IF pensje.EXISTS(i) THEN
            DBMS_OUTPUT.PUT_LINE('Pensja pod indeksem ' || i || ': ' || pensje(i));
        END IF;
    END LOOP;
END;
/
```


#### 3. VARRAY (Variable-size Arrays)

Są to uporządkowane kolekcje o zdefiniowanym z góry maksymalnym rozmiarze.

**Kluczowe cechy:**

* **Indeksowanie**: Zawsze indeksowane kolejnymi liczbami całkowitymi, zaczynając od 1.
* **Rozmiar**: Mają stały, maksymalny rozmiar zadeklarowany podczas tworzenia typu[^3].
* **Gęstość**: Zawsze są gęste. Nie można usuwać pojedynczych elementów, tworząc luki (można jedynie skrócić kolekcję od końca za pomocą `TRIM`)[^3].
* **Inicjalizacja**: Wymagają inicjalizacji, podobnie jak tabele zagnieżdżone.
* **Użycie**: Przydatne, gdy znamy maksymalną liczbę elementów, które będziemy przechowywać. Mogą być składowane w kolumnach tabel[^5].

**Składnia:**

```plsql
TYPE nazwa_typu IS VARRAY(maksymalny_rozmiar) OF typ_elementu;
```

**Przykład:**

```plsql
DECLARE
    TYPE t_lista_imion IS VARRAY(5) OF VARCHAR2(50);
    imiona t_lista_imion;
BEGIN
    -- Inicjalizacja z dwoma wartościami
    imiona := t_lista_imion('Jan', 'Anna');
    
    -- Rozszerzenie o jeden element
    imiona.EXTEND;
    imiona(3) := 'Piotr';
    
    DBMS_OUTPUT.PUT_LINE('Maksymalny rozmiar (limit): ' || imiona.LIMIT);
    DBMS_OUTPUT.PUT_LINE('Aktualna liczba elementów: ' || imiona.COUNT);
END;
/
```


### Porównanie typów kolekcji

| Cecha | Tablica asocjacyjna (Associative Array) | Tabela zagnieżdżona (Nested Table) | VARRAY |
| :-- | :-- | :-- | :-- |
| **Typ indeksu** | `PLS_INTEGER` lub `VARCHAR2` | Tylko `PLS_INTEGER` (sekwencyjny od 1) | Tylko `PLS_INTEGER` (sekwencyjny od 1) |
| **Limit rozmiaru** | Brak | Brak | Zdefiniowany przy tworzeniu |
| **Gęstość (luki)** | Może być rzadka (sparse) | Może stać się rzadka po operacji `DELETE` | Zawsze gęsta (dense) |
| **Inicjalizacja** | Nie wymaga | Wymaga konstruktora | Wymaga konstruktora |
| **Przechowywanie w bazie** | Nie | Tak | Tak |
| **Główne zastosowanie** | Szybkie tabele przeglądowe w pamięci, cachowanie | Duże, dynamiczne zbiory danych | Zbiory danych o znanym, małym rozmiarze |


[^1]: https://docs.oracle.com/cd/A97630_01/appdev.920/a96624/05_colls.htm

[^2]: https://www.examtopics.com/discussions/oracle/view/103640-exam-1z0-149-topic-1-question-11-discussion/

[^3]: https://dev.to/mrcaption49/plsql-collections-associative-arrays-nested-tables-varrays-3i6j

[^4]: https://www.williamrobertson.net/documents/collection-types.html

[^5]: https://stackoverflow.com/questions/14934514/purpose-of-using-different-types-of-pl-sql-collections-in-oracle

[^6]: skrypt.txt

[^7]: https://stackoverflow.com/questions/44009066/populate-an-associative-array-inside-a-nested-table

[^8]: https://www.slideshare.net/slideshow/optymalizacja-rodowiska-open-source-w-celu-zwikszenia-oszczdnoci-i-kontroli/41866335

[^9]: https://dokumen.pub/oracle-database-12c-programowanie-w-jzyku-pl-sql-9788324699506-9788324699230-8324699236-8324699503.html

[^10]: https://www.ceneo.pl/96144620

