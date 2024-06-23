# Real-Time-Indoor-Temperature-Humidity-Gas-Dust-Monitoring-

#define BLYNK_TEMPLATE_ID "TMPL6o0HmkNZh"

#define BLYNK_TEMPLATE_NAME "Air Quality 2"

#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

// Blynk Auth Token
char auth[] = "bwY36Vrf5wFF56JSms-KitPNHyAumhiV";

// WiFi credentials
char ssid[] = "APA";  // Replace with your WiFi SSID
char pass[] = "wokwiiii";  // Replace with your WiFi Password

// Pin definitions
#define DHTPIN D5
#define DHTTYPE DHT11
#define MQ135PIN A0
#define LEDPIN D0
#define BUZZERPIN D6
#define RELAYPIN D2
#define DUST_LED_PIN D7
#define DUST_ANALOG_PIN D3

// Create instances
DHT dht(DHTPIN, DHTTYPE);

int temperatureThreshold = 30;
int gasThreshold = 150;
int dustThreshold = 250;

BlynkTimer timer;

// Function to read dust sensor
int readDustSensor() {
  digitalWrite(DUST_LED_PIN, LOW); // Turn on the dust sensor LED
  delayMicroseconds(280);
  int dustValue = analogRead(DUST_ANALOG_PIN);
  delayMicroseconds(40);
  digitalWrite(DUST_LED_PIN, HIGH); // Turn off the dust sensor LED
  delayMicrosecon
