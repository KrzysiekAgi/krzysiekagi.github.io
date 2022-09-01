---
layout: post
title: "Wprowadzenie do bezpieczeństwa aplikacji webowych"
author: "Kris"
tags: security, TestWarez, workshop, intro, polski
---

Ten post jest krótkim wprowadzeniem do bezpieczeństwa aplikacji webowych i powtórką z podstawowych zagadnień związanych z aplikacjami w Internecie.


# Table of contents
1. [Protokół HTTP](#Protokół HTTP)
    1. [Adresy URL](#url)
    2. [Zapytania](#zapytania)
    3. [Metody](#metody)
    4. [Nagłówki](#nagłówki)
    5. [Statusy](#statusy)
2. [Bazy danych i SQL](#sql)
    1. [Składnia SQL](#składnia)
    2. [SELECT](#select)
    3. [Union](#union)
3. [HTML, JS i narzędzia deweloperskie](#html)

## Protrokół HTTP <a name="Protokół HTTP"></a>
HTTP to najbardziej popularny obecnie prtokół komunikacji między dwoma serwisami (stronami internetowymi, serwerami itp.) w Internecie. Jest to protokół głównie tekstowy i bezstanowy, co oznacza że każde zapytanie jest wysyłane w formie tekstowej (o określonej strukturze, omówionej niżej) oraz że serwer nie pamięta poprzednich zapytań od danego klienta. Tę ostatnią część można obejść za pomocą ciasteczek, ale jest to poza zakresem tego wprowadzenia.

### Adresy URL <a name="url"></a>
Adresy URL to ciągi znaków wpisywane w pasku adresu. Mają ustaloną budowę. Przykładowy adres URL wygląda następująco

```
http://192.168.1.1:80/test/index.php?action=show&id=2
```

Składa się on następująco z: 
- protokołu (`http://`)
- adresu serwera (`192.168.1.1` lub `nazwastrony.pl`)
- ścieżki do zasobu (`test/index.php`)
- parametrów (`?action=show&id=2`)


### Zapytania HTTP <a name="zapytania"></a>
Zapytania HTTP składają się z trzech części: metody, nagłówków i ciała. Metoda jest pierwszą informacją w zapytaniu. Następnie występują nagłówki, a na końcu opcjonalne ciało, które musi być oddzielone pustą linią. Nagłówki muszą następować zaraz po sobie, w kolejnych liniach. Wszystko co będzie się znajdowało po pierwszej pustej linii, zostanie zinterpretowane jako ciało zapytania. Przykładowe zapytanie `POST` wygląda następująco 

```
POST /login.php HTTP/1.1                        <- Metoda, zasób, wersja HTTP
Host: www.bodge.it                              <- nagłówki
User-Agent: Mozilla/5.0                         
Content-Length: 25                              
Content-Type: application/x-www-form-urlencoded 
                                                <- wymagana pusta linia oddzielająca nagłówki od ciała
login=bob&password=secret                       <- ciało zapytania
```

A przykładowa odpowiedź 
```
HTTP/1.1 200 OK                     <- wersja HTTP, status i nazwa odpowiedzi
Content-Length: 58                  <- nagłówki
Connection: close               
Content-Type: text/html         
                                    <- wymagana pusta linia oddzielająca nagłówki od ciała
<html><body>                        <- ciało odpowiedzi w HTML 
<h1>Wrong login or password</h1> 
</body><html> 
```

### Metody HTTP <a name="metody"></a>

```
| Metoda  |  Ciało zapytania  |  Ciało odpowiedzi  | Opis                                        |
|---------|-------------------|--------------------|---------------------------------------------|
| GET     |  niedozwolone     |  opcjonalne        | Pobierz dany zasób                          |
| HEAD    |  niedozwolone     |  niedozwolone      | Pobiera same nagłówki (bez ciała odpowiedzi)|
| POST    |  opcjonalne       |  opcjonalne        | Utworzenie lub zastąpienie zasobu           |
| PUT     |  opcjonalne       |  opcjonalne        | Utworzenie lub modyfikacja zasobu           |
| DELETE  |  opcjonalne       |  opcjonalne        | Usuwanie zasobu na serwerze                 |
```

<br>

Istnieją również inne metody HTTP (np. CONNECT, czy TRACE), ale nie są one bardzo powszechne w praktyce testera. Pełną specyfikację można znaleźć w [standardzie RFC 9110](https://www.rfc-editor.org/rfc/rfc9110.html#name-methods)


### Nagłówki HTTP <a name="nagłówki"></a>
Nagłówki HTTP występują zarówno w zapytaniach, jak i odpowiedziach. Są oddzielone od ciała pustą linią. Zawsze przyjmują postać `klucz: wartość`, niemniej pole `klucz` nie jest zdefiniowane żadnym standardem. Pozwala to wysyłać dowolne nagłówki między serwerami. 

Nagłówki informują o takich rzeczach jak długość i rodzaj wiadomości, user agent, rodzaj odpowiedzi, informacje jak traktować odebraną wiadomość i inne. To tutaj wysyłane są ciasteczka.

### Statusy odpowiedzi HTTP <a name="statusy"></a>
Statusy HTTP to trzycyfrowe liczby zwracane w odpowiedzi dla każdego zapytania. Informują one, czy dane zapytanie się powiodło i z jakim skutkiem, a jeśli nie, to wskazują na możliwą przyczynę błędu. Wyróżniamy 5 klas statusów odpowiedzi.

```
| Klasa |                          Opis                            |
|-------|----------------------------------------------------------|
| 1xx   | Informacyjne. Rzadko spotykane przez użytkowanika        |
| 2xx   | Zapytanie zakończone sukcesem                            |
| 3xx   | Przekierowanie. Zapytanie należy kierować pod inny adres |
| 4xx   | Błąd użytkownika                                         |
| 5xx   | Błąd serwera                                             |
```


Najpopularniejsze kody odpowiedzi to:
- 200 OK
- 201 Created
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 500 Internal Server Error

Więcej informacji znajduje się np. w [dokumentacji Mozilli](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

Ulubionym statusem każdego programisty jest jednak [status 418](https://www.google.com/teapot).

## Bazy danych i SQL <a name="sql"></a>
To nie jest pełne wprowadzenie do języka SQL. Tutaj skupimy się tylko na przedstawieniu tej części, która jest przydatna na codzień w praktyce (pen)testera do wyszukiwania błędów. Oczywiście jeśli zdarzy się znaleźć miejsce w aplikacji pozwalające wykonać dowolne polecenia SQL, zaczyna etap wykorzystania podatności, gdzie szersza wiedza z zakresu baz danych jest bardzo przydatna.

### Składnia języka SQL <a name="składnia"></a>
Składnia SQLa jest stosunkowo prosta, mimo że różni się w zależności od zapytania. Każde zaczyna się od słowa kluczowego oznaczającego rodzaj operacji. W każdym musi się też znaleźć nazwa tabeli, do której się odwołujemy (poza kwerendami tworzącymi bazy danych itp.). Najbardziej popularne zapytanie to prawdopodobnie `SELECT`, służące do wybierania danych z tablei. Przyjrzymy się bliżej w następnym podrozdziale. 

Jedna uwaga - język SQL nie zwraca uwagi na wielkość liter. `SELECT` i `select` (oraz np. `sElEcT`) oznaczają to samo. Niemniej, konwencją jest pisać zapytania wielkimi literami.

### Zapytanie SELECT <a name="select"></a>
Zapytanie `SELECT` wygląda następująco:

```
SELECT column1, column2 FROM table1
```

Po słowie kluczowym `SELECT` następują nazwy kolumn, które chcemy pobrać, słowo `FROM`, oraz nazwa tabeli. Jeśli chcielibyśmy pobrać wszystkie kolumny z danej tabeli, można uniknąć ich wypisywania przez użycie gwiazdki (`*`) w miejsce nazw kolumn. 

SQL pozwala również na "mniej sensowne" z punktu biznesowego kwerendy. Możemy napisać np. 
```
SELECT 1 FROM table1
```
co spowoduje... wyświetlenie liczby 1.

Umożliwia też filtrowanie wyników przez użycie słowa `WHERE`. 

```
SELECT * FROM table1 WHERE column1 like '%TestWarez%`
```

Powyższe zapytanie wybierze wszystkie kolumny z tabeli table1, ale pokaże tylko te wiersze, które w kolumnie column1 zawierają ciąg znaków "TestWarez". Znaki `%` oznaczają w tym kontekście, że z danej strony mogą (ale nie muszą) występować inne znaki. Także wszyskie z poniższych przykładów zostaną znalezione jako spełniające warunek
- To jest ciąg znaków na TestWarez
- TestWarez
- SuperTestWarez
- TestWarez!
- ` TestWarez ` (spacje na początku i końcu)

Jeśli chcielibyśmy znaleźć dokładne dopasowanie do ciągu znaków, należy pominąć znaki procentu. Analogiczny warunek można dopisać do innych typów danych (np. `WHERE column2=5`)


### Union - łączenie zapytań <a name="union"></a>
W ramach jednego zapytania można łączyć kilka tabel. W SQLu są na to dwa sposoby, `JOIN` i `UNION`. Różnią się one jednak zachowaniem. `JOIN` pozwoli połączyć kolumny z tabel tab1 i tab2 i wyświetlić je razem (np. wyświetlić nazwiska użytwkowników z jednej tabeli, obok numerów ich kart kredytowych z drugiej tabeli). 

`UNION` z kolei "doklei" wynik drugiego zapytania do wyniku pierwszego pod warunkiem, że liczba kolumn w obu zapytaniach będzie się zgadzać (nie musi się zgadzać ich typ!).
Tak więc 
```
SELECT col1, col2 FROM tab1 UNION SELECT col3, col4 FROM tab2
```
spowoduje wyświetlenie kolumn col1 oraz col2, a pod nimi kolumn col3 ("doklejonej" do col1) i col4 ("doklejonej" do col2). Ponieważ `UNION` łączy dwa pełnoprawne zapytania `SELECT`, możemy oprócz (lub zamiast) kolumn wybrać dane (`SELECT 1 [...]`) lub funkcje (`max`, `min`, `avg`, etc).

## HTML, JS i narzędzia deweloperskie <a name="html"></a>
HTML i język JavaScript stanowią podstawę dzisiejszych stron internetowych. Kod źródłowy strony można podejrzejć naciskając w przeglądarce Crtl+u (skrót Firefoxa) lub klikając PPM i wybierając "Pokaż źródło strony". 

Podstawowe elementy HTMLa, które są przydatne to m.in. `<head>, <body>, <h1>, <div>, <script>`
Element `<script>` pozwala na wstrzykiwanie kodu [JavaScript](https://www.w3schools.com/js/DEFAULT.asp).

Oprócz kodu źródłowego przeglądarka pozwal nam również zobaczyć konsolę, ruch sieciowy na danej stronie, czy otworzyć inspektora i podejrzeć konkretne elementy. To wszystko można zrobić za pomocą narzędzi deweloperskich (Crtl+Shift+C lub menu przeglądarki -> więcej narzędzi -> Web Dev tools)