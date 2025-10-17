# PlantPulse

In this manual, I show how I tried to retrieve the watering information of a plant from the Perenual API.


## things you need
1. NodeMCU (ESP8266)
2. Arduino IDE


## Step 1 Create API Key
The first step is to create an account with Perenual so that you can get your own API key.
Go to the Perenual website at [https://perenual.com/docs/api](https://perenual.com/docs/api) and click the **GET API KEY & ACCESS** button.
Create an account by clicking **Sign Up**.
<img width="1440" height="804" alt="Scherm­afbeelding 2025-10-16 om 14 47 35" src="https://github.com/user-attachments/assets/2dc7e57b-7a1d-418c-b39f-a0fa858af432" />
Once you’ve created an account, you’ll be taken to this page, where you need to click the **Generate New Key** button.
<img width="1440" height="804" alt="Developer" src="https://github.com/user-attachments/assets/01dde758-081c-4eb2-b927-367629fb8dca" />
You’ll then need to fill in what you’re going to use it for ,for example, a school project. Once you’ve entered that information, you’ll receive your API key.
Make sure to remember where you saved it, as you’ll need it later.
<img width="1440" height="804" alt="Scherm­afbeelding 2025-10-16 om 14 49 55" src="https://github.com/user-attachments/assets/89562dfa-ccb5-4b81-b09a-46d38def9de5" />


## Step 2 Install libraries and basic code
You need to install the **HTTPClient** library by *Adrian McEwen*, as shown in the picture.
In the same way, you also need to install the following libraries: **ArduinoJson** by *Benoit Blanchon*, **WiFiClientSecure**, and **ESP8266WiFi**.
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 11 14" src="https://github.com/user-attachments/assets/d4fa2c74-4a73-49a1-86fb-0f50b6a40ad4" />

Paste this code into your sketch to check if you can connect to the API.
Make sure to enter your own Wi-Fi name and password, it’s often best to connect through your mobile hotspot, as not every Wi-Fi network will work.
You’ll also need to enter your own API key, which you should have just created.


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

If you see this message in your serial monitor, it means the connection was successful.
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 19 34" src="https://github.com/user-attachments/assets/499f0968-e92d-4637-83c9-0bc375274ec8" />


## Step 3 Retrieve information from the API

Now we’re going to make it so that you can type a plant name in the serial monitor and see its watering requirements.
I wasn’t able to get this part fully working, but I’ll explain what I did and where it went wrong.

Paste this code into your sketch:
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
In your serial monitor, you can enter the name of the plant, as shown in this picture:
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 57 17" src="https://github.com/user-attachments/assets/e177dbaa-42ee-4631-a4a4-cf9f687655a3" />
Unfortunately, I got an error at this point:
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-16 om 15 58 02" src="https://github.com/user-attachments/assets/4aeada68-efcf-4fdf-a3ab-610d1a26be2f" />

I tried several things to fix the problem, but unfortunately, I wasn’t able to solve it. I’ll go over what I did.

The first issue I addressed was that Perenual has switched to API v2, so you also need to request access to that version.
Therefore, you need to modify the **String url** by adding **v2** to it.
<img width="960" height="603" alt="Scherm­afbeelding 2025-10-17 om 09 00 02" src="https://github.com/user-attachments/assets/638202ea-32e8-4970-b092-b968dba51af9" />

You can also make sure that no capital letters or spaces are sent to the API, as it’s very sensitive to those.
To do this, add the following:
```
naam.toLowerCase();
naam.replace(" ", "%20");
```
Make sure that the name is the same as the one you used at the top in the ```String plantName = "";``` so in this case, you would use: 
```
plantName.toLowerCase();
plantName.replace(" ", "%20");
```
And in the String url, you also need to change it to: ``` String url = "https://perenual.com/api/v2/species-list?key=" + String(apiKey) + "&q=" + plantName;```

Now it is possible to look up the plant in the API, but it is still not possible to look up the water requirements. Here is an example:
<img width="480" height="301" alt="Scherm­afbeelding 2025-10-17 om 09 09 10" src="https://github.com/user-attachments/assets/02d10a39-bd39-452b-945a-00d2768cc91f" />
<img width="480" height="301" alt="Scherm­afbeelding 2025-10-17 om 09 09 32" src="https://github.com/user-attachments/assets/ac3073ef-5654-47e1-a535-69b59a48069f" />
It says that the water requirements are unknown, but they are in the database.
So I tried to retrieve it from the API in a different way.

At the very bottom, you need to add this piece,
<img width="960" height="803" alt="Scherm­afbeelding 2025-10-17 om 09 15 51" src="https://github.com/user-attachments/assets/bc9b5fbb-b2b3-40db-a69c-b8793e766f40" />
replace with this:
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
Below that, you need to add a new void:
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

      DynamicJsonDocument doc(16384);
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

Now the first fetch operation works, but the second one does not. So converting from name to ID number works, but from ID number to water requirements does not. Then you get the message 'connection failed'.
<img width="953" height="392" alt="Scherm­afbeelding 2025-10-17 om 15 20 46" src="https://github.com/user-attachments/assets/4f02adc2-2d74-4cd2-96bc-bcb1a14994c3" />

De vertaling naar Engels is:

I tried a lot more with ChatGPT afterwards, but nothing worked, so I have to give up because there was no more time. 
Here is a link to my chat: https://chatgpt.com/share/68f1ff39-7514-8000-92a7-ebe3213b80d8 

De vertaling naar Engels is:

Here are the links to my ChatGPT chats for translating the text: 
https://chatgpt.com/share/68f24ce3-bd34-8000-9d14-0d948412d499
https://chatgpt.com/share/68f24cce-ecf8-8000-aa3d-3b6d208fb570


