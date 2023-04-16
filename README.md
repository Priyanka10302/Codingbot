#include <LiquidCrystal_I2C.h>
#include <Wire.h> 

LiquidCrystal_I2C lcd(0x27, 16, 2);

#define trig 6
#define echo 7
#define relay 5
#define buzzer A0



#include<Servo.h>
#include <SPI.h>
#include <MFRC522.h>

#define RST_PIN         9          // Configurable, see typical pin layout above
#define SS_PIN          10         // Configurable, see typical pin layout above

MFRC522 mfrc522(SS_PIN, RST_PIN);  // Create MFRC522 instance
byte accessUID[4] = {0x06, 0x55, 0xCF, 0x1B};
Servo Myservo1;
Servo Myservo2;
int pos;
void setup() {
  pinMode(A0,OUTPUT);
  pinMode(relay,OUTPUT);
  pinMode(trig,OUTPUT);
  pinMode(echo,INPUT);
  Myservo1.attach(4);
  Myservo2.attach(8);
  lcd.begin(16,2);
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0,0);
  lcd.print(" Welcome to ");
  lcd.setCursor(0,1);
  lcd.print("  Army Cantt.");
  delay(3000);
  lcd.clear();

  Myservo1.write(0);
  Myservo2.write(0);
  digitalWrite(relay,0);
	Serial.begin(9600);		// Initialize serial communications with the PC
	while (!Serial);		// Do nothing if no serial port is opened (added for Arduinos based on ATMEGA32U4)
	SPI.begin();			// Init SPI bus
	mfrc522.PCD_Init();		// Init MFRC522
	delay(4);				// Optional delay. Some board do need more time after init to be ready, see Readme
	mfrc522.PCD_DumpVersionToSerial();	// Show details of PCD - MFRC522 Card Reader details
	Serial.println(F("Scan PICC to see UID, SAK, type, and data blocks..."));
}

void ultrasonic()
{
  digitalWrite(trig,0);
  delayMicroseconds(2);
  digitalWrite(trig,1);
  delayMicroseconds(10);
  digitalWrite(trig,0);

  float time = pulseIn(echo,1);
  float distance = time * 0.034/2;
  Serial.print("Distance = ");
  Serial.println(distance);
  delay(200);

if(distance<8 && distance>1)
{
Myservo2.write(90);
delay(2000);
Myservo2.write(0);
}
else
{
  //digitalWrite(13,0);
  Myservo2.write(0);
}
}

void loop() 
{
   ultrasonic();
 
	// Reset the loop if no new card present on the sensor/reader. This saves the entire process when idle.
	if ( ! mfrc522.PICC_IsNewCardPresent()) {
		return;
	}

	// Select one of the cards
	if ( ! mfrc522.PICC_ReadCardSerial()) {
		return;
	}

	// Dump debug info about the card; PICC_HaltA() is automatically called
if(mfrc522.uid.uidByte[0] ==accessUID[0]&& mfrc522.uid.uidByte[1] ==accessUID[1]&&mfrc522.uid.uidByte[2] ==accessUID[2]&&mfrc522.uid.uidByte[3] ==accessUID[3])
{
digitalWrite(A0,0);
digitalWrite(relay,0);
lcd.setCursor(0,0);
lcd.print("Enter Now");
lcd.setCursor(0,1);
lcd.print("Door Open");
Serial.println("Access Granted");
Serial.println("Door Open");
Myservo1.write(90);
delay(2000);
Myservo1.write(0);

}
else
{
 digitalWrite(relay,1);
 Serial.println("Access Denied");  
 lcd.setCursor(0,0);
lcd.print("Not Allowed");
lcd.setCursor(0,1);
lcd.print("Invalid Card");
digitalWrite(A0,1);
delay(1000);
digitalWrite(A0,0);
delay(1000);
digitalWrite(A0,1);
delay(1000);
digitalWrite(A0,0);
delay(1000);

 
}

mfrc522.PICC_HaltA(); 

 
 }
