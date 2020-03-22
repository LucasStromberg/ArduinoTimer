# Arduino Timer
A timer with an OLED display and capacitive touch buttons, powered by an Arduino Nano.

At its current state the code is neither commented or structured very well, this might change in the future.

More information: https://lucasstromberg.com. Contact me if you're interested in getting a pcb.

## Schematic and PCB

## 3D-Printed case

## Code
```c
//INCLUDE STUFF:
#include <Adafruit_GFX.h>
#include <Adafruit_SPITFT.h>
#include <Adafruit_SPITFT_Macros.h>
#include <gfxfont.h>
#include <Adafruit_SSD1306.h>
#include <splash.h>
#include <Wire.h>
#include <SPI.h>

//DEFINE STUFF:
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 32

#define OLED_RESET 4

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

//PINS
int buttonPin1 = 6;
int buttonPin2 = 5;
int piezo = A3;
int drawPin = 7;

//VARIABLES:
int currentPage = 1;
int countDown = 0;
unsigned long reference = 0;
unsigned long drawReference = 0;
int delayAmount = 300;
int drawLength = 1000;
int drawDelay = 15000;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(9600);

  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // Address 0x3C for 128x32
    Serial.println(F("SSD1306 allocation failed"));
    for (;;); // Don't proceed, loop forever
  }

  pinMode(buttonPin1, INPUT);
  pinMode(buttonPin2, INPUT);
  pinMode(piezo, OUTPUT);
  pinMode(drawPin, OUTPUT);

  display.clearDisplay();

  animateRect();
  rePrint(0, 0);
  
  reference = millis();
}

void loop() {
  // put your main code here, to run repeatedly:
  unsigned long timeState = millis();
  
  if (countDown > 0 && (timeState - reference > 3000)){
    startTimer(countDown);
  }

  if (digitalRead(buttonPin1)){
    reference = millis();
    if (countDown < 99){
      countDown++;
    }
    if (delayAmount > 50){
      delayAmount = delayAmount - 30;
    }
    delay(delayAmount);
  } else if (digitalRead(buttonPin2)){
    reference = millis();
    if (countDown != 0){
      countDown--;
    }
    delay(delayAmount);
    if (delayAmount > 50){
      delayAmount = delayAmount - 30;
    }
  } else {
    delayAmount = 300;
  }
  
  rePrint(countDown, 0);

  drawCurrent();
}

void drawCurrent(){
  digitalWrite(drawPin, millis() - drawReference < drawLength);
  Serial.println(countDown);
  if (millis() - drawReference > drawDelay){drawReference += drawDelay;}
  
}

void animateRect() {
  for (int i = 0; i <= SCREEN_WIDTH; i++) {
    display.drawPixel(i, 0, WHITE);
    if ((i % 8) == 0) {
      display.display();
    }
  }
  for (int i = 0; i <= SCREEN_HEIGHT; i++) {
    display.drawPixel(SCREEN_WIDTH - 1, i, WHITE);
    if ((i % 8) == 0) {
      display.display();
    }
  }
  for (int i = SCREEN_WIDTH; i >= 0; i--) {
    display.drawPixel(i, SCREEN_HEIGHT - 1, WHITE);
    if ((i % 8) == 0) {
      display.display();
    }
  }
  for (int i = SCREEN_HEIGHT; i >= 0; i--) {
    display.drawPixel(0, i, WHITE);
    if ((i % 8) == 0) {
      display.display();
    }
  }
}

void drawRect(){
  display.drawRect(0,0,SCREEN_WIDTH, SCREEN_HEIGHT,WHITE);
}

void drawBoxes() {
  display.fillRect(0, 0, 14, 18, WHITE);
  display.drawRect(15, 0, 14, 18, WHITE);
  display.drawRect(30, 0, 14, 18, WHITE);
  display.drawRect(45, 0, 14, 18, WHITE);
}

void printBoxText() {
  display.setTextSize(1);
  display.setTextColor(BLACK);

  display.setCursor(5, 5);
  display.println(F("1"));

  display.setTextColor(WHITE);

  display.setCursor(20, 5);
  display.println(F("2"));

  display.setCursor(35, 5);
  display.println(F("3"));

  display.setCursor(50, 5);
  display.println(F("4"));

}

void printTimer(int minutes, int seconds) {
  display.setTextSize(3);
  display.setTextColor(WHITE);

  if (minutes < 10){
    display.setCursor(90, 5);
  } else {
    display.setCursor(73, 5);
  }

  display.println(minutes);

  display.setTextSize(1);
  display.setTextColor(WHITE);
  display.setCursor(110, 5);

  if (seconds < 10) {
    display.println("0" + String(seconds));
  } else {
    display.println(seconds);
  }

}

void rePrint(int minutes, int seconds){
  display.clearDisplay();

  drawRect();
 
  drawBoxes();

  printBoxText();

  printTimer(minutes, seconds);

  display.display();
}

void blinkPrint(int minutes, int seconds){
  for (int i = 0; i <=6; i++){
    display.clearDisplay();
    drawRect();
    drawBoxes();
    printBoxText();
    if (i % 2 == 0){
      printTimer(minutes, seconds);
    }
    display.display();
    delay(50);
  }
}

void startTimer(int minutes) {
  blinkPrint(minutes, 0);
  
  int currentMinute = minutes;
  int currentSecond = 0;

  bool atStart = true;

  unsigned long milliSecond = 1000UL;
  unsigned long milliMinute = milliSecond * 60UL;
  unsigned long milliSeconds = minutes * milliMinute;
  unsigned long startReference = millis();
  unsigned long secondReference = millis();

  bool sound = true;

  while (true) {

    drawCurrent();

    if (digitalRead(buttonPin1) || digitalRead(buttonPin2)){
      rePrint(0,0);
      sound = false;
      countDown = 0;
      blinkPrint(0, 0);
      break;
    }

    secondReference = millis();
    
    if (secondReference - startReference > milliSecond) {
      if (currentSecond == 0) {
        if (currentMinute == 0){
          break;
        }
        currentSecond = 59;
        if (atStart){
          currentMinute--;
          atStart = false;
        }
      } else {
        currentSecond--;
      }
      rePrint(currentMinute, currentSecond);
      milliSecond += 1000UL;
    }
    if (secondReference - startReference > milliMinute && currentSecond == 59) {
      currentMinute--;
      rePrint(currentMinute, currentSecond);
      milliMinute += 1000UL * 60UL;
    }
  }
  if (sound){
    playSound();
  }
}

void playSound(){
  while (!digitalRead(buttonPin1) || !digitalRead(buttonPin2)){

    drawCurrent();

    tone(piezo, 800);
    delay(50);
    noTone(piezo);
    delay(50);
    tone(piezo, 800);
    delay(50);
    noTone(piezo);
    delay(50);
    tone(piezo, 800);
    delay(50);
    noTone(piezo);

    delay(800);
  }
}
```
