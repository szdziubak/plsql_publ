**Zadanie 1: Podstawowe zmienne i wyświetlanie danych**

Napisz anonimowy blok PL/SQL, który:

1. W sekcji `DECLARE` zadeklaruje trzy zmienne:
    * `v_imie` typu `VARCHAR2(50)` z Twoim imieniem jako wartością domyślną.
    * `v_rok_studiow` typu `PLS_INTEGER` z numerem Twojego roku studiów.
    * `v_data_zapisu` typu `DATE`, której przypiszesz dzisiejszą datę za pomocą funkcji `SYSDATE`.
2. W sekcji `BEGIN` zmodyfikuje wartość zmiennej `v_imie`, dodając do niej Twoje nazwisko.
3. Wyświetli na ekranie jeden komunikat w formacie: `Student [imię i nazwisko] zapisał się na zajęcia dnia [data] na [rok] roku studiów.` używając funkcji `DBMS_OUTPUT.PUT_LINE` i operatora konkatenacji (`||`).

**Przykład oczekiwanego wyniku:**

```
Student Jan Kowalski zapisał się na zajęcia dnia 22-06-25 na 3 roku studiów.
```

**Zadanie 2: Atrybut %TYPE i obliczenia na danych z tabeli**

Napisz anonimowy blok PL/SQL, który pobierze dane pracownika i obliczy jego roczną pensję oraz premię.

1. W sekcji `DECLARE` zadeklaruj:
    * Stałą `c_premia_roczna` typu `NUMBER` z wartością `0.15` (oznaczającą 15% premii).
    * Zmienną `v_nazwisko_pracownika` o typie zakotwiczonym w kolumnie `last_name` z tabeli `employees` (użyj `%TYPE`).
    * Zmienną `v_pensja_miesieczna` o typie zakotwiczonym w kolumnie `salary` z tabeli `employees`.
    * Zmienne `v_pensja_roczna` i `v_wartosc_premii` typu `NUMBER(10,2)`.
2. W sekcji `BEGIN` wykonaj zapytanie `SELECT ... INTO`, które pobierze nazwisko i pensję pracownika o `employee_id = 104`.
3. Oblicz roczną pensję (`pensja miesięczna * 12`) i wartość premii (`pensja roczna * premia`).
4. Wyświetl na ekranie trzy oddzielne komunikaty:
    * `Pracownik: [nazwisko]`
    * `Roczna pensja: [wartość]`
    * `Należna premia: [wartość]`

**Przykład oczekiwanego wyniku:**

```
Pracownik: Ernst
Roczna pensja: 72000
Należna premia: 10800
```

**Zadanie 3: Zasięg zmiennych i bloki zagnieżdżone**

Napisz blok PL/SQL, który zademonstruje działanie zasięgu zmiennych w blokach zagnieżdżonych.

1. W bloku zewnętrznym zadeklaruj zmienną `v_globalny_opis` typu `VARCHAR2(50)` i przypisz jej wartość `'Jestem w bloku głównym'`.
2. Wewnątrz sekcji wykonawczej bloku zewnętrznego, utwórz blok zagnieżdżony.
3. W bloku zagnieżdżonym zadeklaruj zmienną `v_lokalny_opis` typu `VARCHAR2(50)` i przypisz jej wartość `'Jestem w bloku lokalnym'`.
4. W sekcji wykonawczej bloku zagnieżdżonego wyświetl wartości **obu** zmiennych (`v_globalny_opis` i `v_lokalny_opis`).
5. Po zakończeniu bloku wewnętrznego, w sekcji wykonawczej bloku zewnętrznego, ponownie wyświetl wartość zmiennej `v_globalny_opis`.

**Oczekiwany wynik na ekranie:**

```
Jestem w bloku głównym
Jestem w bloku lokalnym
Jestem w bloku głównym
```

*(Pytanie dodatkowe: Dlaczego próba odwołania się do zmiennej `v_lokalny_opis` w bloku zewnętrznym zakończyłaby się błędem?)*
