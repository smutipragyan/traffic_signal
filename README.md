// C++ code for Arduino
//

int red = 12, yellow = 10, green = 8, button = 7;
int TRIG = 6, ECHO = 5;
long duration;
int distance;
bool vehicleDetected = false;

void setup() {
    pinMode(red, OUTPUT);
    pinMode(yellow, OUTPUT);
    pinMode(green, OUTPUT);
    pinMode(button, INPUT_PULLUP);
    pinMode(TRIG, OUTPUT);
    pinMode(ECHO, INPUT); 
  
   Serial.begin(9600); // For debugging distance readings
}

// Function to get distance from Ultrasonic Sensor
int getDistance() {
    digitalWrite(TRIG, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG, LOW);
    duration = pulseIn(ECHO, HIGH, 30000); // 30ms timeout
    if (duration == 0) return 1000; // If no echo received, return a large distance
    return duration * 0.034 / 2;
}

void loop() {
    distance = getDistance();
    Serial.print("Distance: ");
    Serial.println(distance);
    if (distance < 10) { // If vehicle detected (within 10cm)
        vehicleDetected = true;
    } else {
        // If no vehicle for 30 seconds, stay red to save power
        unsigned long startTime = millis();
        while (millis() - startTime < 30000) {
            if (getDistance() < 10) {
                vehicleDetected = true;
                break;
            }
        }
        vehicleDetected = false;
    }
    if (!vehicleDetected) {
        // No vehicle detected, stay Red
        digitalWrite(red, HIGH);
        digitalWrite(yellow, LOW);
        digitalWrite(green, LOW);
    } else {
        // Vehicle detected, normal traffic cycle
        if (digitalRead(button) == LOW) {
            // Pedestrian crossing: Turn Red immediately
            digitalWrite(green, LOW);
            digitalWrite(yellow, LOW);
            digitalWrite(red, HIGH);
            delay(5000); // Pedestrian crosses
        } else {
            // Normal Traffic Light Cycle
            digitalWrite(green, HIGH);
            digitalWrite(yellow, LOW);
            digitalWrite(red, LOW);
            delay(5000);
            digitalWrite(green, LOW);
            digitalWrite(yellow, HIGH);
            delay(2000);
            digitalWrite(yellow, LOW);
            digitalWrite(red, HIGH);
            delay(5000);
        }
    }
}
