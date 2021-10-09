# Blog 1 Describing the hardware

## Introduction

At the heart of this project is a hardware device. But before that let's explain what is this term called IoT. The Internet of Things (IoT) is a term that refers to the network of devices that connect to the internet. The Internet of Things refers to when commonplace items or things are connected to the internet via sensors and IP addresses. Anything from your shoes to your automobile to your coffee pot may be considered an everyday object. Connecting these gadgets to the internet allows data and user analytics to be extracted. The data and analytics gathered may be utilised for a variety of purposes, including companies, surveys, alerts, and just improving user experiences.

This means we have two systems, a frontend system, which will be our hardware device and a backend system which will be our server. Our IoT device will constitute the JKUAT SES development board based on the esp32 WROOM chips. So if you have any esp32 chip this will work out fine. We used the DHT11 module as our temperature and humidity sensor. With this, we can send temperature and humidity data to our backend services for monitoring purposes.

For connectivity, we are using the WIFI module which is inbuilt on the esp32 WROOM chip. Even for production purposes, if we are to deploy it to homes, I guess most people have WIFI in their homes.

## Circuit

https://circuits4you.com/wp-content/uploads/2018/12/ESP32-Pinout.jpg

https://raw.githubusercontent.com/AchimPieters/esp32-homekit-camera/master/Images/ESP32-38%20PIN-DEVBOARD.png

The DHT11 has the following wires:
- GND is a common ground for both the dht11 and microcontroller.
- 5 V is a positive voltage that powers the dht11.
- Control transmit data

Wiring the DHT11 to the ses dev board is easy. Connect the Red wire or VCC pin to the 5V on the ses dev board and Black wire or GND pin to the ground. Finally, connect the Orange wire or Data pin to pin 15. You can use any GPIO pins as the Data Pin

## DHT11

The electrical resistance between two electrodes is measured by the DHT11 to detect water vapour. A moisture-holding substrate with electrodes placed to the surface serves as the humidity sensor component. When water vapour is absorbed by the substrate, ions are released, increasing the conductivity between the electrodes. The relative humidity affects the change in resistance between the electrodes. The resistance between the electrodes reduces as the relative humidity rises, whereas the resistance rises as the relative humidity falls.

The DHT11 has a built-in thermistor that monitors temperature. The calibration coefficients are stored and controlled by an IC on the device.

## MQTT

You're not incorrect if you think the Internet of Things (IoT) has gone from zero to pervasive hype in a short period. Message Queuing Telemetry Transport, or MQTT, is, I believe, one of the most critical elements affecting the condition of the IoT today.

Message Queuing Telemetry Transport (MQTT) is an acronym for Message Queuing Telemetry Transport. It's a lightweight messaging protocol designed for usage in situations where clients require a minimal code footprint and are linked to unreliable networks or networks with restricted capacity. It's mostly utilised for M2M (machine-to-machine) communication and Internet of Things connectivity.

MQTT was developed in 1999 by Dr Andy Stanford-Clark and Arlen Nipper. The communication method's original aim was to allow monitoring equipment in the oil and gas sector to communicate data to remote servers.

MQTT uses a PUSH/SUBSCRIBE architecture to run on top of TCP/IP. There are two sorts of systems in MQTT architecture: clients and brokers. The server with which the clients communicate is known as a broker. Client messages are received by the broker, who then forwards them to other clients. Clients connect to the broker rather than communicating directly with one another.

Another method MQTT reduces transmissions is by using a well-defined, compact message structure. In comparison to HTTP, each message has a fixed header of only 2 bytes.



## OTA

Instead of needing the user to connect the ESP32 to a computer via USB to execute the update, OTA programming allows the user to update/upload new software to the ESP32 via Wi-Fi.

When there is no physical access to the ESP module, the OTA capability comes in handy. It helps to cut down on the time spent upgrading each ESP module during maintenance.

One of the most useful features of OTA is that it allows a single central location to deliver an update to many ESPs on the same network.

The only drawback is that you must include an additional OTA code with each drawing you publish to use OTA in the next update.

Ways To Implement OTA In ESP32
There are two ways to implement OTA functionality in ESP32.

Basic OTA – Over-the-air updates are sent through Arduino IDE.
Web Updater OTA – Over-the-air updates are sent through a web browser.

On an OTA architecture, there are two key components:
The remote device is in charge of checking for updates, downloading the new version, and installing it on its system.
The cloud server is in charge of creating, delivering, and managing updates to linked devices.

## Code

### Importations

Here we import the necessary libraries we will need for our project

```cpp
#include <Arduino.h>
#include <DNSServer.h>
#include <WebServer.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <Update.h>
#include <WiFiManager.h>
#include <Arduino.h>
#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>
#include "./config.h" //contains our passwords
#include <PubSubClient.h> //For publishing messages to the MQTT Server
#include <ArduinoJson.h> //To create the payload so that we can publish it to the subscribed topics
```

### Credentials

Let's get the credentials for starting the connecting. We need the Wifi credentials also with the MQTT credentials to publish messages to our MQTT server

```cpp
#define WIFI_SSID WIFI_SSID
#define WIFI_PASSWORD WIFI_PASSWORD
#define MQTT_SERVER MQTT_SERVER
#define MQTT_USER MQTT_USER
#define MQTT_PASS MQTT_PASS
```

### Declaration

Here we declare variables and initialize classes.

```cpp

#define CURRENT_VERSION "1.0.0" // The current version of the firmware running
#define DOWNLOAD_URL SERVER_URL // The server url where we will get the latest firmware version
#define DHTPIN 2      // Digital pin connected to the DHT sensor
#define DHTTYPE DHT11 // DHT 11
#define LED 4 // Digital pin connected to the inbuilt LED

#define ARDUINOJSON_USE_LONG_LONG 1
#define ARDUINOJSON_USE_DOUBLE 1


WiFiClient espClient; //Initializing the WiFiClient
PubSubClient client(espClient); // Initializing the PubSubClient

WebServer server(80); // Initializing the WebServer

DHT_Unified dht(DHTPIN, DHTTYPE);

int ledState = LOW; // Set the ledstate to be off at the first instance
const long interval = 1000; // This is the interval we will be using to check for a new version
unsigned long previousMillis = 0; // previous timings in milliseconds
unsigned long currentMillis; // current timings in milliseconds
bool success; // download success
```

### Setup

```cpp
void setup(){

    Serial.begin(115200);// begin Serial at 115200 baud rate

    setup_wifi(); // function to setup up wifi
    espClient.setServer(mqtt_server, 1883); //Setup the MQTT server
    
    // Initialize device.
    dht.begin();
    Serial.println(F("DHTxx Unified Sensor Example"));
    // Print temperature sensor details.
    sensor_t sensor;
    dht.temperature().getSensor(&sensor);
    Serial.println(F("------------------------------------"));
    Serial.println(F("Temperature Sensor Present"));
    
    // Print humidity sensor details.dht.humidity().getSensor(&sensor);
    Serial.println(F("------------------------------------"));
    Serial.println(F("Humidity Sensor Present"));

    Serial.setDebugOutput(true); // Set debug to true in order to print more serial output
    pinMode(LED, OUTPUT); // Initialize the inbuilt led as an output device

    delay(3000); // Set a delay of 3 seconds (optional)

    String version = String("<p>Current Version - v") + String(CURRENT_VERSION) + String("</p>");
    Serial.println(version);

    // Setup Wifi Manager
    WiFiManager wm;
    WiFiManagerParameter versionText(version.c_str());
    wm.addParameter(&versionText);

    if (!wm.autoConnect()){
        Serial.println("failed to connect and hit timeout");
        ESP.restart();
        delay(1000);
    }

    // Check if we need to download a new version
    String downloadUrl = getDownloadUrl();
    if (downloadUrl.length() > 0){
        success = downloadUpdate(downloadUrl);
        if (!success){
            Serial.println("Error updating device");
        }
    }

    server.on("/", handleRoot);
    server.begin();
    Serial.println("HTTP server started");

    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
}
```

### Loop

```cpp
void loop() {
    // Get temperature event and print its value.
    sensors_event_t event;
    dht.temperature().getEvent( & event);
    if (isnan(event.temperature)) {
        Serial.println(F("Error reading temperature!"));
    } else {
        Serial.print(F("Temperature: "));
        Serial.print(event.temperature);
        Serial.println(F("°C"));
    }
    // Get humidity event and print its value.
    dht.humidity().getEvent( & event);
    if (isnan(event.relative_humidity)) {
        Serial.println(F("Error reading humidity!"));
    } else {
        Serial.print(F("Humidity: "));
        Serial.print(event.relative_humidity);
        Serial.println(F("%"));
    }

    currentMillis = millis();

    if (currentMillis - previousMillis >= interval) {
        previousMillis = currentMillis;
        ledState = ledState == LOW ? HIGH : LOW;
        digitalWrite(4, ledState);
    }
    // Just chill
    server.handleClient();
    delay(1000);

    char msg[200];
    if (!espClient.connected()) {
        reconnect();
    }

    StaticJsonBuffer < 300 > JSONbuffer;
    JsonObject & JSONencoder = JSONbuffer.createObject();

    JSONencoder["time"] = millis();
    JSONencoder["temperature"] = event.temperature;
    JSONencoder["humidity"] = event.relative_humidity;

    char JSONmessageBuffer[100];
    JSONencoder.printTo(JSONmessageBuffer, sizeof(JSONmessageBuffer));
    Serial.println(JSONmessageBuffer);

    if (espClient.publish("dht/user_id892", JSONmessageBuffer) == true) {
        Serial.println("Success sending message");
    } else {
        Serial.println("Error sending message");
    }

    espClient.loop();

}

```

### Handle server requests

```cpp
void handleRoot() {
    server.send(200, "text/plain", "v" + String(CURRENT_VERSION));
}
```
### Get download link

```cpp
String getDownloadUrl() {
    HTTPClient http;
    String downloadUrl;
    Serial.print("[HTTP] begin...\n");

    String url = DOWNLOAD_URL;
    http.begin(url);

    Serial.print("[HTTP] GET...\n");
    // start connection and send HTTP header
    int httpCode = http.GET();

    // httpCode will be negative on error
    if (httpCode > 0) {
        // HTTP header has been send and Server response header has been handled
        Serial.printf("[HTTP] GET... code: %d\n", httpCode);

        // file found at server
        if (httpCode == HTTP_CODE_OK) {
            String payload = http.getString();
            Serial.println(payload);
            downloadUrl = payload;
        } else {
            Serial.println("Device is up to date!");
        }
    } else {
        Serial.printf("[HTTP] GET... failed, error: %s\n", http.errorToString(httpCode).c_str());
    }

    http.end();
    Serial.println(downloadUrl);
    return downloadUrl;
}
```

### Download binary firmware

```cpp
/*
   Download binary image and use Update library to update the device.
*/
/*
   Download binary image and use Update library to update the device.
*/
bool downloadUpdate(String url) {
    HTTPClient http;
    Serial.print("[HTTP] Download begin...\n");

    http.begin(url);

    Serial.print("[HTTP] GET...\n");
    // start connection and send HTTP header
    int httpCode = http.GET();
    if (httpCode > 0) {
        // HTTP header has been send and Server response header has been handled
        Serial.printf("[HTTP] GET... code: %d\n", httpCode);

        // file found at server
        if (httpCode == HTTP_CODE_OK) {

            int contentLength = http.getSize();
            Serial.println("contentLength : " + String(contentLength));

            if (contentLength > 0) {
                bool canBegin = Update.begin(contentLength);
                if (canBegin) {
                    WiFiClient stream = http.getStream();
                    Serial.println("Begin OTA. This may take 2 - 5 mins to complete. Things might be quite for a while.. Patience!");
                    size_t written = Update.writeStream(stream);

                    if (written == contentLength) {
                        Serial.println("Written : " + String(written) + " successfully");
                    } else {
                        Serial.println("Written only : " + String(written) + "/" + String(contentLength) + ". Retry?");
                    }

                    if (Update.end()) {
                        Serial.println("OTA done!");
                        if (Update.isFinished()) {
                            Serial.println("Update successfully completed. Rebooting.");
                            ESP.restart();
                            return true;
                        } else {
                            Serial.println("Update not finished? Something went wrong!");
                            return false;
                        }
                    } else {
                        Serial.println("Error Occurred. Error #: " + String(Update.getError()));
                        return false;
                    }
                } else {
                    Serial.println("Not enough space to begin OTA");
                    client.flush();
                    return false;
                }
            } else {
                Serial.println("There was no content in the response");
                client.flush();
                return false;
            }
        } else {
            return false;
        }
    } else {
        return false;
    }
}

```
### Setup WIFI function

```cpp
void setup_wifi() {
    // Connecting to a WiFi network
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("WiFi connected");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
}
```

### Reconnect function

```cpp
void reconnect() {
    // Loop until we're reconnected
    Serial.println("In reconnect...");
    while (!espClient.connected()) {
        Serial.print("Attempting MQTT connection...");
        // Attempt to connect
        if (espClient.connect("SES_DHT", MQTT_USER, MQTT_PASS)) {
            Serial.println("connected");
        } else {
            Serial.print("failed, rc=");
            Serial.print(espClient.state());
            Serial.println(" try again in 5 seconds");
            delay(5000);
        }
    }
}
```

## References
1. https://blackd0t.gitbook.io/africastalking-eris-devkit-playground/arduino/dht11
2. https://lastminuteengineers.com/esp32-ota-web-updater-arduino-ide/
3. https://microcontrollerslab.com/esp32-ota-over-the-air-updates-asyncelegantota-library-arduino/
4. https://www.survivingwithandroid.com/esp32-mqtt-client-publish-and-subscribe/
5. https://www.allacronyms.com/MQTT/Message_Queuing_Telemetry_Transport
6. https://horasvilla.com/iot-and-iiot-what-is-the-difference-between-them/