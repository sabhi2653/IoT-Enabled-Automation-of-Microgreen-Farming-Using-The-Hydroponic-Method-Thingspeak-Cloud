# IoT-Enabled-Automation-of-Microgreen-Farming-Using-The-Hydroponic-Method-Thingspeak-Cloud
#include <DHT.h>
#include <ThingSpeak.h>
#include <ESP8266WiFi.h>
const int trigPin1 = D5;
const int echoPin1 = D6;
String apiKey = "1G8PO42RMLPB39EF";     //  Enter your Write API key here
const char* server = "api.thingspeak.com";
const char *ssid = "Abhisingh";     // Enter your WiFi Name
const char *pass =  "Abhi@2002"; // Enter your WiFi Password
#define DHTPIN D3          // GPIO Pin where the dht11 is connected
DHT dht(DHTPIN, DHT11);
WiFiClient client;

const int moisturePin = A0;             // moisteure sensor pin
const int motorPin = D0;
const int fan = D1;
const int buzzer = D8;
const int LED1=D4;
const int LED2=D7;
unsigned long interval = 1000;
unsigned long previousMillis = 0;
unsigned long interval1 = 1000;
unsigned long previousMillis1 = 0;
float moisturePercentage;              //moisture reading
float h;                  // humidity reading
float t;                  //temperature reading
long duration1;
int distance1;
int height;

void setup()
{
  Serial.begin(115200);
  delay(10);
  pinMode(trigPin1, OUTPUT);
  pinMode(echoPin1, INPUT);
  pinMode(motorPin, OUTPUT);
  pinMode(fan, OUTPUT);
  pinMode(buzzer, OUTPUT);
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);
  digitalWrite(motorPin, LOW); // keep motor off initally
  dht.begin();
  Serial.println("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, pass);
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(500);
    Serial.print(".");              // print ... till not connected
  }
  Serial.println("");
  Serial.println("WiFi connected");
}

void loop()
{
  unsigned long currentMillis = millis(); // grab current time

  h = dht.readHumidity();     // read humiduty
  t = dht.readTemperature();     // read temperature

  if (isnan(h) || isnan(t))
  {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }

  moisturePercentage = 2*( 100.00 - ( (analogRead(moisturePin) / 1023.00) * 100.00 ) );

  if ((unsigned long)(currentMillis - previousMillis1) >= interval1) {
    Serial.print("Soil Moisture is  = ");
    Serial.print(moisturePercentage);
    Serial.println("%");
    previousMillis1 = millis();
  }

if (moisturePercentage < 25) {
  digitalWrite(motorPin, LOW);   // tun on motor
  digitalWrite(LED1, HIGH); 
 
}
if (moisturePercentage > 25 && moisturePercentage < 30) {
  digitalWrite(motorPin, LOW);        //turn on motor pump
   digitalWrite(LED1, HIGH);
 
}
if (moisturePercentage > 31) {
  digitalWrite(motorPin, HIGH);          // turn off mottor
   digitalWrite(LED1, LOW);

}
if (t < 25) {
  digitalWrite(fan, HIGH);         // tun off fan
  digitalWrite(LED2, LOW);
 
}
if (t > 25 && t < 28) {
  digitalWrite(fan, HIGH);        //turn off fan
  digitalWrite(LED2, LOW);
 
}
if (t > 28) {
  digitalWrite(fan, LOW);          // turn on fan
  digitalWrite(LED2, HIGH);

}
 {
  digitalWrite(trigPin1, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin1, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin1, LOW);
  duration1 = pulseIn(echoPin1, HIGH);
  distance1 = duration1 * 0.034 / 2;
  height= 20-distance1;
  //erial.println(distance1);
  delayMicroseconds(100);
 }
 if (height < 5) {
  digitalWrite(buzzer, LOW);         // tun off BUZZER
 
}
if (height > 5 && height < 6) {
  digitalWrite(buzzer, LOW);        //turn off BUZZER
 
}
if (height > 6) {
  digitalWrite(buzzer, HIGH);          // turn on BUZZER

}


if ((unsigned long)(currentMillis - previousMillis) >= interval) {

  sendThingspeak();           //send data to thing speak
  previousMillis = millis();
  client.stop();
}

}

void sendThingspeak() {
  if (client.connect(server, 80))
  {
    String postStr = apiKey;              // add api key in the postStr string
    postStr += "&field1=";
    postStr += String(moisturePercentage);    // add mositure readin
    postStr += "&field2=";
    postStr += String(t);                 // add tempr readin
    postStr += "&field3=";
    postStr += String(h);                  // add humidity readin
    postStr += "&field4=";
    postStr += String(height);            // add distance readin
    postStr += "\r\n\r\n";

    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(postStr.length());           //send lenght of the string
    client.print("\n\n");
    client.print(postStr);                      // send complete string
    Serial.print("Moisture Percentage: ");
    Serial.print(moisturePercentage);
    Serial.print("%. Temperature: ");
    Serial.print(t);
    Serial.print(" C, Humidity: ");
    Serial.print(h);
    client.print("%\n\n");
    Serial.print(" distance1, Height of Microgreen: ");
    Serial.print(height);
    Serial.println("cm. Sent to Thingspeak.");
  }
  delay(500);
client.stop();
Serial.println("Waiting...");
 
// thingspeak needs minimum 15 sec delay between updates.
delay(1500);
}
