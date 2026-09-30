# Сесия 2 — Nmap и ffuf

## Сценарий
GameStudio Neo OOD — лабораторна проверка на тестов Game Server и уеб заместител на Jenkins.

## Обхват
- `10.202.0.10` — `srv`, лабораторен Game Server заместител.
- `10.200.0.1` — `gw`, уеб заместител на Jenkins за упражнението.

## Изпълнени проверки
- TCP Nmap: `22/tcp` е `closed`; `7777/tcp` е `open` и връща `GameStudio Neo test server`.
- UDP Nmap: `7777/udp` е `open`.
- TCP Nmap: `10.200.0.1:8080` е `open`.
- ffuf с wordlist `index.html`, `builds.txt`, `admin`, `missing` откри `index.html` и `builds.txt` с HTTP status `200`.

## Ограничения
- `7777` е Python тестов listener, а не реален production Game Server.
- `8080` е Python HTTP server и не доказва наличието на Jenkins.
- `10.200.0.10` (CI/CD адресът от заданието) не е реализиран като отделна VM.
- `22/tcp` е затворен; SSH deployment не е демонстриран.
- QA клиентите, В. Търново мрежата и firewall правилата от урок 9 предстои да бъдат реализирани.

## Команди
```bash
sudo nmap -sT -sV -p 22,7777 -oN nmap-game-tcp.txt 10.202.0.10
sudo nmap -sU -p 7777 -oN nmap-game-udp.txt 10.202.0.10
nmap -sT -p 8080 -oN nmap-web-tcp.txt 10.200.0.1
ffuf -w paths.txt -u http://10.200.0.1:8080/FUZZ -mc 200 -o ffuf-web.json -of json
```
