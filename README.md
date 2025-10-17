# PlantPulse

In deze manual laat ik zien hoe ik heb geprobeert de watering van een plant te achterhalen van de API van Perenual.

## Stap 1 API Key aanmaken
De eerste stap is het aanmaken van een account bij Perenual zodat je een eigen API Key krijgt.
ga naar de website van Perenual https://perenual.com/docs/api en druk dan op de knop GET API KEY & ACCESS.
Maak een account aan door op Sing Up te kliken.
<img width="1440" height="804" alt="Scherm­afbeelding 2025-10-16 om 14 47 35" src="https://github.com/user-attachments/assets/2dc7e57b-7a1d-418c-b39f-a0fa858af432" />
Als je een account hebt gemaakt kom je op deze pagina waar je op de knop Generate New Key moet kliken. 
<img width="1440" height="804" alt="Developer" src="https://github.com/user-attachments/assets/01dde758-081c-4eb2-b927-367629fb8dca" />
Je moet dan invullen waarvoor je het gaat gebruiken, dan kan bijvoorbeeld zijn voor een school opdracht. Als je dat hebt ingevult zal je je API Key krijgen. Ondhoud waar je deze hent staan, want je hebt het later nodig.
<img width="1440" height="804" alt="Scherm­afbeelding 2025-10-16 om 14 49 55" src="https://github.com/user-attachments/assets/89562dfa-ccb5-4b81-b09a-46d38def9de5" />


## Stap 2 library's installeren en basis code
Je moet de library HTTPClient by Adrian McEwen instaleren zoals op de foto. Ook moet je op de zelfde manier de library's ArduinoJson by Benoit Blanchon en de WiFiClientSecure en de ESP8266WiFi installeren.
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 11 14" src="https://github.com/user-attachments/assets/d4fa2c74-4a73-49a1-86fb-0f50b6a40ad4" />

Plak deze code in je sketch om te kijken of je kan verbinden met de API. Voer wel nog jouw eigen WiFi naam en wachtwoordin, het is het slim om het te laten verbinden met je hotspot want niet elke WiFi werkt. Ook moet je je eigen API Key invullen die heb je als het goed is net aangemaakt:

```
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecure.h>
#include <ArduinoJson.h>

const char* ssid = "JE_WIFI_NAAM";
const char* password = "JE_WIFI_WACHTWOORD";
const char* apiKey = "YOUR_API_KEY_HERE";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);

  Serial.print("Verbinden met WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" Verbonden!");

  // Gebruik een beveiligde client
  WiFiClientSecure client;
  client.setInsecure(); // Accepteert alle certificaten (handig voor test)

  HTTPClient http;
  String url = "https://perenual.com/api/species-list?key=" + String(apiKey);

  Serial.println("Verbinding maken met:");
  Serial.println(url);

  if (http.begin(client, url)) {  // HTTPS-verbinding
    int httpCode = http.GET();

    if (httpCode > 0) {
      Serial.printf("HTTP code: %d\n", httpCode);
      String payload = http.getString();
      Serial.println("Response:");
      Serial.println(payload);
    } else {
      Serial.printf("Fout bij verbinding: %s\n", http.errorToString(httpCode).c_str());
    }

    http.end();
  } else {
    Serial.println("Kon verbinding niet starten (HTTPS)");
  }
}

void loop() {
  // niets hier
}
```

Als je dit in je serial monitor ziet staan is die goed geconnect.
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 19 34" src="https://github.com/user-attachments/assets/499f0968-e92d-4637-83c9-0bc375274ec8" />


## Stap 3 informatie ophalen uit de API

Dit is mij niet helemaal gelukt, ik zal uitleggen wat ik heb gedaan en dan waar het fout gaat. 

Het stuk onder "HTTPClient http;" gaan moet je veranderen naar:
```
String url = "https://perenual.com/api/species-list?key=" + String(apiKey) + "&q=" + String(plantName);

  Serial.println("Verbinding maken met:");
  Serial.println(url);

  if (http.begin(client, url)) {
    int httpCode = http.GET();

    if (httpCode > 0) {
      Serial.printf("HTTP code: %d\n", httpCode);
      String payload = http.getString();
      Serial.println("Ontvangen data:");

      // JSON parser
      DynamicJsonDocument doc(8192); // genoeg buffer voor Perenual-data
      DeserializationError error = deserializeJson(doc, payload);

      if (error) {
        Serial.print("JSON parse error: ");
        Serial.println(error.c_str());
        return;
      }

      // Controleer of er resultaten zijn
      if (doc["data"].size() > 0) {
        String name = doc["data"][0]["common_name"] | "Onbekend";
        String watering = doc["data"][0]["watering"] | "Onbekend";

        Serial.println("------------------------------");
        Serial.println("Plant gevonden!");
        Serial.print("Naam: ");
        Serial.println(name);
        Serial.print("Waterbehoefte: ");
        Serial.println(watering);
        Serial.println("------------------------------");
      } else {
        Serial.println("Geen planten gevonden met die naam.");
      }

    } else {
      Serial.printf("Fout bij verbinding: %s\n", http.errorToString(httpCode).c_str());
    }

    http.end();
  } else {
    Serial.println("Kon verbinding niet starten (HTTPS)");
  }
}

void loop() {
  // niks nodig in de loop
}
```
Voeg helemaal bovenin onder de const char* deze regel:
```
const char* plantName = "monstera";
```

Bij mij kwam toen dit staan:
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 32 50" src="https://github.com/user-attachments/assets/8f2924c6-5029-4621-b355-000ea7c0f56f" />

Dit heb ik op verschilende manieren proberen optelossen. Ik heb als eerst de JSON-buffer vergroot zodat je meer kB kan ophalen. Verandere het naar 16384.
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 35 55" src="https://github.com/user-attachments/assets/256d6e32-18f1-4737-8351-3a9703e538ab" />



