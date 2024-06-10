# Systemy Operacyjne 2 Projekt

## Wydział Informatyki i Telekomunikacji

### Autorzy:
- Michał Kucharek - 264169
- Michał Ćwirko - 269178

### Prowadzący:
dr inż. Tomasz Szandała

---

## Tower Defense

Gra została stworzona przy użyciu języka Python, biblioteki Pygame oraz wątków. Celem gry jest ochrona swojej bazy przed wrogami, budując wieże obronne. Wrogowie pojawiają się co pewien czas falami i poruszają się po wyznaczonej ścieżce. Gracz zdobywa pieniądze za pokonywanie wrogów, które może przeznaczyć na budowę dodatkowych wież. Gra oferuje prostą, ale wciągającą mechanikę, która wymaga szybkiego myślenia i skutecznej strategii.

### Wątki:
#### 1. Wątek “SpawnEnemy”
- **Reprezentacja**: Wątek ten jest odpowiedzialny za ciągłe tworzenie przeciwników zdefiniowanych w falach. Przeciwnicy są generowani w interwałach określonych dla każdej fali i są dodawani do listy “enemies”, którą wykorzystuje główny wątek gry do zarządzania interakcjami i logiką przeciwników.

- **Inicjacja**: Wątek jest uruchamiany za każdym razem, gdy rozpoczyna się nowa fala przeciwników, co jest kontrolowane przez “spawn_event”. Nowy wątek jest tworzony i startowany z funkcji “spawn_enemies”.

- **Zależności i synchronizacja**:
  - Używa “enemies_lock” do synchronizacji dostępu do listy przeciwników, co zapobiega konfliktom i błędom związanym z jednoczesnym modyfikowaniem tej listy przez różne wątki.
  - Odczytuje flagę “game_over_flag” oraz “stop_event”, aby sprawdzić, czy gra została zakończona lub czy wątek powinien zakończyć działanie przed rozpoczęciem nowej fali.

- **Typ**: Standardowy wątek z Pythona (threading.Thread).

#### 2. Wątek “MusicThread”
- **Reprezentacja**: Odpowiada za odtwarzanie muzyki w tle gry.

- **Inicjacja**: Jest uruchamiany raz podczas startu gry, aby zapętlić odtwarzanie ścieżki dźwiękowej.

- **Zależności i synchronizacja**:
  - Operuje niezależnie od innych wątków, ponieważ zarządzanie muzyką nie wpływa bezpośrednio na mechanikę gry i jest odseparowane od głównego przepływu logiki.
  - Nie używa żadnych mechanizmów synchronizacji z innymi wątkami, ponieważ jego zadania są izolowane.

- **Typ**: Standardowy wątek z Pythona (threading.Thread).

### Sekcje Krytyczne:
#### 1. Modyfikacja listy przeciwników
- **Opis**: W funkcji “spawn_enemies”, przeciwnicy są tworzeni i dodawani do listy “enemies”. Wątek odpowiedzialny za generowanie przeciwników musi uzyskać blokadę (enemies_lock) przed modyfikacją listy, aby zapewnić, że żaden inny wątek (np. główny wątek gry podczas aktualizacji stanu przeciwników) nie będzie modyfikować listy jednocześnie.

- **Typ Blokady**: Mutex (threading.Lock).

#### 2. Przeglądanie i modyfikacja listy przeciwników
- **Opis**: W głównym wątku gry, podczas rysowania i aktualizacji stanu przeciwników, lista “enemies” jest iterowana, a przeciwnicy są aktualizowani, rysowani lub usuwani. Ta operacja również wymaga blokady, ponieważ w innym przypadku mogą wystąpić konflikty z wątkiem generującym przeciwników.

- **Typ Blokady**: Mutex (threading.Lock).

#### 3. Flagi synchronizacyjne:
-  **Flaga “spawn_event”**: Kontroluje, kiedy nowi przeciwnicy powinni być generowani. Wątek generujący przeciwników czeka na to zdarzenie, aby rozpocząć generowanie nowej fali przeciwników i ustawia je na zakończenie fali, informując główny wątek gry o możliwości rozpoczęcia nowej fali.
-  **Flaga “game_over_flag”**: Jest ustawiana, gdy gra się kończy (np. gracz przegrał wszystkie życia). Ta flaga jest sprawdzana przez różne wątki, aby zdecydować, czy kontynuować działanie, czy zakończyć i oczyścić zasoby.

- **Typ**: Zdarzenia (threading.Event).
