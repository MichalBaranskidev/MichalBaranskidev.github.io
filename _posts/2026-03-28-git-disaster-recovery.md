---
layout: default
title: "Git Disaster Recovery – case study"
date: 2026-03-28
categories: [Git, DevOps]
tags: [git, disaster-recovery, reflog, tag, detached-head]
---
---

# Git Disaster Recovery – case study

## SYTUACJA WYJŚCIOWA
## SYTUACJA WYJŚCIOWA

- Pracowałem 5 dni nad notatkami w folderze `08-Git/`
- Pliki były commitowane i śledzone na branchu `main/recovery`
- Zobaczyłem "wiszące" commity (`git fsck --lost-found`)
- Chciałem je "uratować" tworząc z nich nowe branche

---

## CO POSZŁO NIE TAK

**Krok 1:** Byłem na `recovery` (tam były notatki)

git checkout -b detacze2 56230ff
text


**Krok 2:** Commit `56230ff` pochodził z epoki ZANIM powstał folder `08-Git`

- W tym commitcie NIE BYŁO moich notatek
- Git przełączając mnie na ten commit usunął folder `08-Git` z dysku
- Working directory = stan z commita = bez moich plików

**Krok 3:** Przełączałem się między branchami

git checkout detacze # tu były notatki (commit 5d66893)
git checkout detacze2 # tu nie było notatek (commit 56230ff)
text


**Krok 4:** Przy każdym wejściu na `detacze2` git kasował folder `08-Git`

- Pliki fizycznie znikały z dysku
- Ale w repo były bezpieczne na innych branchach

---

## DLACZEGO SIĘ STAŁO

- Commit bez plików + checkout = usunięcie z working directory
- Nie zrobiłem tagu/backupu PRZED skokiem w przeszłość
- Myślałem że "ratuję" commity, a wszedłem w alternatywną rzeczywistość

---

## PROCES ODZYSKIWANIA

**Krok 1:** ZACHOWAJ SPOKÓJ

**Krok 2:** ZRÓB BACKUP CAŁEGO REPO

git bundle create backup.bundle --all
mv backup.bundle ~/Desktop/
text


**Krok 3:** SPRAWDŹ GDZIE JESTEŚ

git status
git branch
git reflog --date=format:%Y-%m-%d_%H:%M:%S
text


**Krok 4:** ZNAJDŹ COMMITY Z TWOIMI PLIKAMI

git branch -a | while read b; do
git ls-tree $b --name-only 2>/dev/null | grep "08-Git" && echo "MA: $b"
done
text


**Krok 5:** PRZYWRÓĆ PLIKI Z INNEGO BRANCHA

git checkout recovery
git checkout e1025e0 -- 08-Git/"git tag"
text


**Krok 6:** ZABEZPIECZ

git tag stan-po-odzysku-$(date +%Y%m%d_%H%M)
text


**Krok 7:** DODAJ PLIKI Z POWROTEM DO REPO

git add 08-Git/
git commit -m "recovery: przywrócone notatki po katastrofie"
text


---

## KLUCZOWE KOMENDY

| Komenda | Zastosowanie |
|---------|--------------|
| `git bundle create backup.bundle --all` | Pełny backup repo |
| `git reflog --date=format:%Y-%m-%d_%H:%M:%S` | Historia z datami |
| `git fsck --lost-found` | Znajduje wiszące commity |
| `git ls-tree <hash> --name-only` | Sprawdza pliki w commitcie |
| `git checkout <hash> -- <file>` | Przywraca plik z commita |
| `git tag nazwa` | Znacznik (niezmienny punkt) |
| `git merge --no-ff` | Merge z widoczną historią |

---

## ZASADY GITA

**Zasada #1:** Git podmienia working directory do stanu z commita przy każdym checkout

**Zasada #2:** Commit bez pliku = git usunie ten plik z dysku przy przełączeniu

**Zasada #3:** Git nie ostrzega przy tworzeniu brancha z commita – ostrzega DOPIERO przy przełączaniu

**Zasada #4:** Tagi są niezmienne (immutable), branche się zmieniają – tag = backup, branch = praca

---

## SCENARIUSZE

| Akcja | Jesteś na | Skutek |
|-------|-----------|--------|
| `git checkout 56230ff` | Detached HEAD | Możesz stracić zmiany |
| `git checkout -b detacze2 56230ff` | Nowy branch | Wchodzisz do innej rzeczywistości |
| `git checkout recovery` | Branch | Wracasz do swoich plików |
| `git checkout detacze2` (mając pliki) | Branch bez plików | Git kasuje pliki z dysku |

---

## LEKCJE NA PRZYSZŁOŚĆ

**Zanim przełączysz branch/commit:**

git status # musi być czysto!
git tag zapas-$(date +%Y%m%d) # zrób znacznik
text


**Zanim wejdziesz na stary commit:**

nigdy nie wchodź bezpośrednio!

git checkout -b nowy-branch stary-commit
text


**Jak sprawdzić co jest w commitcie zanim wejdziesz:**

git ls-tree --name-only <hash> | grep "08-Git"
git diff --name-only <hash>..HEAD
text


---

## WNIOSKI

1. Git nie kasuje danych – one są w historii, trzeba wiedzieć gdzie szukać
2. Reflog to pamięć – zawiera wszystko co robiłeś przez ostatnie miesiące
3. Tagi to twoi przyjaciele – rób je przed każdym ryzykownym ruchem
4. Branch z commita to nowa rzeczywistość – nie ten sam branch
5. Working directory to nie repo – to tylko migawka z commita

---

## MANTRY NA DZIŚ

- "Commit zanim przełączysz"
- "Tag zanim skoczysz"
- "Status zanim cokolwiek zrobisz"

---

**KONIEC NOTATKI**

---

*Michał Barański – IT professional transitioning into DevOps*
