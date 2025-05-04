
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

Krok 5  Skanowanie Podatności Obrazu:
![image](https://github.com/user-attachments/assets/cd5b5092-bb2b-40e4-b96b-ba59dc9e7fc4)

Dodano do Dockerfile:
/opt/venv/bin/pip install --no-cache-dir --upgrade pip==23.3 setuptools==70.0.0
Oraz zaktualizowano wersje w requirements.txt na nowsze
Zbudowano ponownie obraz
![image](https://github.com/user-attachments/assets/3a6cd666-5818-4bd2-b30b-7899960fe83a)
