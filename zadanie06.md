# 1. Własne testy

## Wynik `docker compose exec backend pytest -v`:
```
================================================ test session starts ================================================
platform linux -- Python 3.12.16, pytest-8.4.2, pluggy-1.7.0 -- /usr/local/bin/python3.12
cachedir: .pytest_cache
rootdir: /app
collected 7 items                                                                                                   

tests/test_check.py::test_leaked_hash_is_found PASSED                                                         [ 14%]
tests/test_check.py::test_unknown_hash_is_not_found PASSED                                                    [ 28%]
tests/test_check.py::test_missing_hash_field PASSED                                                           [ 42%]
tests/test_check.py::test_plaintext_password_is_rejected PASSED                                               [ 57%]
tests/test_check.py::test_uppercased_hash_is_found PASSED                                                     [ 71%]
tests/test_check.py::test_hash_length_39_is_rejected PASSED                                                   [ 85%]
tests/test_check.py::test_signs_outside_range_are_rejected PASSED                                             [100%]

```

## Wynik `docker compose exec backend pytest`:
```
================================================ test session starts ================================================
platform linux -- Python 3.12.16, pytest-8.4.2, pluggy-1.7.0
rootdir: /app
collected 7 items                                                                                                   

tests/test_check.py ....F..                                                                                   [100%]

===================================================== FAILURES ======================================================
___________________________________________ test_uppercased_hash_is_found ___________________________________________

client = <FlaskClient <Flask 'app'>>

    def test_uppercased_hash_is_found(client):
        response = client.post("/check", json={"hash": LEAKED.upper()})
>       assert response.status_code == 200
E       assert 400 == 200
E        +  where 400 = <WrapperTestResponse streamed [400 BAD REQUEST]>.status_code

tests/test_check.py:27: AssertionError
============================================== short test summary info ==============================================
FAILED tests/test_check.py::test_uppercased_hash_is_found - assert 400 == 200
============================================ 1 failed, 6 passed in 0.12s ============================================

```

## Dlaczego testy z zajęć nie wykryły usunięcia `.lower()`?
Bo w testach z zajęć nie było napisanego testu, który sprawdzałby czy hash napisany wielkimi literami przechodzi. Bez `.lower()` hash nie zostanie rozpoznany i nowo dodany test to wykryje.

# 2. Awaria blokady

## Wynik `curl`:
```
$ curl -s -o blad.html -w "HTTP %{http_code}\n" -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '{"hash":"7c4a8d09ca3762af61e59520943dc26494f8941b"}'
HTTP 500
```

## Wynik `docker compose ps -a` i `docker inspect` po restarcie:
```
$ docker compose ps -a
NAME                        IMAGE                     COMMAND                  SERVICE    CREATED          STATUS                     PORTS
breach-checker-backend-1    breach-checker-backend    "flask --app app run…"   backend    50 minutes ago   Exited (1) 6 seconds ago   
breach-checker-db-1         postgres:16-alpine        "docker-entrypoint.s…"   db         50 minutes ago   Exited (0) 5 minutes ago   
breach-checker-frontend-1   breach-checker-frontend   "/docker-entrypoint.…"   frontend   50 minutes ago   Up 50 minutes              0.0.0.0:8080->80/tcp, [::]:8080->80/tcp
```

```
$ docker inspect --format '{{.State.Status}} {{.State.ExitCode}}' breach-checker-backend-1
exited 1
```

## Tabela porównująca tę awarię z blokadą z kroku 4 w pięciu wierszach:
| | Ta awaria | Krok 4 |
|-| -------|------- |
| STATUS | Exited (1) (db), up (backend) | Up |
| CPU | - (db) 0.17% (backend) | 0.19% |
| Ostanie 3 linie logów: 1. | backend-1  \|     conn = _connect(dsn, connection_factory=connection_factory, **kwasync) | backend-1  \| 127.0.0.1 - - [11/Nov/2026 13:54:54] "GET /health HTTP/1.1" 200 - |
| 2. | backend-1  \| psycopg2.OperationalError: could not translate host name "db" to address: Temporary failure in name resolution | backend-1  \| check: hash=8cb22... |
| 3. | backend-1  \| 127.0.0.1 - - [11/Nov/2026 14:02:37] "GET /health HTTP/1.1" 200 - | backend-1  \| 127.0.0.1 - - [11/Nov/2026 13:54:59] "GET /health HTTP/1.1" 200 - |


## Co zawierał `blad.html` i dlaczego taka odpowiedź na serwerze produkcyjnym byłaby problemem bezpieczeństwa?
`blad.html` zawierał stack trace aplikacji i szczegóły techniczne dotyczące infrastruktury aplikacji, takie jak użyte bibliotaki,ścieżki i fragmenty kodu źródłowego. Taka odpowiedź byłaby problemem bezpieczeńtwa, ponieważ ujawnia dużo wrażliwych informacji, co ułatwia ewentualny rekonesans przeprowadzany przez atakujących, tym samym ułatwiając dokonanie skutecznego ataku na aplikację. 

# 3. Frontend w trybie deweloperskim

## Wynik `curl http://localhost:8080/nginx.conf` przed poprawką:
```
$ curl http://localhost:8080/nginx.conf
server {
    listen 80;

    # Pliki strony (index.html, style.css, script.js)
    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    # Wszystko pod /api/ nginx przekazuje do usługi backend w sieci Compose
    location /api/ {
        proxy_pass http://backend:5000/;
    }
}
```

## Wynik `curl -I http://localhost:8080/nginx.conf` po poprawce:
```
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.31.6</center>
</body>
</html>
```

## Dlaczego obraz produkcyjny frontendu nie ma tego problemu (wskazówka: frontend/Dockerfile) i dlaczego montowanie całego katalogu projektu do katalogu serwowanego przez serwer WWW jest ryzykowne?
Obraz produkcyjny nie ma tego problemu, bo COPY przenosi do katalogu serwera tylko potrzebne pliki statyczne i pomija konfigurację. Montowanie całego katalogu projektu sprawia, że wszystkie znajdujące się w nim pliki, włącznie z konfiguracyjnymi, są dostępne z poziomu przeglądarki, co prowadzi do ryzyka wycieku danych wrażliwych.
