---
layout: post
title: Mobile App Controlled Relays through ESP8266 via MQTT+HTTP
video-top: 7B2xMqvkf_I
---
The system is based on a very cheap $3 wifi module - ESP8266 which connects to the Node.js REST API server via Mosquitto's free online MQTT Broker (test.mosquitto.org). The Android Mobile App is built using Ionic Framework utilizing AngularJs.
The REST API Server is hosted on a cloud server(heroku) which enables the relays to be controlled online via internet.

![Test](https://lh3.googleusercontent.com/MT3cCGJZptMq1ZqHtQfZnXR6KrfGNixHuM3Td_rG25A=w3360-h1890-no "esp8266")

The diagram below illustrates the connection flow of the system. The ESP8266 Wifi Module should be connected to the internet through wifi for this to work.

![Test](https://lh3.googleusercontent.com/5qLDksi8lvpFFtA0XhiToL0T-ywVQpjcK9AI6W-7fxU=w684-h392-no "esp8266 - diagram")

## Requirements



### Hardware
- ESP8266
- 5V Power Supply ( at least 1A )
- 3.3V Voltage Regulator
- 5V Relays
- 2N2222A Transistor
- Resistors
- 1N4108 Diodes
- LEDs

### Software
- Node.js
- Ionic Framework
- Arduino IDE

## Assembling the Circuit

![Test](https://lh3.googleusercontent.com/9D8iGzvVCNlwAAgmo1qCA_KNw6jdZB9iLx7QF2MHOLc=w975-h912-no "esp8266 - circuit")

## Preparing the Software

1.) Download Node.js here [https://nodejs.org/download/](https://nodejs.org/download/). The installation procedure depends on your operating system.
If you are using Ubuntu Linux, you can use this tutorial [https://elizarpepino.com/install-latest-nodejs-on-ubuntu-box/](https://elizarpepino.com/install-latest-nodejs-on-ubuntu-box/). To test if node has been installed properly, you should be able to run these commands and display the current version.

```bash
node -v
npm -v
```

2.) Install Ionic Framework from here [http://ionicframework.com/getting-started/](http://ionicframework.com/getting-started/). Ionic Framework is dependent with node.js, so you must install it first.

3.) In setting up the Arduino IDE for ESP8266, you can go to my previous article [http://vinceelizaga.com/2015/05/28/flashing-esp8266-using-arduino-ide/](http://vinceelizaga.com/2015/05/28/flashing-esp8266-using-arduino-ide/).

## Diving into the code
There are three code modules which are the MQTT Client + REST API Server, ESP8266 Arduino, and the Ionic Mobile App Framework.

You can get the codes in these github repos.

- MQTT Client + REST API Server - [https://github.com/vynci/MQTT-REST-API](https://github.com/vynci/MQTT-REST-API)
- ESP8266 Arduino - [https://github.com/vynci/esp8266-relay](https://github.com/vynci/esp8266-relay)
- Ionic Mobile App - [https://github.com/vynci/esp8266-ionic](https://github.com/vynci/esp8266-ionic)

### MQTT Client + Rest API Server
This is the server that has been deployed to heroku(http://esp8266-relays.herokuapp.com). You can create your own server locally or by deploying it in the cloud. This node application requirest two modules which are the hapi.js and mqtt.js. Hapi.js is a REST API framework, while MQTT.js is an mqtt client.

```javascript
var Hapi = require('hapi');
var mqtt = require('mqtt');

var server = new Hapi.Server();
server.connection({ port: 4444, routes: { cors: true } });

var client  = mqtt.connect('mqtt://test.mosquitto.org:1883');

var mqttPublish = function(topic, msg){
  client.publish(topic, msg, function() {
    console.log('msg sent: ' + msg);
  });
}

server.route([
  {
    method: 'POST',
    path: '/device/control',
    handler: function (request, reply) {
      var deviceInfo = 'dev' + request.payload.deviceNum + '-' + request.payload.command;
      reply(deviceInfo);
      mqttPublish('device/control', deviceInfo, {
        'qos' : 2
      });
    }
  }
]);

server.start();
```

### ESP8266 Arduino Code
This code block right here connects to the wifi, and to the MQTT broker.

```c++
#include <PubSubClient.h>
#include <ESP8266WiFi.h>

const char* ssid = "your-wifi-ssid";
const char* password = "your-wifi-passwd";

char* topic = "device/control";
char* topicPublish = "device/sensor";
char* server = "ip-address-of-your-server";

WiFiClient wifiClient;
PubSubClient client(server, 1883, callback, wifiClient);
.....

```
### Ionic - Mobile App
This file(services.js) right here connects to the REST API server, doing a POST method
containing the command information in the payload.

```javascript
angular.module('starter.services', [])

.factory('Devices', function($http) {

  var ipServer = 'http://esp8266-relays.herokuapp.com';

  return {
    deviceCommand: function(data) {
      console.log(data);
      return $http.post(ipServer + '/device/control', data);
    }
  };
});
```
