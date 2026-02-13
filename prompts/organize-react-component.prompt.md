---
name: organize-react-component
description: "Uporządkuj plik TS/TSX z komponentem React (Tailwind, Biome, named exports). Bez zmian zachowania."
argument-hint: "Uruchom na otwartym pliku .tsx/.ts; opcjonalnie: zaznacz fragment."
agent: 'agent'
---

# Zadanie
Uporządkuj kod w bieżącym pliku `${fileBasename}` (lub w zaznaczeniu), zgodnie z najlepszymi praktykami React + TypeScript w tym repo.

- Jeśli istnieje zaznaczenie (`${selection}`), uporządkuj **tylko zaznaczenie**.
- W przeciwnym razie uporządkuj **cały plik** (`${file}`).

# Kontekst projektu (obowiązuje)
- React (bez Next.js / bez RSC).
- Stylowanie: Tailwind (`className`).
- Formatowanie i sortowanie importów: **Biome** (nie skupiaj się na ręcznym sortowaniu importów).
- Komponenty: wyłącznie funkcyjne, w formie `const ... = () =>`.
- Eksport: zawsze **named export** (żadnych `export default`).

# Twarde ograniczenia
- Nie zmieniaj zachowania komponentu (brak zmian funkcjonalnych); dopuszczalne są refaktoryzacje porządkujące i drobne poprawki jakości.
- Nie dodawaj nowych zależności.
- Nie rozbijaj na nowe pliki.
- Nie zmieniaj publicznego API (propsy, nazwy eksportów) **poza wymuszeniem named export**; zachowaj nazwę komponentu.
- Nie zmieniaj stylu typów (`type` vs `interface`) jeżeli działa.

# Docelowa struktura pliku
Ułóż kod w tej kolejności (dodaj lekkie nagłówki sekcji komentarzami, jeśli to pomaga):
1. Importy
2. Eksportowane typy/kontrakty (np. `Props`)
3. Typy lokalne
4. Stałe / konfiguracje
5. Czyste helpery (bez React)
6. Subkomponenty (tylko jeśli realnie poprawiają czytelność)
7. Główny komponent(y)
8. Eksport(y) – wyłącznie named

# Importy (minimalne zasady)
- Usuń nieużywane importy.
- Jeśli import służy tylko do typów, użyj `import type`.
- Nie wykonuj ręcznego sortowania/grupowania importów, o ile nie jest to konieczne do usunięcia duplikatów lub oczywistego bałaganu (Biome i tak to domknie).

# Typy (TypeScript)
- Jeśli komponent ma propsy i nie ma czytelnego typu, dodaj `Props` (w stylu już użytym w pliku/repo; jeśli brak stylu, preferuj `type Props = { ... }`).
- Jeśli używane jest `children`, doprecyzuj typ (`children?: React.ReactNode` lub równoważnie) w sposób najmniej inwazyjny.
- Nie rób “wielkiej” przebudowy typów. Poprawiaj tylko oczywiste rzeczy (np. `any` w handlerach, jeśli łatwo doprecyzować).

# Komponent: wymagana forma (konwencja repo)
- Docelowo główny komponent ma formę:
    - `export const ComponentName = (props: Props) => { ... }`
    - albo, jeśli propsów brak: `export const ComponentName = () => { ... }`
- Nie używaj `function ComponentName()` jako głównej deklaracji.
- Usuń `export default` (zastąp named export; zachowaj nazwę komponentu).
- Nie wprowadzaj `React.FC` (jeśli już jest w pliku, zostaw).

# Wnętrze komponentu: porządek
W komponentach utrzymuj kolejność bloków:
1) destrukturyzacja propsów / wartości domyślne
2) hooki (`useState`, `useRef`, `useReducer`, …)
3) wartości pochodne (czytelne zmienne; `useMemo` tylko gdy ma sens)
4) handlery/callbacki (`useCallback` tylko gdy realnie potrzebne)
5) efekty (`useEffect`/`useLayoutEffect`) z poprawnymi zależnościami
6) przygotowanie danych do renderu (np. flagi, listy, warunki)
7) `return` (JSX)

Zasady dodatkowe:
- Hooki muszą być wywoływane zawsze w tym samym porządku (bez warunków).
- Jeśli JSX jest zbyt „gęsty”, wynieś złożone warunki do czytelnych zmiennych (bez zmiany logiki).
- Dla list w JSX zapewnij stabilne `key`.
- Unikaj tworzenia dużych obiektów/array inline w JSX, jeśli pogarsza to czytelność (wynieś do stałych/zmiennych).
- Nie dodawaj optymalizacji “na siłę” (`memo`, `useMemo`, `useCallback`) bez uzasadnienia.

# Tailwind (className)
- Uporządkuj `className` tak, aby był czytelny:
    - długie klasy rozbij na wielolinijkowy string/template literal, jeśli to pomaga,
    - powtarzające się zestawy klas wynieś do stałych,
    - warunkowe klasy zapisuj konsekwentnie (użyj istniejącego helpera `cn`/`clsx`/`classnames` tylko jeśli już jest w projekcie/pliku; nie dodawaj nowego).
- Nie zmieniaj znaczenia klas (nie usuwaj/nie podmieniaj “na oko”).

# Drobne poprawki jakości (mile widziane, jeśli bezpieczne)
- Usuń martwy kod, nieużywane zmienne, nieużywane funkcje, zakomentowane śmieci.
- Uprość oczywiste duplikacje (np. zduplikowane fragmenty JSX → helper renderujący).
- Jeśli `useEffect` ma ewidentnie zły dependency array, popraw go **tylko** gdy nie zmieniasz zachowania; w razie ryzyka zostaw komentarz `// TODO:` z krótkim uzasadnieniem.

# Wynik
1) Zastosuj zmiany w pliku (lub zaznaczeniu).
2) Na końcu wypisz maks. 5 punktów: co zostało uporządkowane / poprawione (krótko, rzeczowo).
