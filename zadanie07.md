# 1. Raport podatności - `SECURITY.md`

## Linie `Total:` z czterech skanów obrazów:
Frontend:
```
breach-checker-frontend:prod (alpine 3.24.4)
============================================
Total: 3 (HIGH: 3, CRITICAL: 0)
```

Frontend `--ignore-unfixed`:
```
breach-checker-frontend:prod (alpine 3.24.4)
============================================
Total: 3 (HIGH: 3, CRITICAL: 0)
```

Backend:
```
breach-checker-backend:prod (debian 13.7)
=========================================
Total: 49 (HIGH: 49, CRITICAL: 0)
```

Backend `--ignore-unfixed`:
```
breach-checker-backend:prod (debian 13.7)
=========================================
Total: 2 (HIGH: 2, CRITICAL: 0)
```

## Czy `--pull` zmienił wynik i dlaczego dla jednego obrazu bardziej niż dla drugiego?
`--pull` zminił wynik skanowania dla obrazu frontend, ponieważ zaktualizował wersję Alpine o łatki bezpieczeństwa dla wielu podatności. W przypadku backendu wynik się nie zmienił, bo wersja Debiana nie została zaktualizowana i podatności nie zostały naprawione.

# 2. Utwardzanie kontenera w działaniu

## Ostatnia linia błędu z logów backendu przed dodaniem `tmpfs`:
```
backend-1  | FileNotFoundError: [Errno 2] No usable temporary directory found in ['/tmp', '/var/tmp', '/usr/tmp', '/app']
```

## Wynik trzech poleceń:
```
$ docker compose exec backend touch /etc/test
touch: cannot touch '/etc/test': Read-only file system
```

```
$ docker compose exec backend touch /tmp/test
```

```
$ docker compose exec backend grep CapEff /proc/1/status
CapEff:	0000000000000000
```

## Wynik `docker compose exec backend pytest`:
```
$ docker compose exec backend pytest
================================================ test session starts ================================================
platform linux -- Python 3.12.16, pytest-8.4.2, pluggy-1.7.0
rootdir: /app
collected 7 items                                                                                                   

tests/test_check.py .......                                                                                   [100%]

================================================= 7 passed in 0.03s =================================================
```

## Przed czym chroni każde z trzech ustawień i czym `read-only` różni się od `USER appuser` z zajęć?
`read-only` chroni system plików kontenera przed modyfikacją. `USER appuser` chroni przed eskalacją uprawnień wewnątrz kontenera, daje uprawnienia zwykłego użytkownika a nie root. `cap_drop ALL` i `no-new-privileges` chronią przed nadużyciem uprawnień jądra systemu, odcinają niepotrzebne uprawnienia.

`read-only` różni się od `USER appuser` tym, że działa na poziomie systemu plików, blokuje możliwość modyfikacji plików nawet przez procesy root, a `USER appuser` działa na poziomie uprawnień systemowych i określa z jakim indetyfikatorem UID/GID uruchamiany jest proces aplikacji i sam w sobie nie zabrania modyfikacji plików, do których proces ma uprawnienia.

# 3. Sekret w usuniętym pliku

## Wynik `docker history sekret-demo:plik`:
```
$ docker history sekret-demo:plik
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
b098e0089e6b   30 seconds ago   RUN /bin/sh -c rm /tmp/klucz.txt # buildkit     0B        buildkit.dockerfile.v0
<missing>      30 seconds ago   RUN /bin/sh -c test -s /tmp/klucz.txt && ech…   0B        buildkit.dockerfile.v0
<missing>      31 seconds ago   COPY klucz.txt /tmp/klucz.txt # buildkit        14B       buildkit.dockerfile.v0
<missing>      12 days ago      CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      12 days ago      ADD alpine-minirootfs-3.22.8-x86_64.tar.gz /…   8.3MB     buildkit.dockerfile.v0
```

## Odzyskana zawartość pliku:
```
$ for f in obraz/blobs/sha256/*; do tar -tf "$f" 2>/dev/null | grep -qx 'tmp/klucz.txt' && tar -xOf "$f" tmp/klucz.txt; done
tajny-klucz-V
```

## Dlaczego `RUN rm` nie usunął sekretu z obrazu i jak należało przekazać ten klucz?
`RUn rm` nie usunął sekretu z obrazu, bo dodał nową warstwę z oznaczeniem usunięcia, podczas gdy plik fizycznie nadal znajdował się w poprzedniej, niezmienialnej warstwie `COPY`. Klucz należało przekazać za pomocą `--secret`, który pozwala bezpiecznie użyć sekretów bez utrwalania ich w warstwie obrazu.
