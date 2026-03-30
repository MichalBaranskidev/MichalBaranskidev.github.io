---
layout: post
title: Git Disaster Recovery – case study
date: 2026-03-28
categories: [Git, DevOps]
tags: [git, disaster-recovery, reflog, tag, detached-head]
---

<style>
  .note-box {
    background: #f6f8fa;
    border-left: 4px solid #0969da;
    padding: 1rem;
    margin: 1rem 0;
    border-radius: 6px;
  }
  .warning-box {
    background: #fff8e7;
    border-left: 4px solid #e6a700;
    padding: 1rem;
    margin: 1rem 0;
    border-radius: 6px;
  }
  .tip-box {
    background: #e6ffec;
    border-left: 4px solid #2da44e;
    padding: 1rem;
    margin: 1rem 0;
    border-radius: 6px;
  }
  pre {
    background: #f6f8fa;
    padding: 1rem;
    border-radius: 6px;
    overflow-x: auto;
    border: 1px solid #d0d7de;
  }
  code {
    background: #f6f8fa;
    padding: 0.2rem 0.4rem;
    border-radius: 4px;
    font-size: 0.9em;
  }
  table {
    border-collapse: collapse;
    width: 100%;
    margin: 1rem 0;
  }
  th, td {
    border: 1px solid #d0d7de;
    padding: 0.5rem;
    text-align: left;
  }
  th {
    background: #f6f8fa;
  }
  hr {
    margin: 2rem 0;
  }
  h1, h2, h3 {
    margin-top: 1.5rem;
  }
</style>

# Git Disaster Recovery – case study

## 📋 Spis treści

1. [Sytuacja wyjściowa](#sytuacja-wyjsciowa)
2. [Co poszło nie tak](#co-poszlo-nie-tak)
3. [Dlaczego Git tak działa](#dlaczego-git-tak-dziala)
4. [Proces odzyskiwania](#proces-odzyskiwania)
5. [Kluczowe komendy](#kluczowe-komendy)
6. [Zasady Gita](#zasady-gita)
7. [Scenariusze](#scenariusze)
8. [Lekcje na przyszłość](#lekcje-na-przyszlosc)
9. [Wnioski](#wnioski)
10. [Mantry na dziś](#mantry-na-dzis)
11. [Podsumowanie](#podsumowanie)

---

## <span id="sytuacja-wyjsciowa">Sytuacja wyjściowa</span>

- Pracowałem 5 dni nad notatkami w folderze `08-Git/`
- Pliki były commitowane i śledzone na branchu `main/recovery`
- Zobaczyłem "wiszące" commity (`git fsck --lost-found`)
- Chciałem je "uratować" tworząc z nich nowe branche

---

## <span id="co-poszlo-nie-tak">Co poszło nie tak</span>

**Krok 1:** Byłem na `recovery` (tam były notatki)

```bash
git checkout -b detacze2 56230ff

Krok 2: Commit 56230ff pochodził z epoki ZANIM powstał folder 08-Git

    W tym commitcie NIE BYŁO moich notatek

    Git przełączając mnie na ten commit usunął folder 08-Git z dysku

    Working directory = stan z commita = bez moich plików

Krok 3: Przełączałem się między branchami
bash

git checkout detacze   # tu były notatki (commit 5d66893)
git checkout detacze2  # tu nie było notatek (commit 56230ff)

Krok 4: Przy każdym wejściu na detacze2 git kasował folder 08-Git

    Pliki fizycznie znikały z dysku

    Ale w repo były bezpieczne na innych branchach

<span id="dlaczego-git-tak-dziala">Dlaczego Git tak działa</span>

    Commit bez plików + checkout = usunięcie z working directory

    Nie zrobiłem tagu/backupu PRZED skokiem w przeszłość

    Myślałem że "ratuję" commity, a wszedłem w alternatywną rzeczywistość

<span id="proces-odzyskiwania">Proces odzyskiwania</span>

Krok 1: ZACHOWAJ SPOKÓJ

Krok 2: ZRÓB BACKUP CAŁEGO REPO
bash

git bundle create backup.bundle --all
mv backup.bundle ~/Desktop/

Krok 3: SPRAWDŹ GDZIE JESTEŚ
bash

git status
git branch
git reflog --date=format:%Y-%m-%d_%H:%M:%S

Krok 4: ZNAJDŹ COMMITY Z TWOIMI PLIKAMI
bash

git branch -a | while read b; do
  git ls-tree $b --name-only 2>/dev/null | grep "08-Git" && echo "MA: $b"
done

Krok 5: PRZYWRÓĆ PLIKI Z INNEGO BRANCHA
bash

git checkout recovery
git checkout e1025e0 -- 08-Git/

Krok 6: ZABEZPIECZ
bash

git tag stan-po-odzysku-$(date +%Y%m%d_%H%M)

Krok 7: DODAJ PLIKI Z POWROTEM DO REPO
bash

git add 08-Git/
git commit -m "recovery: przywrócone notatki po katastrofie"

<span id="kluczowe-komendy">Kluczowe komendy</span>
Komenda	Zastosowanie
git bundle create backup.bundle --all	Pełny backup repo
git reflog --date=format:%Y-%m-%d_%H:%M:%S	Historia z datami
git fsck --lost-found	Znajduje wiszące commity
git ls-tree <hash> --name-only	Sprawdza pliki w commitcie
git checkout <hash> -- <file>	Przywraca plik z commita
git tag nazwa	Znacznik (niezmienny punkt)
git merge --no-ff	Merge z widoczną historią
<span id="zasady-gita">Zasady Gita</span>

Zasada #1: Git podmienia working directory do stanu z commita przy każdym checkout

Zasada #2: Commit bez pliku = git usunie ten plik z dysku przy przełączeniu

Zasada #3: Git nie ostrzega przy tworzeniu brancha z commita – ostrzega DOPIERO przy przełączaniu

Zasada #4: Tagi są niezmienne (immutable), branche się zmieniają – tag = backup, branch = praca
<span id="scenariusze">Scenariusze</span>
Akcja	Jesteś na	Skutek
git checkout 56230ff	Detached HEAD	Możesz stracić zmiany
git checkout -b detacze2 56230ff	Nowy branch	Wchodzisz do innej rzeczywistości
git checkout recovery	Branch	Wracasz do swoich plików
git checkout detacze2 (mając pliki)	Branch bez plików	Git kasuje pliki z dysku
<span id="lekcje-na-przyszlosc">Lekcje na przyszłość</span>

Zanim przełączysz branch/commit:
bash

git status                    # musi być czysto!
git tag zapas-$(date +%Y%m%d) # zrób znacznik

Zanim wejdziesz na stary commit:
bash

# nigdy nie wchodź bezpośrednio!
git checkout -b nowy-branch stary-commit

Jak sprawdzić co jest w commitcie zanim wejdziesz:
bash

git ls-tree --name-only <hash> | grep "08-Git"
git diff --name-only <hash>..HEAD

<span id="wnioski">Wnioski</span>

    Git nie kasuje danych – one są w historii, trzeba wiedzieć gdzie szukać

    Reflog to pamięć – zawiera wszystko co robiłeś przez ostatnie miesiące

    Tagi to twoi przyjaciele – rób je przed każdym ryzykownym ruchem

    Branch z commita to nowa rzeczywistość – nie ten sam branch

    Working directory to nie repo – to tylko migawka z commita

<span id="mantry-na-dzis">Mantry na dziś</span>
<div class="tip-box"> ✅ "Commit zanim przełączysz"<br> ✅ "Tag zanim skoczysz"<br> ✅ "Status zanim cokolwiek zrobisz" </div>
<span id="podsumowanie">Podsumowanie – czego się nauczyłem</span>
Czego się nauczyłem	Jak sprawdzić / komenda
Sprawdzać detached HEAD	git symbolic-ref HEAD
Robić TAG przed skokiem	git tag backup-$(date)
Wchodzić na stary commit przez branch	git checkout -b nowy stary
Nie ufać branchom jako backup	Bo się zmieniają
Bundle trzymać POZA repo	mv bundle ~/Desktop/
Sprawdzać co jest w commitcie	git ls-tree <hash>
Szukać plików po całym repo	git branch -a | while read b
Porównywać daty	stat vs git log
VSC zmienia nazwy	Spacje → myślniki
Flaga --no-ff	Zostawia ślad w historii
Rozwiązywać konflikt	git checkout --theirs/--ours
Przerwać merga	git merge --abort

Michał Barański – IT professional transitioning into DevOps