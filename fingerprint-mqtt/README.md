# fingerprint-mqtt.ino

This is the default sketch and is the one used in the guide. You won't need to manually change anything to run this and only need to install the libraries mentioned in the root README file.

# fingerprint-mqtt-led.ino

This version is a slight variation on the default sketch but allows you to enable and disable the onboard fingerprint LED light via an MQTT topic. I have Home Assistant turn the sensor on/off when motion is detected in the same room. 

At time of writing the Adafruit fingerprint library doesn't have LED functionality so we'll need to modify the library files to add this. Luckily it's pretty easy, find the library folder and follow the below steps.

Note I've only tested this with the sensor in the guide.

# fingerprint-mqtt-led-touch.ino

Same as fingerprint-mqtt-led.ino, but in addition implements module's built-in touch sensor to only turn on LED when a finger touches the glass. Module can still be disabled (regardless of finger detection) using the MQTT topic.

# fingerprint-mqtt-led-touch-oled.ino

Same as fingerprint-mqtt-led-touch.ino, but in addition implements OLED functionality.  Note the following important changes!

- MQTT structure differs from other methods:
  - State topic publishes the state (ie: idle, reading, matched, not matched, learning, deleting), and the ID. Ex: {"state":"idle","id":"0"}
  - Request topic is subscribed. Send "learn" or "delete" with the id for that action. Ex: {"request":"learn","id":"6"}
  - Reply topic is subscribed. Send a message to display on the OLED to confirm your automation. This topic provides for 2 lines of text as a response. Ex: You want to use your right thumb (id 1) to arm your security system, and your left thumb (id 6) to disarm it. You setup an automation to arm on recieving from the state topic {"state":"matched","id":"1"}. In the automation, you publish an MQTT message to the reply topic {"line1":"HELLO IAN","line2":"SYSTEM ARMED"} to display on your OLED that the automation has fired. Repeat for left thumb (id 6), but with a disarmed message.
- If using the included automation for the reply topic for Home Assistant, note that this requires you to be on at least 0.114 as it uses the "choose" condition. If you don't want to upgrade you can redo the automation to avoid using the "choose" functionality.
- You'll need to add in the appropriate libraries for the OLED...see https://everythingsmarthome.co.uk/esp8266/adding-an-ssd1306-oled-display-to-any-project/

**Note**: You must wire pins 4 and 5 of the FPM10A for this to work. Wire T-Out to D3 and T-3v to 3v. 


## Adafruit_Fingerprint.h

Below the initial define section add this

```
#define FINGERPRINT_OPENLED 0x50
#define FINGERPRINT_CLOSELED 0x51
```

Then declare the new methods

```
//The first three are already there, just placed here as a reference
  uint8_t getImage(void);
  uint8_t image2Tz(uint8_t slot = 1);
  uint8_t createModel(void);
  uint8_t OpenLED(void);
  uint8_t CloseLED(void);
```

## Adafruit_Fingerprint.cpp

Add the new methods:

```
uint8_t Adafruit_Fingerprint::image2Tz(uint8_t slot) { // this one is already in your cpp file, I'm just showing it here as a placemente reference
  SEND_CMD_PACKET(FINGERPRINT_IMAGE2TZ,slot);
}

// Custom function
/**************************************************************************/
/*!
    @brief   Ask the sensor to turn on the LED
    @returns <code>FINGERPRINT_OK</code> on success
    @returns anything else for operation failed
*/
/**************************************************************************/
uint8_t Adafruit_Fingerprint::OpenLED(void) {
  SEND_CMD_PACKET(FINGERPRINT_OPENLED);
}

// Custom function
/**************************************************************************/
/*!
    @brief   Ask the sensor to turn off the LED
    @returns <code>FINGERPRINT_OK</code> on success
    @returns anything else for operation failed
*/
/**************************************************************************/
uint8_t Adafruit_Fingerprint::CloseLED(void) {
  SEND_CMD_PACKET(FINGERPRINT_CLOSELED);
}
```

You'll then be able to send `on` or `off` to the `/fingerprint/enabled` topic to enable/disable the sensor. You will however need the LED on in order to take a fingerprint reading. 
