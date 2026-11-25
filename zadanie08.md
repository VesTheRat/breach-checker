# 1. Frontend w pipeline

## Odnośnik do uruchomienia workflow, w którym `build-frontend` się nie powiódł z linią błędu `htmlhint`:
[odnośnik do czerwonego](https://github.com/VesTheRat/breach-checker/actions/runs/45618012184)
ERROR: failed to build: failed to solve: process "/bin/sh -c htmlhint index.html" did not complete successfully: exit code: 1

## Odnośnik do następnego zielonego uruchomienia:
[odnośnik do zielonego](https://github.com/VesTheRat/breach-checker/actions/runs/45618378284) 

## Dlaczego w tym zielonym uruchomieniu weryfikacja sprawdza obrazy, które trafią na produkcję, a we wcześniejszej wersji pipeline'u tylko jeden z nich?
W obecnej wersji job pobiera obrazy obu usług oznaczone hashem commita i testuje oba przed trafieniem na produkcje. We wcześniejszej wersji pobierany i testowany był tylko backend.


# 2. `latest` tylko po weryfikacji:

## Odnośnik do uruchomienia z nieudanym `verify` i pominiętym `promote`:
[odnośnik do nieudanego verify](https://github.com/VesTheRat/breach-checker/actions/runs/45625235601) 

## Odnośnik do następnego, zielonego uruchomienia:
[odnośnik do zielonego uruchomienia](https://github.com/VesTheRat/breach-checker/actions/runs/45625673437)

## Na co wskazywał `latest` backendu po nieudanym uruchomieniu? Dlaczego to ważne dla kogoś, kto pobiera obraz bez podawania tagu?
Po nieudanym uruchomieniu tag `latest` dalej wskazywał na ostatni obraz z poprawnie zweryfikowanego commita, ponieważ job `promote` został pominięty i tag nie został dopisany do obrazu ze złego commita. Jest to ważne dla kogoś, kto pobiera obraz bez podawania tagu, bo domyślnie pobrany zostanie obraz z tagiem `latest` i nie zostanie pobrany zepsuty obraz.


# 3. Publiczny pakiet - odczyt bez tokenu, zapis z tokenem 

## Wynik 4 poleceń:
```
$ docker pull ghcr.io/vestherat/breach-checker-backend:latest
latest: Pulling from vestherat/breach-checker-backend
83a14706aa6d: Pull complete 
30cf67d7101c: Pull complete 
e3e428093f5e: Pull complete 
286d8c291daa: Pull complete 
f498c9e8b98c: Pull complete 
d1f2e03cfc28: Pull complete 
b267c6d3b038: Pull complete 
1e45c7f2395a: Pull complete 
ea22df6c7d7b: Pull complete 
Digest: sha256:f690bf84754a100c381df0730d4a621b376d99098a16b703145168e9b49e4189
Status: Downloaded newer image for ghcr.io/vestherat/breach-checker-backend:latest
ghcr.io/vestherat/breach-checker-backend:latest
```

```
$ docker pull ghcr.io/vestherat/breach-checker-backend:2138ec4b89f396961656ed4209906b5d9720da3b
2138ec4b89f396961656ed4209906b5d9720da3b: Pulling from vestherat/breach-checker-backend
Digest: sha256:f690bf84754a100c381df0730d4a621b376d99098a16b703145168e9b49e4189
Status: Downloaded newer image for ghcr.io/vestherat/breach-checker-backend:2138ec4b89f396961656ed4209906b5d9720da3b
ghcr.io/vestherat/breach-checker-backend:2138ec4b89f396961656ed4209906b5d9720da3b
```

```
$ docker images --digests ghcr.io/vestherat/breach-checker-backend
REPOSITORY                                 TAG                                        DIGEST                                                                    IMAGE ID       CREATED          SIZE
ghcr.io/vestherat/breach-checker-backend   2138ec4b89f396961656ed4209906b5d9720da3b   sha256:f690bf84754a100c381df0730d4a621b376d99098a16b703145168e9b49e4189   6377beb0a43e   14 minutes ago   142MB
ghcr.io/vestherat/breach-checker-backend   latest                                     sha256:f690bf84754a100c381df0730d4a621b376d99098a16b703145168e9b49e4189   6377beb0a43e   14 minutes ago   142MB
```

```
$ docker push ghcr.io/vestherat/breach-checker-backend:latest
The push refers to repository [ghcr.io/vestherat/breach-checker-backend]
3b3094459035: Layer already exists 
50000b9dfc1d: Layer already exists 
74e1d21e8d3b: Layer already exists 
2ee6c4afefe6: Layer already exists 
c896b1839c90: Layer already exists 
7283aa262939: Layer already exists 
e1beed03d15a: Layer already exists 
dc8af65e94dc: Layer already exists 
54ae9782e24f: Layer already exists 
unauthorized: unauthenticated: User cannot be authenticated with the token provided.
```

## Co pokazują digesty obu tagów?
Identyczny digest obu tagów potwierdza, że `latest` oraz tag commita wskazują na ten sam manifestu obrazu

## Dlaczego `pull` zadziałał, a `push` nie i do czego służy token z zakresem `write:packages` w kroku 1.?
`Pull` zadziałał bez logowania, bo pakier w ghcr jest publiczny i pozwala na anonimowy odczyt, natomiast `push` został odrzucony, bo wymaga uwierzytelniania. Token z zakresem `write:packages` w jobach budujących służy do uwierzytelniania w ghcr i pushowania zbudowanych lub promowanych obrazów do rejestru.
