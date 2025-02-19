# bot_face_expression
 A table bot that roams around, reacts to objects, and changes facial expressions dynamically.


#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Servo.h>

// Motor Pins
#define MOTOR1_IN1 3
#define MOTOR1_IN2 4
#define MOTOR2_IN1 5
#define MOTOR2_IN2 6

// Servo Motor Pin
#define SERVO_PIN 9

// Ultrasonic Sensor Pins
#define TRIG_PIN 12
#define ECHO_PIN 13

// Create Servo and Display Objects
Servo servoMotor;
Adafruit_SSD1306 display(128, 64, &Wire, -1);

// Display dimensions
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64

// Movement Variables
long duration;
int distance;
bool obstacleDetected = false; // Flag for obstacle detection

// Eye and mouth properties
int eyeSize = 16;  
int leftEyeX = 30;
int rightEyeX = 82;
int eyeY = 20;
int mouthY = 45;
int mouthWidth = 40;
int mouthHeight = 8;

void setup() {
  Serial.begin(9600);
  servoMotor.attach(SERVO_PIN);

  pinMode(MOTOR1_IN1, OUTPUT);
  pinMode(MOTOR1_IN2, OUTPUT);
  pinMode(MOTOR2_IN1, OUTPUT);
  pinMode(MOTOR2_IN2, OUTPUT);

  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    while (true);
  }
  display.clearDisplay();
  display.display();

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
}

void loop() {
  display.clearDisplay();
  
  distance = getUltrasonicDistance();
  
  if (distance < 10) {
    obstacleDetected = true;
    drawEyes(2);   // Crying face
    drawMouth(2);  // Sad mouth
    moveRobotRandomly();
  } else {
    obstacleDetected = false;
    drawEyes(1);   // Happy face
    drawMouth(1);  // Smiling mouth
    moveRobotRandomly();
  }

  display.display();
  moveServoRandomly();
  delay(100);
}

void moveRobotRandomly() {
  int direction = random(0, 4);
  switch (direction) {
    case 0: moveForward(); break;
    case 1: moveBackward(); break;
    case 2: turnLeft(); break;
    case 3: turnRight(); break;
  }
  delay(1000);
}

void moveForward() {
  digitalWrite(MOTOR1_IN1, HIGH);
  digitalWrite(MOTOR1_IN2, LOW);
  digitalWrite(MOTOR2_IN1, HIGH);
  digitalWrite(MOTOR2_IN2, LOW);
}

void moveBackward() {
  digitalWrite(MOTOR1_IN1, LOW);
  digitalWrite(MOTOR1_IN2, HIGH);
  digitalWrite(MOTOR2_IN1, LOW);
  digitalWrite(MOTOR2_IN2, HIGH);
}

void turnLeft() {
  digitalWrite(MOTOR1_IN1, LOW);
  digitalWrite(MOTOR1_IN2, HIGH);
  digitalWrite(MOTOR2_IN1, HIGH);
  digitalWrite(MOTOR2_IN2, LOW);
}

void turnRight() {
  digitalWrite(MOTOR1_IN1, HIGH);
  digitalWrite(MOTOR1_IN2, LOW);
  digitalWrite(MOTOR2_IN1, LOW);
  digitalWrite(MOTOR2_IN2, HIGH);
}

void moveServoRandomly() {
  int angle = random(0, 180);
  servoMotor.write(angle);
  delay(500);
}

long getUltrasonicDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  
  duration = pulseIn(ECHO_PIN, HIGH);
  return duration * 0.034 / 2;
}

void drawEyes(int exp) {
  switch (exp) {
    case 1: display.fillRect(leftEyeX, eyeY, eyeSize, eyeSize, SSD1306_WHITE);
            display.fillRect(rightEyeX, eyeY, eyeSize, eyeSize, SSD1306_WHITE);
            break;
    case 2: display.drawCircle(leftEyeX + eyeSize / 2, eyeY + eyeSize / 2, eyeSize / 2, SSD1306_WHITE);
            display.drawCircle(rightEyeX + eyeSize / 2, eyeY + eyeSize / 2, eyeSize / 2, SSD1306_WHITE);
            break;
  }
}

void drawMouth(int exp) {
  int mouthCenterX = (SCREEN_WIDTH - mouthWidth) / 2;
  switch (exp) {
    case 1: display.drawLine(mouthCenterX, mouthY, mouthCenterX + mouthWidth, mouthY, SSD1306_WHITE);
            break;
    case 2: display.drawLine(mouthCenterX, mouthY, mouthCenterX + mouthWidth, mouthY, SSD1306_WHITE);
            display.drawLine(mouthCenterX, mouthY, mouthCenterX + mouthWidth / 2, mouthY + 10, SSD1306_WHITE);
            display.drawLine(mouthCenterX + mouthWidth, mouthY, mouthCenterX + mouthWidth / 2, mouthY + 10, SSD1306_WHITE);
            break;
  }
}