# Endpoint `GET /health`

## Odpowiedź `/health`
`{"status":"ok"}`

## linia z `docker logs backend`:
`172.18.0.1 - - [22/Oct/2026 14:28:26] "GET /health HTTP/1.1" 200 -`

# Kontener diagnostyczny `healthcheck-probe`
## Polecenia `docker run`
1. `docker run --rm --network breach-net healthcheck-probe` przy działającym backendzie:
```
$ docker run --rm --network breach-net healthcheck-probe
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
100     16 100     16   0      0 {"status":"ok"}                            0
  3565      0                              0
backend: OK
```

2. `docker run --rm healthcheck-probe` (bez --network):
```
$ docker run --rm healthcheck-probe
curl: (6) Could not resolve host: backend (Domain name not found)
backend: UNREACHABLE
```

3. ponownie w sieci `breach-net`, ale po `docker stop backend`:
```
$ docker run --rm --network breach-net healthcheck-probe
curl: (6) Could not resolve host: backend (Domain name not found)
backend: UNREACHABLE
```

## Dlaczego bez `--network` nazwa `backend` nie działa, mimo że backend jest uruchomiony?
Domyślna sieć w Dockerze nie posiada wbudowanego DNS przetłumaczającego nazwy kontenerów na ich adresy IP. Kontener uruchomiony poza wewnętrzną siecią nie rozpoznaje nazwy `backend` i nie jest w stanie uzyskać jego adresu IP, zatem nie jest w stnaie się z nim połączyć, mimo że kontener ten jest uruchomiony.


# Trwałość danych - co przetrwa restart, a co usunięcie kontenera
## Wynik obu `docker exec backend cat /tmp/notatka.txt` (po restarcie i po rm + run)
1. Po restarcie:
```
$ docker exec backend cat /tmp/notatka.txt
notatka
```
2. Po `rm` + `run`:
```
$ docker exec backend cat /tmp/notatka.txt
cat: /tmp/notatka.txt: No such file or directory
```

## Zawartość `historia.txt`
Mon Oct 26 13:53:14 UTC 2026
Mon Oct 26 13:53:24 UTC 2026

## Dlaczego plik w `/tmp` przetrwał restart, ale nie `rm`; dlaczego `historia.txt` przetrwała, choć oba kontenery zostały usunięte; czy lista wycieków przetrwałaby `docker rm` i dlaczego?
Plik w `/tmp` przetrwał restart, ale nie przetrwał `rm`, ponieważ warstwa zapisywalna kontenera zachowuje swoje dane podczas restartu, ale jest usuwana przy `rm` i traci po nim dane bezpowrotnie.

Plik `historia.txt` przetrwał usunięcie kontenerów, ponieważ nie został zapisany w kontenerach, a w osobnym wolumenie, który jest niezależny od cyklu życia kontenerów i nie został usunięty.

Lista wycieków przetrwałaby `docker rm`, ponieważ znajduje się w oddzielnym wolumenie, który jest montowany do kontenera, a taki wolumen nie zostanie usunięty przez to polecenie
