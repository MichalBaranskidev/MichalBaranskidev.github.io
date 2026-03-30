---
layout: post
title: Git Disaster Recovery – case study
date: 2026-03-28
categories: [Git, DevOps]
tags: [git, disaster-recovery, reflog, tag, detached-head]
---

## 📋 Spis treści

**1.** [Sytuacja wyjściowa](#sytuacja-wyjsciowa)  
**2.** [Co poszło nie tak](#co-poszlo-nie-tak)  
**3.** [Dlaczego Git tak działa](#dlaczego-git-tak-dziala)  
**4.** [Proces odzyskiwania](#proces-odzyskiwania)  
**5.** [Kluczowe komendy](#kluczowe-komendy)  
**6.** [Zasady Gita](#zasady-gita)  
**7.** [Scenariusze](#scenariusze)  
**8.** [Lekcje na przyszłość](#lekcje-na-przyszlosc)  
**9.** [Wnioski końcowe](#wnioski-koncowe)  
**10.** [Podsumowanie – czego się nauczyłem](#podsumowanie---czego-sie-nauczylem)

---

## Sytuacja wyjściowa

- Pracowałem 5 dni nad notatkami w folderze `08-Git/`
- Pliki były commitowane i śledzone na branchu `main/recovery`
- Zobaczyłem "wiszące" commity (`git fsck --lost-found`)
- Chciałem je "uratować" tworząc z nich nowe branche

---

## Co poszło nie tak

**Krok 1:** Byłem na `recovery` (tam były notatki)

    git checkout -b detacze2 56230ff

**Krok 2:** Commit `56230ff` pochodził z epoki ZANIM powstał folder `08-Git`

- W tym commitcie NIE BYŁO moich notatek
- Git przełączając mnie na ten commit usunął folder `08-Git` z dysku
- Working directory = stan z commita = bez moich plików

**Krok 3:** Przełączałem się między branchami

    git checkout detacze   # tu były notatki (commit 5d66893)
    git checkout detacze2  # tu nie było notatek (commit 56230ff)

**Krok 4:** Przy każdym wejściu na `detacze2` git kasował folder `08-Git`

- Pliki fizycznie znikały z dysku
- Ale w repo były bezpieczne na innych branchach

---

## Dlaczego Git tak działa

- Commit bez plików + checkout = usunięcie z working directory
- Nie zrobiłem tagu/backupu PRZED skokiem w przeszłość
- Myślałem że "ratuję" commity, a wszedłem w alternatywną rzeczywistość

---

## Proces odzyskiwania

**Krok 1:** ZACHOWAJ SPOKÓJ

**Krok 2:** ZRÓB BACKUP CAŁEGO REPO

    git bundle create backup.bundle --all
    mv backup.bundle ~/Desktop/

**Krok 3:** SPRAWDŹ GDZIE JESTEŚ

    git status
    git branch
    git reflog --date=format:%Y-%m-%d_%H:%M:%S

**Krok 4:** ZNAJDŹ COMMITY Z TWOIMI PLIKAMI

    git branch -a | while read b; do
      git ls-tree $b --name-only 2>/dev/null | grep "08-Git" && echo "MA: $b"
    done

**Krok 5:** PRZYWRÓĆ PLIKI Z INNEGO BRANCHA

    git checkout recovery
    git checkout e1025e0 -- 08-Git/

**Krok 6:** ZABEZPIECZ

    git tag stan-po-odzysku-$(date +%Y%m%d_%H%M)

**Krok 7:** DODAJ PLIKI Z POWROTEM DO REPO

    git add 08-Git/
    git commit -m "recovery: przywrócone notatki po katastrofie"

---

## Kluczowe komendy

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

## Zasady Gita

**Zasada #1:** Git podmienia working directory do stanu z commita przy każdym checkout

**Zasada #2:** Commit bez pliku = git usunie ten plik z dysku przy przełączeniu

**Zasada #3:** Git nie ostrzega przy tworzeniu brancha z commita – ostrzega DOPIERO przy przełączaniu

**Zasada #4:** Tagi są niezmienne (immutable), branche się zmieniają – tag = backup, branch = praca

---

## Scenariusze

| Akcja | Jesteś na | Skutek |
|-------|-----------|--------|
| `git checkout 56230ff` | Detached HEAD | Możesz stracić zmiany |
| `git checkout -b detacze2 56230ff` | Nowy branch | Wchodzisz do innej rzeczywistości |
| `git checkout recovery` | Branch | Wracasz do swoich plików |
| `git checkout detacze2` (mając pliki) | Branch bez plików | Git kasuje pliki z dysku |

---

## Lekcje na przyszłość

**Zanim przełączysz branch/commit:**

    git status                     # musi być czysto!
    git tag zapas-$(date +%Y%m%d)  # zrób znacznik

**Zanim wejdziesz na stary commit:**

    # nigdy nie wchodź bezpośrednio!
    git checkout -b nowy-branch stary-commit

**Jak sprawdzić co jest w commitcie zanim wejdziesz:**

    git ls-tree --name-only <hash> | grep "08-Git"
    git diff --name-only <hash>..HEAD

---

## Wnioski końcowe

1. Git nie kasuje danych – one są w historii, trzeba wiedzieć gdzie szukać
2. Reflog to pamięć – zawiera wszystko co robiłeś przez ostatnie miesiące
3. Tagi to twoi przyjaciele – rób je przed każdym ryzykownym ruchem
4. Branch z commita to nowa rzeczywistość – nie ten sam branch
5. Working directory to nie repo – to tylko migawka z commita

---

## Podsumowanie – czego się nauczyłem

| Czego się nauczyłem | Jak sprawdzić / komenda |
|---------------------|------------------------|
| Sprawdzać detached HEAD | `git symbolic-ref HEAD` |
| Robić TAG przed skokiem | `git tag backup-$(date)` |
| Wchodzić na stary commit przez branch | `git checkout -b nowy stary` |
| Nie ufać branchom jako backup | Bo się zmieniają |
| Bundle trzymać POZA repo | `mv bundle ~/Desktop/` |
| Sprawdzać co jest w commitcie | `git ls-tree <hash>` |
| Szukać plików po całym repo | `git branch -a \| while read b` |
| Porównywać daty | `stat` vs `git log` |
| VSC zmienia nazwy | Spacje → myślniki |
| Flaga --no-ff | Zostawia ślad w historii |
| Rozwiązywać konflikt | `git checkout --theirs/--ours` |
| Przerwać merga | `git merge --abort` |

---

**Michał Barański – IT professional transitioning into DevOps**