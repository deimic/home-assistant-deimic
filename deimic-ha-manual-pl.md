## Wymagane
1. Moduł Deimic Master v3
1. Zainstalowane poniższe aplikacje (zalecany system operacyjny Windows)
   * [MQTT Explorer](https://github.com/thomasnordquist/MQTT-Explorer/releases)
   * [DEIMIC Configurator](https://www.deimic.pl/wsparcie,do-pobrania.html)
1. Skonfigurowana instancja Home Assistant w sieci lokalnej w wersji HA OS (np. na Raspberry Pi). Instrukcje instalacji można znaleźć [na stronie projektu](https://www.home-assistant.io/installation/). 
    > Zalecamy instalację HA OS, dlatego że instalacja typu _container_ nie umożliwia instalowacji dodatków (Studio Code Server).

## Wstęp
Integracja jest możliwa za pośrednictwem protokołu MQTT. Moduł Deimic Master ogłasza każdą zmianę stanu w dedykowanym temacie, który może być subskrybowany przez Home Assistant. Komunikacja działa w obie strony, więc Home Assistant może ustawić stan urządzenia na Master (włączyć / wyłączyć światło, otworzyć / zamknąć zasłonę itp.)

## Krok 1 - dodanie integracji MQTT
1. Należy otworzyć ustawienia, przejść do zakładki "Devices & services", a nastepnie kliknąć "Add integration"
    ![Home Assistant - settings - add itegration 2](/assets/image.png)
    ![Home Assistant - settings - add itegration 2](/assets/image-1.png)
1. Następnie należy wyszukać "MQTT, wybrać integrację z listy, a w następnym okienku wybrać "MQTT" jeszcze raz. 
    ![Home Assistant - settings - add MQTT](/assets/image-2.png)
    ![Home Assistant - settings - add MQTT](/assets/image-3.png)
1. W następnym oknie należy podać adres IP modułu Deimic Master (1), numer portu, jeśli jest inny, niż domyślny (2) oraz potwierdzić opcje (3).
    ![Home Assistant - configure MQTT](/assets/image-5.png)

## Krok 2 - instalacja Studio Code Server
Jest to dodatek do Home Assistant, który pozwoli na późniejszą edycję plików konfiguracyjnych bezpośrednio z Home Assistant.

1. Należy przejść do ustawień (1) i wybrać "Dodatki" (2)
    ![Home Assistant - addons](/assets/image-4.png)
1. W następnym oknie należy kliknąć "Wyszukaj dodatek" w prawym dolnym rogu, następnie wpisać "Studio Code Server" i wybrać z listy.
    ![Home Assistant - add Studio Code Server](/assets/image-6.png)
1. W następnym oknie należy potwierdzić instalację
    ![Home Assistant - add Studio Code Server](/assets/image-7.png)
1. Po zainstalowaniu dodatku, po lewej stronie pojawi się nowa ikona, która umożliwi edycję konfiguracji.
    ![Home Assistant - Studio Code Server yaml preview](/assets/image-8.png)

## Krok 3 - konfiguracja urządzeń
Urządzenia dodaje się edytując konfigurację (plik `configuration.yaml`). Plik ten ma określoną strukturę, którą należy utrzymać.

### Opis słów kluczowych
* `mqtt` - musi znajdować się nad całą konfiguracją urządzeń, oznacza że wszystkie wiersze poniżej dotyczą integracji MQTT (patrz krok 1)
* `binary_sensor` / `sensor` / `switch` / etc.  - typ klasy urządzenia; więcej informacji [w dokumentacji Home Assistant](https://www.home-assistant.io/docs/configuration/customizing-devices/#device-class)
* `unique_id` – unikalny identyfikator każdego urządzenia; można stosować następujący standard nazewnictwa: `<nr seryjny modułu>_<typ gniazda><numer>`, np. "1a2c1953_o00", gdzie "1a2c1953" to nr seryjny mastera.
* `name` – nazwa, pod jaką urządzenie będzie wyświetlane
* `state_topic` - Temat MQTT z aktualnym stanem urządzenia
* `payload_on`, `payload_off` - wartości, które ustawiają dany stan urządzenia: włączony (`ON`) lub wyłączony (`OFF`) 
* `device_class` – konfiguracja typu urządzenia (ustawienie konkretnego czujnika: wilgotności, temperatury, ruchu) - definiuje sposób działania i wygląd urządzenia w Home Assistant; każda klasa urządzeń ma swoje własne typy - więcej szczegółów w dokumentacji Home Assistant, np. [tutaj](https://www.home-assistant.io/integrations/sensor/#device-class)
  
### Skąd brać adresy tematów 
Do podglądu komunikacji MQTT służy aplikacja **MQTT Explorer**. Za każdym razem, gdy stan urządzenia zmienia się, odpowiedni temat MQTT zmienia się. W ten sposób można dowiedzieć się, jakie kolejki należy ustawić w konfiguracji Home Assistant.
![MQTT Explorer deimic example](/assets/image-9.png)

Aplikacja **Deimic Configurator** umożliwia sprawdzenie, do jakiego wyjścia/wejścia podłączone jest dane urządzenie. Wiedząc gdzie szukać, prościej odnaleźć odpowiedni temat w **MQTT Explorer**. Aby sprawdzić daną akcję, rozwiń pole opisu klikając na trójkąt w aplikacji MQTT Explorer.

### Przykładowy kod
```yaml
mqtt:
  switch:
    - unique_id: "1a2c1953_o00"
      name: "Output 00"
      state_topic: "system/1a2c1953/outputs/0/get"
      command_topic: "system/1a2c1953/outputs/0/set"
      payload_on: "ON"
      payload_off: "OFF"
      state_on: "ON"
      state_off: "OFF"
      optimistic: false
      qos: 0
      retain: true

    - unique_id: "1a2c1953_o01"
      name: "Output 01"
      state_topic: "system/1a2c1953/outputs/1/get"
      command_topic: "system/1a2c1953/outputs/1/set"
      payload_on: "ON"
      payload_off: "OFF"
      state_on: "ON"
      state_off: "OFF"
      optimistic: false
      qos: 0
      retain: true

  sensor:
    - unique_id: "Temp_sensor"
      name: "Temperature"
      state_topic: "system/1a2c1953/analog_smart_inputs/30/0/get"
      unit_of_measurement: "° C"
      device_class: "temperature"

    - unique_id: "Humidity_sensor"
      name: "Humidity"
      state_topic: "system/1a2c1953/analog_smart_inputs/24/3/get"
      unit_of_measurement: "%"
      device_class: "moisture"

  binary_sensor:
    - unique_id: "Motion_sensor"
      name: "Motion sensor"
      state_topic: "system/1a2c1953/smart_inputs/24/2/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"
      device_class: "motion"

    - unique_id: "smart_sensor01"
      name: "Smart sensor 01"
      state_topic: "system/1a2c1953/smart_inputs/22/0/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"

    - unique_id: "smart_sensor02"
      name: "Smart sensor 02"
      state_topic: "system/1a2c1953/smart_inputs/22/1/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"

    - unique_id: "smart_sensor01"
      name: "Smart sensor 01"
      state_topic: "system/<nr seryjny Master>/smart_inputs/22/0/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"
```

## Krok 4
Rozwinięciem instalacji może być integracja Home Brdige na iOS.
[HomeKit Bridge](https://www.home-assistant.io/integrations/homekit/)
