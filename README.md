# Smart-lock-with-OTP-authentication
A smart lock system using Arduino that generates a 4-character OTP and sends it via SMS using the SIM800A GSM module. The user enters the OTP on a 4x4 keypad. If correct, a servo motor unlocks the lock. An I2C LCD displays system messages. This project enhances security with mobile-based authentication.


# 🔐 Smart Lock with OTP Authentication (Arduino Based)

This project is a smart locking system that uses **OTP (One-Time Password)** authentication via **SIM800A GSM module**, **4x4 Keypad**, **LCD Display**, and **Servo Motor**. It sends an OTP to a registered mobile number, and access is granted only if the correct OTP is entered through the keypad.

## 📸 Circuit Diagram

![Circuit Diagram](https://github.com/obulsai/Smart-lock-with-OTP-athuntication/blob/main/circuit_image%20(1).png)

## ⚙️ Hardware Used

- Arduino Uno
- SIM800A GSM Module
- 4x4 Keypad
- 16x2 I2C LCD Display
- Servo Motor
- Jumper wires
- Power supply

## 🧠 Working

1. When the Arduino starts, it generates a **random 4-character OTP** using characters A-D and digits 0-9.
2. The OTP is sent to a pre-registered phone number using the **SIM800A module**.
3. The user is prompted on the LCD to enter the OTP using the keypad.
4. If the entered password matches the OTP:
   - **Access is granted**, and the servo motor rotates (simulating unlocking).
5. If incorrect:
   - **Access is denied**, and the LCD displays an error message.

## 🔢 OTP Format

OTP includes characters from:
Example: `B9C3`

## 💻 Code

```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <SoftwareSerial.h>
#include <Keypad.h>
#include <Servo.h>

// LCD setup
const int lcdColumns = 16;
const int lcdRows = 2;
const int lcdAddress = 0x27;
LiquidCrystal_I2C lcd(lcdAddress, lcdColumns, lcdRows);

// Keypad setup
SoftwareSerial sim800(10, 11); // TX, RX
Servo myServo;
String password;
String inputPassword = "";

const byte ROWS = 4;
const byte COLS = 4;
char keys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};
byte rowPins[ROWS] = {9, 8, 7, 6};
byte colPins[COLS] = {5, 4, 3, 2};
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

void setup() {
  Serial.begin(9600);
  sim800.begin(9600);
  myServo.attach(12);
  myServo.write(0);

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Enter Password:");
  lcd.setCursor(0, 1);
  lcd.print("Input: ");

  Serial.println("Testing SIM800A Module");
  delay(3000);

  sim800.println("AT+CMGF=1");
  delay(1000);

  generatePassword();
  sendOTP("9550675001", "Your password is " + password);
}

void loop() {
  char key = keypad.getKey();

  if (key) {
    if (key == '#') {
      if (inputPassword == password) {
        Serial.println("Correct Password!");
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Correct Password!");
        rotateServo();
        delay(2000);
        lcd.clear();
      } else {
        Serial.println("Incorrect Password.");
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print("Access Denied");
        delay(2000);
        lcd.clear();
      }
      inputPassword = "";
      lcd.setCursor(0, 0);
      lcd.print("Enter Password:");
      lcd.setCursor(0, 1);
      lcd.print("Input: ");
    } else if (key == '*') {
      inputPassword = "";
      lcd.setCursor(7, 1);
      lcd.print("        ");
      Serial.println("Password Cleared");
    } else {
      inputPassword += key;
      lcd.setCursor(7, 1);
      lcd.print(inputPassword);
      Serial.println("Current Input: " + inputPassword);
    }
  }
}

void generatePassword() {
  randomSeed(analogRead(0));
  char characters[] = "ABCD0123456789";

  password = "";
  for (int i = 0; i < 4; i++) {
    int randomIndex = random(0, sizeof(characters) - 1);
    password += characters[randomIndex];
  }

  Serial.println("Generated Password: " + password);
}

void sendOTP(const char* phoneNumber, String message) {
  sim800.print("AT+CMGS=\"");
  sim800.print(phoneNumber);
  sim800.println("\"");
  delay(1000);

  sim800.print(message);
  delay(500);

  sim800.write(26);
  delay(5000);

  while (sim800.available()) {
    Serial.write(sim800.read());
  }
}

void rotateServo() {
  Serial.println("Rotating Servo Motor...");
  myServo.write(90);
  delay(2000);
  myServo.write(0);
  delay(1000);
}

