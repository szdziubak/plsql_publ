# Temat 3. Współpraca SQL i PL/SQL. Select, DML, Sekwencje w PL/SQL

PL/SQL został stworzony jako proceduralne rozszerzenie języka SQL. Jego siła tkwi w ścisłej integracji z silnikiem SQL bazy danych Oracle, co pozwala na płynne przeplatanie deklaratywnych zapytań SQL z logiką proceduralną[^1][^2]. Każde polecenie SQL wykonane wewnątrz bloku PL/SQL jest wysyłane do silnika SQL do przetworzenia, a wyniki (lub status operacji) wracają do środowiska PL/SQL.

### Użycie instrukcji `SELECT` w bloku PL/SQL

W przeciwieństwie do standardowego użycia SQL, w PL/SQL nie można po prostu wykonać zapytania `SELECT`, aby wyświetlić wyniki. Każde zapytanie `SELECT` musi pobierać dane *do* zmiennych lub struktur danych za pomocą klauzuli `INTO`[^1][^3].

**Kluczowe zasady:**

* Zapytanie `SELECT ... INTO` musi zwrócić **dokładnie jeden wiersz**.
* Jeśli zapytanie nie zwróci żadnego wiersza, PL/SQL zgłosi wyjątek `NO_DATA_FOUND`[^1][^3].
* Jeśli zapytanie zwróci więcej niż jeden wiersz, zostanie zgłoszony wyjątek `TOO_MANY_ROWS`[^1][^3].
* Liczba, kolejność i typy danych kolumn w zapytaniu muszą odpowiadać liczbie, kolejności i typom danych zmiennych w klauzuli `INTO`[^3].

**Składnia:**

```plsql
SELECT kolumna1, kolumna2, ...
INTO zmienna1, zmienna2, ...
FROM nazwa_tabeli
WHERE warunek;
```

**Przykład 1: Pobieranie wielu kolumn do zmiennych**
W tym przykładzie pobieramy imię, nazwisko i pensję pracownika o ID 100 do odpowiednich zmiennych[^1][^4][^5].

```plsql
SET SERVEROUTPUT ON;

DECLARE
    v_first_name employees.first_name%TYPE;
    v_salary     employees.salary%TYPE;
BEGIN
    SELECT first_name, salary 
    INTO v_first_name, v_salary
    FROM employees
    WHERE employee_id = 100;
    
    DBMS_OUTPUT.PUT_LINE('Pracownik: ' || v_first_name || ', pensja: ' || v_salary);
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('Nie znaleziono pracownika o podanym ID.');
    WHEN TOO_MANY_ROWS THEN
        DBMS_OUTPUT.PUT_LINE('Zapytanie zwróciło więcej niż jednego pracownika.');
END;
/
```

**Przykład 2: Pobieranie całego wiersza do rekordu**
Aby uprościć kod, zamiast wielu zmiennych, można użyć rekordu zdefiniowanego za pomocą atrybutu `%ROWTYPE`[^2].

```plsql
DECLARE
    r_emp employees%ROWTYPE; -- r_emp jest rekordem o strukturze tabeli employees
BEGIN
    SELECT *
    INTO r_emp
    FROM employees
    WHERE employee_id = 101;
    
    DBMS_OUTPUT.PUT_LINE('Pracownik: ' || r_emp.first_name || ' ' || r_emp.last_name);
END;
/
```


### Użycie instrukcji DML (INSERT, UPDATE, DELETE)

Instrukcje DML (Data Manipulation Language) takie jak `INSERT`, `UPDATE`, `DELETE` i `MERGE` mogą być używane bezpośrednio w sekcji wykonawczej bloku PL/SQL bez żadnych dodatkowych klauzul[^1][^6].

**Przykład 1: Użycie `INSERT` w pętli**
Poniższy kod tworzy kopię tabeli `employees`, a następnie w pętli `FOR` dodaje do niej 10 nowych, tymczasowych pracowników[^1].

```plsql
CREATE TABLE employees_copy AS SELECT * FROM employees;

BEGIN
    FOR i IN 207..216 LOOP
        INSERT INTO employees_copy (employee_id, first_name, last_name, email, hire_date, job_id, salary)
        VALUES (i, 'employee#'||i, 'temp', 'email'||i||'@test.com', SYSDATE, 'IT_PROG', 5000);
    END LOOP;
    COMMIT; -- Zatwierdzenie transakcji
END;
/
```

**Przykład 2: Użycie `UPDATE` i `DELETE`**
Poniższy blok najpierw podnosi pensję nowo dodanym pracownikom, a następnie ich usuwa[^1][^6].

```plsql
DECLARE
    v_salary_increase NUMBER := 500;
BEGIN
    -- Aktualizacja pensji
    UPDATE employees_copy
    SET salary = salary + v_salary_increase
    WHERE employee_id BETWEEN 207 AND 216;
    
    DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' wierszy zaktualizowano.');

    -- Usunięcie pracowników
    DELETE FROM employees_copy
    WHERE employee_id BETWEEN 207 AND 216;
    
    DBMS_OUTPUT.PUT_LINE(SQL%ROWCOUNT || ' wierszy usunięto.');
    
    ROLLBACK; -- Wycofanie zmian
END;
/
```

**Wskazówka:** Można aktualizować cały wiersz za pomocą rekordu, co jest bardzo wydajne[^2].

```plsql
DECLARE
    r_emp employees_copy%ROWTYPE;
BEGIN
    SELECT * INTO r_emp FROM employees_copy WHERE employee_id = 100;
    
    r_emp.salary := r_emp.salary * 1.1; -- Zwiększ pensję o 10%
    
    UPDATE employees_copy
    SET ROW = r_emp -- Aktualizacja całego wiersza na podstawie rekordu
    WHERE employee_id = 100;
END;
/
```


### Sekwencje w PL/SQL

Sekwencje to obiekty bazy danych służące do generowania unikalnych, sekwencyjnych liczb, najczęściej używanych do tworzenia wartości kluczy głównych[^7][^8]. W PL/SQL można odwoływać się do sekwencji za pomocą pseudokolumn `NEXTVAL` i `CURRVAL`.

* **`sekwencja.NEXTVAL`**: Zwraca następną dostępną wartość z sekwencji.
* **`sekwencja.CURRVAL`**: Zwraca ostatnią wartość pobraną przez `NEXTVAL` w bieżącej sesji.

**Przykład 1: Użycie `NEXTVAL` bezpośrednio w instrukcji `INSERT`**
To najczęstszy i najbardziej efektywny sposób użycia sekwencji[^1][^7].

```plsql
CREATE SEQUENCE emp_copy_seq START WITH 300 INCREMENT BY 1;

BEGIN
    FOR i IN 1..5 LOOP
        INSERT INTO employees_copy (employee_id, first_name, last_name, email, hire_date, job_id)
        VALUES (emp_copy_seq.NEXTVAL, 'New', 'Employee'||i, 'new'||i||'@test.com', SYSDATE, 'SA_REP');
    END LOOP;
END;
/
```

**Przykład 2: Pobieranie wartości sekwencji do zmiennej**
Czasami konieczne jest pobranie wygenerowanego ID do zmiennej, np. w celu użycia go w innych tabelach (klucz obcy).

```plsql
DECLARE
    v_new_emp_id NUMBER;
BEGIN
    -- Sposób 1: Bezpośrednie przypisanie (bardziej wydajne)
    v_new_emp_id := emp_copy_seq.NEXTVAL;
    DBMS_OUTPUT.PUT_LINE('Nowe ID (sposób 1): ' || v_new_emp_id);

    -- Sposób 2: Użycie SELECT INTO (wymaga dodatkowego przełączenia kontekstu do silnika SQL)
    SELECT emp_copy_seq.NEXTVAL INTO v_new_emp_id FROM DUAL;
    DBMS_OUTPUT.PUT_LINE('Nowe ID (sposób 2): ' || v_new_emp_id);
END;
/
```

[^1]: skrypt.txt

[^2]: https://docs.oracle.com/cd/B13789_01/appdev.101/b10807/06_ora.htm

[^3]: https://www.oracletutorial.com/plsql-tutorial/plsql-select-into/

[^4]: https://docs.oracle.com/en/database/other-databases/timesten/22.1/plsql-developer/use-sql-pl-sql-programs.html

[^5]: https://amysimpsongrange.com/2022/12/12/oracle-pl-sql-select-into/

[^6]: https://www.youtube.com/watch?v=_mV1KdWh7sM

[^7]: https://blog.devart.com/oracle-create-sequence-statement.html

[^8]: https://www.uslugi-komputerowe.eu/download/pau/SQL_poradnik_O_Oracle_ludzkim_glosem_podstawy_Andrzej_Klusiewicz.pdf

[^9]: https://www.databasejournal.com/oracle/execute-immediate-option-for-dynamic-sql-and-pl-sql/

[^10]: https://www.youtube.com/watch?v=ZDKl2xhUv8g

[^11]: https://stackoverflow.com/questions/75666539/how-to-write-pl-sql-that-determines-the-sql-dml-operation-being-performed

