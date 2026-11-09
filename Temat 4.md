## Temat 4: Bloki zagnieżdżone. Obsługa wyjątków. Propagacja wyjątków w blokach zagnieżdżonych. RAISE_APPLICATION_ERROR.

### Bloki zagnieżdżone (Nested Blocks)

W PL/SQL każdy blok może być zagnieżdżony wewnątrz innego bloku. Blok zewnętrzny nazywany jest blokiem nadrzędnym (parent/enclosing block), a blok wewnętrzny – blokiem podrzędnym lub zagnieżdżonym (child/nested block)[^1][^2]. Zagnieżdżanie jest powszechną praktyką, która pozwala na:

* **Modularyzację kodu**: Logikę mniejszych części programu można zamknąć w osobnych blokach, co poprawia czytelność[^3].
* **Ograniczenie zasięgu zmiennych**: Zmienne zadeklarowane w bloku wewnętrznym nie są widoczne w bloku zewnętrznym[^4].
* **Osobną obsługę błędów**: Blok wewnętrzny może mieć własną sekcję `EXCEPTION`, co pozwala na lokalne przechwycenie i obsłużenie błędu bez przerywania działania całego bloku nadrzędnego[^3].

**Struktura bloku zagnieżdżonego:**
Blok wewnętrzny umieszcza się w sekcji wykonawczej (`BEGIN...END`) bloku zewnętrznego[^3].

```plsql
-- Blok zewnętrzny (nadrzędny)
DECLARE
    v_zmienna_zewnetrzna VARCHAR2(50) := 'Jestem w bloku zewnętrznym';
BEGIN
    -- Blok wewnętrzny (zagnieżdżony)
    DECLARE
        v_zmienna_wewnetrzna VARCHAR2(50) := 'Jestem w bloku wewnętrznym';
    BEGIN
        DBMS_OUTPUT.PUT_LINE(v_zmienna_zewnetrzna); -- Dostęp do zmiennej zewnętrznej
        DBMS_OUTPUT.PUT_LINE(v_zmienna_wewnetrzna); -- Dostęp do zmiennej wewnętrznej
    END;
    
    DBMS_OUTPUT.PUT_LINE(v_zmienna_zewnetrzna);
    -- Poniższa linia spowoduje błąd, gdyż zmienna wewnętrzna nie jest widoczna na zewnątrz
    -- DBMS_OUTPUT.PUT_LINE(v_zmienna_wewnetrzna);
END;
/
```

Jeśli zmienna w bloku wewnętrznym ma taką samą nazwę jak w bloku zewnętrznym, PL/SQL da pierwszeństwo zmiennej lokalnej (wewnętrznej)[^1][^4].

### Obsługa wyjątków

Sekcja `EXCEPTION` pozwala na eleganckie zarządzanie błędami, które mogą wystąpić w trakcie wykonywania programu. Kiedy w sekcji wykonawczej (`BEGIN`) pojawi się błąd, normalne działanie bloku jest przerywane, a sterowanie przechodzi do sekcji `EXCEPTION` w poszukiwaniu odpowiedniej procedury obsługi.

Wyróżniamy trzy główne typy wyjątków[^4]:

1. **Wbudowane (Predefined)**: Najczęściej spotykane błędy serwera Oracle, które mają predefiniowane nazwy, np. `NO_DATA_FOUND`, `TOO_MANY_ROWS`, `VALUE_ERROR`.
2. **Niezdefiniowane (Non-Predefined)**: Błędy serwera Oracle, które nie mają domyślnych nazw. Można im przypisać nazwę za pomocą dyrektywy kompilatora `PRAGMA EXCEPTION_INIT`.
3. **Zdefiniowane przez użytkownika (User-Defined)**: Wyjątki tworzone na potrzeby logiki biznesowej aplikacji. Są deklarowane przez programistę i wywoływane jawnie za pomocą instrukcji `RAISE`.

**Przykład obsługi różnych typów wyjątków:**

```plsql
DECLARE
    -- Wyjątek zdefiniowany przez użytkownika
    e_za_wysoka_pensja EXCEPTION;
    -- Powiązanie błędu Oracle z nazwą wyjątku
    e_cannot_update_to_null EXCEPTION;
    PRAGMA EXCEPTION_INIT(e_cannot_update_to_null, -01407);

    v_pensja employees.salary%TYPE;
BEGIN
    SELECT salary INTO v_pensja FROM employees WHERE employee_id = 100;
    
    -- Warunek logiki biznesowej
    IF v_pensja > 20000 THEN
        RAISE e_za_wysoka_pensja; -- Ręczne wywołanie wyjątku
    END IF;

    -- Próba operacji, która może zgłosić błąd Oracle (non-predefined)
    UPDATE employees_copy SET email = NULL WHERE employee_id = 100;
EXCEPTION
    WHEN e_za_wysoka_pensja THEN
        DBMS_OUTPUT.PUT_LINE('Błąd logiki biznesowej: pensja jest zbyt wysoka.');
    WHEN e_cannot_update_to_null THEN
        DBMS_OUTPUT.PUT_LINE('Błąd danych: nie można przypisać wartości NULL do tej kolumny.');
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Błąd wbudowany: nie znaleziono pracownika.');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Wystąpił nieoczekiwany błąd.');
        DBMS_OUTPUT.PUT_LINE('Kod: ' || SQLCODE || ', Opis: ' || SQLERRM);
END;
/
```


### Propagacja wyjątków

Jeśli w bloku, w którym wystąpił błąd, nie ma odpowiedniej sekcji `EXCEPTION` do jego obsługi, PL/SQL **propaguje** (przekazuje) ten wyjątek do bloku nadrzędnego[^5][^6]. Proces ten powtarza się, aż wyjątek zostanie obsłużony lub dotrze do środowiska hosta (np. SQL Developer), co skutkuje zwróceniem nieobsłużonego błędu[^6].

**Scenariusz 1: Wyjątek obsłużony w bloku wewnętrznym**
Błąd jest przechwytywany i obsłużony lokalnie. Nie jest propagowany na zewnątrz, a wykonanie kodu jest kontynuowane w bloku zewnętrznym po zakończeniu bloku wewnętrznego[^5].

```plsql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Start bloku zewnętrznego.');
    BEGIN
        -- Ten blok spowoduje błąd, ale go obsłuży
        RAISE NO_DATA_FOUND;
    EXCEPTION
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Błąd obsłużony w bloku wewnętrznym.');
    END;
    DBMS_OUTPUT.PUT_LINE('Koniec bloku zewnętrznego.');
END;
/
```

**Scenariusz 2: Propagacja do bloku zewnętrznego**
Blok wewnętrzny nie obsługuje błędu, więc jest on przekazywany do sekcji `EXCEPTION` bloku zewnętrznego[^5].

```plsql
BEGIN
    DBMS_OUTPUT.PUT_LINE('Start bloku zewnętrznego.');
    BEGIN
        -- Ten blok spowoduje błąd, ale go NIE obsłuży
        RAISE TOO_MANY_ROWS;
    END;
EXCEPTION
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Błąd obsłużony w bloku ZEWNĘTRZNYM.');
END;
/
```

**Scenariusz 3: Wyjątek wywołany w sekcji `EXCEPTION`**
Jeśli wewnątrz sekcji `EXCEPTION` bloku wewnętrznego zostanie wywołany nowy wyjątek za pomocą `RAISE`, jest on **zawsze** propagowany do bloku nadrzędnego[^7].

```plsql
DECLARE
    v_zmienna VARCHAR2(5);
BEGIN
    BEGIN
        v_zmienna := 'Too long'; -- To spowoduje błąd VALUE_ERROR
    EXCEPTION
        WHEN VALUE_ERROR THEN
            DBMS_OUTPUT.PUT_LINE('Wewnętrzny handler: przechwycono VALUE_ERROR.');
            RAISE NO_DATA_FOUND; -- Wywołanie nowego wyjątku, który zostanie przekazany na zewnątrz
        WHEN NO_DATA_FOUND THEN
            DBMS_OUTPUT.PUT_LINE('Ten handler nigdy się nie wykona.');
    END;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Zewnętrzny handler: przechwycono NO_DATA_FOUND.');
END;
/
```

**Wynik:** Na ekranie pojawi się "Wewnętrzny handler...", a następnie "Zewnętrzny handler...", ponieważ `RAISE NO_DATA_FOUND` przekazało sterowanie do sekcji `EXCEPTION` bloku nadrzędnego[^7].

### Procedura `RAISE_APPLICATION_ERROR`

Procedura `RAISE_APPLICATION_ERROR` pozwala na zatrzymanie wykonania programu i zwrócenie do aplikacji klienckiej niestandardowego błędu z własnym numerem i komunikatem[^8][^9]. Jest to preferowany sposób sygnalizowania błędów logiki biznesowej[^10].

**Składnia:**

```plsql
RAISE_APPLICATION_ERROR(numer_błędu, komunikat, [zachowaj_stare_błędy]);
```

* `numer_błędu`: Liczba całkowita z zakresu od -20000 do -20999[^8][^11].
* `komunikat`: Ciąg znaków o długości do 2048 bajtów[^9].
* `zachowaj_stare_błędy` (opcjonalnie):
    * `FALSE` (domyślnie): Nowy błąd zastępuje wszystkie poprzednie na stosie błędów.
    * `TRUE`: Nowy błąd jest dodawany do stosu istniejących błędów[^8][^9].

**Przykład:**

```plsql
DECLARE
    v_pensja employees.salary%type;
BEGIN
    SELECT salary INTO v_pensja FROM employees WHERE employee_id = 100;
    
    IF v_pensja > 20000 THEN
        -- Zatrzymuje wykonanie i zwraca błąd do klienta
        RAISE_APPLICATION_ERROR(-20001, 'Pensja pracownika przekracza dozwolony próg 20000.');
    END IF;
    
    DBMS_OUTPUT.PUT_LINE('Pensja jest w normie.');
EXCEPTION
    -- Sekcja EXCEPTION w tym bloku nie jest konieczna, 
    -- ponieważ błąd jest zwracany bezpośrednio do aplikacji
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Błąd: ' || SQLERRM);
END;
/
```

**Ważne:** `RAISE_APPLICATION_ERROR` natychmiast zatrzymuje wykonanie bloku, ale **nie cofa** automatycznie zmian dokonanych przez instrukcje DML (`INSERT`, `UPDATE`, `DELETE`). Aby anulować zmiany, należy jawnie wywołać `ROLLBACK`[^9].


[^1]: https://www.plsqltutorial.com/plsql-nested-block/

[^2]: https://www.guru99.com/nested-blocks-pl-sql.html

[^3]: https://www.databasestar.com/pl-sql-tutorial-chapter-7/

[^4]: skrypt.txt

[^5]: https://docs.oracle.com/en/database/oracle/oracle-database/23/lnpls/exception-propagation.html

[^6]: https://www.oracletutorial.com/plsql-tutorial/plsql-exception-propagation/

[^7]: https://stackoverflow.com/questions/20218009/oracle-nested-blocks-and-exception-handling

[^8]: https://docs.oracle.com/en/database/oracle/oracle-database/19/lnpls/plsql-error-handling.html

[^9]: https://www.oracletutorial.com/plsql-tutorial/oracle-raise_application_error/

[^10]: https://dev.to/mrcaption49/raiseapplicationerror-and-pragma-exceptioninit-in-plsql-except-handling-1k3f

[^11]: https://dev.to/mrcaption49/e-5hfe

[^12]: https://vinish.dev/pl-sql-raise-application-error-example

[^13]: https://docs.oracle.com/html/B28370_05/block.htm

[^14]: https://forums.oracle.com/ords/apexds/post/pl-sql-nested-blocks-2901

[^15]: https://www.rebellionrider.com/declare-user-define-exception-using-raise_application_error-method-in-oracle-database/

[^16]: https://rules.sonarsource.com/plsql/tag/error-handling/rspec-2450/

[^17]: https://blogs.oracle.com/connect/post/building-with-blocks

[^18]: https://stackoverflow.com/questions/2334659/when-should-i-nest-pl-sql-begin-end-blocks

[^19]: https://stackoverflow.com/questions/40249993/nested-ifs-in-pl-sql

[^20]: https://www.cs.utahtech.edu/it/4310/plSQL/Variables-6.pdf

[^21]: https://hobsoft.com/guides/mastering-pl-sql-variable-scope-nested-code-blocks/

[^22]: https://docs.oracle.com/cd/B14117_01/appdev.101/b10807/07_errs.htm

[^23]: https://groups.google.com/g/oracle-plsql/c/EJ9ftLnvsw8

[^24]: https://docstore.mik.ua/orelly/oracle/prog2/ch08_04.htm

[^25]: https://www.scribd.com/doc/128845542/Nested-Block-in-PLSQL

[^26]: https://flylib.com/books/en/4.476.1.129/1/

[^27]: https://www.ibm.com/docs/en/db2/11.1.0?topic=plsql-raise-application-error

[^28]: https://stackoverflow.com/questions/60710866/pl-sql-raise-application-error-parameter-question

[^29]: http://www.360doc.com/content/17/1018/08/41075863_695893045.shtml

