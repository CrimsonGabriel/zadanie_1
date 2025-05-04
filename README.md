# Zadanie 1 Sprawozdanie
 Realizacja zadanka

#Krok 1 Tworzenie Aplikacji Webowej
Utworzono folder zadanie1_app zawierający potrzebne pliki
zadanie1_app/
├── app.py            # Główny plik aplikacji
├── requirements.txt  # Zależności Python
└── templates/
    └── index.html    # Szablon HTML dla interfejsu
    
Zainstalowano potrzebne biblioteki potrzebne biblioteki: pip install Flask requests python-dotenv

Utworzono plik.env z kluczem API pogody

# Kod app.py

import os
import logging
from datetime import datetime
from flask import Flask, render_template, request, jsonify
import requests
from dotenv import load_dotenv # Do wczytania klucza API z .env

load_dotenv() # Wczytaj zmienne środowiskowe z pliku .env

# Konfiguracja Aplikacji
app = Flask(__name__)
PORT = int(os.getenv('PORT', 8080)) # Pobierz port z zmiennej środowiskowej lub użyj 8080
AUTHOR_NAME = "Gabriel Piątek" 
WEATHER_API_KEY = os.getenv('WEATHER_API_KEY') # Pobierz klucz API

# Konfiguracja Logowania
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Przykładowa predefiniowana lista miast
PREDEFINED_LOCATIONS = {
    "Poland": ["Warsaw", "Krakow", "Gdansk", "Wroclaw","Lublin","Pulawy","Opole Lubelskie","Laziska","Janiszów"],
    "Germany": ["Berlin", "Munich", "Hamburg"],
}

@app.route('/', methods=['GET', 'POST'])
def index():
    weather_data = None
    error_message = None
    selected_country = request.form.get('country')
    selected_city = request.form.get('city')

    if request.method == 'POST':
        if not selected_country or not selected_city:
             error_message = "Proszę wybrać kraj i miasto."
        elif not WEATHER_API_KEY:
            error_message = "Brak klucza API do serwisu pogodowego."
            logging.error("WEATHER_API_KEY is not set.")
        else:
            try:
                # Zapytanie do API OpenWeatherMap
                base_url = "http://api.openweathermap.org/data/2.5/weather"
                params = {
                    'q': f'{selected_city},{selected_country}',
                    'appid': WEATHER_API_KEY,
                    'units': 'metric', 
                    'lang': 'pl'      
                }
                response = requests.get(base_url, params=params)
                response.raise_for_status() # Rzuć wyjątkiem dla złych odpowiedzi (4xx lub 5xx)
                data = response.json()

                # Przetwarzanie danych pogodowych
                weather_data = {
                    'city': data['name'],
                    'country': data['sys']['country'],
                    'temp': data['main']['temp'],
                    'description': data['weather'][0]['description'],
                    'humidity': data['main']['humidity'],
                    'pressure': data['main']['pressure'],
                    'icon': data['weather'][0]['icon'] # Kod ikony pogody
                }
            except requests.exceptions.RequestException as e:
                logging.error(f"Błąd API pogody: {e}")
                error_message = f"Nie można pobrać pogody dla {selected_city}. Sprawdź nazwę lub spróbuj później."
            except KeyError as e:
                 logging.error(f"Błąd przetwarzania danych API: brak klucza {e}")
                 error_message = "Wystąpił błąd podczas przetwarzania danych pogodowych."

    return render_template('index.html',
                           locations=PREDEFINED_LOCATIONS,
                           weather=weather_data,
                           error=error_message,
                           selected_country=selected_country,
                           selected_city=selected_city)

@app.route('/health')
def health_check():
    # Prosty endpoint dla HEALTHCHECK Dockera
    return "OK", 200

if __name__ == '__main__':
    # Logowanie informacji przy starcie
    startup_log_message = (
        f"Aplikacja uruchomiona {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}. "
        f"Autor: {AUTHOR_NAME}. Nasłuchiwanie na porcie TCP: {PORT}"
    )
    logging.info(startup_log_message)
    # Uruchomienie serwera Flask
    # Użyj host='0.0.0.0' dostępny z zewnątrz kontenera
    app.run(host='0.0.0.0', port=PORT, debug=False) # Wyłącz debug w produkcji/kontenerze

# Kod index.html w templates

<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <title>Pogoda</title>
    <style>
        body { font-family: sans-serif; margin: 20px; }
        .weather-info { margin-top: 20px; padding: 15px; border: 1px solid #ccc; border-radius: 5px; }
        .error { color: red; font-weight: bold; }
        label, select, button { display: block; margin-bottom: 10px; }
    </style>
</head>
<body>
    <h1>Sprawdź pogodę</h1>

    <form method="POST">
        <label for="country">Wybierz kraj:</label>
        <select name="country" id="country" required>
            <option value="">-- Wybierz kraj --</option>
            {% for country in locations %}
                <option value="{{ country }}" {% if country == selected_country %}selected{% endif %}>{{ country }}</option>
            {% endfor %}
        </select>

        <label for="city">Wybierz miasto:</label>
        <select name="city" id="city" required>
             <option value="">-- Wybierz miasto --</option>
             {% if selected_country and selected_country in locations %}
                 {% for city_option in locations[selected_country] %}
                     <option value="{{ city_option }}" {% if city_option == selected_city %}selected{% endif %}>{{ city_option }}</option>
                 {% endfor %}
             {% endif %}
        </select>

        <button type="submit">Pokaż pogodę</button>
    </form>

    {% if error %}
        <p class="error">{{ error }}</p>
    {% endif %}

    {% if weather %}
        <div class="weather-info">
            <h2>Pogoda dla: {{ weather.city }}, {{ weather.country }}</h2>
            <p>Temperatura: {{ weather.temp }} °C</p>
            <p>Warunki: {{ weather.description }} <img src="http://openweathermap.org/img/wn/{{ weather.icon }}.png" alt="Ikona pogody"></p>
            <p>Wilgotność: {{ weather.humidity }}%</p>
            <p>Ciśnienie: {{ weather.pressure }} hPa</p>
        </div>
    {% endif %}

    {# Prosty JS do aktualizacji listy miast po wybraniu kraju #}
    <script>
        const countrySelect = document.getElementById('country');
        const citySelect = document.getElementById('city');
        const locations = {{ locations | tojson }}; // Przekaż dane z Flaska do JS
        const selectedCity = '{{ selected_city or "" }}'; // Zapamiętaj wybrane miasto

        countrySelect.addEventListener('change', function() {
            const selectedCountry = this.value;
            // Wyczyść obecne opcje miast
            citySelect.innerHTML = '<option value="">-- Wybierz miasto --</option>';

            if (selectedCountry && locations[selectedCountry]) {
                locations[selectedCountry].forEach(city => {
                    const option = document.createElement('option');
                    option.value = city;
                    option.textContent = city;
                    // Zaznacz miasto, jeśli było wcześniej wybrane dla tego kraju
                    if (city === selectedCity && selectedCountry === '{{ selected_country or "" }}') {
                       option.selected = true;
                    }
                    citySelect.appendChild(option);
                });
            }
        });
         // Wywołaj zmianę przy ładowaniu strony, aby wypełnić miasta jeśli kraj jest już wybrany
         if (countrySelect.value) {
             countrySelect.dispatchEvent(new Event('change'));
         }
    </script>
</body>
</html>

Uruchomiono w terminalu w katalogu zadanie1_app: python app.py
Przetestowano lokalnie http://localhost:8080 działanie API

# Krok 2: Tworzenie Pliku Dockerfile
Utworzono Plik Dockerfile

# Etap 1: Budowanie zależności (Build Stage)
# Użyj obrazu Python do zainstalowania zależności, aby warstwa była cache'owana
FROM python:3.10-slim as builder

# Ustaw katalog roboczy
WORKDIR /app_builder

# Skopiuj tylko plik wymagań, aby wykorzystać cache Dockera
COPY requirements.txt .

# Zainstaluj zależności (bez zapisywania cache pip i bez plików .pyc)
# Użyj virtualenv dla lepszej izolacji
RUN python -m venv /opt/venv && \
    /opt/venv/bin/pip install --no-cache-dir -r requirements.txt

# ---
# Etap 2: Obraz wynikowy (Runtime Stage)
# Użyj minimalnego obrazu Python
FROM python:3.10-slim

# Metadane OCI - Informacje o autorze
LABEL org.opencontainers.image.authors="Gabriel Piątek <gabriel.piatek.biznes@gmail.com>"

# Ustaw zmienne środowiskowe
ENV PYTHONDONTWRITEBYTECODE 1 
# Nie twórz plików .pyc
ENV PYTHONUNBUFFERED 1        
# Logi od razu na stdout/stderr
ENV FLASK_APP=app.py         
# Wskazanie aplikacji Flask (choć uruchamiamy bezpośrednio)
ENV PORT=8080               
# Domyślny port w kontenerze
ENV WEATHER_API_KEY=""     
# Pusta wartość - zostanie przekazana przy `docker run`

# Ustaw katalog roboczy
WORKDIR /app

# Skopiuj wirtualne środowisko z zależnościami z etapu budowania
COPY --from=builder /opt/venv /opt/venv

# Skopiuj kod aplikacji
COPY . .
# Uwaga: Plik .env NIE jest kopiowany (i dobrze!), klucz API podano przez zmienną środowiskową przy uruchamianiu

# Uruchom aplikację jako użytkownik nie-root dla bezpieczeństwa
RUN useradd --create-home appuser && \
    chown -R appuser:appuser /app /opt/venv
USER appuser

# Wystaw port, na którym nasłuchuje aplikacja w kontenerze
EXPOSE ${PORT}

# Sprawdzenie stanu kontenera (Healthcheck)
# Sprawdza co 30s, timeout 5s, 3 próby zanim oznaczy jako niezdrowy
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:${PORT}/health || exit 1
# Uwaga: curl musi być dostępny w obrazie bazowym (python:slim go zawiera)

# Komenda uruchamiająca aplikację przy starcie kontenera
# Użyj venv/bin/python do uruchomienia, aby użyć zainstalowanych pakietów
CMD ["/opt/venv/bin/python", "app.py"]

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


