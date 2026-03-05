# Alfabet - komunikator szpitalny

## Cel
Aplikacja mobilna (PWA) umozliwiajaca komunikacje osobom, ktore nie moga mowic. Mobile-first z obsluga tabletu (foldable). Interfejs po polsku.

## Uklad ekranu

Ekran podzielony na 3 strefy:

1. **Pasek zdania** (gorne ~15%) - skladane zdanie jako edytowalne kafelki slow + przycisk cofnij + przycisk wyczysc
2. **Strefa tresci** (srodkowe ~65%) - zmienia sie w zaleznosci od trybu
3. **Nawigacja trybow** (dolne ~20%) - 3 zakladki: Szybkie frazy, Slowa, Alfabet

Przyciski min. 60px wysokosci, duze odstepy, wysoki kontrast, czcionka min. 18px.

## Pasek zdania

- Kazde slowo jako osobny kafelek
- Tapniecie na kafelek = tryb edycji: sugestie zamiennikow, "Wpisz inne" (tryb literowy), "Usun"
- Przycisk cofnij = usuwa ostatnie slowo
- Przycisk wyczysc = czysci zdanie (z potwierdzeniem)

## Tryb 1: Szybkie frazy (ekran startowy)

Gotowe frazy pogrupowane w kategorie:

- **Potrzeby** - "Chce pic", "Chce jesc", "Zimno mi", "Goraco mi", "Chce spac"
- **Bol** - "Boli mnie glowa", "Boli mnie brzuch", "Boli mnie reka", "Cos mnie boli"
- **Pomoc** - "Zawolaj pielegniarke", "Prosze podniesc lozko", "Prosze polozyc lozko", "Potrzebuje pomocy"
- **Rozmowa** - "Jak tam u ciebie?", "Co slychac?", "Dziekuje", "Kocham cie", "Tak", "Nie"
- **Samopoczucie** - "Czuje sie dobrze", "Czuje sie zle", "Lepiej mi", "Gorzej mi"

Tapniecie na fraze = wstawia ja do paska zdania jako gotowe zdanie.

## Tryb 2: Slowa (predykcja)

1. Na starcie - najczestsze slowa startowe ("Ja", "Czy", "Jak", "Chce", "Prosze", "Nie", "Gdzie", "Kiedy", "Co")
2. Po wyborze slowa - sugestie nastepnego slowa
3. Zrodla sugestii:
   - Offline: wbudowany slownik 1000 najczestszych polskich slow + bigramy
   - Online (opcjonalnie): API modelu jezykowego
4. Brak odpowiedniego slowa = przycisk "Wpisz literkami" -> tryb Alfabet
5. Po wpisaniu literkami -> wraca do trybu Slowa

Sugestie jako siatka duzych przyciskow (3 kolumny, scroll).

## Tryb 3: Alfabet

1. Klawiatura A-Z ulozona alfabetycznie (5-6 liter w rzedzie)
2. Pole wpisywania nad klawiatura
3. Filtrowanie na zywo - lista pasujacych slow ze slownika
4. Tapniecie na slowo z listy = wstawia do paska i wraca do trybu Slowa
5. Przycisk polskich znakow (acelnoszz)
6. Spacja = konczy wpisywanie i dodaje slowo

## Techniczne

- Jeden plik index.html (HTML + CSS + JS), zero zaleznosci
- PWA: manifest.json + service worker
- Slownik offline: ~1000 najczestszych polskich slow + bigramy w JS
- AI predykcja: opcjonalne API, klucz w localStorage, fallback na offline
- Hosting: GitHub Pages
- Stan w pamieci, bez bazy danych
