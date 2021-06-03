---
permalink: /projects/WirelessThermocoupleReader/
title: "Wireless Thermocouple Reader"
toc: true
toc_label: "Contents"
toc_icon: "cog"
---

![Wireless Thermocouple](/assets/images/WirelessThermocouple_foto.jpg){:height="400px" width="700px"}

## Purpose

When you run a chemical experiment, the Temperature is one of the key parameter. A bulb-thermometer is quite always in contact with the reacting chemicals. Thermocouples are valid alternatives expecially because are the most common solutions in the industrial plants. On the small scale,however, they may cause several problems. Thermocouples require cables to connect them to some electronic device that needs cables again to be powered and transfer data. If you need temperature measurements in several points of your experiment, immediately you realize that **the number of wires growth up soo much that limits the feasibility (and the safety!) of the experiment itself.** A partial solution of this problem is the adption of a wireless thermocouple reader. The number of cables is reduced by the absence of the communication wires that send the temperature data remotely. With this solution, you do not need to have neither a PC close to the expereiment, neither a Ethernet port. Data are fast and easily sent to a router as soon as they generated. A separate PC connection with the reuter allows data storing and visualization. Basically, the preferred characteristics of a wireless temperature reader in running a chemical experiment, are:

* **use a thermocouple and not a thermistor.** This is because thermistors have narrower temperature range, they are not chemically protected for aggressive chemicals, they lose lineary. Thermistors are very common nowaday because they are used in IoT, but very limited on high temperatures.

* **send data to a router in a open format.** Data and router must be managed and stored by a open protocol that can be eventually customized. Many wireless readers send data to a vendor server and then they are made accessible by an internet connection. However, this way is unfeasible in case of data generated for proprietary experiments.

* **send but also display data.** Data are collected and stored via the network. However people running the expriment need to know the exact value of the temperature in real time. A display is wellcome benefits for a wireless reader, although limited in the commercial product offer.

* **run under a low continuous voltage.** Due to the driving maket of Iot many commercial wireless reader are powered with alternate high tension that is quite common at home. This is not the case, in chemical experiemnts where liquids and direct contact of operator is the standard.

The new interesting possibilities given by the new low cost developping wirelss boards, suggest that the optimal wireless thermocouple reader is much simply built than bought. The following sections implement our actual solution. However with the wireless connectivity, we prospect a quite high and continous activity...

## Circuit
![Wireless Thermocouple Circuit](/assets/images/WirelessThermocouple_circuit.jpg){:height="400px" width="700px"}

## Code


	// Load max6675 library
	#include "max6675.h"
	// Load Wi-Fi library
	#include <WiFi.h>
	// Load LCD library
	#include <LiquidCrystal.h>
	// Replace asteriscs with your network credentials
	const char* ssid = "**************";
	const char* password = "***********";
	// Set web server port number to 80
	WiFiServer server(80);
	// Variable to store the HTTP request
	String header;
	// Assign output variables to GPIO pins
	int thermoCLK = 25;
	int thermoCS = 26;
	int thermoDO = 27;
	MAX6675 thermocouple(thermoCLK, thermoCS, thermoDO);
	float tempC;
	// Current time
	unsigned long currentTime = millis();
	// Previous time
	unsigned long previousTime = 0; 
	// Define timeout time in milliseconds (example: 2000ms = 2s)
	const long timeoutTime = 2000;
	// Assign LCD function
	LiquidCrystal lcd(15,23,18,19,21,22);

	void setup()
	{
	  lcd.begin(16, 2);
	  Serial.begin(115200);
	  // Connect to Wi-Fi network with SSID and password
	  Serial.print("Connecting to ");
	  Serial.println(ssid);
	  WiFi.begin(ssid, password);
	  while (WiFi.status() != WL_CONNECTED) {
		delay(500);
		Serial.print(".");
	  }
	  // Print local IP address and start web server
	  Serial.println("");
	  Serial.println("WiFi connected.");
	  Serial.println("IP address: ");
	  Serial.println(WiFi.localIP());
	  server.begin();
	}

	void loop()
	{
	  // Read sensor data
	  float tempC;
	  tempC = thermocouple.readCelsius();     // return Celsius
	  lcd.clear();
	  lcd.setCursor(0,1); 
	  lcd.print (tempC);
	 // Listen for incoming clients
	  WiFiClient client = server.available();
	  if(client) {
		// get a client
		Serial.println("New client");
		// an http request ends with a blank line
		bool currentLineIsBlank = true;
		// string to hold incoming data from client
		String currentLine = "";
		while(client.connected()) {
		  if(client.available()){
			// reading from the client
			char c = client.read();
			Serial.write(c);
			if (c == '\n' && currentLineIsBlank) {
			  client.println("HTTP/1.1 200 OK");
			  client.println("Content-type:text/html");
			  client.println();
			  Serial.print("Temperature (C) ");
			  Serial.println(tempC);
			  client.print("Temperature (C) ");
			  client.print(tempC);
			  client.println();
			  break;
			} 
			if (c == '\n') {
			  // you're starting a new line
			  currentLineIsBlank = true;
			  currentLine = "";
			} else if (c != '\r') {
			  // you've gotten a character on the current line
			  currentLineIsBlank = false;
			  currentLine += c;
			}
		  } 
		}
		// close the connection
		client.stop();
		Serial.println("Client disconnected");
	  }
	  delay(500);
	}

## Running Example
