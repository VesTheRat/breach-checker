# Healthcheck backendu
## Pełny wynik `docker compose up -d --build`:
```
[+] up 6/6
 ✔ Image breach-checker-frontend       Built                                                                     1.4s
 ✔ Image breach-checker-backend        Built                                                                     1.4s
 ✔ Network breach-checker_default      Created                                                                   0.1s
 ✔ Container breach-checker-db-1       Healthy                                                                   5.7s
 ✔ Container breach-checker-backend-1  Healthy                                                                  11.4s
 ✔ Container breach-checker-frontend-1 Started                                                                  11.6s

```

## Wynik `docker compose ps`:
```
$ docker compose ps
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED          STATUS                    PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    25 seconds ago   Up 19 seconds (healthy)   5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         25 seconds ago   Up 25 seconds (healthy)   5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   25 seconds ago   Up 13 seconds             0.0.0.0:8080->80/tcp, [::]:8080->80/tcp

```

# 2. Trwałość bazy
## Wynik odpowiedzi API po `INSERT`:
```
{"found":true}

```

## Wynik odpowiedzi API po `down` + `up`:
```
{"found":true}

```

## Wynik odpowiedzi API po `down -v` + `up`:
```
{"found":false}

```

## Dlaczego skrót przetrwał `down`, a nie przetrwał `down -v`?
Skrót został zapisany w wolumenie kontenera, dlatego przetrwał `down`, które nie usuwa tego wolumenu, ale nie przetrwał `down -v`, ponieważ po tym poleceniu wolumen kontenera został usunięty

## Dlaczego po `down -v` w bazie znów są 89 skróty z próbki, choć cała baza została skasowana?
Ponieważ komenda `up` wystartowała kontener z domyślnego skryptu inicjalizacyjnego aplikacji, który automatycznie utworzył bazę danych z zestawem 89 próbek testowych

# 3. Skalowanie

## Wynik `docker compose ps`:
```
$ docker compose ps
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED         STATUS                   PORTS
breach-checker-backend-1    breach-checker-backend    "python app.py"          backend    6 days ago      Up 3 minutes (healthy)   5000/tcp
breach-checker-backend-2    breach-checker-backend    "python app.py"          backend    3 minutes ago   Up 3 minutes (healthy)   5000/tcp
breach-checker-backend-3    breach-checker-backend    "python app.py"          backend    3 minutes ago   Up 3 minutes (healthy)   5000/tcp
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         6 days ago      Up 3 minutes (healthy)   5432/tcp
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   6 days ago      Up 3 minutes             0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

## Wynik `nslookup backend`:
```
$ docker compose exec frontend nslookup backend
Server:		127.0.0.11
Address:	127.0.0.11:53

** server can't find backend.tail06a64e.ts.net: NXDOMAIN

** server can't find backend.tail06a64e.ts.net: NXDOMAIN
```

## Kilka linii `POST /check` z logów przed restartem frontendu:
```
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:49] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:49] "POST /check HTTP/1.1" 200 -
```

## Kilka linii `POST /check` z logów po restarcie frontendu:
```
backend-3  | 172.19.0.4 - - [04/Nov/2026 19:46:19] "POST /check HTTP/1.1" 200 -
backend-3  | check: hash=bc454...
backend-3  | 172.19.0.4 - - [04/Nov/2026 19:46:19] "POST /check HTTP/1.1" 200 -
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-1  | 172.19.0.4 - - [04/Nov/2026 19:40:48] "POST /check HTTP/1.1" 200 -
backend-1  | check: hash=bc454...
backend-2  | 172.19.0.4 - - [04/Nov/2026 19:46:19] "POST /check HTTP/1.1" 200 -
```

## Dlaczego przed restartem frontendu ruch trafiał do jednej repliki?
Nginx zapisuje adresy IP kontenerów tylko w momencie uruchomienia, później ich nie aktualizuje aż do czasu restartu. Dlatego na poćzątku nginx miał adres IP tylko jednej repliki i cały ruch trafiał do niej. Po restarcie baza adresów IP została zaktualizowana o zeskalowane kontenery, dlatego ruch zaczął trafiać też do pozostałych replik

## Dlaczego skalowanie byłoby niemożliwe, gdyby backend nadal miał `ports:"5000:5000"`?
Taki zapis wymusza zajęcie dokładnie tego portu zewnętrznego przez każdy uruchomiony kontener. Przy próbie uruchomienia replik powstałby konflikt, gdyż wszyskie repliki chciałyby nadawać na tym samym porcie, przez co wystartowanie ich byłoby niemożliwe
