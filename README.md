# PlantPulse

In deze manual laat ik zien hoe ik heb geprobeert de watering van een plant te achterhalen van de API van Perenual.

## Stap 1 API Key aanmaken
De eerste stap is het aanmaken van een account bij Perenual zodat je een eigen API Key krijgt.
ga naar de website van Perenual https://perenual.com/docs/api en druk dan op de knop GET API KEY & ACCESS.
Maak een account aan door op Sing Up te kliken.
<img width="1440" height="804" alt="Scherm­afbeelding 2025-10-16 om 14 47 35" src="https://github.com/user-attachments/assets/2dc7e57b-7a1d-418c-b39f-a0fa858af432" />
Als je een account hebt gemaakt kom je op deze pagina waar je op de knop Generate New Key moet kliken. 
<img width="1440" height="804" alt="Developer" src="https://github.com/user-attachments/assets/01dde758-081c-4eb2-b927-367629fb8dca" />
Je moet dan invullen waarvoor je het gaat gebruiken, dan kan bijvoorbeeld zijn voor een school opdracht. Als je dat hebt ingevult zal je je API Key krijgen.
<img width="1440" height="804" alt="Scherm­afbeelding 2025-10-16 om 14 49 55" src="https://github.com/user-attachments/assets/60b579d3-546b-4baf-b338-cbecb03ed918" />

## Stap 2 library's installeren 
Je moet de library HTTPClient by Adrian McEwen instaleren zoals op de foto. Ook moet je op de zelfde manier de library ArduinoJson by Benoit Blanchon en de  installeren.
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 11 14" src="https://github.com/user-attachments/assets/d4fa2c74-4a73-49a1-86fb-0f50b6a40ad4" />

voer boven in je Arduino sktech de volgende #include:

``` #include <ESP8266WiFi.h>``` 

```#include <ESP8266HTTPClient.h>```

```#include <WiFiClientSecure.h>```

```#include <ArduinoJson.h>```



