#include "DHT.h"

#include <Wire.h> 

#define DHTPIN A1    

#define DHTTYPE DHT22   // DHT 22  (AM2302)

#include <DS1307.h>

DS1307 rtc(SDA, SCL); 

#include <LiquidCrystal.h>

LiquidCrystal lcd(12, 11, 5, 4, 3, 2); 

int Hours;

int Minutes;

int buzzer = 6;

int butsetO = 9;

int butsetM = 8;

int butsetU = 10;

int relay1= 13;

int sensorPin = A0;

int sensorValue = 0; 

int percent = 0;

int setHours = 15;

int setMin = 30;

int setUp = 10;

boolean readingbutsetH = HIGH;

boolean readingbutsetM = HIGH;

boolean readingbutsetU = HIGH;

Time ti;

DHT dht(DHTPIN, DHTTYPE);

void setup() {
 // Define o LCD com 20 colunas e 4 linhas
    pinMode(relay1,OUTPUT);

    pinMode(butsetO,INPUT_PULLUP);

    pinMode(butsetM,INPUT_PULLUP);

    pinMode(butsetU,INPUT_PULLUP);

  lcd.begin(20, 4);

    dht.begin();
    // Initialize the rtc object
    rtc.begin();
    // Set the clock to run-mode
    rtc.halt(false);
    // The following lines can be uncommented to set the time
    // rtc.setDOW(MONDAY);        // Set Day-of-Week to SUNDAY
    //rtc.setTime(11, 46, 0);     // Set the time to 12:00:00 (24hr format)
    // rtc.setDate(20, 8, 2018);   // Set the date to October 3th, 2010
}
void loop() {
  sensorValue = analogRead(sensorPin);

  percent = convertToPercent(sensorValue);

  printValuesToSerial();

  delay(1000);
}

int convertToPercent(int value)
{
  int percentValue = 0;

  percentValue = map(value, 1023, 465, 0, 100);

  return percentValue; 
}

void printValuesToSerial()
{
    readingbutsetH = digitalRead(butsetO);

    delay (100);

    readingbutsetH = digitalRead(butsetO);

    if (readingbutsetH ==LOW)
    {
       setHours++;

        if (setHours>23)
            {
                setHours=0; 
                delay(10);  
            }
    }

    readingbutsetM = digitalRead(butsetM);

    delay (100);

    readingbutsetM = digitalRead(butsetM);

        if (readingbutsetM ==LOW)
         {
            setMin++;

            if (setMin>59)
                {
                    setMin=0; 
                    delay(10);  
                }
         }

    readingbutsetU = digitalRead(butsetU);

       delay (100);

    readingbutsetU = digitalRead(butsetU);

         if (readingbutsetU ==LOW)
        {
            setUp++;

            if (setUp>70)
            {
                setUp=0; 

                delay(10);  
            }
        }
    ti = rtc.getTime();

    Hours = ti.hour;

    Minutes = ti.min;

    if ( setHours>=Hours& setUp>=percent)
    {
    //lcd.clear();
    //lcd.setCursor(2,1); // ziua 
    //lcd.print("ACTIVARE POMPA");
    digitalWrite(buzzer,HIGH);

    tone(6,400,300);

    delay (1000);

    digitalWrite(buzzer,HIGH);

    tone(6,400,300);

    delay (1000);

    lcd.clear();

    lcd.setCursor(4,0); // ziua 

    lcd.print("POMPA ACTIVA");

    lcd.setCursor(0,1); // ziua

    lcd.print(rtc.getTimeStr());

    lcd.setCursor(10,1); // ziua

    lcd.print(rtc.getDateStr());

    lcd.setCursor(0,2); // ziua

    lcd.print("Umiditate Pamant:");

    lcd.print(percent);

    lcd.print("%");

    lcd.setCursor(3,3); // ziua 

    lcd.print("ASTEPTARE 5 MIN");

    digitalWrite(relay1,HIGH);

    delay(10000);

    digitalWrite(relay1,LOW);

    }

    float h = dht.readHumidity();

    float t = dht.readTemperature();

  // curata ecrnu de eruri
    lcd.clear();

    lcd.setCursor(0,0); // ziua 

    lcd.print(rtc.getDOWStr());

    lcd.setCursor(11,2); // Umiditate aier

    lcd.print("Ua:"); 

    lcd.print(h);

    lcd.print("%");

    lcd.setCursor(10,1);  // Temperatura aier

    lcd.print("Ta:");

    lcd.print(t);

    lcd.print(" C"); 

    lcd.setCursor(10,0); /// dara

    lcd.print(rtc.getDateStr());

    lcd.setCursor(0,1); // Ora

    lcd.print(rtc.getTimeStr());

    lcd.setCursor(0,2); // Umiditae pamant

    lcd.print("Up:");

    lcd.print(percent);

    lcd.print("%");

    lcd.setCursor(1,3); // Umiditae pamant

    //lcd.print("SetO.");

    lcd.print(setHours);

    lcd.print(":");

    lcd.setCursor(4,3);

    lcd.print(setMin);

    lcd.setCursor(15,3);

    lcd.print(setUp);
     
    lcd.print("%");
}
