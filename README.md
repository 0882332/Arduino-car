# Arduino-car
#include "SoftwareSerial.h"
SoftwareSerial mySerial(12, 13);
# define Start_Byte 0x7E
# define Version_Byte 0xFF
# define Command_Length 0x06
# define End_Byte 0xEF
# define Acknowledge 0x00 //Returns info with command 0x41 [0x01: info, 0x00: no info]
#define echoPin 6 // Echo Pin
#define trigPin 7 // Trigger Pin
# define ACTIVATED LOW


int buttonPause = 3;

boolean isPlaying = false;

int motor_left[] = {10, 11};
int motor_right[] = {8, 9};
int play();
int pause();
int execute_CMD;
int maximumRange = 10; // Maximum range needed
int minimumRange = 0; // Minimum range needed
long duration, distance; // Duration used to calculate distance

void setup () {

pinMode(buttonPause, INPUT);
digitalWrite(buttonPause,HIGH);


mySerial.begin (9600);
delay(1000);
int playFirst();
isPlaying = true;
pinMode(trigPin, OUTPUT);
pinMode(echoPin, INPUT);

// Setup motors
int i;
for(i = 0; i < 2; i++){
pinMode(motor_left[i], OUTPUT);
pinMode(motor_right[i], OUTPUT);
}

}

void loop () { 
digitalWrite(trigPin, LOW); 
 delayMicroseconds(2); 

 digitalWrite(trigPin, HIGH);
 delayMicroseconds(10); 
 
 digitalWrite(trigPin, LOW);
 duration = pulseIn(echoPin, HIGH);
 
 //Calculate the distance (in cm) based on the speed of sound.
 distance = duration/58.2;
 
 if (distance >= maximumRange || distance <= minimumRange){
 /* Send a negative number to computer and Turn LED 
 to indicate "out of range" */
 Serial.println("-1");
digitalWrite(motor_left[0], LOW);
digitalWrite(motor_left[1], HIGH);

digitalWrite(motor_right[0], LOW);
digitalWrite(motor_right[1], HIGH);
 delay(25);
}
else {
  Serial.println(distance);
  digitalWrite(motor_left[0], HIGH);
  digitalWrite(motor_left[1], LOW);
  digitalWrite(motor_right[0], LOW);
  digitalWrite(motor_right[1], LOW);
  delay (2000);
}

delay(50);

if (distance >= maximumRange || distance <= minimumRange){ 

     {isPlaying = true;
      play();}}
else
 {isPlaying = false;
 pause();}
      
 
  }
 
  
  void playFirst()
{
  execute_CMD(0x3F, 0, 0);
  delay(500);
  setVolume(20);
  delay(500);
  execute_CMD(0x11,0,1); 
  delay(500);
}
}

void pause()
{
  execute_CMD(0x0E,0,0);
  delay(500);
}

void play()
{
  execute_CMD(0x0D,0,1); 
  delay(500);
}
void setVolume(int volume)
{
  execute_CMD(0x06, 0, volume); // Set the volume (0x00~0x30)
  delay(2000);
}

void execute_CMD(byte CMD, byte Par1, byte Par2)
// Excecute the command and parameters
{
// Calculate the checksum (2 bytes)
word checksum = -(Version_Byte + Command_Length + CMD + Acknowledge + Par1 + Par2);
// Build the command line
byte Command_line[10] = { Start_Byte, Version_Byte, Command_Length, CMD, Acknowledge,
Par1, Par2, highByte(checksum), lowByte(checksum), End_Byte};
//Send the command line to the module
for (byte k=0; k<10; k++)
{
mySerial.write( Command_line[k]);
}
}
