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

Nu gaan we er voor zorgen dat je in je serial monitor een planten naam kan typen en dat dan de waterbehoefte te zien is. Dit is mij niet helemaal gelukt, ik zal uitleggen wat ik heb gedaan en dan waar het fout gaat. 

Plak deze code in scetch:
```
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecure.h>
#include <ArduinoJson.h>

const char* ssid = "JE_WIFI_NAAM";
const char* password = "JE_WIFI_WACHTWOORD";
const char* apiKey = "YOUR_API_KEY_HERE";

String plantName = "";  // wordt ingevuld via Serial

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  Serial.print("Verbinden met WiFi...");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println(" Verbonden!");

  Serial.println();
  Serial.println("Typ de naam van een plant en druk op Enter:");
}

void loop() {
  // check of er iets via Serial is ingevoerd
  if (Serial.available() > 0) {
    plantName = Serial.readStringUntil('\n');
    plantName.trim();  // spaties en \r verwijderen
    if (plantName.length() > 0) {
      zoekPlant(plantName);
      Serial.println("\nTyp nog een plantnaam om opnieuw te zoeken:");
    }
  }
}

void zoekPlant(String naam) {
  WiFiClientSecure client;
  client.setInsecure(); // accepteer alle certificaten

  HTTPClient http;
  http.setTimeout(15000);

  String url = "https://perenual.com/api/species-list?key=" + String(apiKey) + "&q=" + naam;
  Serial.println();
  Serial.println("Verbinding maken met:");
  Serial.println(url);

  if (http.begin(client, url)) {
    int httpCode = http.GET();
    if (httpCode > 0) {
      Serial.printf("HTTP code: %d\n", httpCode);
      String payload = http.getString();
      Serial.print("Ontvangen lengte: ");
      Serial.println(payload.length());

      DynamicJsonDocument doc(16384);
      DeserializationError error = deserializeJson(doc, payload);
      if (error) {
        Serial.print("JSON parse error: ");
        Serial.println(error.c_str());
        return;
      }

      if (doc["data"].size() > 0) {
        String name = doc["data"][0]["common_name"] | "Onbekend";
        String watering = doc["data"][0]["watering"] | "Onbekend";
        Serial.println("------------------------------");
        Serial.println("Plant gevonden:");
        Serial.println("Naam: " + name);
        Serial.println("Waterbehoefte: " + watering);
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

```
In je serial monitor kan je de naam van de plant invullen zoals op deze foto:
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 57 17" src="https://github.com/user-attachments/assets/e177dbaa-42ee-4631-a4a4-cf9f687655a3" />
Helaas kreeg ik hier een error:
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 58 02" src="https://github.com/user-attachments/assets/4aeada68-efcf-4fdf-a3ab-610d1a26be2f" />

Ik heb meerdere dingen geprobeerd om het probleem te kunnen oplossen maar helaas is het mij niet gelukt. Ik zal doornemen wat ik heb gedaan. 

Het eerste probleem wat ik heb aangepakt is dat Perenual is overgestapt naar de API v2 dus moet je ook die aanvragen. 
Dus je moet de String url aanpassen van door er v2 in te zetten
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-17 om 09 00 02" src="https://github.com/user-attachments/assets/638202ea-32e8-4970-b092-b968dba51af9" />

Ook kan je er voor zorgen dat het geen hoofdletters en spaties stuurt naar de API want daar is de API erg gevoelig voor. Voeg daarom dit toe:
```
naam.toLowerCase();
naam.replace(" ", "%20");
```
Zorg er voor de de naam het zelfde is als de naam die je bovenin gebruikt bij de ```String plantName = "";``` dus in dit geval zou je: 
```
plantName.toLowerCase();
plantName.replace(" ", "%20");
```
moeten gebruiken. En ook in de String url moet je het veranderen naar: ``` String url = "https://perenual.com/api/v2/species-list?key=" + String(apiKey) + "&q=" + plantName;```

Nu lukt het om de plant op te zoeken in de API alleen is het nog niet mogelijk om de waterbehoefte op te zoeken. Hier een voorbeeld: 
<img width="480" height="301" alt="Scherm­afbeelding 2025-10-17 om 09 09 10" src="https://github.com/user-attachments/assets/02d10a39-bd39-452b-945a-00d2768cc91f" />
<img width="480" height="301" alt="Scherm­afbeelding 2025-10-17 om 09 09 32" src="https://github.com/user-attachments/assets/ac3073ef-5654-47e1-a535-69b59a48069f" />
Er staat dat de waterbehoefte onbekend is maar het staat wel in de database. 

Dus heb ik het geprobeert om het via een andere manier uit de API te halen. 
Helemaal onderaan gaan moet je dit stukje, 
<img width="960" height="803" alt="Scherm­afbeelding 2025-10-17 om 09 15 51" src="https://github.com/user-attachments/assets/bc9b5fbb-b2b3-40db-a69c-b8793e766f40" />
veranderen met dit: 
```
// Check of er resultaten zijn
      if (doc["data"].size() > 0) {
        int id = doc["data"][0]["id"] | -1;
        String commonName = doc["data"][0]["common_name"] | "Onbekend";
        Serial.printf("Eerste resultaat: %s (ID: %d)\n", commonName.c_str(), id);

        if (id != -1) {
          haalPlantDetails(id);
        }
      } else {
        Serial.println("Geen planten gevonden met die naam.");
      }

    } else {
      Serial.printf("Fout bij verbinding: %s\n", http.errorToString(httpCode).c_str());
    }
    http.end();
  } else {
    Serial.println("Kon verbinding niet starten (HTTPS).");
  }
}
```
daar onder moet je een nieuw void toevoegen:
```
void haalPlantDetails(int id) {
  WiFiClientSecure client;
  client.setInsecure();

  HTTPClient http;
  String url = "https://perenual.com/api/v2/species/details/" + String(id) + "?key=" + String(apiKey);
  Serial.println("\nOphalen van details:");
  Serial.println(url);

  if (http.begin(client, url)) {
    int httpCode = http.GET();

    if (httpCode > 0) {
      Serial.printf("HTTP code: %d\n", httpCode);
      String payload = http.getString();

      DynamicJsonDocument doc(32768);
      DeserializationError error = deserializeJson(doc, payload);

      if (error) {
        Serial.print("JSON parse error (details): ");
        Serial.println(error.c_str());
        http.end();
        return;
      }

      String name = doc["common_name"] | "Onbekend";
      String watering = doc["watering"] | "Onbekend";

      Serial.println("------------------------------");
      Serial.println("Plant gevonden (details):");
      Serial.println("Naam: " + name);
      Serial.println("Waterbehoefte: " + watering);
      Serial.println("------------------------------");

    } else {
      Serial.printf("Fout bij ophalen details: %s\n", http.errorToString(httpCode).c_str());
    }
    http.end();
  }
}
```
