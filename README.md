/*
 * Bluetooth Controlled Robot
 * RX = 0, TX = 1 (Hardware Serial)
 * Servos = 8,9,10,11
 * Motor Driver = A0–A5
 */

#include <Servo.h>

// ================== SERVO OBJECTS ==================
Servo servo1;  // Base (continuous)
Servo servo2;  // Shoulder
Servo servo3;  // Elbow
Servo servo4;  // Gripper (continuous)

// ================== SERVO PINS ==================
#define SERVO1_PIN 8
#define SERVO2_PIN 9
#define SERVO3_PIN 10
#define SERVO4_PIN 11

// ================== SERVO POSITIONS ==================
int S1curntPos = 90;
int S2curntPos = 90;
int S3curntPos = 90;
int S4curntPos = 90;

// ================== MOTOR DRIVER (A0–A5) ==================
#define ENABLE_A A0
#define MOTOR_A_IN1 A1
#define MOTOR_A_IN2 A2
#define MOTOR_B_IN3 A3
#define MOTOR_B_IN4 A4
#define ENABLE_B A5

int motorSpeed = 200;

// ================== SETUP ==================
void setup() {
  Serial.begin(9600);  // Bluetooth + USB Serial

  servo1.attach(SERVO1_PIN);
  servo2.attach(SERVO2_PIN);
  servo3.attach(SERVO3_PIN);
  servo4.attach(SERVO4_PIN);

  servo1.write(S1curntPos);
  servo2.write(S2curntPos);
  servo3.write(S3curntPos);
  servo4.write(S4curntPos);

  pinMode(ENABLE_A, OUTPUT);
  pinMode(MOTOR_A_IN1, OUTPUT);
  pinMode(MOTOR_A_IN2, OUTPUT);
  pinMode(ENABLE_B, OUTPUT);
  pinMode(MOTOR_B_IN3, OUTPUT);
  pinMode(MOTOR_B_IN4, OUTPUT);

  stopMotors();

  Serial.println("Robot Ready!");
}

// ================== LOOP ==================
void loop() {
  if (Serial.available()) {
    String command = Serial.readString();
    command.trim();

    if (command.startsWith("S1:")) controlBaseServo(command);
    else if (command.startsWith("S2:")) controlServo(servo2, command, S2curntPos);
    else if (command.startsWith("S3:")) controlServo(servo3, command, S3curntPos);
    else if (command.startsWith("S4:")) controlGripperServo(command);

    else if (command == "F") moveForward();
    else if (command == "B") moveBackward();
    else if (command == "L") turnLeft();
    else if (command == "R") turnRight();
    else if (command == "S") stopMotors();
  }
}

// ================== SERVO CONTROLS ==================
void controlBaseServo(String command) {
  String action = command.substring(command.indexOf(':') + 1);
  action.toUpperCase();

  if (action == "CW") servo1.write(180);
  else if (action == "CCW") servo1.write(0);
  else servo1.write(90);
}

void controlGripperServo(String command) {
  String action = command.substring(command.indexOf(':') + 1);
  action.toUpperCase();

  if (action == "CLOSE") servo4.write(180);
  else if (action == "OPEN") servo4.write(0);
  else servo4.write(90);
}

void controlServo(Servo &myServo, String command, int &currentPos) {
  int target = command.substring(command.indexOf(':') + 1).toInt();
  target = constrain(target, 0, 180);

  if (target > currentPos) {
    for (int i = currentPos; i <= target; i++) {
      myServo.write(i);
      delay(15);
    }
  } else {
    for (int i = currentPos; i >= target; i--) {
      myServo.write(i);
      delay(15);
    }
  }
  currentPos = target;
}

// ================== MOTOR CONTROLS ==================
void moveForward() {
  digitalWrite(MOTOR_A_IN1, HIGH);
  digitalWrite(MOTOR_A_IN2, LOW);
  digitalWrite(MOTOR_B_IN3, HIGH);
  digitalWrite(MOTOR_B_IN4, LOW);
  analogWrite(ENABLE_A, motorSpeed);
  analogWrite(ENABLE_B, motorSpeed);
}

void moveBackward() {
  digitalWrite(MOTOR_A_IN1, LOW);
  digitalWrite(MOTOR_A_IN2, HIGH);
  digitalWrite(MOTOR_B_IN3, LOW);
  digitalWrite(MOTOR_B_IN4, HIGH);
  analogWrite(ENABLE_A, motorSpeed);
  analogWrite(ENABLE_B, motorSpeed);
}

void turnLeft() {
  digitalWrite(MOTOR_A_IN1, LOW);
  digitalWrite(MOTOR_A_IN2, HIGH);
  digitalWrite(MOTOR_B_IN3, HIGH);
  digitalWrite(MOTOR_B_IN4, LOW);
  analogWrite(ENABLE_A, motorSpeed);
  analogWrite(ENABLE_B, motorSpeed);
}

void turnRight() {
  digitalWrite(MOTOR_A_IN1, HIGH);
  digitalWrite(MOTOR_A_IN2, LOW);
  digitalWrite(MOTOR_B_IN3, LOW);
  digitalWrite(MOTOR_B_IN4, HIGH);
  analogWrite(ENABLE_A, motorSpeed);
  analogWrite(ENABLE_B, motorSpeed);
}

void stopMotors() {
  digitalWrite(MOTOR_A_IN1, LOW);
  digitalWrite(MOTOR_A_IN2, LOW);
  digitalWrite(MOTOR_B_IN3, LOW);
  digitalWrite(MOTOR_B_IN4, LOW);
  analogWrite(ENABLE_A, 0);
  analogWrite(ENABLE_B, 0);
}
