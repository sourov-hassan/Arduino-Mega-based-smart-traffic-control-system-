# Arduino-Mega-based-smart-traffic-control-system-
# This project implements a density-based smart traffic control system using Arduino Mega 2560. IR sensors are used to detect vehicle presence, and the system  dynamically assigns green signal timing based on traffic density. LCD display and potentiometer enhance real-time monitoring and control.

#include <LiquidCrystal.h>

// ===== IR SENSOR PINS =====
#define IR_N A0   // North IR
#define IR_S A1   // South IR
#define IR_E A2   // East IR
#define IR_W A3   // West IR

// ===== TRAFFIC LIGHT PINS =====
// North
#define N_R 22
#define N_Y 23
#define N_G 24

// South
#define S_R 25
#define S_Y 26
#define S_G 27

// East
#define E_R 28
#define E_Y 29
#define E_G 30

// West
#define W_R 31
#define W_Y 32
#define W_G 33

int density[4];

// ===== LCD SETUP =====
// RS, E, D4, D5, D6, D7
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

void setup() {
  // LCD
  lcd.begin(16, 2);
  lcd.clear();

  // IR sensors
  pinMode(IR_N, INPUT);
  pinMode(IR_S, INPUT);
  pinMode(IR_E, INPUT);
  pinMode(IR_W, INPUT);

  // Traffic lights
  int lights[] = {N_R,N_Y,N_G,S_R,S_Y,S_G,E_R,E_Y,E_G,W_R,W_Y,W_G};
  for(int i=0;i<12;i++) pinMode(lights[i], OUTPUT);
}

void loop() {

  // Count cars (LOW = car present)
  density[0] = (digitalRead(IR_N)==LOW);
  density[1] = (digitalRead(IR_S)==LOW);
  density[2] = (digitalRead(IR_E)==LOW);
  density[3] = (digitalRead(IR_W)==LOW);

  // Display densities on LCD
  displayDensity();

  // Find max density road
  int maxRoad = -1;
  int maxValue = 0;
  for(int i=0;i<4;i++){
    if(density[i] > maxValue){
      maxValue = density[i];
      maxRoad = i;
    }
  }

  // If no cars anywhere → all RED
  if(maxRoad == -1){
    allRed();
    delay(2000);
    return;
  }

  // Turn all RED
  allRed();

  // Green timing
  int greenTime = 5000 + (density[maxRoad] * 5000);

  // GREEN ON
  greenOn(maxRoad);
  delay(greenTime);

  // YELLOW ON
  yellowOn(maxRoad);
  delay(2000);
}

// ---------------- FUNCTIONS ----------------

void allRed(){
  digitalWrite(N_R,HIGH); digitalWrite(S_R,HIGH);
  digitalWrite(E_R,HIGH); digitalWrite(W_R,HIGH);

  digitalWrite(N_Y,LOW); digitalWrite(S_Y,LOW);
  digitalWrite(E_Y,LOW); digitalWrite(W_Y,LOW);

  digitalWrite(N_G,LOW); digitalWrite(S_G,LOW);
  digitalWrite(E_G,LOW); digitalWrite(W_G,LOW);
}

void greenOn(int r){
  if(r==0) digitalWrite(N_G,HIGH);
  if(r==1) digitalWrite(S_G,HIGH);
  if(r==2) digitalWrite(E_G,HIGH);
  if(r==3) digitalWrite(W_G,HIGH);

  if(r==0) digitalWrite(N_R,LOW);
  if(r==1) digitalWrite(S_R,LOW);
  if(r==2) digitalWrite(E_R,LOW);
  if(r==3) digitalWrite(W_R,LOW);
}

void yellowOn(int r){
  if(r==0) digitalWrite(N_Y,HIGH);
  if(r==1) digitalWrite(S_Y,HIGH);
  if(r==2) digitalWrite(E_Y,HIGH);
  if(r==3) digitalWrite(W_Y,HIGH);

  if(r==0) digitalWrite(N_G,LOW);
  if(r==1) digitalWrite(S_G,LOW);
  if(r==2) digitalWrite(E_G,LOW);
  if(r==3) digitalWrite(W_G,LOW);
}

// ---------------- LCD FUNCTION ----------------
void displayDensity(){
  lcd.setCursor(0,0);
  lcd.print("R1:");
  lcd.print(density[0] ? "High " : "Low  ");

  lcd.setCursor(8,0);
  lcd.print("R2:");
  lcd.print(density[1] ? "High " : "Low  ");

  lcd.setCursor(0,1);
  lcd.print("R3:");
  lcd.print(density[2] ? "High " : "Low  ");

  lcd.setCursor(8,1);
  lcd.print("R4:");
  lcd.print(density[3] ? "High " : "Low  ");
}
