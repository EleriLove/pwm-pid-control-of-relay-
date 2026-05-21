#include <PID_v1.h>

const int relay = D6;
int error = 0;
int userinput;
const double sensorPin = A1;
const double samples = 45;
const double A = 0.5529545762e-3;
const double B = 2.151848957e-4;
const double C = 2.120919370e-7;
const float Vin = 3.3;
const float pullup = 100000.0;
float Vout;
float R;
float sensorValue;
float prevRead;
float sensorFinal;
float smoothing = .2;
double temp;
double setpoint = 0;
double logR;
double output;
double Kp=61.2, Ki=1.3, Kd=3;
char serialread[24];
char command[10];
float P, I, D;
String input;


PID pidTemp(&temp, &output, &setpoint, Kp, Ki, Kd, DIRECT);

int timeScale = 2000;
unsigned long timeStart;

void setup() {
  Serial.begin(115200);

  Serial.println();

  pinMode(relay, OUTPUT);

  timeStart = millis();

  pidTemp.SetOutputLimits(0, timeScale);

  pidTemp.SetMode(AUTOMATIC);

}

void panic () {
  while (true) {
    if (digitalRead(relay) == 1|| setpoint > 0 ) {
      digitalWrite(relay, 0);
      setpoint = 0;
      Serial.println("PANIC");
    Serial.println(temp);
    }
    delay(500);
    Serial.println(temp);
  }
 
}

void loop() {
  
  // averaging samples to reduce random noise
  int total = 0;
  input = Serial.readString();
  input.toCharArray(serialread, 24);

  if ( input[1] == 'e') {
    sscanf(serialread, "%5s %d", command, &userinput);
    if (strcmp(command, "temp=") == 0 && userinput <=250) {
      setpoint = userinput;
      Serial.println();
      Serial.print("temperature changed: ");
      Serial.print(setpoint);
      Serial.println();
    }
  }

  if ( input[1] == 'u') {
    sscanf(serialread, "%5s %f %f %f", command, &P, &I, &D);
    if (strcmp(command, "tune=") == 0) {
      
      Kp = P;
      Ki = I;
      Kd = D;
      Serial.println();
      Serial.print("PID changed: ");
      Serial.print(Kp);
      Serial.print(" ");
      Serial.print(Ki);
      Serial.print(" ");
      Serial.println(Kd);
      pidTemp.SetTunings(Kp, Ki, Kd);
    }
  }
  

  prevRead = sensorValue;

  for(int i =0; i < samples; i++) {

    int n = analogReadMilliVolts(sensorPin);

    if (error >= 3) {
    panic();
  }
    if (n > 8 && n < 1865) {
      error = 0;
      total += n;
    }
      else {
        error++;
        Serial.println("sensor fault");
        i--;
    }
    delay(10);
  } 
  sensorValue = total / samples;
  sensorFinal = smoothing * sensorValue + (1-smoothing) * prevRead;


  // don't try understanding this math, just know it takes voltage and turns it into celcius, this takes way too much math.... 
  Vout = sensorFinal / 1000.0;
  R = pullup * 1 / ((Vin/Vout)-1);
  logR= log(R);
  temp = 1.0 / (A + B * logR + (C * logR * logR * logR));
  temp = temp - 273.15;
  Serial.println(temp);
  
//don't thermal runaway, don't thermal runaway, don't thermal runaway..... oh gods it's going to thermal runaway... 
  pidTemp.Compute();

  if(timeScale < millis() - timeStart){
    timeStart += timeScale;
  }
  if (output < millis() - timeStart) {
    digitalWrite(relay, 0);
  }
    else digitalWrite(relay, 1);

}
