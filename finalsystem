#include <Arduino.h>
#include <SPI.h>
#include "C:\csc113\fall18\UtilLib\RFRadio.h"
#include "C:\csc113\fall18\UtilLib\Message.h"
#include "C:\csc113\fall18\UtilLib\DNS.h"
#include <avr/sleep.h>


csc113::RFRadio radio(7,8);
static csc113::Message msg;

const byte SENSORPIN = 2;			// sensor pin
volatile byte state = LOW;			// by default, no motion detected
volatile byte interrupState;
const int BUZZERPIN = 3;			// buzzer pin

const int SILENTMODE = 1;
const int ALARMMODE = 2;
const String DEFAULTMODE = "ALARMMODE";	// alarm mode by default
int mode = ALARMMODE;					// change DEFAULTMODE and mode to change default throughout code



// from RFRadioExp code
void transmitMessageHelper(const csc113::Message &msg)
{
	unsigned long sendTime = micros();
	if (!radio.write(&msg, sizeof(csc113::Message))) {
		Serial.println(F("failed to send data"));
	}
	unsigned long ackTime = micros();
	Serial.print(F("Sent data: "));
	Serial.println(msg.m_strData);
	Serial.print(F("Send delay: "));
	Serial.println(ackTime - sendTime);
}


// info to display upon start up for user
void startInfo()
{
	Serial.println("\nThis system is in " + DEFAULTMODE);
	Serial.println("Choose from the options below to change the current mode: ");
}

// input options
void printOptions()
{
	Serial.println(F("*** Serial Interface Options: "));
	Serial.println(F("(A) Alarm Mode "));
	Serial.println(F("(S) Silent Mode "));
	Serial.println(F("(P) Power Down system "));
}

// code to turn on buzzer
void buzzerOn()
{
	tone(3, 400);
	delay(300);
	noTone(3);
	delay(300);
	tone(3, 400);
	delay(300);
	tone(3, 400);
	delay(300);
	noTone(3);
	delay(300);
	tone(3, 400);
	delay(300);
	noTone(BUZZERPIN);
}

// sends message to gateway that alarm is on
void notifyOn()
{
	// Serial.println("entering Notify on");
	String str = ("alarm-is-on");
	msg.init(radio.getSrcAddress(),MESSAGE_TYPE_DATA,str);
	transmitMessageHelper(msg);
	if (mode == ALARMMODE) { buzzerOn(); }
}

// sends message to gateway that alarm is off
void notifyOff()
{
	String str = ("alarm-is-off");
	msg.init(radio.getSrcAddress(),MESSAGE_TYPE_DATA,str);
	transmitMessageHelper(msg);
	// Serial.println("leaving Notify off");
}

void alarm() {
	interrupState = digitalRead(SENSORPIN);
}


void setup()
{
	Serial.begin(115200);					// initialize serial
	pinMode(SENSORPIN, INPUT_PULLUP);		// initialize sensor as an input
	attachInterrupt(digitalPinToInterrupt(SENSORPIN),alarm, CHANGE);	// restrict interrupt to sleep mode
	radio.begin();

	// Set the PA Level low to prevent power supply related issues since this is a
	// getting_started sketch, and the likelihood of close proximity of the devices. RF24_PA_MAX is default.
	radio.setPALevel(RF24_PA_MAX);

	uint8_t srcAddress = csc113::DNS::nameToAddress("");
	uint8_t destAddress = csc113::DNS::nameToAddress("");
	radio.open(srcAddress,destAddress);

	startInfo();
	::printOptions();
}


// sleep mode
void powerSystemDown() {
	radio.powerDown();
	Serial.println(F("*** Begin Sleep"));
	delay(100);
	set_sleep_mode(SLEEP_MODE_PWR_DOWN);
    sleep_enable();
    sleep_bod_disable();
    sei();
    sleep_cpu();
    sleep_disable();
	Serial.println(F("*** End Sleep"));
	radio.powerUp();
	radio.startListening();
}



void loop()
{
	if (Serial.available())
	{
		char c = toupper(Serial.read());

		switch(c)
		{
			case 'A':
			{
				mode = ALARMMODE;
				Serial.println(F("*** In alarm mode"));
			}
			break;

			case 'S':
			{
				mode = SILENTMODE;
				Serial.println(F("*** In silent mode"));
			}
			break;

			case 'P':
				powerSystemDown();
			break;

			default:
			{
				Serial.println(F("*** Unknown option "));
				::printOptions();
			}
		}
	}

	else
	{
		state = digitalRead(SENSORPIN);
		if (state == HIGH)
		{
			notifyOn();
			do { state = digitalRead(SENSORPIN); }
			while (state == HIGH);
			notifyOff();
		}
	}
}

