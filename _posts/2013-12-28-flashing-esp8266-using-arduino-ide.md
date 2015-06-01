---
layout: post
title: Flashing ESP8266 using Arduino IDE
---

ESP8266 can be used in two ways, by using dependently with a microcontoller through USART - AT Commands, and by using it stand alone independently, while utilising its own GPIOs and other features that a microcontroller have such as Software PWM, Serial, I2C, SPI, EEPROM and etc.

Recently, there has been a lot of developments on frameworks supporting ESP8266 such as NodeMCU, Sming, and Arduino. And a bunch of communities which are very active in unraveling the very cheap SoC wifi module that only costs around $3-$5.

In this article, I will be sharing to you on how I got my setup working using the ESP8266 board on stand alone using the Arduino IDE. I used Mac OS and Ubuntu Linux when I was doing my experiment with the chip.

## Installing the IDE

Installing the IDE is very easy and straightforward

Download the IDE in this repo [https://github.com/esp8266/Arduino](https://github.com/esp8266/Arduino). Click the release suitable to your operating system.

![Test](http://imageshack.com/a/img910/2821/yiaGSh.png "Test")

Extract the downloaded compressed file. This will directly give you the executable arduino file.

![Test](http://imageshack.com/a/img908/1702/Y3GlFu.png "Test")

As you can see, it has new Board Settings such as Arduino ESP8266, Generic ESP8266 Board, and WIFIO

![Test](http://imageshack.com/a/img538/7822/7a7URA.png "Test")

As well as a new Programmer settings, esptool.

## Preparing the Hardware

### Materials
- ESP8266
- FTDI USB to TTL
- 3.3V Power Supply
- LED

Assemble the circuit according to the schematic shown below

![Test](https://lh3.googleusercontent.com/DubSNaAqPduElEWo09JyltjLVkNpxkMFCzqUBaeAhjk=w849-h471-no "Test")

The ESP8266 board is not 5V tolerant, so it is advisable to use 3.3V supply to prevent it from damaging the chip.

Some FTDI USB to TTL Adapters doesn’t have a sufficient current on their 3.3V output, and some doesn’t have a 3.3V at all.

![Test](http://41.media.tumblr.com/b91005ffb6b0e58551632c4e49bd51a6/tumblr_inline_noisdwllI61t9mq4x_400.png "Test")

In my experiment in this article, I used a USB to UART (Serial TTL) Converter Prolific PL2303 which I purchased in e-gizmo (shown above). The 3.3V output on this board was not enough to power the ESP8266 so I used an external power supply.

> Insufficient current supply on the ESP8266 can cause malfunction on the board. The power indicator will still light up, and the SSID of the WIFI can still be detected but it will not work as it is supposed to.

## Flashing the code to ESP8266

In this stage of the ESP8266 - Arduino IDE development, the blah blah.

1. **Set the ESP8266 to bootloader mode**. While putting the GPIO_0 to ground, reset the power on the ESP8266 by turning it off and on again. You can also put the reset pin from Vcc high to ground.
2. **Configure the Settings**.. Open the Arduino IDE,  Select “Generic ESP8266 Board” from the Tools → Boards menu, and the serial port your board is connected to from the Tools → Ports menu. Finally select “esptool” from the Tools → Programmer menu.
3. **Open and Edit Sample Sketch**.. Now go to the File → Examples → ESP8266WiFi menu and load the “WiFiWebServer” sketch into an editor window. Replace the placeholders values in the script for “ssid” and “password” with the SSID and password for your WiFi network.
4. **Upload the Sketch**. Hit the upload button. If all goes well you you should see that it is “Uploading…” and then, after a while, that it is “Done uploading.” If you encounter an error then you should double check and ensure that the CH_PD pin is pulled high, and the GPIO_0 pin is pulled low. Then toggle the power on the board again to make sure it’s in bootloader mode. You should probably also check that RX is wired to TX, and TX to RX.

## Running the ESP8266

Now go ahead and open the Serial Monitor, remove the jumper between the GPIO_0 pin and GND, you toggle the power to the board again. If you do not toggle the power to the board at this stage, you sketch won’t be loaded.

However if all is still going well you should see something like this,

>Connecting to Wireless Network
.......
WiFi connected
Server started
10.1.2.45

which tells you that the board has connected to the network, and rather crucially, what the board’s IP address ended up as when it negotiated its connection to the DHCP server.

Now you can just go to your browser and use the endpoint http://10.1.2.45/gpio/1 to pull GPIO_2 high, and turn the LED on, or http://10.1.2.45/gpio/0 to pull GPIO_2 low, and turn the LED off. If you’re still connected to the Serial Console you should see something like this scroll by if you attempt to turn the LED on,

>new client>
GET /gpio/1 HTTP/1.1
Client disconnected
new client
GET /favicon.ico HTTP/1.1
>invalid request

Here you can safely ignore the invalid request error message — that’s just your browser asking for the icon that accompanies the web pages its retrieving, it doesn’t necessarily expect a response — however at this point the LED connected to GPIO_2 should be on. Go ahead and try turning it off again.

## Conclusion

The ESP8266 module should have a sufficient current supply. It is advisable to use an external 3.3V power supply for it to work properly. The trickiest part in the process of making ESP8266 to work is the flashing of the code.You should first put the board into boot loader mode by pulling GPIO_0 down to ground, and then restart the power of the board.

ESP8266 using Arduino IDE is still in experimental stage, there are still a lot of unstable features that you can find compared to a generic Arduino (AVR) micro controller. However this is still a work in progress, and achieved already a lot considering its early stage.

Aside from the Arduino IDE, there are also other solutions to make ESP8266 work stand alone like NodeMCU and Sming Framework.
