# Water-tank-level-indicator-
#define BLYNK_TEMPLATE_ID "TMPL3b97-M_LQ"
#define BLYNK_TEMPLATE_NAME "water level final"
#define BLYNK_AUTH_TOKEN "ox_5mwOvK0uJIGRUQLAN_FmBusHAzX9G"
#include <LiquidCrystal_I2C.h>
#define BLYNK_PRINT Serial
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

char auth[] = BLYNK_AUTH_TOKEN; 
char ssid[] = "BinuCandace";       
char pass[] = "binu2412";       

BlynkTimer timer;
#define trig D7
#define echo D8
#define LED2 D3
#define relay 3
int MaxLevel = 25;

int Level1 = 20; // 75% of MaxLevel
int Level2 = 16; // 65% of MaxLevel
int Level3 = 14; // 55% of MaxLevel
int Level4 = 11; // 45% of MaxLevel
int Level5 = 5;  // 35% of MaxLevel

void setup() {
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
  pinMode(trig, OUTPUT);
  pinMode(echo, INPUT);
  pinMode(LED2, OUTPUT);
  pinMode(relay, OUTPUT);
  digitalWrite(relay, HIGH);
  Blynk.begin(auth, ssid, pass, "blynk.cloud", 80);

  lcd.setCursor(0, 0);
  lcd.print("Water level");
  lcd.setCursor(4, 1);
  lcd.print("Monitoring");
  delay(4000);
  lcd.clear();

  //Call the functions
  timer.setInterval(1000L, ultrasonic); // Adjusted interval to 1000ms for stability
}
void ultrasonic() {
  digitalWrite(trig, LOW);
  delayMicroseconds(4);
  digitalWrite(trig, HIGH);
  delayMicroseconds(10);
  digitalWrite(trig, LOW);
  long t = pulseIn(echo, HIGH);
  int distance = t / 29 / 2;

  // Cap the distance at 25 cm for display purposes
  int displayDistance = distance > 25 ? 25 : distance;

  int blynkDistance = (distance - MaxLevel) * -1;
  if (distance <= MaxLevel) {
    Blynk.virtualWrite(V0, blynkDistance);
  } else {
    Blynk.virtualWrite(V0, 0);
  }

  // Display the capped distance on the LCD
  lcd.setCursor(0, 0);
  lcd.print("LEVEL: ");
  lcd.print(displayDistance);
  lcd.print(" cm  ");

  // Determine and display the water level status based on the actual distance
  lcd.setCursor(0, 1);
  if (Level1 <= distance) {
    lcd.print("Very Low  ");
    digitalWrite(LED2, HIGH);
  } else if (Level2 <= distance && Level1 > distance) {
    lcd.print("Low      ");
    digitalWrite(LED2, LOW);
  } else if (Level3 <= distance && Level2 > distance) {
    lcd.print("Medium   ");
    digitalWrite(LED2, LOW);
  } else if (Level4 <= distance && Level3 > distance) {
    lcd.print("High     ");
    digitalWrite(LED2, LOW);
  } else if (Level5 >= distance) {
    lcd.print("Full     ");
    digitalWrite(LED2, HIGH);
  }
}


//Get the button value
BLYNK_WRITE(V1) {
  bool Relay = param.asInt();
  if (Relay == 1) {
    digitalWrite(relay, LOW);
    lcd.setCursor(0, 1);
    lcd.print("Motor is    ON ");
  } else {
    digitalWrite(relay, HIGH);
    lcd.setCursor(0, 1);
    lcd.print("Motor is   OFF");
  }
}

void loop() {
  Blynk.run();
  timer.run();
}
