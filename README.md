# Warning-Code
```c
#define BLYNK_TEMPLATE_ID "TMPL3NLw3Kr-5"
#define BLYNK_TEMPLATE_NAME "AIR Quality"
#define BLYNK_AUTH_TOKEN "Z29qqK8mbQH9sf2iGEojdfhLvmAg6AFN"
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>
#include <DHT.h>
#include <BlynkSimpleEsp8266.h>

BlynkTimer timer;
#define BLYNK_PRINT Serial    // Comment this out to disable prints and save space
char auth[] = BLYNK_AUTH_TOKEN;

char ssid[] = "TRYME"; // Enter WIFI Name Here
char pass[] = "12345678"; // Enter WIFI Password Here
const char* gasIFTTTKey = "bm3rXYTDiawPYVrzmRvBmY";

int mq2 = A0; // smoke sensor is connected with the analog pin A0
int data = 0;
#define DHTPIN 2    // DHT11 data pin is connected to GPIO 2
#define DHTTYPE DHT11 // DHT type is DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  Blynk.begin(auth, ssid, pass);
  timer.setInterval(1000L, getSendData);
    dht.begin();

}

void loop() {
  timer.run(); // Initiates SimpleTimer
  Blynk.run();
}

void getSendData() {
  data = analogRead(mq2);
  Blynk.virtualWrite(V2, data);

  Serial.print("Sensor Reading: ");
  Serial.println(data);

  if (data > 100) {
    Serial.println("Gas Leak Detected!");
    sendIFTTTEvent();  }
}


void sendIFTTTEvent() {
  std::unique_ptr<BearSSL::WiFiClientSecure> client(new BearSSL::WiFiClientSecure);
  client->setInsecure();

  HTTPClient https;

  Serial.print("[HTTPS] begin...\n");
  if (https.begin(*client, "https://maker.ifttt.com/trigger/gas_leakd/with/key/" + String(gasIFTTTKey))) {
    Serial.print("[HTTPS] GET...\n");
    int httpCode = https.GET();

    // httpCode will be negative on error
    if (httpCode > 0) {
      Serial.printf("[HTTPS] GET... code: %d\n", httpCode);
      if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
        String payload = https.getString();
        Serial.println(payload);
      }
    } else {
      Serial.printf("[HTTPS] GET... failed, error: %s\n", https.errorToString(httpCode).c_str());
    }

    https.end();
  } else {
    Serial.printf("[HTTPS] Unable to connect\n");
  }
}

```
