***********************************************************
*/Gesture-Controlled-robotic-car-for-blind


easymathsforyou@gmail.com


https://www.youtube.com/channel/UCY5O7tihQh7VwO4iWww3iCg 


*/


***********************************************************
Transmitter code
************************************************************

#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>
#include "Wire.h"       
#include "I2Cdev.h"     
#include "MPU6050.h"
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <NewPing.h>
#include "printf.h"
#define screen_width 128
#define screen_height 64
Adafruit_SSD1306 display(screen_width, screen_height);
#define TRIGGER_PIN A5
#define ECHO_PIN A4
int vibPin=5;  
//Define variables for Gyroscope and Accelerometer data
MPU6050 mpu;
int16_t ax, ay, az;
int16_t gx, gy, gz;

RF24 radio(9, 10); // CE, CSN
const byte addresses[][6] = {"00001", "00002"};
//boolean buttonState = 0;
struct MyData {
  byte X;
  byte Y;
   long duration;
  int Distance;
};

MyData data;

void setup() {
  pinMode(5, OUTPUT);
  Serial.begin(9600);
  Wire.begin();
  display.begin(SSD1306_SWITCHCAPVCC, 0x3c);
 //current_distance = sonar1.ping_cm();
 //pinMode(buzzPin,OUTPUT);
  mpu.initialize();
  radio.begin();
  radio.openWritingPipe(addresses[1]); // 00002
  radio.openReadingPipe(1, addresses[0]); // 00001
  radio.setPALevel(RF24_PA_MIN);
}
void loop() {
  delay(5);
  radio.stopListening();
  //int potValue = analogRead(A0);
  //int angleValue = map(potValue, 0, 1023, 0, 180);
  mpu.getMotion6(&ax, &ay, &az, &gx, &gy, &gz);

  data.X = map(ax, -17000, 17000, 0, 255 ); //Send X axis data
  data.Y = map(ay, -17000, 17000, 0, 255);  //Send Y axis data
Serial.println("X: ");
Serial.println(data.X);
Serial.println("Y: ");
Serial.println(data.Y);

  radio.write(&data, sizeof(MyData));
  if(data.X > 155){     
  display.clearDisplay();
  display.setTextSize(3);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0); 
  display.println( " ");
  display.println( " Right ");
  display.setCursor(100,100);
  //display.print(millis());
  display.display();
    }
  if (data.Y < 90) {
  display.clearDisplay();
  display.setTextSize(3);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
     display.println( " ");
  display.println( "Forward");
  display.setCursor(100,100);
  //display.print(millis());
  display.display();
}
  if (data.Y > 170){
  display.clearDisplay();
  display.setTextSize(3);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
   display.println( " ");
  display.println( "Reverse ");
  display.setCursor(100,100);
  //display.print(millis());
  display.display();
 }
    if(data.X < 80){
     display.clearDisplay();
  display.setTextSize(3);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  display.println( " ");
  display.println( " Left ");
  display.setCursor(100,100);
  //display.print(millis());
  display.display();
   }
   if(data.X > 100 && data.X < 170 && data.Y > 100 && data.Y < 140){
       display.clearDisplay();
  display.setTextSize(3);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0,0);
  display.println( " ");
  display.println( "  STOP ");
  display.setCursor(100,100);
  //display.print(millis());
  display.display();
    }
  delay(5);
 radio.startListening();
 while(!radio.available());
 radio.read(&data, sizeof(MyData)); 
  
 if (data.Y < 80 && data.Distance > 30) {
    digitalWrite(vibPin, LOW);
    delay(10);
  }
 else if(data.Y < 80 && data.Distance < 30){
      digitalWrite(vibPin, HIGH);
      delay(10);
      Serial.println("low");
    }
else{
  digitalWrite(vibPin, LOW);
}
}

***************************************************************
Receiver Side
***************************************************************

#include <SPI.h>
#include <nRF24L01.h>
#include <RF24.h>
#include <Servo.h>
#include <NewPing.h>
#include "Wire.h" 
//#define button 4
const int IN4 = A0;    
const int IN3 = A1;    
const int IN2 = A2;     
const int IN1 = A3; 
//const int enabA = 5;
//const int enabB = 6;
int buzPin=8;
#define TRIGGER_PIN A5
#define ECHO_PIN A4
int MAX_DISTANCE=50;
NewPing sonar1(2, 3, 50);
NewPing sonar2(6,5, 50);
//NewPing sonar3(A7,A6, 30);
//int speed = 100;

RF24 radio(9, 10); // CE, CSN

const byte addresses[][6] = {"00001", "00002"};
//Servo myServo;
///boolean buttonState = 0;
 
struct MyData {
  byte X; 
  byte Y;
  long duration;
int Distance;
};
MyData data;

void setup() {
  //pinMode(button, INPUT);
   //pinMode(vibPin, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);
//  pinMode(enabA, OUTPUT);
  //pinMode(enabB, OUTPUT);
  // analogWrite(enabA, speed);
//analogWrite(enabB, speed);
pinMode(TRIGGER_PIN, OUTPUT);
pinMode(ECHO_PIN, INPUT);
pinMode(buzPin,OUTPUT);
Serial.begin(9600);
  radio.begin();
  radio.openWritingPipe(addresses[0]); // 00001
  radio.openReadingPipe(1, addresses[1]); // 00002
  radio.setPALevel(RF24_PA_MIN);
}
  void Left(){
 digitalWrite(IN1, LOW);
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
      Serial.println("left");
       delay(100);
  }
  void Right(){
     digitalWrite(IN1, HIGH);
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, HIGH);
//      analogWrite(enabA, 80);
//analogWrite(enabB, 80);
      Serial.println("Right");
      delay(100);
  }
  void Reverse(){
    digitalWrite(IN1, LOW);
      digitalWrite(IN2, HIGH);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, HIGH);
      Serial.println("Forward");
       delay(100);
  }
  void Forward(){
    digitalWrite(IN1, HIGH);
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, HIGH);
      digitalWrite(IN4, LOW);
      Serial.println("Reverse");
       delay(100);
  }
  void Stop(){
    digitalWrite(IN1, LOW);
      digitalWrite(IN2, LOW);
      digitalWrite(IN3, LOW);
      digitalWrite(IN4, LOW);
      //Serial.println("Stop");
       delay(100);
  }
  void buz()
{
 digitalWrite(buzPin,HIGH);
 delay(100);
 digitalWrite(buzPin,LOW);
}
void buzzer()
{
 digitalWrite(buzPin,HIGH);
 delay(10);
 digitalWrite(buzPin,LOW);
}

  int Middle_Distance_test(){
  digitalWrite(TRIGGER_PIN,LOW);
  delayMicroseconds(2);
  digitalWrite(TRIGGER_PIN,HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIGGER_PIN,LOW);
 data.duration=pulseIn(ECHO_PIN,HIGH);
 data.Distance=data.duration/58.2;
  return data.Distance;
  }
void loop() {
  delay(5);
 radio.startListening();
 if (radio.available()) {
 while (radio.available()) {
      //int angleV = 0;
 radio.read(&data, sizeof(MyData));
  
 if (data.X > 200){//Left
      Right();
      Serial.println("Right");
    }
    if (data.Y > 200 && sonar2.ping_cm()==0){//Left
      Reverse();
       Serial.println("Reverse");
     }
      if (data.Y > 190 && sonar2.ping_cm()!=0){//Left
      Stop();
      buz();
       Serial.println("Reverse");
       }
    if (data.X < 70 && sonar1.ping_cm()==0){ //Reverse
      Left();
       Serial.println("Reverse");
    }
     if (data.X < 70 && sonar1.ping_cm()!=0 ){ //Reverse
      Stop();
      buz();
       Serial.println("Stop buzz");
     }
   if(data.X > 90 && data.X < 190 && data.Y > 80 && data.Y < 150){  //stop 
      Stop();
       Serial.println("Stop");
    }
    }
  delay(5);
 radio.stopListening();
data.Distance= Middle_Distance_test();
if (data.Y < 70 && data.Distance > 50)
  {
      Forward();
        Serial.println(data.Distance);
      //digitalWrite(vibPin,HIGH);
  }
  } 
  else if (data.Y < 70 && data.Distance < 50){ //Forward
            Stop();
             Serial.println("Obstacle");
    }
        radio.write(&data, sizeof(MyData));
}    
