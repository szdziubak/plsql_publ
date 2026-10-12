### Zadania do Tematu 2: Instrukcje sterowania i pętle

**Zadanie 1: Instrukcja `IF` - Kategoryzacja pensji**

Napisz anonimowy blok PL/SQL, który pobierze pensję pracownika o `employee_id = 104` i na podstawie jej wartości wyświetli jedną z poniższych kategorii:

* Jeśli pensja jest mniejsza niż 5000: 'Niska pensja'
* Jeśli pensja jest w przedziale od 5000 do 9000 (włącznie): 'Średnia pensja'
* Jeśli pensja jest wyższa niż 9000: 'Wysoka pensja'

Użyj do tego konstrukcji `IF-ELSIF-ELSE`.

**Oczekiwany wynik:**

```
Średnia pensja
```

**Zadanie 2: `CASE EXPRESSION` - Obliczanie premii**

Napisz anonimowy blok PL/SQL, który pobierze `job_id` pracownika o `employee_id = 101` i na podstawie tej wartości przypisze do zmiennej `v_procent_premii` odpowiedni procent premii, zgodnie z poniższymi zasadami:

* 'AD_PRES': 30% (`0.3`)
* 'SA_MAN': 25% (`0.25`)
* 'IT_PROG': 20% (`0.2`)
* Dla wszystkich pozostałych: 10% (`0.1`)

Użyj do tego prostego wyrażenia `CASE` (`Simple CASE Expression`). Na koniec wyświetl obliczony procent premii.

**Oczekiwany wynik:**

```
Procent premii: .1
```

**Zadanie 3: `CASE STATEMENT` - Symulacja operacji**

Napisz anonimowy blok PL/SQL, który na podstawie wartości zmiennej `v_operacja` typu `VARCHAR2` wykona odpowiednią akcję. Użyj wyszukiwanej instrukcji `CASE` (`Searched CASE Statement`).

1. Zadeklaruj zmienną `v_operacja` i przypisz jej wartość `'UPDATE'`.
2. Zadeklaruj zmienną `v_tabela` i przypisz jej wartość `'employees'`.
3. W `CASE STATEMENT` sprawdź wartość zmiennej `v_operacja`:
    * Jeśli ma wartość 'SELECT', wyświetl: `Operacja SELECT na tabeli [nazwa_tabeli].`
    * Jeśli ma wartość 'INSERT' lub 'UPDATE', wyświetl: `Operacja DML na tabeli [nazwa_tabeli].`
    * Jeśli ma wartość 'DELETE', wyświetl: `Operacja DELETE na tabeli [nazwa_tabeli] jest zabroniona!`
    * W każdym innym przypadku (klauzula `ELSE`) wyświetl: `Nieznana operacja.`

**Oczekiwany wynik:**

```
Operacja DML na tabeli employees.
```

**Zadanie 4: Pętla `LOOP` - Osiągnięcie progu**

Napisz anonimowy blok PL/SQL, który użyje prostej pętli `LOOP` do symulacji wzrostu kapitału.

1. Zadeklaruj zmienną `v_kapital` i przypisz jej wartość początkową `1000`.
2. Zadeklaruj zmienną `v_licznik_lat` i przypisz jej wartość `0`.
3. W pętli `LOOP` w każdej iteracji:
    * Zwiększaj `v_kapital` o 10% (`v_kapital := v_kapital * 1.1;`).
    * Zwiększaj `v_licznik_lat` o 1.
    * Użyj klauzuli `EXIT WHEN`, aby przerwać pętlę, gdy `v_kapital` przekroczy `2000`.
4. Po pętli wyświetl komunikat: `Kapitał przekroczył 2000 po [liczba] latach.`

**Oczekiwany wynik:**

```
Kapitał przekroczył 2000 po 8 latach.
```

**Zadanie 5: Pętla `FOR` - Tabliczka mnożenia**

Napisz anonimowy blok PL/SQL, który za pomocą pętli `FOR` wyświetli tabliczkę mnożenia dla liczby 7 w zakresie od 1 do 10.

**Oczekiwany wynik:**

```
7 x 1 = 7
7 x 2 = 14
...
7 x 10 = 70
```

**Zadanie 6: Pętla `WHILE` - Obliczanie silni**

Napisz anonimowy blok PL/SQL, który obliczy silnię dla liczby 5.

1. Zadeklaruj zmienne: `v_liczba` z wartością `5` oraz `v_silnia` z wartością `1`.
2. Użyj pętli `WHILE` do pomnożenia przez siebie kolejnych liczb od 5 w dół, aż do 1.
3. Po zakończeniu pętli wyświetl wynik w formacie: `Silnia z 5 wynosi: 120.`

**Oczekiwany wynik:**

```
Silnia z 5 wynosi: 120
```

**Zadanie 7: Instrukcja `CONTINUE` - Pomijanie liczb parzystych**

Napisz anonimowy blok PL/SQL, który w pętli `FOR` od 1 do 20 wypisze tylko te liczby, które są podzielne przez 3. Użyj instrukcji `CONTINUE WHEN`, aby pominąć iteracje dla liczb, które nie spełniają tego warunku.

**Oczekiwany wynik:**

```
Liczba podzielna przez 3: 3
Liczba podzielna przez 3: 6
Liczba podzielna przez 3: 9
Liczba podzielna przez 3: 12
Liczba podzielna przez 3: 15
Liczba podzielna przez 3: 18
```

**Zadanie 8: Instrukcja `GOTO` - Przerwanie pętli zagnieżdżonej**

Napisz blok PL/SQL z dwiema zagnieżdżonymi pętlami `FOR`. Pętla zewnętrzna powinna iterować od 1 do 5, a wewnętrzna od 1 do 5.

* Wewnątrz pętli wewnętrznej sprawdzaj, czy iloczyn liczników obu pętli (`licznik_zewnętrzny * licznik_wewnętrzny`) jest większy niż 20.
* Jeśli warunek jest spełniony, użyj instrukcji `GOTO`, aby natychmiast przeskoczyć do etykiety `<<koniec_bloku>>` umieszczonej na końcu sekcji wykonawczej.
* W każdej iteracji pętli wewnętrznej wyświetlaj aktualne wartości obu liczników.
* Na końcu bloku, pod etykietą, wyświetl komunikat 'Zakończono działanie.'.

**Oczekiwany wynik (lub podobny, zależny od momentu przerwania):**

```
Zewnętrzny: 1, Wewnętrzny: 1
Zewnętrzny: 1, Wewnętrzny: 2
...
Zewnętrzny: 4, Wewnętrzny: 5
Zewnętrzny: 5, Wewnętrzny: 1
Zewnętrzny: 5, Wewnętrzny: 2
Zewnętrzny: 5, Wewnętrzny: 3
Zewnętrzny: 5, Wewnętrzny: 4
Zakończono działanie.
```

[^1]: skrypt.txt

