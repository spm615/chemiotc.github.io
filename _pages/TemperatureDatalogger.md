---
permalink: /projects/TemperatureDatalogger/
title: "Temperature Data Logger"
toc: true
toc_label: "Contents"
toc_icon: "cog"
---

![Temperature Datalogger](/assets/images/TemperatureDatalogger_foto.jpg){:height="700px" width="400px"}

## Purpose

Temperature is a very important parameter for chemistry. A chemical reaction may start and develop only if reactants have a defined temperature. Then, of course is important to control the tempertaure but this is a secondary task although important as well. But let's now focalize on a reliable temperature measure. **Tempertaure dataloggers are common tools so why do we project a new one?** To be used in a chemical experiemnt, you need to have all the following characteristics:

* **use a thermocouple and not a thermistor.** This is because thermistors have narrower temperature  range, they are not chemically protected for aggressive chemicals, they lose lineary. Termocouple are cheap and reliable and very common in chemistry. They are also available with different format (micro, small dish, anular ring, ...)

* **record data with a settable time-frame.** Some reactions are slow some others are fast. If you have to follow a reaction, you must be able to set the rate of data collection.

* **display and save data.** Some commercial dataloggers do not display data, but this a key point if you are running a reaction and see if it runs as you like.

* **save and make data available by a serial communication.** This is quite uncommon in commercial dataloggers. Why is it important? Saving data on a SDcard is important for data safety. Do not forget that our purpose focalize on the chemical reaction and we do not like to re-run it because we missed the temperature values. For the same reason, chemists are interested to visualize the Temperature profile (even more than the actual temperature value) and the possibility to connect the datalogger to a PC for immediate visualization is a wellcome plus.

* **have more than one possibility to power the datalogger.** Batteries are usefull in outdoor experiments or is number of electrical plugs is short, but their cost and short life are not commonly appreciated.

* **the datalogger should be easily integrable in open source software platform.** Commercial dataloggers are almost all integrable with proprietary software for the data download and visualization.

* **read more than one thermocouple.** In chemistry, Temperature is important but a Temperature Difference is important as well. Actually sometime, the Temperature Difference is more important since it gives to the chemist the "driving force" of a process.

Based on the previous points, we realized that now it is muche easy and quick to build our ideal Temperature Datalogger than look in the market to find a compulsory one. We also realized that is more chaep and of course funny...

## Circuit

![Temperature Datalogger Circuit](/assets/images/TemperatureDatalogger_circuit.jpg){:height="1000px" width="1200px"}


## Code

	#include <SPI.h>
	#include <SD.h>
	#include <LiquidCrystal.h>
	#include "max6675.h"
	#define FILE_BASE_NAME "Data"

	 
	// initialize the library with the numbers of the interface pins
	LiquidCrystal lcd(6, 7, 2, 3, 4, 5);
	File myFile;
	const uint8_t BASE_NAME_SIZE = sizeof(FILE_BASE_NAME) - 1;
	char fileName[] = FILE_BASE_NAME "00.csv";

	// change this to match your SD shield or module;
	const int chipSelect = 10;

	// Max6675 chip initialization
	int thermoDO1 = 14;  // analog A0
	int thermoCS1 = 15;  // analog A1 
	int thermoCLK1 = 16; // analog A2
	int thermoDO2 = 17;  // analog A3
	int thermoCS2 = 19;  // analog A4 
	int thermoCLK2 = 18; // analog A5
	MAX6675 thermocouple1(thermoCLK1, thermoCS1, thermoDO1);
	MAX6675 thermocouple2(thermoCLK2, thermoCS2, thermoDO2);


	void setup()
	{
	  // set up the LCD's number of columns and rows:
	  lcd.begin(16, 2);
	  
	  // Open serial communications and wait for port to open:
	  Serial.begin(9600);
	  while (!Serial) {
		; // wait for serial port to connect. Needed for Leonardo only
	  }
	  Serial.print("Initializing SD card...");

	  if (!SD.begin()) {
		Serial.println("initialization failed!");
		lcd.clear();
		lcd.setCursor(0, 0);
		lcd.print("No card");
		delay(50000);
		return;
	  }
	  Serial.println("initialization done.");

	//  // open the file. note that only one file can be open at a time,
	//  // so you have to close this one before opening another.
	//  myFile = SD.open("datalog.dat", FILE_WRITE);
	//    // close the file:
	//  myFile.close();
	  while (SD.exists(fileName)) {
		if (fileName[BASE_NAME_SIZE + 1] != '9') {
		  fileName[BASE_NAME_SIZE + 1]++;
		} else if (fileName[BASE_NAME_SIZE] != '9') {
		  fileName[BASE_NAME_SIZE + 1] = '0';
		  fileName[BASE_NAME_SIZE]++;
		} else {
		  Serial.println(F("Can't create file name"));
		  lcd.clear();
		  lcd.setCursor(0, 0);
		  lcd.print("No File");
		  delay(50000);
		  return;
		}
	  }
	  myFile = SD.open(fileName, FILE_WRITE);
	  if (!myFile) {
		Serial.println(F("open failed"));
		return;
	  }
	  Serial.print(F("opened: "));
	  Serial.println(fileName);
	  myFile.close();
	  delay(500);
	  
	  Serial.println("MAX6675 test");
	  // wait for MAX chip to stabilize
	  delay(500);
	 
	  // Print a message to the LCD.
	  lcd.print("Start DataLog");
	  delay(500);

	}

	void loop()
	{
	  String dataString = "";
	  // print the number of seconds since reset:
	  float actual_time = millis()/1000;
	  float tempC1 = thermocouple1.readCelsius();
	  float tempC2 = thermocouple2.readCelsius();
	  dataString += String(actual_time)+","+String(tempC1)+","+String(tempC2);
	  lcd.clear();
	  lcd.setCursor(0, 0);
	  lcd.print("Time "+ String(int(actual_time)));
	  lcd.setCursor(0, 1);
	  lcd.print("T "+ String(tempC1)+","+String(tempC2));
	  // if the file opened okay, write to it:
	  myFile = SD.open(fileName, FILE_WRITE);
	  if (myFile) {
		Serial.println(dataString);
		myFile.println(dataString);
		// close the file:
		myFile.close();
	  } else {
		// if the file didn't open, print an error:
		Serial.println("error opening datalog.dat");
	  }
	  delay(2000);
	}

## Running Example
