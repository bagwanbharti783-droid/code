# Portable Ultrasonic Trench Logger for OFC Cable

## Overview
This project is a **Portable Ultrasonic Trench Logger** designed for **Optical Fiber Cable (OFC)** trench monitoring.  
The system measures trench depth using an ultrasonic sensor and logs GPS coordinates along with depth data into an SD card.

---

# Components Used

1. Arduino UNO  
2. HC-SR04 Ultrasonic Sensor  
3. NEO-6M GPS Module  
4. 16x2 I2C LCD  
5. SD Card Module  
6. Push Button  
7. Buzzer  

---

# Connections

## HC-SR04 Ultrasonic Sensor

| HC-SR04 Pin | Arduino UNO |
|---|---|
| VCC | 5V |
| GND | GND |
| TRIG | D7 |
| ECHO | D8 |

---

## GPS Module (NEO-6M)

| GPS Pin | Arduino UNO |
|---|---|
| TX | D2 |
| RX | D3 |
| VCC | 5V |
| GND | GND |

---

## SD Card Module

| SD Module Pin | Arduino UNO |
|---|---|
| CS | D10 |
| MOSI | D11 |
| MISO | D12 |
| SCK | D13 |

---

## LCD I2C

| LCD Pin | Arduino UNO |
|---|---|
| SDA | A4 |
| SCL | A5 |

---

## Push Button

| Button Side | Connection |
|---|---|
| One Side | D4 |
| Other Side | GND |

---

## Buzzer

| Buzzer Pin | Connection |
|---|---|
| + | D5 |
| - | GND |

---

# Arduino Code

```cpp
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
```

---

# Features

- Real-time trench depth measurement
- GPS location tracking
- SD card data logging
- LCD live display
- Push-button data recording
- Buzzer confirmation alert

---

# Output CSV Format

```csv
Record,Depth(cm),Latitude,Longitude
1,45.23,12.345678,77.123456
2,47.10,12.345690,77.123470
```

---

# Applications

- Optical Fiber Cable (OFC) trench monitoring
- Underground cable depth logging
- Smart infrastructure surveying
- Field data collection systems

---