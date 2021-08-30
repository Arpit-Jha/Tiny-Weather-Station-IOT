# Tiny-Weather-Station

# Hardware
The hardware for this project is fairly simple. We use the Arduino Nano 33 IoT which handles WiFi, a BME280 sensor board with temperature, humidity and barometric pressure all in one unit, and a 1 inch OLED display. All three run fine at 3.3 volts. The display is obviously optional - the goal is to get the weather station displaying on your phone. But it makes it much easier to get everything up and running on the OLED display first, before dealing with Blynk and your smartphone.

The schematic below shows how the hardware is interconnected. Both the sensor and the display interface with I2C. In theory, one I2C can support multiple devices, but the libraries of the sensor and display had some conflicts, so I ended up with two different I2C ports. The sensor uses the normal default I2C port at analog pins 4 and 5. The display uses a secondary I2C port set up at digital pins 4 and 5. (And yes, they are reversed - SDA is digital pin 5))

![image](https://user-images.githubusercontent.com/77734479/131398284-2c40fffb-2b19-4c5e-942b-27e3b0dccae5.png)

The hardware is enclosed in a small plastic case with a clear front, so that the display can be viewed. It is plastic, as it needs to be transparent to RF for WiFi. It needs to be relatively weather-proof, but the sensor is exposed to the outside weather by a 1/2" hole at the bottom. The sensor is attached so that it sits just inside that hole. I attached everything inside the case with hot glue, but you might want to use epoxy instead if you are planning to put it outside in the summer heat.

Power is supplied through the USB cable. That way, it can be programmed or powered by a single cable coming out of the box. That cable can go to your computer when programming or to a plug-in USB power supply when in use as a weather station.

# Software 

The software running on the Arduino is just a little more than a mash-up of the library examples for the sensor, the display and Blynk. The libraries involved are the Adafruit_BME280_Library for the sensor, the ss_oled library for the display, and the Blynk library for Blynk. All three can be downloaded directly from Arduino's library manager. You may want to experiment with each of these libraries separately to get a better understanding of each.

Blynk has a bunch of examples of how to configure hardware to interface with it: https://examples.blynk.cc/ Unfortunately, it doesn't list the Nano 33 IoT as one of its supported Arduino's. But Blynk does support WiFiNINA, which is used by the Nano 33 IoT and several other Arduino processors. So adding these two includes solved that problem: #include<WiFiNINA.h> and #include<BlynkSimpleWiFiNINA.h

One other addition you will see in my code is: #include <avr/dtostrf.h> This was necessary to convert the numbers coming out of the sensor into strings suitable to display on the OLED display. One little twist on this that gave me some grief was that Blynk is happy to display the data whether sent as a string or a number. At first I sent Blynk the same string I was sending to the OLED. That worked fine to display the numbers, but failed totally when I added graphs. To get the graphs to work, Blynk obviously needed actual numbers.

Note - barometric pressure is normally reported corrected to sea level. At higher elevations, barometric pressure obviously drops, so to correct values from the sensor to their equivalent reading at sea level, you must add in a value to the measured value of roughly 1 inch Hg for every 1000 feet of elevation. In my own code attached, you will see I added 1.3 to compensate for my own elevation of 1300 ft. above sea level. You will want to change this value to match your own elevation. The 1" Hg per 1000 feet is just a rough estimate. If you want a precise correction, there are tables online that will give you the exact correction for your elevation.

# Blynk

Blynk is easy to install on your iPhone or Android and comes with detailed documentation: http://docs.blynk.cc/ But there is so much information there and so many options that I found things a little confusing at first, so I will provide here my own version of how to get started with Blynk.

I used Blynk with an iPhone, but I think the experience is similar enough with Android that you can follow my instructions with either one. Once you have the app, you need to create an account. From there, within the app, you create a new project. Your project is provided with an authentication code that is used to link your hardware with your project. Your first job is to get your hardware connected through WiFi to your Blynk project. You can accomplish this and test it out using Blynk's default sketch/program called Blynk Blink at examples.blynk.cc, which allows you to turn on and off the onboard LED on your Nano 33 IoT. My suggestion is you try this and get it working before trying to get the weather station working with Blynk.

We have already talked a little about configuring our software to get our Nano 33 IoT to work with Blynk. If you open examples.blynk.cc, it defaults to a ESP8266 board. Find #include <ESP8266WiFi.h> and #include <BlynkSimpleEsp8266.h> and replace them with #include<WiFiNINA.h> and #include<BlynkSimpleWiFiNINA.h for our Nano 33 IoT board.

In addition to adding WiFiNINA support for the Nano 33 IoT, as we already talked about, you need to add your WiFi credentials and your Blynk project's authentication code. We will explain turning on and off the LED in a minute, but first, let's just check the connection. With the Blynk Blink sketch properly configured and running, open Arduino's Serial Monitor, and you should be able to watch the connection being made to the Blynk cloud server. At this point, we can set aside our hardware and work with the Blynk app on the phone.

![image](https://user-images.githubusercontent.com/77734479/131398441-7bd6fd7e-4419-45e6-a4a3-f01fa684db4d.png)

To get control of our onboard LED, we need to enter Edit mode in the app. You will then see a blank screen. If you swipe left, it will move aside to reveal a toolbox of widgets. Select a button by clicking on it and it will now be on the main screen. Click on it there and it will open for configuration. Move it from push mode to switch mode. Use pin select to select the onboard LED - digital pin 13 on our Nano 33 IoT. Now click OK, and hit the top right icon to exit Edit mode. Your button should now control the LED.

# Weather Station

We are now ready to link our weather station to our Blynk project. Open my attached software, uncomment the Blynk.begin() line in Setup, add your project authentication code and WiFi credentials, and upload to Arduino.

Open Blynk on your phone. If you have followed this tutorial, your button is still there, and should still be able to turn the onboard LED on and off. Go to edit mode, click the button to configure it, and delete it with Delete at the very bottom. Now go to the toolbox and select a Labeled Value. It's down the list a way under Displays. Back on display page, click on the Labeled Value display to configure it. Click on pin, then select Virtual pin V3. We will explain virtual pins in the next paragraph. Now for the label, where it says "e.g. Temp" type "Temp /pin.#/ deg.F". The.# tells the app to display one place past the decimal point. Select the large text size, and leave Refresh Interval on Push and Text Color on Green. Now click OK. Your Labeled Value is now almost ready to display the temperature, but it is too narrow. Select it slowly - slow enough that it does not reenter configuration mode. The outline of the label will light up. It can now be stretched to display the whole line. It can also be moved around, though we won't do that here. Stretch it about 3/4 of the way across the the screen. Then exit Edit mode and you should see the temperature displayed on your phone.
You will need two more Labeled Value displays, one for Humidity in % (Virtual pin V4) and one for Pressure in " in. HG" or inches of mercury (Virtual pin V5). To make it look like mine, you will want humidity in gold and pressure in red. For humidity, I showed one place past the decimal point; for pressure, I showed two places past the decimal point.

If you now have the three Labeled Value displays showing temperature, humidity and barometric pressure, you can add the graph of all three. This is accomplished with the addition of a SuperChart widget. First, stretch it downward so that it fills the rest of the screen. Then click on it to configure. I turned on Show x axis values and I chose resolutions of live, 1 hr, 6 hr, 1 day, 1 wk, 1 mo. and 3 mo. We need three data steams - one for each of our 3 variables. To configure each data stream, click the icon to the right of it. We need to again select the virtual pin for each. For y axis scaling, choose height. Then for temperature, set height to 67 - 100. For humidity, set height to 34 - 66. For pressure, set height to 0 - 33. Set the colors to match the labeled values. Turn on Show Y axis. That's about it. Get out of Edit mode and your phone display should look like mine. It takes a little time for the graph to get started. For fast results, view live or 1 hour resolution.

![image](https://user-images.githubusercontent.com/77734479/131398665-c20ff920-4a7f-4fd2-9d43-86c990a844d9.png)


