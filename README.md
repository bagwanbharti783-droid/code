/*
---------------------------------------------------------
Portable Ultrasonic Trench Logger for OFC Cable
---------------------------------------------------------

Components Used:
1. Arduino UNO
2. HC-SR04 Ultrasonic Sensor
3. NEO-6M GPS Module
4. 16x2 I2C LCD
5. SD Card Module
6. Push Button
7. Buzzer

---------------------------------------------------------
CONNECTIONS
---------------------------------------------------------

HC-SR04:
VCC  -> 5V
GND  -> GND
TRIG -> D7
ECHO -> D8

GPS MODULE:
TX -> D2
RX -> D3
VCC -> 5V
GND -> GND

SD CARD MODULE:
CS   -> D10
MOSI -> D11
MISO -> D12
SCK  -> D13

LCD I2C:
SDA -> A4
SCL -> A5

BUTTON:
One side -> D4
Other side -> GND

BUZZER:
+ -> D5
- -> GND

---------------------------------------------------------
*/

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <TinyGPS++.h>
#include <SoftwareSerial.h>
#include <SPI.h>
#include <SD.h>

// ---------------- LCD ----------------
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ---------------- GPS ----------------
TinyGPSPlus gps;
SoftwareSerial gpsSerial(2, 3); // RX, TX

// ---------------- Ultrasonic ----------------
const int trigPin = 7;
const int echoPin = 8;

// ---------------- SD Card ----------------
const int chipSelect = 10;

// ---------------- Button & Buzzer ----------------
const int buttonPin = 4;
const int buzzerPin = 5;

// ---------------- Variables ----------------
long duration;
float distance;

File dataFile;
int recordNumber = 1;

// ------------------------------------------------
// SETUP
// ------------------------------------------------
void setup()
{
  Serial.begin(9600);

  // GPS serial
  gpsSerial.begin(9600);

  // LCD initialize
  lcd.init();
  lcd.backlight();

  // Pin Modes
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(buttonPin, INPUT_PULLUP);

  pinMode(buzzerPin, OUTPUT);

  // Welcome Message
  lcd.setCursor(0, 0);
  lcd.print("Trench Logger");
  lcd.setCursor(0, 1);
  lcd.print("Initializing");
  delay(2000);

  // SD Card Initialization
  if (!SD.begin(chipSelect))
  {
    lcd.clear();
    lcd.print("SD Failed!");
    while (1);
  }

  lcd.clear();
  lcd.print("SD Card Ready");
  delay(1500);

  // Create CSV Header
  dataFile = SD.open("GPSLOG.CSV", FILE_WRITE);

  if (dataFile)
  {
    dataFile.println("Record,Depth(cm),Latitude,Longitude");
    dataFile.close();
  }

  lcd.clear();
}

// ------------------------------------------------
// LOOP
// ------------------------------------------------
void loop()
{
  // ---------------- GPS Reading ----------------
  while (gpsSerial.available() > 0)
  {
    gps.encode(gpsSerial.read());
  }

  // ---------------- Ultrasonic Reading ----------------
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH);

  // Distance Calculation
  distance = duration * 0.034 / 2;

  // ---------------- LCD Display ----------------
  lcd.setCursor(0, 0);
  lcd.print("Depth:");
  lcd.print(distance);
  lcd.print("cm ");

  lcd.setCursor(0, 1);

  if (gps.location.isValid())
  {
    lcd.print("GPS OK       ");
  }
  else
  {
    lcd.print("Waiting GPS  ");
  }

  // ---------------- Button Press ----------------
  if (digitalRead(buttonPin) == LOW)
  {
    saveData();
    delay(1000);
  }

  delay(500);
}

// ------------------------------------------------
// SAVE DATA FUNCTION
// ------------------------------------------------
void saveData()
{
  float latitude = gps.location.lat();
  float longitude = gps.location.lng();

  dataFile = SD.open("GPSLOG.CSV", FILE_WRITE);

  if (dataFile)
  {
    dataFile.print(recordNumber);
    dataFile.print(",");

    dataFile.print(distance);
    dataFile.print(",");

    dataFile.print(latitude, 6);
    dataFile.print(",");

    dataFile.println(longitude, 6);

    dataFile.close();

    // LCD Message
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Data Saved!");

    // Buzzer Beep
    digitalWrite(buzzerPin, HIGH);
    delay(200);
    digitalWrite(buzzerPin, LOW);

    delay(1000);

    recordNumber++;
  }
  else
  {
    lcd.clear();
    lcd.print("Save Failed!");
    delay(1000);
  }

  lcd.clear();
}
