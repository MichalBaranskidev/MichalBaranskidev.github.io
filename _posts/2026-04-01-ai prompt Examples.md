BASH – DZIEŃ 2: awk (ciąg dalszy)
Przygotuj pliki testowe (5 min)
bash
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
awk '{print $1, $2}' /var/log/firewall.log
DevOps: Z logu aplikacji potrzebujesz timestamp i poziom logowania.
awk '{print $1, $3}' app.log
bash
# 1. Wyciągnij adres IP i ścieżkę (kolumny 1 i 3)
awk '{print $1, $3}' access.log

# 🔍 Pytania:
# 2. Co się dzieje, gdy w linii jest więcej spacji? Sprawdź.
echo "192.168.1.1     GET      /" | awk '{print $1, $3}'
# 3. A gdy brakuje kolumny? Co wyświetli?
echo "192.168.1.1 GET" | awk '{print $1, $3}'
# 4. Jak wyciągnąć tylko adres IP i kod odpowiedzi?
# ✍️ Wniosek: awk automatycznie dzieli linie na kolumny po spacjach/tabach. $0 to cała linia, $1 pierwsza kolumna itd.

📘 awk -F',' – zmiana separatora
Network: Plik konfiguracyjny routera jest w formacie CSV z przecinkami.
awk -F',' '{print $2, $3}' /config/routers.csv
DevOps: Przetwarzasz plik CSV z listą użytkowników.
awk -F',' '{print $2, $3, $4}' users.csv
bash
# 1. Wyciągnij imię i nazwisko z CSV
awk -F',' '{print $2, $3}' users.csv

# 🔍 Pytania:
# 2. Porównaj z domyślnym separatorem (spacja) – co się dzieje?
awk '{print $2, $3}' users.csv
# 3. Użyj innego separatora, np. średnik – jak to zrobić?
echo "Adam;Kowalski;30" | awk -F';' '{print $1, $2}'
# 4. Jak wyciągnąć tylko miasto (ostatnia kolumna) z CSV?
# ✍️ Wniosek: -F zmienia separator pól. Domyślnie to spacja/tab. Dla CSV trzeba ustawić -F','.

📘 awk '$4 > 30' – filtrowanie po warunku
Network: Chcesz zobaczyć tylko te połączenia, które trwały dłużej niż 0.5 sekundy.
awk '$6 > 0.5' access.log
DevOps: Wyciągasz tylko użytkowników powyżej 30 lat.
awk -F',' '$4 > 30 {print $2, $3, $4}' users.csv
bash
# 1. Pokaż tylko linie, gdzie kolumna 5 (rozmiar) > 500
awk '$5 > 500' access.log

# 🔍 Pytania:
# 2. Pokaż tylko linie, gdzie kod odpowiedzi to 404
awk '$4 == 404' access.log
# 3. Połącz warunki – kod 404 i rozmiar > 300
awk '$4 == 404 && $5 > 300' access.log
# 4. Pokaż tylko użytkowników z Warszawy (użyj CSV)
# ✍️ Wniosek: awk pozwala filtrować linie przez warunki na kolumnach. Można używać ==, >, <, &&, ||.

📘 awk '$1 ~ /192\.168\.1\.10/' – dopasowanie do wzorca
Network: Chcesz zobaczyć tylko ruch z konkretnego adresu IP.
awk '$1 ~ /192\.168\.1\.10/' access.log
DevOps: Wyciągasz tylko użytkowników z Krakowa.
awk -F',' '$5 ~ /Kraków/' users.csv
bash
# 1. Pokaż tylko linie z IP 192.168.1.10
awk '$1 ~ /192\.168\.1\.10/' access.log

# 🔍 Pytania:
# 2. Pokaż linie, gdzie ścieżka zawiera "api"
awk '$3 ~ /api/' access.log
# 3. Odwróć wzorzec – wszystko oprócz api
awk '$3 !~ /api/' access.log
# 4. Pokaż użytkowników z miast na literę "W"
# ✍️ Wniosek: ~ oznacza "pasuje do wzorca", !~ oznacza "nie pasuje". Wzorce to wyrażenia regularne.

📘 awk '{print $NF}' – ostatnia kolumna
Network: Logi mają zmienną liczbę kolumn, ale chcesz zawsze wyciągnąć czas odpowiedzi.
awk '{print $NF}' access.log
DevOps: Ostatnia kolumna to kod błędu lub czas wykonania.
awk '{print $NF}' app.log
bash
# 1. Wyciągnij ostatnią kolumnę (czas)
awk '{print $NF}' access.log

# 🔍 Pytania:
# 2. A co z przedostatnią?
awk '{print $(NF-1)}' access.log
# 3. Sprawdź na pliku ze zmienną liczbą kolumn
echo "a b c d e" | awk '{print $NF}'
echo "a b c" | awk '{print $NF}'
# 4. Jak wyciągnąć 3 ostatnią kolumnę?
# ✍️ Wniosek: NF to liczba pól w bieżącej linii. $NF to ostatnie pole, $(NF-1) to przedostatnie.

📘 awk '{print NR, $0}' – numerowanie linii
Network: Przeglądasz konfigurację serwera i chcesz widzieć numery linii.
awk '{print NR, $0}' /etc/nginx/nginx.conf
DevOps: Debugując skrypt, chcesz widzieć numery linii błędów.
awk '{print NR, $0}' app.log | grep "ERROR"
bash
# 1. Wyświetl log z numerami linii
awk '{print NR, $0}' access.log

# 🔍 Pytania:
# 2. Znajdź błędy 404 z numerami linii
awk '{print NR, $0}' access.log | grep 404
# 3. Zapisz do pliku z numerami
awk '{print NR, $0}' access.log > numerowany.log
# 4. Jak zacząć numerowanie od 100?
# ✍️ Wniosek: NR to numer bieżącej linii (liczony od 1). Przydatne do odnajdywania miejsc w pliku.

📘 awk '{sum += $5} END {print sum}' – sumowanie
Network: Chcesz policzyć, ile łącznie danych przesłał Twój serwer WWW.
awk '{sum += $5} END {print "Suma bajtów:", sum}' access.log
DevOps: Sumujesz całkowity czas odpowiedzi API.
awk '{sum += $6} END {print "Czas całkowity:", sum}' access.log
bash
# 1. Policz sumę rozmiarów odpowiedzi
awk '{sum += $5} END {print "Suma bajtów:", sum}' access.log

# 🔍 Pytania:
# 2. Policz sumę tylko dla kodów 200
awk '$4 == 200 {sum += $5} END {print sum}' access.log
# 3. Policz średnią (suma / liczba)
awk '{sum += $5; count++} END {print "Średnia:", sum/count}' access.log
# 4. Policz minimalny i maksymalny rozmiar
# ✍️ Wniosek: awk może wykonywać obliczenia. Blok END wykonuje się po przeczytaniu wszystkich linii.

📘 awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' – zliczanie unikalnych
Network: Które IP najczęściej łączy się z serwerem?
awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' access.log | sort -nr
DevOps: Który typ błędu występuje najczęściej?
awk '{hits[$4]++} END {for (code in hits) print hits[code], code}' access.log | sort -nr
bash
# 1. Policz, ile razy wystąpiło każde IP
awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' access.log

# 🔍 Pytania:
# 2. Posortuj od najczęstszego
awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' access.log | sort -nr
# 3. Policz kody odpowiedzi
awk '{hits[$4]++} END {for (code in hits) print hits[code], code}' access.log | sort -nr
# 4. Które ścieżki (kolumna 3) są najczęściej odwiedzane?
# ✍️ Wniosek: awk może tworzyć tablice asocjacyjne (np. hits[ip]) i iterować po nich w bloku END.

📘 awk 'length($0) > 50' – filtrowanie po długości linii
Network: Chcesz znaleźć nietypowo długie linie w logach (mogą wskazywać na atak).
awk 'length($0) > 500' /var/log/apache2/access.log
DevOps: Szukasz w logach błędów z długim stack trace.
awk 'length($0) > 200' app.log
bash
# 1. Pokaż linie dłuższe niż 50 znaków
awk 'length($0) > 50' access.log

# 🔍 Pytania:
# 2. Pokaż linie krótsze niż 30 znaków
awk 'length($0) < 30' access.log
# 3. Policz, ile jest długich linii
awk 'length($0) > 50 {count++} END {print count}' access.log
# 4. Znajdź najdłuższą linię
# ✍️ Wniosek: length($0) zwraca długość bieżącej linii w znakach.
