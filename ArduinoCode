#include <Keypad.h>
#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>

// ------------------- KEYPAD SETUP -------------------
const byte ROWS = 4;
const byte COLS = 4;

// Wiring: 8,7,6,5,4,A0,A1,A2
byte rowPins[ROWS] = {8, 7, 6, 5};
byte colPins[COLS] = {4, A0, A1, A2};

char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);

// ------------------- RFID SETUP -------------------
#define SS_PIN 10
#define RST_PIN 9
MFRC522 rfid(SS_PIN, RST_PIN);

// ------------------- SERVO SETUP -------------------
Servo myServo;
int servoPin = 3; 

// ------------------- VARIABLES -------------------
String correctCode = "1234";  // Predefined correct code
String enteredCode = "";      // To store the entered code
bool isChangingCode = false;  // Flag to check if the user is changing the code

void setup() {
  Serial.begin(9600);
  myServo.attach(servoPin);   // Attach the servo to pin 3

  // Keypad setup
  Serial.println("Keypad + RFID ready...");

  // RFID setup
  SPI.begin();
  rfid.PCD_Init();
}

void loop() {
  // ----------- KEYPAD HANDLER -----------
  char key = keypad.getKey();
  if (key) {
    handleKeypadInput(key);
  }

  // ----------- RFID HANDLER -----------
  if ( ! rfid.PICC_IsNewCardPresent()) return;
  if ( ! rfid.PICC_ReadCardSerial())   return;

  Serial.print("RFID UID: ");
  String cardUID = "";
  for (byte i = 0; i < rfid.uid.size; i++) {
    cardUID += String(rfid.uid.uidByte[i], HEX);  // Concatenate UID without spaces
  }

  // Convert to uppercase to ensure the comparison is case-insensitive
  cardUID.toUpperCase();
  Serial.println(cardUID);  // Print the UID without spaces

  // Check for specific RFID card UID (make sure this matches the actual UID)
  if (cardUID == "3A2BAE16") {  // Match against the card UID
    Serial.println("RFID Card recognized! Servo will move.");
    moveServo();  // Move the servo for recognized card
  } else {
    Serial.println("Unknown card.");
  }

  rfid.PICC_HaltA();
}

// Handle keypad input
void handleKeypadInput(char key) {
  Serial.print("Key pressed: ");
  Serial.println(key);

  // If the user is changing the code, process new code entry
  if (isChangingCode) {
    if (key >= '0' && key <= '9') {
      enteredCode += key;  // Add the digit to the entered code

      // If 4 digits have been entered, allow user to press '*' to set the new code
      if (enteredCode.length() == 4) {
        Serial.print("New Code Entered: ");
        Serial.println(enteredCode);
      }
    }

    // If user presses '*', set the new code
    if (key == '*') {
      correctCode = enteredCode;  // Set the new correct code
      Serial.print("New code saved: ");
      Serial.println(correctCode);
      isChangingCode = false;  // Exit code-changing mode
      enteredCode = "";  // Clear the entered code
      Serial.println("Code change complete. Enter the new code to test.");
    }
  } else {
    // If the user is not changing the code, handle the normal code entry
    if (key >= '0' && key <= '9') {
      enteredCode += key;  // Add the digit to the entered code
    }

    // If the user presses '#', enter the change code mode
    if (key == '#') {
      if (enteredCode == correctCode) {
        Serial.println("Old code accepted. Please enter a new 4-digit code.");
        isChangingCode = true;  // Enter change code mode
        enteredCode = "";  // Clear the entered code for the new code input
      } else {
        Serial.println("Incorrect old code. Try again.");
        enteredCode = "";  // Reset entered code if it's incorrect
      }
    }

    // If the user presses '*', validate the entered code
    if (key == '*') {
      Serial.print("Entered Code: ");
      Serial.println(enteredCode);

      // Check if the entered code matches the correct code
      if (enteredCode == correctCode) {
        Serial.println("Correct code!");
        moveServo();  // Move the servo
      } else {
        Serial.println("Incorrect code.");
      }

      // Clear entered code after checking
      enteredCode = "";
    }

    // If the user presses 'D', clear the entered code
    if (key == 'D') {
      Serial.println("Input cleared.");
      enteredCode = "";  // Clear the entered code
    }
  }
}

// Function to move the servo (40 degrees for 100 ms, then 70 degrees)
void moveServo() {
  // Move to 40 degrees
  myServo.write(40);
  delay(100); // Wait for 100 ms
  
  // Move back to 70 degrees
  myServo.write(70);
  delay(1000); // Wait for the servo to reach the position
}
