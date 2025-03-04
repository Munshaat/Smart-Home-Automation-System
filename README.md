# Smart-Home-Automation-System
int pirPin = 3;           // PIR sensor output
int ldrPin = A0;          // LDR input (analog)
int relayPowerControl = 6; // Controls power to the relay module
int fanRelay = 7;         // Relay channel 1 (Fan)
int ledRelay = 8;         // Relay channel 2 (LED)
int motionDetected = 0;   // Variable to store PIR state
int threshold = 100;        // Adjust LDR threshold for night detection
unsigned long lastMotionTime = 0; // Stores the last time motion was detected
int motionTimeout = 3000; // 3-second delay after motion stops

void setup() {
    Serial.begin(9600);
    pinMode(pirPin, INPUT);
    pinMode(ldrPin, INPUT);
    pinMode(relayPowerControl, OUTPUT);
    pinMode(fanRelay, OUTPUT);
    pinMode(ledRelay, OUTPUT);
    
    // Ensure all devices are OFF initially
    digitalWrite(relayPowerControl, LOW);
    digitalWrite(fanRelay, HIGH);  // Relays use LOW to turn ON
    digitalWrite(ledRelay, HIGH);
}

void loop() {
    motionDetected = digitalRead(pirPin);  // Read PIR sensor
    int lightLevel = analogRead(ldrPin);   // Read LDR value
    
    Serial.print("PIR: ");
    Serial.print(motionDetected);
    Serial.print(" | LDR: ");
    Serial.println(lightLevel);

    if (motionDetected == HIGH) {
        lastMotionTime = millis(); // Update last motion time

        // Fan turns ON for motion detection (independent of light condition)
        digitalWrite(fanRelay, LOW); // Turn ON fan

        // LED turns ON only if it's dark
        if (lightLevel <= threshold) {
            digitalWrite(ledRelay, LOW); // Turn ON LED
        } else {
            digitalWrite(ledRelay, HIGH); // Turn OFF LED if not dark
        }

        // Activate relay power (since at least one device is ON)
        digitalWrite(relayPowerControl, HIGH);
    } 
    else {
        // Check if 3 seconds have passed since last motion
        if (millis() - lastMotionTime >= motionTimeout) {
            digitalWrite(fanRelay, HIGH); // Turn OFF fan
            digitalWrite(ledRelay, HIGH); // Turn OFF LED

            // Turn OFF relay power only if both devices are OFF
            digitalWrite(relayPowerControl, LOW);
        }
    }

    delay(100); // Small delay for stability
}
