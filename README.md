
# Zadanie 1 Sprawozdanie
 Realizacja zadanka

# CZĘŚĆ OBOWIĄZKOWA (1-3)
#Krok 1 Tworzenie Aplikacji Webowej
Utworzono folder zadanie1_app zawierający potrzebne pliki
zadanie1_app/
├── app.py            # Główny plik aplikacji
├── requirements.txt  # Zależności Python
└── templates/
    └── index.html    # Szablon HTML dla interfejsu
    
Zainstalowano potrzebne biblioteki potrzebne biblioteki: pip install Flask requests python-dotenv --break-system-packages


Utworzono plik.env z kluczem API pogody

# Kod app.py

![image](https://github.com/user-attachments/assets/d67eadea-bcb5-48cc-8400-7b8626b05033)
![image](https://github.com/user-attachments/assets/0cfc7ac0-5e25-4db5-a037-66fecf9b2c5d)


# Kod index.html w templates

![image](https://github.com/user-attachments/assets/0ccae35e-b851-490d-b0fc-a8b420a752f9)
![image](https://github.com/user-attachments/assets/774ff587-77b3-44b2-b967-91b9103b49dd)


Uruchomiono w terminalu w katalogu zadanie1_app: python3 app.py
Przetestowano lokalnie http://localhost:8080 działanie API

# Krok 2: Tworzenie Pliku Dockerfile
Utworzono Plik Dockerfile

![image](https://github.com/user-attachments/assets/ba053b30-bcab-451b-a8ad-dda6c1285151)


Wyjaśnienia Optymalizacji w Dockerfile:
Wieloetapowe budowanie (multi-stage build): builder instaluje zależności, a finalny obraz kopiuje tylko potrzebne venv i kod. Obraz wynikowy nie zawiera narzędzi budowania.
Minimalny obraz bazowy: python:3.10-slim jest mniejszy niż standardowy python:3.10. Można rozważyć alpine dla jeszcze mniejszego rozmiaru, ale może wymagać kompilacji niektórych pakietów.
Optymalizacja Cache: Kopiowanie requirements.txt i instalacja zależności przed kopiowaniem reszty kodu (COPY . .) sprawia, że warstwa z zależnościami jest budowana ponownie tylko gdy requirements.txt się zmieni.
Kolejność warstw: Mniej zmienne instrukcje (np. FROM, WORKDIR, COPY requirements.txt, RUN pip install) są na początku.
.dockerignore: Zapobiega kopiowaniu niepotrzebnych plików.
Użytkownik nie-root: Uruchamianie jako appuser zwiększa bezpieczeństwo.
HEALTHCHECK: Pozwala Dockerowi monitorować stan aplikacji.
Metadane OCI: LABEL zawiera standardową informację o autorze.
Zmienne środowiskowe: ENV do konfiguracji i przekazywania sekretów (klucz API).

# Krok 3: Polecenia Docker i Weryfikacja

a) Budowanie obrazu:
docker build -t crimsongabriel/zadanie1-app:v1.0 .
![builderlog](https://github.com/user-attachments/assets/39400b98-c9de-40dc-a2e6-50ff4ea85f7b)
b) Uruchomienie kontenera:
docker run -d --name pogoda-app -p 8080:8080 -e PORT=8080 -e WEATHER_API_KEY="<api_klucz_here>" crimsongabriel/zadanie1-app:v1.0

c) Sprawdzenie logów startowych:
docker logs pogoda-app

d) Sprawdzenie warstw i rozmiaru obrazu:
Pokazuje historię budowania obrazu, każda linia odpowiada mniej więcej jednej warstwie). Lub bardziej szczegółowo (w formacie JSON)
docker history crimsongabriel/zadanie1-app:v1.0
docker image inspect crimsongabriel/zadanie1-app:v1.0 --format '{{json .RootFS.Layers}}'

Wyświetla listę obrazów, w tym rozmiar zbudowanego obrazu
docker images crimsongabriel/zadanie1-app:v1.0>
![działaniekontenera](https://github.com/user-attachments/assets/8b118faf-4721-4140-aed2-aeaa34f7b204)


Weryfikacja działania aplikacji w kontenerze:
![dzialanieapizkontenera](https://github.com/user-attachments/assets/896c77b1-f2cb-4624-809f-be8a52deb847)

#CZĘŚĆ NIEOBOWIĄZKOWA (DODATKOWA - Opcja 3)

# Krok 4  Skanowanie Podatności Obrazu:
![image](https://github.com/user-attachments/assets/cd5b5092-bb2b-40e4-b96b-ba59dc9e7fc4)

Dodano do Dockerfile:
/opt/venv/bin/pip install --no-cache-dir --upgrade pip==23.3 setuptools==70.0.0
Oraz zaktualizowano wersje w requirements.txt na nowsze
Zbudowano ponownie obraz
![image](https://github.com/user-attachments/assets/3a6cd666-5818-4bd2-b30b-7899960fe83a)

Przeanalizowano nowy obraz:
![image](https://github.com/user-attachments/assets/4574a331-f5b4-4a53-bffb-71bf57352596)

Podatności związane z setuptools (CVE-2024-6345) i pip (CVE-2023-5752) występują w obrazie bazowym python:3.10-slim, ale można je naprawiono poprzez aktualizację tych pakietów w kontenerze

Scout pokazuje całą historię warstw obrazu.
Warstwa python:3.10-slim nadal zawiera stare wersje pip i setuptools, zanim je nadpisano.
Aplikacja ich już nie używa, bo działa z /opt/venv/..., czyli z zaktualizowanym środowiskiem, co można już pominąć.

# Krok 5 Konfiguracja Buildera Docker Buildx:

Do budowania obrazów wieloplatformowych użyto Docker Buildx z driverem docker-container.

docker buildx create --name multiarch-builder --driver docker-container --use
Tworzy buildera o nazwie multiarch-builder, używa sterownika docker-container (który uruchamia kontener BuildKit) i ustawia go jako domyślny builder (--use)).

Oraz sprawdzono buildera
docker buildx inspect --bootstrap
Pokazuje informacje o aktywnym builderze, w tym obsługiwane platformy.

![image](https://github.com/user-attachments/assets/0aa19dfe-7bda-40a8-9655-50c20f418c65)

# Krok 6 Modyfikacja Dockerfile dla Dostępu do GitHub i BuildKit:

Wymaga: Skonfigurowania kluczy SSH i agenta SSH na maszynie, która wykonuje docker buildx build. Klucz publiczny musi być dodany do konta GitHub. 

Modyfikacja Dockerfile:
![image](https://github.com/user-attachments/assets/31d3092e-e35a-4cd3-8e4e-3f47243db859)

Budowanie Obrazu:
(Trochę będzie rwane i cache'owane etapy bo starałem się naprawić błędy na bieżąco - zrefreshowanie klucza ssh na nowo z lab6, bo coś nie działało i ignorowało) 
![image](https://github.com/user-attachments/assets/e232fc54-51e0-4565-b9b7-37ab32dfbc96)
![image](https://github.com/user-attachments/assets/707bab05-6ad5-4102-a234-1445e727ccb7)
![image](https://github.com/user-attachments/assets/62739ae4-ca95-4d64-95eb-15fe87c2635d)
![image](https://github.com/user-attachments/assets/5813b8b6-d0db-4918-8449-84b90237321c)
![image](https://github.com/user-attachments/assets/3b925218-b7a5-4a3e-a51c-ed00c73481ac)

Standardo przy pierwszym budowaniu cache manifestu nie może zostać wczytany, będzie dopiero stworzony

Kontener i obraz z buildx w dockerze oraz repozytorium
![image](https://github.com/user-attachments/assets/9ff41f02-493c-4a0d-9cea-d25c81d1483d)
![image](https://github.com/user-attachments/assets/b46cce88-89e9-48c4-b233-192580372170)
![image](https://github.com/user-attachments/assets/fe32440c-a4c6-4688-85a4-0b27957d060f)

Sprawdzenie na DockerHub:
![Zrzut ekranu 2025-05-04 164955](https://github.com/user-attachments/assets/d38d2892-dbbc-44c2-8396-162c4612db1f)
Sprawdzenie lokalnie
![image](https://github.com/user-attachments/assets/28ddf1b6-2cbe-4acc-bf9c-509bad19503e)

Ponowne zbudowanie obrazu (skuteczność i działanie Cache'a)
![image](https://github.com/user-attachments/assets/d23ed4f1-a380-4692-83b6-3d77b748ee44)
![image](https://github.com/user-attachments/assets/206a3ff1-900d-4a31-9b3b-a1f24aa790ff)


