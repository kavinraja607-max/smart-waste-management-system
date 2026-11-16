# smart-waste-management-system

#include <ESP32Servo.h>

// Sensor pins
#define MOISTURE_PIN 34
#define METAL_PIN 27

// Ultrasonic pins
#define TRIG_PIN 5
#define ECHO_PIN 18

// Output pins
#define SERVO_PIN 13
#define BUZZER_PIN 26

Servo sortingServo;

// Calibration values
int moistureLimit = 2000;

// Servo positions
int wetPosition = 30;
int dryPosition = 90;
int metalPosition = 150;
int centerPosition = 90;

long duration;
float distance;

void setup() {

  Serial.begin(115200);

  pinMode(METAL_PIN, INPUT);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);

  sortingServo.attach(SERVO_PIN);
  sortingServo.write(centerPosition);

  Serial.println("Smart Waste Management Started");

  delay(1000);
}

float getDistance() {

  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);

  digitalWrite(TRIG_PIN, LOW);

  duration = pulseIn(ECHO_PIN, HIGH, 30000);

  if (duration == 0) {
    return -1;
  }

  distance = duration * 0.0343 / 2;

  return distance;
}

void sortWaste(int position, String type) {

  Serial.print("Waste Type: ");
  Serial.println(type);

  sortingServo.write(position);

  delay(1500);

  // Return to center
  sortingServo.write(centerPosition);

  digitalWrite(BUZZER_PIN, HIGH);
  delay(300);
  digitalWrite(BUZZER_PIN, LOW);

  delay(1000);
}

void loop() {

  // Read moisture sensor
  int moistureValue =
    analogRead(MOISTURE_PIN);

  // Read metal sensor
  // Assumes LOW means metal detected
  int metalValue =
    digitalRead(METAL_PIN);

  // Read bin level distance
  float binDistance = getDistance();

  Serial.println("--------------------");

  Serial.print("Moisture: ");
  Serial.println(moistureValue);

  Serial.print("Metal Sensor: ");
  Serial.println(metalValue);

  Serial.print("Bin Distance: ");
  Serial.println(binDistance);

  // Check bin level
  if (binDistance > 0 && binDistance < 5) {

    Serial.println("BIN FULL!");

    digitalWrite(BUZZER_PIN, HIGH);
    delay(1000);
    digitalWrite(BUZZER_PIN, LOW);

    delay(2000);
    return;
  }

  // Waste classification
  if (metalValue == LOW) {

    sortWaste(metalPosition, "METAL");

  }

  else if (moistureValue < moistureLimit) {

    sortWaste(wetPosition, "WET WASTE");

  }

  else {

    sortWaste(dryPosition, "DRY WASTE");

  }

  delay(1000);
}
