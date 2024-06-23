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
  delayMicroseconds(9680);
  return dustValue;
}

// Function to send sensor data to Blynk
void sendSensorData() {
  float h = dht.readHumidity();
  float t = dht.readTemperature(); // Read temperature as Celsius
  int gasValue = analogRead(MQ135PIN); // Read gas sensor value
  int dustValue = readDustSensor(); // Read dust sensor value

  Serial.print("Temperature: ");
  Serial.println(t);
  Serial.print("Humidity: ");
  Serial.println(h);
  Serial.print("Gas Value: ");
  Serial.println(gasValue);
  Serial.print("Dust Value: ");
  Serial.println(dustValue);

  // Send to Blynk
  Blynk.virtualWrite(V2, t);
  Blynk.virtualWrite(V3, h);
  Blynk.virtualWrite(V4, dustValue);
  Blynk.virtualWrite(V1, gasValue);

  // Control fan based on temperature threshold
  if (t > temperatureThreshold || gasValue > gasThreshold || dustValue > dustThreshold) {
    digitalWrite(RELAYPIN, LOW);  // Turn on the fan
  } else {
    digitalWrite(RELAYPIN, HIGH);  // Turn off the fan
  }

  // Control buzzer and LED based on gas and dust thresholds
  if (gasValue > gasThreshold || dustValue > dustThreshold) {
    digitalWrite(BUZZERPIN, HIGH);  // Turn on the buzzer
    digitalWrite(LEDPIN, HIGH); // Turn on the LED 
  } else {
    digitalWrite(BUZZERPIN, LOW);  // Turn off the buzzer
    digitalWrite(LEDPIN, LOW); // Turn off the LED
  }
}

// Blynk functions to update thresholds
BLYNK_WRITE(V0) {
  temperatureThreshold = param.asInt();
}

void setup() {
  // Debug console
  Serial.begin(115200);
  Serial.println("Starting...");

  // Initialize the DHT sensor
  dht.begin();
  Serial.println("DHT sensor initialized");

  // Set pin modes
  pinMode(LEDPIN, OUTPUT);
  pinMode(BUZZERPIN, OUTPUT);
  pinMode(RELAYPIN, OUTPUT);
  pinMode(DUST_LED_PIN, OUTPUT);
  pinMode(MQ135PIN, INPUT);

  // Turn off fan, LED, and buzzer initially
  digitalWrite(RELAYPIN, HIGH);
  digitalWrite(LEDPIN, LOW);
  digitalWrite(BUZZERPIN, LOW);
  digitalWrite(DUST_LED_PIN, HIGH); // Ensure the dust sensor LED is off initially

  // Connect to WiFi and Blynk
  Blynk.begin(auth, ssid, pass);

  // Setup timer to read sensor data every 2 seconds
  timer.setInterval(2000L, sendSensorData);
}

void loop() {
  Blynk.run();
  timer.run();
}
