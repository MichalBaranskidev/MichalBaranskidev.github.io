---
layout: post
title: "Git Rebase -i – jak oczyścić historię commitów przed rekruterem"
date: 2026-04-01
categories: [Git, DevOps]
tags: [git, rebase, history, cleanup, tutorial]
---

## Dlaczego historia commitów jest ważna?

Twój GitHub to Twoje CV. Rekruterzy techniczni patrzą na:

- Jak często commitujesz
- Co piszesz w commitach
- Czy umiesz utrzymać czystą historię
- Czy używasz konwencji nazewnictwa

**Commity typu `omg`, `kurwa`, `fix` nie robią dobrego wrażenia.**

---

## Czego się nauczysz?

1. Jak zrobić backup przed rebase
2. Jak zmienić nazwy commitów
3. Jak usunąć niechciane foldery z historii
4. Jak bezpiecznie wypchnąć zmienioną historię
5. Jak posprzątać po sobie

---

## Krok 1: Backup – zanim cokolwiek zrobisz

```bash
# Stwórz lokalny branch backup
git branch backup-przed-rebase

# Opcjonalnie: wypchnij na GitHub (ale potem usuń!)
# git push origin backup-przed-rebase

    ⚠️ Uwaga: Jeśli wypchniesz backup na GitHub, Twoje kompromitujące commity będą tam widoczne! Lepiej trzymaj backup lokalnie.

Krok 2: Sprawdź co chcesz zmienić
bash

git log --oneline

Przykładowy log przed zmianą:
text

bc28aa8 omg
e2233c7 notattata
9b29e3a kuva
215297b kurwa2
807cdbf kurwa
1cfbe56 final - notatka gotowa do CVs

Krok 3: Uruchom interaktywny rebase

Jeśli chcesz zmienić commity od konkretnego miejsca:
bash

# Znajdź hash commita SPRZED tych do zmiany
git rebase -i 1cfbe56

Co się dzieje? Otwiera się edytor z listą commitów od wskazanego miejsca.
Krok 4: Zmień pick na reword w edytorze

Zobaczysz coś takiego:
text

pick 807cdbf kurwa
pick 215297b kurwa2
pick 9b29e3a kuva
pick e2233c7 notattata
pick bc28aa8 omg

W Vimie – szybka zamiana wszystkich:
vim

:1,5s/^pick/reword/g
:wq

Co to robi: Zamienia wszystkie pick na reword w pierwszych 5 liniach.
Krok 5: Zmień nazwy commitów

Git otworzy edytor dla KAŻDEGO commita. Dla każdego:

    Usuń starą nazwę – w Vimie użyj cc (usuwa linię i włącza tryb edycji)

    Wpisz nową nazwę – zgodnie z konwencją

    Zapisz i wyjdź – :wq

Mapowanie starych → nowych nazw:
Stary commit	Nowa nazwa
kurwa	chore: initial setup
kurwa2	chore: backup old files
kuva	chore: initial blog structure
notattata	style: update blog layout
omg	feat: add AI prompt system post
Krok 6: Konwencja commitów
Typ	Zastosowanie	Przykład
feat	Nowa funkcja	feat: add dark mode toggle
fix	Poprawka błędu	fix: broken link in about page
docs	Dokumentacja	docs: update README
style	Formatowanie, CSS	style: format markdown files
chore	Narzędzia, konfiguracja	chore: update Jekyll version
refactor	Zmiana struktury kodu	refactor: simplify layout logic
Krok 7: Sprawdź efekt
bash

git log --oneline -10

Powinieneś zobaczyć:
text

a1b2c3d feat: add AI prompt system post
e4f5g6h style: update blog layout
i7j8k9l chore: initial blog structure
m0n1p2q chore: backup old files
r3s4t5u chore: initial setup

Krok 8: Wymuś push na GitHub
bash

git push --force origin main

    🔐 Bezpieczniejsza opcja:
    bash

    git push --force-with-lease origin main

Dlaczego --force? Zmieniłeś historię, więc normalny push nie zadziała.
Krok 9: Posprzątaj po sobie
bash

# Usuń lokalny backup
git branch -D backup-przed-rebase

# Jeśli wypchnąłeś backup na GitHub – usuń go!
# git push origin --delete backup-przed-rebase

🚨 BONUS: Jak usunąć niechciany folder z repozytorium

Czasem zdarzy się, że folder backup-stare-pliki lub kurwa trafi na GitHub. Oto jak go usunąć:
Opcja 1: Usuń i popraw ostatni commit
bash

# Usuń folder
git rm -r backup-stare-pliki

# Dodaj do ostatniego commita
git commit --amend -m "feat: add AI prompt system post"

# Wymuś push
git push --force origin main

Opcja 2: Usuń folder z całej historii (radykalne)
bash

git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch -r backup-stare-pliki" \
  --prune-empty --tag-name-filter cat -- --all

git push --force origin main

🔧 Przydatne komendy w Vimie dla rebase
Co chcesz zrobić	Komenda
Usunąć linię i przejść do edycji	cc
Usunąć całą linię	dd
Usunąć jeden wyraz	dw
Zapisać i wyjść	:wq
Wyjść bez zapisu	:q!
Cofnąć	u
📋 Checklist przed rebase

    Zrobiłem backup (git branch backup)

    Sprawdziłem które commity chcę zmienić

    Mam listę nowych nazw commitów

    Wiem jak używać Vima (lub innego edytora)

    Jestem na swoim branchu (nie współdzielonym)

⚠️ Najczęstsze błędy
Błąd	Rozwiązanie
error: cannot rebase: Your index contains uncommitted changes	Zrób git stash lub git commit
error: the branch is not fully merged	Użyj git branch -D do wymuszonego usunięcia
failed to push	Użyj --force-with-lease zamiast --force
🎯 Podsumowanie

Czysta historia commitów = profesjonalny developer.
Przed	Po
omg	feat: add AI prompt system post
kurwa	chore: initial setup
fix	fix: correct broken link

Narzędzia, które poznałeś:

    git rebase -i – interaktywne zmienianie historii

    git commit --amend – poprawianie ostatniego commita

    git push --force – nadpisywanie historii na GitHub

    git filter-branch – masowe usuwanie plików z historii

📚 Źródła

    Git Documentation - Rebase

    Conventional Commits

P.S. Ta notatka powstała dzięki prawdziwej historii – w moim repozytorium znalazły się commity typu kurwa i omg. Git rebase -i uratował mnie przed wstydem przed rekruterem. Niech i Ciebie uratuje! 😄
text


---