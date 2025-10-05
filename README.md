# IoT_ESP8266_communiceren

## Introduction 
Today we will let 2 ESP8266 boards communicate with each other using the adafruit API combined with a button and a ledstrip. In this guide we will go step by step on how to do this, with potential errors that could occur in the process. This way everyone is able to replicate this! So lets start off with everything you need. 
This guide will be divided into 3 steps. Setting up your boards, the code for the sender, and the code for the receiver. If you left off at one of these steps you can easily navigate back!.

## Parts
For this guide you will need
  - two ESP8266 boards
  - a ledstrip
  - a button module for the ESP8266
  - [Arduino IDE](https://www.arduino.cc/en/software)

## Step 1 - Set-up 
We will first set up the receiver board. This board will receive the signals that the other ESP8266 board sends. For this board we only need to attach a LEDstrip. Attaching the strip should be easy. Attach the yellow cable to D1, the red one to 3V and the black one on G.
<img width="500" alt="image" src="https://github.com/user-attachments/assets/a0731280-52d7-42e3-a33f-9091c6f67985" />

For the second board we just need to attach the button module. 
The button module has 3 pins, the left one says "VCC" The middle one "OUT" and the right one "GND". Attach the VCC pin to 3V. 
<br>⚠️Heads up! Dont attach your VCC pin to a 5V pin. This could cause damage to your hardware. </br>
Attach the OUT to D0, and lastly attach GND to G.

That was everything you needed to do to make the boards work! Now we can start with the sender. 

## Step 2 - The sender.
For this step, we will use the board with the button module. Leave the ledstrip board to your side for now.
First install the adafruit neopixel library. It will probably be the third option you see. Make sure to check if you are downloading the correct one. 
⚠️ If you are getting errors later on make sure you have not installed the DMA version by accident. We need the normal one. 

Now that we have done that its time to make ArduinoIDE recognize your board.
You can do this easily by going to tools > board > ESP8266 > NodeMCU 1.0 (ESP-12E Module). You may need to scroll down in the last dropdown menu to find it.
Also make sure to select the correct port. Again, Tools > port > select your port. 

Now that we've added your board we can start with the code. Navigate to File at the top left corner > examples > scroll down till you find Adafruit IO Arduino > Click on adafruitio_06_digital_in. This will open a code snippet which gives us the basics. However the version that we're targeting modified it a lot. 

First of all we need to edit the config.
<br>⚠️ A common error that people make is pasting your config data in the arduino file itself. This is wrong and the ESP8266 won't detect anything in there, so if something is not working that could be a cause.
<img width="1018" height="242" alt="image" src="https://github.com/user-attachments/assets/e08ded81-7c1e-4fb0-8fc2-aed82487108e" />

Instead, look at the top of the file to see a tab that says "config.h". in this file you can place all your data. 
As you can see, arduino asks for 4 different passwords or usernames.
```
#define IO_USERNAME "your_username"
#define IO_KEY "your_key"
#define WIFI_SSID "your_ssid"
#define WIFI_PASS "your_pass"
```
For the top two, you need to create an account on [io.adafruit.com](https://io.adafruit.com)
Once you've done that, you can access your key By clicking on the key icon in the navigation. Copy paste this into the config file.
<img width="2506" height="1118" alt="image" src="https://github.com/user-attachments/assets/7b32b7ff-bde6-4654-9ed1-7695c2fe1628" />

Now we will work on the main file. 
on line 24, it asks you to define the button pin. Because our board is different, we need to change that from 5 to D0. 

Next on line 29 below the booleans with the button states, we add a new state. the LED state. 
```
int ledState = false;
```

In the void loop we made some major changes. You can delete the void loop you have in the file right now, as the changes are so huge that its easier to copy paste the whole thing instantly.
```
void loop() {

  // io.run(); is required for all sketches.
  // it should always be present at the top of your loop
  // function. it keeps the client connected to
  // io.adafruit.com, and processes any incoming data.
  io.run();

  // grab the current state of the button.
  // we have to flip the logic because we are
  // using a pullup resistor.
  if(digitalRead(BUTTON_PIN) == LOW)
    current = true;
  else
    current = false;

  // return if the value hasn't changed
  if(current == last)
    return;

  // save the current state to the 'digital' feed on adafruit io
  Serial.print("sending button -> ");
  Serial.println(current);
  digital->save(current);

  // store last button state
  last = current;

}
```
What this code does is that it sets up a toggle system, which will come into play again later in this guide, when we talk about the receiver. 

This was everything we needed to do for the sender. You can test out if everything works by uploading this code to the board by clicking the arrow on the navigation. If you open the serial monitor (the spyglass with dots at the top right) you can see whether it responds to your clicks. If it does you are all set! 
You can also open the feed to see if the responses are processed online aswell. Via https://io.adafruit.com/youruser/overview

> ⚠️Potential errors that could occur here : 
- If the console log shows anything relating to expecting a { or a ; it could mean that you've pasted the code on the wrong place. Double check everything from step 1 again.
- If it does not show anything in the serial monitor it could mean that your board isnt connected to the internet. Make sure that your wifi password/username are correct, and that you enabled the option to have compatible internet with a 2.4ghz connection.
- If it shows any errors about a port, go back to the beginning of this step and check if your board is really connected to the right port.


## Step 3 - The receiver
For this step we will use the board with the ledstrip! take out the board with the button module to make sure you dont send the same file to both boards. 
We will once again load up an example code. Navigate to File at the top left corner > examples > scroll down till you find Adafruit IO Arduino > Click on adafruitio_07_digital_out. This file is as you can see in the name the counterpart to what we just did. The structure is also the same as file 1, so once again navigate to the config file first, and input the SAME passwords and usernames that you've used in the first step. This is important as we need to make sure that both boards are on the same feed.
This time, we will however also paste something in the config of the main file. so from line 20 and onwards.
```
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
 #include <avr/power.h> // Required for 16 MHz Adafruit Trinket
#endif
```
We include these libraries to make the ledstrip work on your board. We also once again need to redefine the LED_PIN to D0 and add your normal PIN to D1, and the NUMPIXELS to 12. We also need to add 2 more lines to make the LED's work. Please use the code snippet below as a reference.
```
rest of your code
...
/************************** Configuration ***********************************/

// edit the config.h tab and enter your Adafruit IO credentials
// and any additional configuration needed for WiFi, cellular,
// or ethernet clients.
#include "config.h"
#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
 #include <avr/power.h> // Required for 16 MHz Adafruit Trinket
#endif

/************************ Example Starts Here *******************************/

// digital pin 5
#define LED_PIN D0
#define PIN D1 
#define NUMPIXELS 12

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

#define DELAYVAL 500 // Time (in milliseconds) to pause between pixels

...
rest of your code
```

Now that we've done the configuration, its time to jump into the void setup. We need to add 2 things at the top of the void setup and as you can guess, both have something to do with the LEDstip.

```
#if defined(__AVR_ATtiny85__) && (F_CPU == 16000000)
    clock_prescale_set(clock_div_1);
  #endif
    // END of Trinket-specific code.

    pixels.begin(); // INITIALIZE NeoPixel strip object (REQUIRED)
```
Paste this as the top of the void setup. 

The void loop is our last part of the code, and also the last part where we made any changes. These changes are also quite substantial. We've basically reworked the entire handlemessage function, so you can delete that in your file, and paste the following code snippet. 

```
void handleMessage(AdafruitIO_Data *data) {

  Serial.print("received <- ");

  if(data->toPinLevel() == HIGH) {
    Serial.println("HIGH"); 
      for(int i=0; i<NUMPIXELS; i++) {
          pixels.setPixelColor(i, pixels.Color(0, 150, 0));
          pixels.show();
        }
    }
  else {
    Serial.println("LOW");
     pixels.clear(); 
      pixels.show(); 
  }

  digitalWrite(LED_PIN, data->toPinLevel());
}
```

And that's all you have to do! if everythings gone well, this should mean that the code is working!
<img width="4000" height="1800" alt="image" src="https://github.com/user-attachments/assets/6b473670-fe34-44fa-9d07-f094e0e8d907" />
We hope you enjoy this awesome interaction between two ESP2688 boards.

> ⚠️Potential errors that could occur here : 
- Open the serial monitor like you've done in step 2 to check whether your board is receiving signals. If its not, make sure you've put in the correct information in the config file.
- Make sure you have not accidentally left both boards in your device. If something is not working it could be that you accidentally uploaded the same file on both devices.
- If the LEDstrip is not turning on make sure you've added all of the code we added in this step. Check the step again and compare it with your code.




