# ESP32 Sketch

## BOM

* [ESP32](https://www.az-delivery.de/en/products/esp32-developmentboard)
* [Kit RFID RC522](https://www.ebay.fr/itm/133514459432)
* Breadboard
* Wires

## Wiring

* RST -> G22
* SDA -> G21
* MOSI -> G23
* MISO -> G19
* SCK -> G18

## Software

* https://github.com/miguelbalboa/rfid

## IDE

Install :

* VSCode
* [PlatformIO IDE](https://docs.platformio.org/en/latest/integration/ide/vscode.html) plugin

* Fix permissions on /dev/ttyUSB0

```sh
sudo usermod -a -G dialout $(id -un)
```

Create a new project :

* Board Name: **Espressif ESP32 Dev Module**
* Framework: **Arduino**

In PIO Home, add the library **MFRC522 by Miguel André Balboa** (search for **header:MFRC522.h**).

Add the library **PubSubClient by Nick O'Leary**.

For src/main.cpp, use:

```cpp
#include <WiFi.h>   // Utilisation de la librairie WiFi.h
#include <PubSubClient.h>

//WIFI
const char* ssid = "iPhone de Florian";  //  SSID Wifi
const char* password = "XXXXXXX";  //mot de passe Wifi


// MQTT Broker
const char *mqtt_broker = "10.6.82.226";
const char *topic = "esp32";
const char *mqtt_username = "admin";
const char *mqtt_password = "public";
const int mqtt_port = 32574;

WiFiClient espClient;
PubSubClient client(espClient);

void callback(char *topic, byte *payload, unsigned int length) {
  Serial.print("Message arrived in topic: ");
  Serial.println(topic);
  Serial.print("Message:");
  for (int i = 0; i < length; i++) {
      Serial.print((char) payload[i]);
  }
  Serial.println();
  Serial.println("-----------------------");
}


void setup() {
  Serial.begin(115200);   // Initialisation du moniteur série à 115200
  delay(1000);

  Serial.println("\n");
  WiFi.begin(ssid,password);  // Initialisation avec WiFi.begin / ssid et password
  Serial.print("Attente de connexion ...");  // Message d'attente de connexion
  while(WiFi.status() != WL_CONNECTED){   // Test connexion

    Serial.print(".");  // Affiche des points .... tant que connexion n'est pas OK

    delay(100);
  }
  Serial.println("\n");
  Serial.println("Connexion etablie !");  // Affiche connexion établie
  Serial.print("Adresse IP: ");
  Serial.println(WiFi.localIP());  // Affiche l'adresse IP de l'ESP32 avec WiFi.localIP

  //connecting to a mqtt broker
  client.setServer(mqtt_broker, mqtt_port);
  client.setCallback(callback);
  while (!client.connected()) {
      String client_id = "esp32-client-";
      client_id += String(WiFi.macAddress());
      Serial.printf("The client %s connects to the public mqtt broker\n", client_id.c_str());
      if (client.connect(client_id.c_str(), mqtt_username, mqtt_password)) {
          Serial.println("Public emqx mqtt broker connected");
      } else {
          Serial.print("failed with state ");
          Serial.print(client.state());
          delay(2000);
      }
  }
  // publish and subscribe
  client.publish(topic, "Hello c'est flo ^^");
  client.subscribe(topic);
}



void loop() { 
  client.loop();
}
```

Monitor messages on the chosen topic:

```sh
sudo dnf install mosquitto
mosquitto_sub -t esp32 -h '10.6.82.56' -p 31053 -u admin -P public
```
