#include <Arduino.h>

#include <WiFi.h>

const char* ssid = "NET_2GDDEAFE";
const char* password = "B3DDEAFE";
WiFiServer server(8080);

int packetCount = 0;

void setup() {
    Serial.begin(115200);
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Conectando ao Wi-Fi...");
    }

    // Print IP address when connected
    Serial.println("Wi-Fi conectado!");
    Serial.print("Endereço IP: ");
    Serial.println(WiFi.localIP()); // Print the IP address

    server.begin();
}

void loop() {
    WiFiClient client = server.available();
    if (client) {
        Serial.println("Cliente conectado!");
        while (client.connected()) {
            if (client.available()) {
                String data = client.readStringUntil('\n');
                client.printf("Recebido %d bytes. Pacotes recebidos: %d\n", data.length(), ++packetCount);
            }
        }
        client.stop();
        Serial.println("Cliente desconectado.");
    }
}