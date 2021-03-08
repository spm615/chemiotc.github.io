---
title: "Post: Raspberry Pi thermostat"
classes: wide
categories:
  - Industry
tags:
  - Control
  - Sensors
---
A [home thermostat](https://opensource.com/article/21/3/thermostat-raspberry-pi) is built in this project.
At its core, a thermostat is just a type of switch. Once the thermistor (temp sensor) inside the thermostat detects a lower temperature, the switch closes and completes the 24V circuit. Instead of having a thermostat in every room, this project keeps all of them right next to the furnace so that all six-zone valves can be controlled by a relay module using six of the eight relays. The Raspberry Pi acts as the brains of the thermostat and controls each relay independently.

![Home thermostat](https://opensource.com/sites/default/files/uploads/furnacewiring.png)
