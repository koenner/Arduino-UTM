# Arduino-UTM
Universal_Test_Machine.ino
/*
 * Universal Test Machine Arduino Sketch
 * 
 * This sketch supports a universal test machine for tensile and compression testing.
 * 
 * Load Cell Calibration Instructions:
 * - Apply a known weight to the load cell, run the sketch, and observe the serial output.
 * - Adjust the `calibration_factor` variable until the output matches the known weight.
 * - This will ensure accurate readings during testing.
 */

#include <SPI.h>
#include <SD.h>
#include <Wire.h>
#include <SparkFun_Qwiic_Scale_NAU7802_Arduino_Library.h>

// Pin Definitions
#define PUL_MINUS 0
#define STOP_BUTTON 2
#define REVERSE_LED 3
#define TENSILE_LED 4
#define RED_LED 5
#define COMP_LED 6
#define FORWARD_LED 7
#define DIR_MINUS 8
#define SS_PIN 10
#define FORWARD_BUTTON 14
#define COMP_BUTTON 15
#define TENSILE_BUTTON 16
#define REVERSE_BUTTON 17
#define SDA_PIN 18
#define SCL_PIN 19

// Load Cell Settings
NAU7802 myScale;
float calibration_factor = 1.0;  // Update this value during calibration

// SD Card Settings
File dataFile;

// Function Prototypes
void moveForward();
void performCompressionTest();
void stopAll();
void performTensileTest();
void moveReverse();
void pulseMotor();
void logData();

// Setup function
void setup() {
  // Pin Modes
  pinMode(PUL_MINUS, OUTPUT);
  pinMode(STOP_BUTTON, INPUT);
  pinMode(REVERSE_LED, OUTPUT);
  pinMode(TENSILE_LED, OUTPUT);
  pinMode(RED_LED, OUTPUT);
  pinMode(COMP_LED, OUTPUT);
  pinMode(FORWARD_LED, OUTPUT);
  pinMode(DIR_MINUS, OUTPUT);
  pinMode(FORWARD_BUTTON, INPUT);
  pinMode(COMP_BUTTON, INPUT);
  pinMode(TENSILE_BUTTON, INPUT);
  pinMode(REVERSE_BUTTON, INPUT);

  // Initialize I2C for the Load Cell
  Wire.begin();
  if (!myScale.begin()) {
    Serial.begin(9600);
    Serial.println("Load cell initialization failed!");
  }
  myScale.setSampleRate(NAU7802_SPS_80);

  // Initialize SPI for the SD card
  if (!SD.begin(SS_PIN)) {
    Serial.begin(9600);
    Serial.println("SD card initialization failed!");
  }
  
  // Start Serial for debugging
  Serial.begin(9600);
}

// Loop function
void loop() {
  if (digitalRead(FORWARD_BUTTON) == HIGH) {
    moveForward();
  }

  if (digitalRead(COMP_BUTTON) == HIGH) {
    performCompressionTest();
  }

  if (digitalRead(STOP_BUTTON) == HIGH) {
    stopAll();
  }

  if (digitalRead(TENSILE_BUTTON) == HIGH) {
    performTensileTest();
  }

  if (digitalRead(REVERSE_BUTTON) == HIGH) {
    moveReverse();
  }
}

// Function to move the motor forward (CW)
void moveForward() {
  digitalWrite(FORWARD_LED, HIGH);
  digitalWrite(DIR_MINUS, LOW);
  while (digitalRead(FORWARD_BUTTON) == HIGH) {
    pulseMotor();
  }
  digitalWrite(FORWARD_LED, LOW);
}

// Function to perform a compression test
void performCompressionTest() {
  digitalWrite(COMP_LED, HIGH);
  digitalWrite(DIR_MINUS, LOW);
  dataFile = SD.open("testData.csv", FILE_WRITE);
  while (digitalRead(STOP_BUTTON) == LOW) {
    pulseMotor();
    logData();
  }
  dataFile.close();
  digitalWrite(COMP_LED, LOW);
}

// Function to perform a tensile test
void performTensileTest() {
  digitalWrite(TENSILE_LED, HIGH);
  digitalWrite(DIR_MINUS, HIGH);
  dataFile = SD.open("testData.csv", FILE_WRITE);
  while (digitalRead(STOP_BUTTON) == LOW) {
    pulseMotor();
    logData();
  }
  dataFile.close();
  digitalWrite(TENSILE_LED, LOW);
}

// Function to move the motor in reverse (CCW)
void moveReverse() {
  digitalWrite(REVERSE_LED, HIGH);
  digitalWrite(DIR_MINUS, HIGH);
  while (digitalRead(REVERSE_BUTTON) == HIGH) {
    pulseMotor();
  }
  digitalWrite(REVERSE_LED, LOW);
}

// Function to generate motor pulses
void pulseMotor() {
  digitalWrite(PUL_MINUS, HIGH);
  delayMicroseconds(2000);  // Pulse duration
  digitalWrite(PUL_MINUS, LOW);
  delayMicroseconds(2000);
}

// Function to log data from the load cell
void logData() {
  float load = myScale.getWeight() * calibration_factor;
  dataFile.print(millis());
  dataFile.print(", ");
  dataFile.println(load);
}

// Function to stop all motor movement and save data
void stopAll() {
  digitalWrite(RED_LED, HIGH);
  delay(1000);  // Delay to ensure all processes stop
  digitalWrite(RED_LED, LOW);
}
