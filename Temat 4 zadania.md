### Zadania do Tematu 5: Obsługa wyjątków

**Zadanie 1: Obsługa wyjątku wbudowanego (`NO_DATA_FOUND`, `TOO_MANY_ROWS`)**

Napisz anonimowy blok PL/SQL, który próbuje pobrać dane pracownika i obsłuży standardowe wyjątki Oracle.

1. Zadeklaruj zmienną `v_pensja` typu `employees.salary%TYPE`.
2. W sekcji `BEGIN` spróbuj pobrać pensję pracownika:
    * Najpierw dla `employee_id = 9999` (który nie istnieje), co powinno wywołać `NO_DATA_FOUND`.
    * Następnie, zakomentuj powyższą linię i odkomentuj próbę pobrania pensji dla `job_id = 'IT_PROG'`, co powinno wywołać `TOO_MANY_ROWS` (ponieważ jest więcej niż jeden programista).
3. W sekcji `EXCEPTION`:
    * Obsłuż wyjątek `NO_DATA_FOUND`, wyświetlając komunikat: `'Błąd: Nie znaleziono pracownika o podanym ID.'`.
    * Obsłuż wyjątek `TOO_MANY_ROWS`, wyświetlając komunikat: `'Błąd: Znaleziono więcej niż jednego pracownika.'`.
    * Obsłuż ogólny wyjątek `OTHERS`, wyświetlając kod błędu i opis: `'Wystąpił nieoczekiwany błąd: ' || SQLCODE || ' - ' || SQLERRM`.

**Zadanie 2: Obsługa wyjątku niezdefiniowanego (`PRAGMA EXCEPTION_INIT`)**

Napisz anonimowy blok PL/SQL, który spróbuje zaktualizować kolumnę z ograniczeniem `NOT NULL` wartością `NULL` i obsłuży ten błąd jako wyjątek nazwany.

1. W sekcji `DECLARE` zadeklaruj wyjątek `e_cannot_update_to_null` i powiąż go z numerem błędu Oracle dla naruszenia `NOT NULL` (`-01407`) za pomocą `PRAGMA EXCEPTION_INIT`.
2. W sekcji `BEGIN` spróbuj zaktualizować kolumnę `email` w tabeli `employees_copy` na `NULL` dla dowolnego pracownika (np. `employee_id = 100`). Upewnij się, że kolumna `email` ma ograniczenie `NOT NULL`.
3. W sekcji `EXCEPTION` obsłuż zadeklarowany wyjątek `e_cannot_update_to_null`, wyświetlając komunikat: `'Błąd: Nie można przypisać wartości NULL do kolumny EMAIL.'`.

**Zadanie 3: Obsługa wyjątku zdefiniowanego przez użytkownika (`RAISE`)**

Napisz anonimowy blok PL/SQL, który symuluje kontrolę poprawności danych biznesowych.

1. W sekcji `DECLARE` zadeklaruj wyjątek zdefiniowany przez użytkownika o nazwie `e_zbyt_mlody_pracownik`.
2. Zadeklaruj zmienną `v_data_zatrudnienia` typu `employees.hire_date%TYPE` i przypisz jej datę np. `'01-JAN-2050'`.
3. W sekcji `BEGIN`:
    * Jeśli `v_data_zatrudnienia` jest późniejsza niż `SYSDATE` (co oznacza datę z przyszłości), wywołaj wyjątek `e_zbyt_mlody_pracownik` za pomocą `RAISE`.
    * W przeciwnym razie wyświetl komunikat: `'Data zatrudnienia jest poprawna.'`.
4. W sekcji `EXCEPTION` obsłuż wyjątek `e_zbyt_mlody_pracownik`, wyświetlając komunikat: `'Błąd biznesowy: Data zatrudnienia nie może być z przyszłości.'`.

**Zadanie 4: Użycie `RAISE_APPLICATION_ERROR`**

Napisz anonimowy blok PL/SQL, który zastosuje `RAISE_APPLICATION_ERROR` do zgłoszenia błędu logiki biznesowej.

1. Zadeklaruj zmienną `v_cena_produktu` typu `NUMBER` i przypisz jej wartość `5`.
2. W sekcji `BEGIN`:
    * Jeśli `v_cena_produktu` jest mniejsza niż `10`, użyj `RAISE_APPLICATION_ERROR` do zgłoszenia błędu o numerze `-20005` i komunikacie: `'Produkt jest zbyt tani, minimalna cena to 10.'`.
    * W przeciwnym razie wyświetl komunikat: `'Cena produktu jest akceptowalna.'`.
3. **Wskazówka**: `RAISE_APPLICATION_ERROR` natychmiast przerywa wykonanie bloku i zwraca błąd do klienta, więc nie będzie potrzeby sekcji `EXCEPTION` w tym bloku, chyba że chcesz go przechwycić na wyższym poziomie. Po wykonaniu kodu powinien pojawić się błąd w narzędziu klienckim.

**Zadanie 5: Integracja obsługi wyjątków w blokach zagnieżdżonych**

Napisz anonimowy blok PL/SQL z zagnieżdżonym blokiem, który zademonstruje propagację i obsługę różnych wyjątków.

1. **Blok zewnętrzny:**
    * Zadeklaruj zmienną `v_glowny_status` typu `VARCHAR2(100)` i przypisz jej `'Rozpoczynam główny proces...'`.
    * W `BEGIN` wyświetl `v_glowny_status`.
    * W `EXCEPTION` obsłuż `OTHERS`, wyświetlając `'Błąd w bloku głównym: ' || SQLERRM`.
2. **Blok wewnętrzny (zagnieżdżony w `BEGIN` bloku zewnętrznego):**
    * Zadeklaruj zmienną `v_liczba_dzialow` typu `PLS_INTEGER`.
    * W `BEGIN` spróbuj pobrać `COUNT(*)` z tabeli `non_existent_table` (nieistniejącej), co spowoduje błąd.
    * W `EXCEPTION` tego bloku:
        * Obsłuż wyjątek `OTHERS`.
        * Wyświetl komunikat: `'Błąd w bloku wewnętrznym: ' || SQLERRM`.
        * Następnie użyj `RAISE;` (bez parametrów), aby zgłosić ten sam wyjątek do bloku nadrzędnego.
3. Sprawdź, jakie komunikaty pojawią się na ekranie.

[^1]: skrypt.txt

