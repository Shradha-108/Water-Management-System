# Water-Management-System
A water management system using wireless communication, hydrometers, and mechanical level detectors can efficiently monitor and manage water levels, flow rates, and quality in real-time.This system ensures efficient and sustainable water management while minimizing manual effort and resource wastage.

#include <Wire.h>
#include <LiquidCrystal_I2C.h>

#define WATER_SENSOR A0
#define FLOW_SENSOR 2
#define PUMP 3
#define BUZZER 4

LiquidCrystal_I2C lcd(0x27, 16, 2);

volatile int pulseCount = 0;
float flowRate = 0.0;
unsigned long oldTime = 0;

void flowISR() {
    pulseCount++;
}

void setup() {
    pinMode(WATER_SENSOR, INPUT);
    pinMode(FLOW_SENSOR, INPUT);
    pinMode(PUMP, OUTPUT);
    pinMode(BUZZER, OUTPUT);

    attachInterrupt(digitalPinToInterrupt(FLOW_SENSOR), flowISR, RISING);

    lcd.begin();
    lcd.backlight();
    lcd.setCursor(0, 0);
    lcd.print("Water System");
    delay(2000);
    lcd.clear();
}

void loop() {
    int waterLevel = analogRead(WATER_SENSOR);
    float voltage = waterLevel * (5.0 / 1023.0);
    
    lcd.setCursor(0, 0);
    lcd.print("Water Level: ");
    lcd.print(voltage, 2);
    lcd.print("V");

    if (millis() - oldTime > 1000) {
        oldTime = millis();
        flowRate = (pulseCount / 7.5); // Flow rate in LPM
        pulseCount = 0;

        lcd.setCursor(0, 1);
        lcd.print("Flow: ");
        lcd.print(flowRate);
        lcd.print(" LPM");
    }

    if (voltage < 1.0) {
        digitalWrite(PUMP, HIGH);
        digitalWrite(BUZZER, HIGH);
        lcd.setCursor(0, 1);
        lcd.print("Pump: ON ");
    } else if (voltage > 4.0) {
        digitalWrite(PUMP, LOW);
        digitalWrite(BUZZER, LOW);
        lcd.setCursor(0, 1);
        lcd.print("Pump: OFF");
    }

    delay(1000);
}
