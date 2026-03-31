---
layout: post
title: "awk – praktyczne ćwiczenia"
date: 2026-03-31
categories: [Bash, DevOps]
tags: [awk, linux, logs, data-processing]
---

## 📋 Spis treści

1. [Przygotowanie plikow](#przygotowanie-plikow)
2. [Wybieranie kolumn](#wybieranie-kolumn)
3. [CSV i separator](#csv-i-separator)
4. [Filtrowanie warunkowe](#filtrowanie-warunkowe)
5. [Dopasowanie wzorca](#dopasowanie-wzorca)
6. [Ostatnia kolumna](#ostatnia-kolumna)
7. [Numerowanie linii](#numerowanie-linii)
8. [Sumowanie danych](#sumowanie-danych)
9. [Zliczanie danych](#zliczanie-danych)
10. [Dlugosc linii](#dlugosc-linii)
11. [Wnioski](#wnioski)

---
Przygotuj pliki testowe (5 min)
text

cat > access.log << 'EOF'
192.168.1.10 GET /index.html 200 1234 0.123
192.168.1.20 GET /about.html 200 567 0.045
192.168.1.10 POST /api/login 401 234 0.078
192.168.1.30 GET /index.html 200 1234 0.098
192.168.1.10 POST /api/login 200 789 0.234
192.168.1.40 GET /images/logo.png 404 345 0.012
192.168.1.50 GET /css/style.css 200 2345 0.345
192.168.1.20 GET /js/app.js 200 5678 0.567
192.168.1.60 GET /favicon.ico 404 123 0.021
192.168.1.10 POST /api/logout 200 456 0.089
EOF

cat > users.csv << 'EOF'
id,imie,nazwisko,wiek,miasto
1,Adam,Kowalski,30,Wrocław
2,Ewa,Nowak,25,Kraków
3,Piotr,Wiśniewski,35,Warszawa
4,Anna,Wójcik,28,Gdańsk
5,Tomasz,Kamiński,40,Poznań
EOF

📘 awk '{print $1, $3}' – wybieranie kolumn

Network: Z logów firewalla chcesz wyciągnąć adres źródłowy i docelowy.
text

awk '{print $1, $2}' /var/log/firewall.log

DevOps: Z logu aplikacji potrzebujesz timestamp i poziom logowania.
text

awk '{print $1, $3}' app.log

text

# 1. Wyciągnij adres IP i ścieżkę (kolumny 1 i 3)
awk '{print $1, $3}' access.log

🔍 Pytania:

    Co się dzieje, gdy w linii jest więcej spacji? Sprawdź.

text

echo "192.168.1.1     GET      /" | awk '{print $1, $3}'

    A gdy brakuje kolumny? Co wyświetli?

text

echo "192.168.1.1 GET" | awk '{print $1, $3}'

    Jak wyciągnąć tylko adres IP i kod odpowiedzi?

✍️ Wniosek: awk automatycznie dzieli linie na kolumny po spacjach/tabach. $0 to cała linia, $1 pierwsza kolumna itd.
📘 awk -F',' – zmiana separatora

Network: Plik konfiguracyjny routera jest w formacie CSV z przecinkami.
text

awk -F',' '{print $2, $3}' /config/routers.csv

DevOps: Przetwarzasz plik CSV z listą użytkowników.
text

awk -F',' '{print $2, $3, $4}' users.csv

text

# 1. Wyciągnij imię i nazwisko z CSV
awk -F',' '{print $2, $3}' users.csv

🔍 Pytania:

    Porównaj z domyślnym separatorem (spacja) – co się dzieje?

text

awk '{print $2, $3}' users.csv

    Użyj innego separatora, np. średnik – jak to zrobić?

text

echo "Adam;Kowalski;30" | awk -F';' '{print $1, $2}'

    Jak wyciągnąć tylko miasto (ostatnia kolumna) z CSV?

✍️ Wniosek: -F zmienia separator pól. Domyślnie to spacja/tab. Dla CSV trzeba ustawić -F','.
📘 awk '$4 > 30' – filtrowanie po warunku

Network: Chcesz zobaczyć tylko te połączenia, które trwały dłużej niż 0.5 sekundy.
text

awk '$6 > 0.5' access.log

DevOps: Wyciągasz tylko użytkowników powyżej 30 lat.
text

awk -F',' '$4 > 30 {print $2, $3, $4}' users.csv

text

# 1. Pokaż tylko linie, gdzie kolumna 5 (rozmiar) > 500
awk '$5 > 500' access.log

🔍 Pytania:

    Pokaż tylko linie, gdzie kod odpowiedzi to 404

text

awk '$4 == 404' access.log

    Połącz warunki – kod 404 i rozmiar > 300

text

awk '$4 == 404 && $5 > 300' access.log

    Pokaż tylko użytkowników z Warszawy (użyj CSV)

✍️ Wniosek: awk pozwala filtrować linie przez warunki na kolumnach. Można używać ==, >, <, &&, ||.
📘 awk '$1 ~ /192.168.1.10/' – dopasowanie do wzorca

Network: Chcesz zobaczyć tylko ruch z konkretnego adresu IP.
text

awk '$1 ~ /192\.168\.1\.10/' access.log

DevOps: Wyciągasz tylko użytkowników z Krakowa.
text

awk -F',' '$5 ~ /Kraków/' users.csv

text

# 1. Pokaż tylko linie z IP 192.168.1.10
awk '$1 ~ /192\.168\.1\.10/' access.log

🔍 Pytania:

    Pokaż linie, gdzie ścieżka zawiera "api"

text

awk '$3 ~ /api/' access.log

    Odwróć wzorzec – wszystko oprócz api

text

awk '$3 !~ /api/' access.log

    Pokaż użytkowników z miast na literę "W"

✍️ Wniosek: ~ oznacza "pasuje do wzorca", !~ oznacza "nie pasuje". Wzorce to wyrażenia regularne.
📘 awk '{print $NF}' – ostatnia kolumna

Network: Logi mają zmienną liczbę kolumn, ale chcesz zawsze wyciągnąć czas odpowiedzi.
text

awk '{print $NF}' access.log

DevOps: Ostatnia kolumna to kod błędu lub czas wykonania.
text

awk '{print $NF}' app.log

text

# 1. Wyciągnij ostatnią kolumnę (czas)
awk '{print $NF}' access.log

🔍 Pytania:

    A co z przedostatnią?

text

awk '{print $(NF-1)}' access.log

    Sprawdź na pliku ze zmienną liczbą kolumn

text

echo "a b c d e" | awk '{print $NF}'
echo "a b c" | awk '{print $NF}'

    Jak wyciągnąć 3 ostatnią kolumnę?

✍️ Wniosek: NF to liczba pól w bieżącej linii. $NF to ostatnie pole, $(NF-1) to przedostatnie.
📘 awk '{print NR, $0}' – numerowanie linii

Network: Przeglądasz konfigurację serwera i chcesz widzieć numery linii.
text

awk '{print NR, $0}' /etc/nginx/nginx.conf

DevOps: Debugując skrypt, chcesz widzieć numery linii błędów.
text

awk '{print NR, $0}' app.log | grep "ERROR"

text

# 1. Wyświetl log z numerami linii
awk '{print NR, $0}' access.log

🔍 Pytania:

    Znajdź błędy 404 z numerami linii

text

awk '{print NR, $0}' access.log | grep 404

    Zapisz do pliku z numerami

text

awk '{print NR, $0}' access.log > numerowany.log

    Jak zacząć numerowanie od 100?

✍️ Wniosek: NR to numer bieżącej linii (liczony od 1). Przydatne do odnajdywania miejsc w pliku.
📘 awk '{sum += $5} END {print sum}' – sumowanie

Network: Chcesz policzyć, ile łącznie danych przesłał Twój serwer WWW.
text

awk '{sum += $5} END {print "Suma bajtów:", sum}' access.log

DevOps: Sumujesz całkowity czas odpowiedzi API.
text

awk '{sum += $6} END {print "Czas całkowity:", sum}' access.log

text

# 1. Policz sumę rozmiarów odpowiedzi
awk '{sum += $5} END {print "Suma bajtów:", sum}' access.log

🔍 Pytania:

    Policz sumę tylko dla kodów 200

text

awk '$4 == 200 {sum += $5} END {print sum}' access.log

    Policz średnią (suma / liczba)

text

awk '{sum += $5; count++} END {print "Średnia:", sum/count}' access.log

    Policz minimalny i maksymalny rozmiar

✍️ Wniosek: awk może wykonywać obliczenia. Blok END wykonuje się po przeczytaniu wszystkich linii.
📘 awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' – zliczanie unikalnych

Network: Które IP najczęściej łączy się z serwerem?
text

awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' access.log | sort -nr

DevOps: Który typ błędu występuje najczęściej?
text

awk '{hits[$4]++} END {for (code in hits) print hits[code], code}' access.log | sort -nr

text

# 1. Policz, ile razy wystąpiło każde IP
awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' access.log

🔍 Pytania:

    Posortuj od najczęstszego

text

awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' access.log | sort -nr

    Policz kody odpowiedzi

text

awk '{hits[$4]++} END {for (code in hits) print hits[code], code}' access.log | sort -nr

    Które ścieżki (kolumna 3) są najczęściej odwiedzane?

✍️ Wniosek: awk może tworzyć tablice asocjacyjne (np. hits[ip]) i iterować po nich w bloku END.
📘 awk 'length($0) > 50' – filtrowanie po długości linii

Network: Chcesz znaleźć nietypowo długie linie w logach (mogą wskazywać na atak).
text

awk 'length($0) > 500' /var/log/apache2/access.log

DevOps: Szukasz w logach błędów z długim stack trace.
text

awk 'length($0) > 200' app.log

text

# 1. Pokaż linie dłuższe niż 50 znaków
awk 'length($0) > 50' access.log

🔍 Pytania:

    Pokaż linie krótsze niż 30 znaków

text

awk 'length($0) < 30' access.log

    Policz, ile jest długich linii

text

awk 'length($0) > 50 {count++} END {print count}' access.log

    Znajdź najdłuższą linię

✍️ Wniosek: length($0) zwraca długość bieżącej linii w znakach.