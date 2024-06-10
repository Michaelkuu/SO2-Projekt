# Gra Tower Defense

## Przegląd
Niniejszy dokument dostarcza informacji na temat podejścia do wielowątkowości oraz zarządzania sekcjami krytycznymi w naszej grze typu Tower Defense. Gra wykorzystuje wiele wątków do zarządzania generowaniem przeciwników, stanem gry oraz muzyką w tle, co zapewnia płynną i responsywną rozgrywkę.

## Wątki
### 1. Wątek generowania przeciwników (`SpawnEnemy`)
- **Cel**: Zarządza ciągłym tworzeniem i dodawaniem przeciwników zdefiniowanych w kolejnych falach. Przeciwnicy są generowani w określonych interwałach dla każdej fali i dodawani do współdzielonej listy `enemies`.
- **Inicjacja**: Uruchamiany przez `spawn_event` za każdym razem, gdy rozpoczyna się nowa fala przeciwników. Wątek jest tworzony i uruchamiany z funkcji `spawn_enemies`.
- **Zależności i synchronizacja**:
  - Wykorzystuje `enemies_lock` do synchronizacji dostępu do listy przeciwników, zapobiegając konfliktom i błędom związanym z jednoczesnymi modyfikacjami przez różne wątki.
  - Monitoruje `game_over_flag` oraz `stop_event`, aby ustalić, czy gra się zakończyła lub czy wątek powinien zostać zakończony przed rozpoczęciem nowej fali.
- **Typ**: Standardowy wątek z Pythona (`threading.Thread`).

### 2. Wątek muzyki (`MusicThread`)
- **Cel**: Zarządza odtwarzaniem muzyki w tle podczas gry.
- **Inicjacja**: Uruchamiany raz na początku gry, aby zapętlić muzykę w tle.
- **Zależności i synchronizacja**:
  - Działa niezależnie od innych wątków, jako że zarządzanie muzyką nie wpływa bezpośrednio na mechanikę gry i jest odseparowane od głównego przepływu logiki.
  - Nie wykorzystuje żadnych mechanizmów synchronizacji z innymi wątkami, ponieważ jego zadania są izolowane.
- **Typ**: Standardowy wątek z Pythona (`threading.Thread`).

## Sekcje Krytyczne
### Modyfikacja listy przeciwników
- **Opis**: Przeciwnicy są tworzeni i dodawani do listy `enemies` w funkcji `spawn_enemies`. Wątek odpowiedzialny za generowanie przeciwników musi uzyskać blokadę `enemies_lock` przed modyfikacją listy, aby zapewnić, że żaden inny wątek (np. główny wątek gry) nie modyfikuje listy jednocześnie.
- **Typ Blokady**: Mutex (`threading.Lock`).

### Przeglądanie i modyfikacja listy przeciwników
- **Opis**: Główny wątek gry przegląda i aktualizuje stan przeciwników, rysując ich i potencjalnie usuwając z listy. Ta operacja również wymaga użycia `enemies_lock` do synchronizacji dostępu.
- **Typ Blokady**: Mutex (`threading.Lock`).

## Flagi synchronizacyjne
- **Flaga `spawn_event`**: Kontroluje kiedy nowi przeciwnicy powinni być generowani. Wątek generujący przeciwników czeka na to zdarzenie, aby rozpocząć generowanie nowej fali przeciwników, i ustawia je na zakończenie fali, informując główny wątek gry o możliwości rozpoczęcia nowej fali.
- **Flaga `game_over_flag`**: Ustawiana, gdy gra się kończy (np. gracz przegrywa wszystkie życia). Ta flaga jest sprawdzana przez różne wątki, aby zdecydować, czy kontynuować działanie, czy zakończyć i oczyścić zasoby.
- **Typ**: Zdarzenia (`threading.Event`).

## Podsumowanie
Podejście do wielowątkowości w naszej grze Tower Defense zapewnia dynamiczną rozgrywkę i responsywną mechanikę gry. Odpowiednie zarządzanie sekcjami krytycznymi oraz mechanizmami synchronizacji jest kluczowe dla utrzymania stabilności i wydajności gry.
