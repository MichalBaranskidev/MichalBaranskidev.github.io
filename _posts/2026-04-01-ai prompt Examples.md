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

## 📂 Przygotowanie plikow


cat > access.log << 'EOF'
192.168.1.10 GET /index.html 200 1234 0.123
192.168.1.20 GET /about.html 200 567 0.045
192.168.1.10 POST /api/login 401 234 0.078
192.168.1.30 GET /index.html 200 1234 0.098
EOF


---

## 📘 Wybieranie kolumn {#wybieranie-kolumn}


awk '{print $1, $3}' access.log


**Network:** analiza ruchu  
**DevOps:** analiza logów

---

## 📘 CSV i separator {#csv-i-separator}


awk -F',' '{print $2, $3}' users.csv


---

## 📘 Filtrowanie warunkowe {#filtrowanie-warunkowe}


awk '$5 > 500' access.log


---

## 📘 Dopasowanie wzorca {#dopasowanie-wzorca}


awk '$1 ~ /192.168.1.10/' access.log


---

## 📘 Ostatnia kolumna {#ostatnia-kolumna}


awk '{print $NF}' access.log


---

## 📘 Numerowanie linii {#numerowanie-linii}


awk '{print NR, $0}' access.log


---

## 📘 Sumowanie danych {#sumowanie-danych}


awk '{sum += $5} END {print sum}' access.log


---

## 📘 Zliczanie danych {#zliczanie-danych}


awk '{hits[$1]++} END {for (ip in hits) print hits[ip], ip}' access.log


---

## 📘 Dlugosc linii {#dlugosc-linii}


awk 'length($0) > 50' access.log


---

## 🎯 Wnioski {#wnioski}

- awk działa na kolumnach  
- można filtrować dane  
- można agregować dane  
- to narzędzie do analizy logów  