# Podpunkt 1.

## Docker inspect
docker inspect:
{"autor":"Vesper Haponik","data":"2026-10-15","maintainer":"274822031+VesTheRat@users.noreply.github.com"}

## Docker history
docker history:
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
419ddab0d987   22 minutes ago   EXPOSE [80/tcp]                                 0B        buildkit.dockerfile.v0
<missing>      22 minutes ago   RUN /bin/sh -c htmlhint /usr/share/nginx/htm…   0B        buildkit.dockerfile.v0
<missing>      22 minutes ago   COPY . /usr/share/nginx/html # buildkit         1.52kB    buildkit.dockerfile.v0
<missing>      22 minutes ago   RUN /bin/sh -c apk add --no-cache nodejs npm…   92.1MB    buildkit.dockerfile.v0
<missing>      22 minutes ago   LABEL maintainer=274822031+VesTheRat@users.n…   0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      RUN /bin/sh -c set -x     && apkArch="$(cat …   50.1MB    buildkit.dockerfile.v0
<missing>      3 weeks ago      ENV ACME_VERSION=0.4.1                          0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      ENV NJS_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      ENV NJS_VERSION=1.0.1                           0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      STOPSIGNAL SIGQUIT                              0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      EXPOSE map[80/tcp:{}]                           0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      ENTRYPOINT ["/docker-entrypoint.sh"]            0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      COPY 30-tune-worker-processes.sh /docker-ent…   4.62kB    buildkit.dockerfile.v0
<missing>      3 weeks ago      COPY 20-envsubst-on-templates.sh /docker-ent…   3.03kB    buildkit.dockerfile.v0
<missing>      3 weeks ago      COPY 15-local-resolvers.envsh /docker-entryp…   389B      buildkit.dockerfile.v0
<missing>      3 weeks ago      COPY 10-listen-on-ipv6-by-default.sh /docker…   2.14kB    buildkit.dockerfile.v0
<missing>      3 weeks ago      COPY docker-entrypoint.sh / # buildkit          1.62kB    buildkit.dockerfile.v0
<missing>      3 weeks ago      RUN /bin/sh -c set -x     && addgroup -g 101…   4.39MB    buildkit.dockerfile.v0
<missing>      3 weeks ago      ENV DYNPKG_RELEASE=1                            0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      ENV PKG_RELEASE=1                               0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      ENV NGINX_VERSION=1.31.6                        0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      LABEL maintainer=NGINX Docker Maintainers <d…   0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      3 weeks ago      ADD alpine-minirootfs-3.24.2-x86_64.tar.gz /…   8.42MB    buildkit.dockerfile.v0

## Które warstwy pochodzą z obrazu bazowego, ktore z twojego Dockerfile, dlaczego niektóre mają rozmiar `0B` i dlaczego widać tylko twoją wartość `maintainer`?
5 najnowszych wartstw pochodzi z mojego Dockerfile, wszystkie pozostałe z obrazu bazowego. Niektóre warstwy mają `0B`, ponieważ dopisały tylko metadane. Wartość 'maintainer' w pliku `Dockerfile` nadpisuje wartość z obrazu bazowego nginx, ponieważ nie mogą istnieć dwie różne wartości etykiety przypisane do tego samego klucza, dlatego widać tylko moją wartość

# Podpunkt 2 

## Zawartość build.txt 
Kontener 1:
Fri Oct 16 17:07:44 UTC 2026

Kontener 2:
Fri Oct 16 17:07:44 UTC 2026

## Dlaczego czas w `build.txt` jest taki sam w obu kontenerach i nie jest to czas uruchomienia?
Ponieważ komenda `RUN` działa tylko przy budowaniu obrazu kontenera, zatem zawartość pliku `build.txt` zawiera datę i godzinę zbudowania obrazu, a nie uruchomienia kontenera. Czas jest taki sam w obu kontenerach, ponieważ zostały uruchomione z tego samego obrazu

# Podupunkt 3 

## Linia `root` z wypisanej konfiguracji
root   /usr/share/nginx/html;

## Dlaczego kopiujemy pliki właśnie do `/usr/share/nginx/html`?
Ponieważ jest to domyślny folder dokumentów (document root) i to z niego nginx będzie odczytywał pliki strony


# Podpunkt 4 

## Wynik `docker exec ls /usr/share/nginx`
html
index.html
script.js
style.css

## Wynik `docker exec ls /usr/share/nginx/html`
50x.html
build.txt
index.html

## Który plik `index.html` sprawdził `htmlhint` i dlaczego build przeszedł?
`htmlhint` sprawdził plik `index.html` znajdujący się w folderze `/usr/share/nginx/html/index.html` który zawiera wygląd domyślnej strony nginx. Ponieważ nie zawiera on błędów, build przeszedł.

# Podpunkt 5.

## Wynik polecenia `docker rmi breach-checker-frontend:latest`
Untagged: breach-checker-frontend:latest

## Wynik polecenia `docker rmi breach-checker-frontend:v1`
Untagged: breach-checker-frontend:v2
Deleted: sha256:8ec8b2ff7e38f94bf6188fa48aa0dce6573e5a8d6d1b8f94eda7ba4fe313f8ab

## Kiedy `docker rmi` faktycznie usuwa obraz z dysku?
`docker rmi` faktycznie usuwa obraz z dysku, gdy zostaną usunięte wszystkie tagi. W tym przykładzie pierwsze polecenie usunęło jeden z tagów, drugie usunęło ostatni pozostały tag i wraz z nim obraz.
