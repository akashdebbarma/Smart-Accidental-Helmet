# Smart-Accidental-Helmet

An IoT-based Smart Helmet System designed to improve rider safety by automatically detecting accidents and sending emergency alerts with live GPS location.

Features
Accident detection using MPU6050 sensor
Live GPS location tracking
Automatic emergency SMS alert
Automatic emergency calling
Google Maps location sharing
Real-time rider safety monitoring
Components Used
Arduino UNO
MPU6050 Accelerometer & Gyroscope
GPS Neo-6M Module
A7670 4G SIMCOM Module
Working

The MPU6050 sensor continuously monitors helmet movement. If sudden abnormal motion exceeds the threshold value, the system detects an accident, fetches live GPS coordinates, sends an emergency SMS with Google Maps link, and automatically calls the saved emergency contact number.

Applications
Smart bike helmets
Delivery rider safety
Highway accident monitoring
Industrial worker safety
Technologies Used
Embedded C++
Arduino IDE
IoT
GPS & 4G Communication
Future Improvements
AI-based accident prediction
Mobile application integration
Cloud monitoring dashboard
Health monitoring sensors

code
#include <SoftwareSerial.h>
#include <TinyGPS++.h>
#include <Wire.h>
#include <MPU6050.h>

// GSM
SoftwareSerial gsm(7, 8);

// Mobile Number
String phoneNumber = "+917005469421";

// GPS
SoftwareSerial gpsSerial(4, 3);
TinyGPSPlus gps;

// MPU6050
MPU6050 mpu;

int16_t ax, ay, az;
int16_t gx, gy, gz;

bool accidentDetected = false;

// Accident Limit
int accidentThreshold = 17000;

// =======================================

void setup() {

  Serial.begin(9600);

  // GSM Start
  gsm.begin(115200);

  // GPS Start
  gpsSerial.begin(9600);

  // MPU Start
  Wire.begin();
  mpu.initialize();

  Serial.println("SMART HELMET SYSTEM");
  delay(3000);

  // GSM Check
  sendCommand("AT", 1000);
  sendCommand("AT+CPIN?", 1000);
  sendCommand("AT+CSQ", 1000);
  sendCommand("AT+CREG?", 1000);

  // MPU Check
  if (mpu.testConnection()) {
    Serial.println("MPU6050 Connected");
  } else {
    Serial.println("MPU6050 Failed");
  }

  Serial.println("System Ready...");
}

// =======================================

void loop() {

  // GPS Read
  while (gpsSerial.available()) {
    gps.encode(gpsSerial.read());
  }

  // MPU Read
  mpu.getMotion6(&ax, &ay, &az, &gx, &gy, &gz);

  Serial.print("AX: ");
  Serial.print(ax);

  Serial.print(" AY: ");
  Serial.print(ay);

  Serial.print(" AZ: ");
  Serial.println(az);

  // Accident Detect
  if (abs(ax) > accidentThreshold ||
      abs(ay) > accidentThreshold ||
      abs(az) > accidentThreshold) {

    if (!accidentDetected) {

      accidentDetected = true;

      Serial.println("ACCIDENT DETECTED!");

      sendLocationSMS();

      delay(5000);

      makeCall();

      delay(10000);

      accidentDetected = false;
    }
  }

  delay(500);
}

// =======================================

void sendCommand(String cmd, int waitTime) {

  gsm.println(cmd);

  delay(waitTime);

  while (gsm.available()) {
    Serial.write(gsm.read());
  }
}

// =======================================

void sendLocationSMS() {

  Serial.println("Sending SMS...");

  String latitude = "0.000000";
  String longitude = "0.000000";

  if (gps.location.isValid()) {

    latitude = String(gps.location.lat(), 6);
    longitude = String(gps.location.lng(), 6);
  }

  String message = "ACCIDENT DETECTED!\n";
  message += "Need Help Immediately!\n";
  message += "Location:\n";
  message += "https://maps.google.com/?q=";
  message += latitude;
  message += ",";
  message += longitude;

  sendCommand("AT+CMGF=1", 1000);

  gsm.print("AT+CMGS=\"");
  gsm.print(phoneNumber);
  gsm.println("\"");

  delay(1000);

  gsm.print(message);

  delay(500);

  gsm.write(26);

  delay(5000);

  Serial.println("SMS SENT");
}

// =======================================

void makeCall() {

  Serial.println("Calling...");

  gsm.print("ATD");
  gsm.print(phoneNumber);
  gsm.println(";");

  delay(20000);

  gsm.println("ATH");

  Serial.println("Call Ended");
}
