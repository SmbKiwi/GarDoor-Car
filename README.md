# GarDoor-Car:
Version: 2.00R

## ESP8266 MQTT Garage Door Opener, Garage Door and Car Sensor, and Temperature Monitor using Home Assistant/Blynk
This project allows you to control (open/close) a "dumb" garage door opener for a garage door and independently reports the garage door status (open/closed) and presence of a car in the garage via MQTT and Blynk. In addition you can monitor the Temperature and Humidity within your garage over MQTT as well. This project requires two ultrasonic sensors: one pointed at the garage door, and the other pointed at the normal location of the car. This allows the presence of the car to be detected whether the garage door is open or closed.  

The code covered in this repository utilises Home Assistant's [MQTT Cover Component](https://www.home-assistant.io/components/cover.mqtt/) and [MQTT Binary Sensor Component](https://www.home-assistant.io/components/binary_sensor.mqtt/). There is a sample configuration in the repository. GarDoor will respond to HA's open and close commands and reports door/car status to keep HA's GUI in sync with the door and car state. GarDoor should also be controllable via any home automation software that can work with MQTT.  

GarDoor can also be used with the Blynk app (iOS or Android). If you enable this feature, you can monitor the garage door/car status as well as open/close the door with the app. You can also choose to be notified of garage door events on your phone. 

GarDoor requires both hardware and software components. It requires an ESP8266 microcontroller with sensors and a relay. If you select the appropriate parts, you can build a GarDoor-Car device with no soldering! The software component is found in this repo. 

The code is based on DotNetDann's ESP-MQTT-GarageDoorSensor project (which took inspiration from GarHAge and OpenGarage) with added additional features. 

### Supported Features Include
- Reports status of the garage door
- Reports presence of a car in the garage whether the door is open or not
- Reports ultrasonic sensors distance readings
- Reports information about the ESP8266
- Reports information on Temperature and Humidity
- Ability to select Celius or Fahrenheit for Temperature
- Web page for status of the garage door, car presence, and temperature/humidity
- Ability to use the Blynk App (iOS or Android) to open/close the garage door and monitor door/car status
- Ability to receive Blynk notifications when garage door is opened/closed and when door is left open for more than 15 minutes
- Events and status sent via MQTT
- MQTT commands to toggle relay to open/close the garage door
- Ability to select an Active High or Active Low relay
- Over-the-Air (OTA) Upload from the ArduinoIDE!

### How GarDoor-Car Operates

GarDoor-Car subscribes to a configurable MQTT topic for the garage door (by default, gardoor/1/action).

When the OPEN payload is received on this topic, GarDoor-Car momentarily activates the relay connected to the garage door opener to cause the door to open.

When the CLOSE payload is received on this topic, GarDoor-Car momentarily activates the relay connected to the garage door opener to cause the door to close. By default, GarDoor-Car is configured to activate the same relay for the OPEN and CLOSE payloads, as most (if not all) garage door openers operate manually by momentarily closing the same circuit to both open and close.

When the STATE payload is received on this topic, GarDoor-Car publishes the status of the garage door (open or closed) to the topic gardoor/1/status and the occupied status of the car (true or false) to the topic gardoor/2/status. The distance measurements for the door and car sensors are also included in these messages. These messages are published with the "retain" flag set. (Note: To address a current issue in Home Assistant that may result in MQTT platforms not showing the correct garage door and car status after a HA restart, it is recommended that you create an automation in Home Assistant that publishes the STATE payload on HA start. An example is provided in the Configuring Home Assistant section of this documentation.)

When the state of the garage door changes (either because GarDoor-Car (via HA/MQTT or Blynk app) has triggered the door to open or close, or because the door has been opened or closed via a remote, pushbutton switch, key switch, or manually), GarDoor-Car publishes the status (open or closed) and sensor distance measurement of the garage door to gardoor/door/1/status. These messages are published with the "retain" flag set.

When the occupied state of the car changes because the car leaves or enters the garage, GarDoor-Car publishes the occupied status (true or false) and sensor distance measurement of the car to gardoor/2/status. These messages are published with the "retain" flag set.

GarDoor-Car also publishes (topic gardoor/availability) a "birth" message on connection to your MQTT broker, and a "last-will-and-testament" message on disconnection from the broker, so that your home automation software can respond appropriately to GarDoor-Car being online or offline.

GarDoor-Car also publishes to your MQTT broker on topic gardoor/blynkavailability (only if GarDoor-Car is already online; if GarDoor-Car goes offline first, then an automation in your home automation software is required to update the status on the gardoor/blynkavailability topic) a message when GarDoor-Car connects or disconnects from the Blynk server, so that your home automation software can respond appropriately to the GarDoor-Car Blynk connection being online or offline.

GarDoor-Car publishes data about the ESP8266 (topic gardoor/ESPtoMQTT) and temperature/humidity (topics gardoor/temperature and gardoor/humidity) to your MQTT broker.

GarDoor-Car can also connect (if enabled by the user) to a Blynk server. This enables door and car status updates, as well as the door sensor distance measurement to be published to a Blynk server/app. GarDoor-Car can also receieve button requests from the Blynk server/app, which causes GarDoor-Car to momentarily activate the relay connected to the garage door opener to cause the door to open or close. This feature gives a user the ability to press a button in the Blynk app to control the garage door. Notifications can also be published to the Blynk server/app when the garage door open/closes or is left open too long. 

### Hardware Parts List
- [NodeMCU](https://www.aliexpress.com/item/32665100123.html) or other ESP8266-based microcontroller 
- [2 x HC-SR04P Ultrasonic Sensors](https://www.aliexpress.com/item/32711959780.html) May also use HC-SR04 model
- [5V 1 Channel Relay Board](https://www.gearbest.com/relays/pp_226384.html) Active-High or Active-Low supported
- [DHT22 module](https://www.aliexpress.com/item/32899808141.html) (optional)  
- Electrical cable (two-conductor) to connect the relay to your garage door control panel 
- Power source for NodeMCU: MicroUSB cable or 5V power supply  
- Male-to-female (and Male-to-male if you use HC-SR04 model ultrasonic sensors and need to connect four components to 5v power using the breadboard power rails) breadboard jumper wires (Dupont)
- Breadboard 400 tie-point or larger
- Project box or case (optional)

#### Detailed Hardware

#### 1. ESP8266-based microcontroller
I recommend the NodeMCU as GarDoor-Car was developed and is tested on it. Its advantages are:

- it comes with header pins already soldered so that it can plug directly into a solderless breadboard;
- its VIN (or VU on the LoLin NodeMCU v3 variant) port can power the 5v relay module and DHT22 sensor;
- it can be powered and programmed via MicroUSB;
- it has Reset and Flash buttons, making programming easy.

Accordingly, this guide is written with the NodeMCU in mind. But, GarDoor-Car should also work with the Adafruit HUZZAH, Wemos D1, or similar, though you may need to adjust the GPIO ports used by the sketch to match the ESP8266 ports that your microcontroller makes available.

#### 2. Single 5v relay module
A single 5v relay module makes setup easy: just plug jumper wires from the module's VCC, S, and GND pins to the NodeMCU. Because the relay module is powered by 5v, its inputs can be triggered by the NodeMCU's GPIOs.

GarDoor-Car will work with relay modules that are active-high or active-low; if using an active-low relay, be sure to set the relevant configuration parameter in auth.h, described below, and test thoroughly to be sure that your garage door opener(s) are not inadvertently triggered after a momentary power-loss.

#### 3. Two Ultrasonic Sensors (model HC-SR04P recommended)
GarDoor-Car will work with either the HC-SR04 or HC-SR04P model. The HC-SR04P can use 3.3v or 5v power, while the HC-SR04 requires 5v power. When using the HC-SR04P model (as the wiring diagram below assumes), the maximum distance of the sensor is reduced slightly when using 3.3v power. For the purposes of this project, this shouldn't affect the results of the GarDoor-Car. Thus either model can be used, but it is recommended that you use the HC-SR04P powered using 3.3v.  

While a HC-SR04P works using 3.3v power without any other changes to the design (see the wiring diagram below), if using HC-SR04 sensors (which require you to connect them to the 5v VCC for power), a 1k Ω resistor will need to be used between the ECHO pin on each distance sensor and the related input pin on the NodeMCU. 

#### 4. 5v MicroUSB power supply
Power your NodeMCU via the same type of power supply used to charge Android phones or power a RaspberryPi. Powering the NodeMCU via MicroUSB is recommended since the relay module & DHT22 sensor can be powered via the NodeMCU VIN (or VU on the LoLin v3 variant) pin.

#### 5. Solderless breadboard (400 tie-point or larger)
The NodeMCU mounts to this breadboard nicely, leaving a few female ports next to each NodeMCU pin on both sides (important as the two ultrasonic sensors use the same trigger pin, and the relay and DHT22 sensor both use the 5v VIN power pin) making it easy to use male-to-female jumper wires to make connections from the NodeMCU to the relay module and the three sensors. This makes for a clean and solderless installation. Finally, these breadboards often also have an adhesive backing, making mounting in your project box easy.

You could also use male-to-male jumper wires to connect 5v/ground and/or 3.3v/ground NodeMCU pins to each set of outside power rails on the breadboard, and then connect the male end of the male-to-female jumper wires for a relay/sensor's power and ground to a + (for power) or - (for ground) port on the appropriate 3.3V or 5V rail to provide power/ground to components using the rails.  

You may also choose to mount the DHT22 and/or relay directly on the breadboard by inserting the pins vertically into ports above the NodeMCU. If do this you will require male-to-male jumper wires to connect the 3 ports of the DHT22 and 3 ports of the relay to the appropriate ports on the breadboard to link to the NodeMCU.  

#### 6. A DHT22 module (optional)
This sensor provides temperature/humidity readings to GarDoor-Car, but is optional and can be disabled in the auth.h file. The module verion (which is mounted on a board) is recommended, since the board includes a pull-up resister to reduce the output to 3.3v (the DHT22 is powered using 5v). If you use a DHT22 sensor (without a module board), you will need to use a 10k Ω resistor wired to the power input and the data pin of the DHT22 to act as a pull-up.   

#### 7., 8., & 9. Miscellaneous parts
To install GarDoor-Car, you will also require:

- Low voltage/current two-conductor electrical wire of the appropriate length to make connections from the relay where GarDoor-Car is mounted to the garage door opener.
- Male-to-female breadboard jumper wires to make connections from the NodeMCU to the relay module (3 wires required), the DHT22 (3 wires required), and the two ultrasonic sensors (4 wires required for each). You may also require male-to-male jumper wires if you choose to use the outside power fails or insert the DHT22 or relay's pins vertically on the breadboard. 
- A project box to hold the NodeMCU, the relay module, the DHT22 sensor module (if required), and the two ultrasonic sensors. While the ultrasonic sensors can be mounted inside the same box as the nodeMCU, they will need to have unobstructed access outside the box to point at the garage door and car.  

#### Building GarDoor-Car

See the wiring diagram below for an example of how the pins are wired up. 

- Attach your NodeMCU to the middle (between the outside power rails) of the solderless breadboard at one end; the MicroUSB connector should be facing one of the short outside edges of the breadboard (the power rails are on the longer outside edges). Ensure that there are at least two female ports next to each NodeMCU pin on each side of the NodeMCU.
- Mount the solderless breadboard in your project box.
- Mount the relay module in your project box.
- Mount the DHT22 module in your project box (if required). You could insert lengthways the DHT22's three pins into the breadboard above the position of the NodeMCU so that the DHT22 stands up on top of the breadboard. If you do this then you will use male-to-male jumper wires plugged into ports on the breadboard to connect the DHT22's pins to the NodeMCU.  
- Label your two HC-SR04P sensors as sensor 1 (door) and sensor 2 (car). Mount the two HC-SR04P sensors in your project box, ensuring there are holes for the front of the sensors to "see" the outside. Ensure that the direction you mount sensor 1 in will enable it to "see" the garage door once you position the box in its intended location in the garage. Also ensure that sensor 2 will be able to "see" the position of the car when the box is in the garage. You could mount the box above the garage floor or on the side of the garage wall.     
- Plug a jumper wire from VCC / + on the relay module to VIN / VU port on the NodeMCU (or the 5v + power rail).
- Plug a jumper wire from GND / - on the relay module to GND port on the NodeMCU (or the 5v - power rail).
- Plug a jumper wire from S on the relay module to D6 port on the NodeMCU (or Arduino/ESP8266 GPI012).
- Plug a jumper wire from + on the DHT22 module to VIN / VU port on the NodeMCU (or the 5v + power rail).
- Plug a jumper wire from - on the DHT22 module to GND port on the NodeMCU (or the 5v - power rail).
- Plug a jumper wire from Data on the DHT22 module to D7 port on the NodeMCU (or Arduino/ESP8266 GPI013).
- Plug a jumper wire from VCC on the HC-SR04P sensor 1 module to 3.3V port on the NodeMCU (or the 3.3v + power rail).
- Plug a jumper wire from GND on the HC-SR04P sensor 1 to GND port on the NodeMCU (or the 3.3v - power rail).
- Plug a jumper wire from Trig on the HC-SR04P sensor 1 to D5 port on the NodeMCU (or Arduino/ESP8266 GPI014).
- Plug a jumper wire from Echo on the HC-SR04P sensor 1 to D2 port on the NodeMCU (or Arduino/ESP8266 GPI04).
- Plug a jumper wire from VCC on the HC-SR04P sensor 2 module to 3.3V port on the NodeMCU (or the 3.3v + power rail).
- Plug a jumper wire from GND on the HC-SR04P sensor 2 to GND port on the NodeMCU (or the 3.3v - power rail).
- Plug a jumper wire from Trig on the HC-SR04P sensor 2 to D5 port on the NodeMCU (or Arduino/ESP8266 GPI014).
- Plug a jumper wire from Echo on the HC-SR04P sensor 2 to D1 port on the NodeMCU (or Arduino/ESP8266 GPI05).

Done!

#### Wiring Diagram
![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/master/Wiring%20Diagram-RollerDoor.png?raw=true "Wiring Diagram")

### Software

#### 1. Set up the Arduino IDE

You will modify the configuration parameters and upload the sketch to the NodeMCU with the Arduino IDE.

1. Download the Arduino IDE for your platform from [here](https://www.arduino.cc/en/Main/Software) and install it.
2. Add support for the ESP8266 to the Arduino IDE by following the instructions under "Installing with Boards Manager" [here](https://github.com/esp8266/arduino).
3. Add the "PubSubClient", "NewPing", and "Blynk" libraries to the Arduino IDE: follow the instructions for using the Library Manager [here](https://www.arduino.cc/en/Guide/Libraries#toc3), and search for and install the PubSubClient, NewPing, and Blynk libraries.
4. You may need to install a driver for the NodeMCU for your OS - Google for instructions for your specific microcontroller and platform, install the driver if necessary, and restart the Arduino IDE.
5. Select your board from Tools - Boards in the Arduino IDE (e.g. "Generic ESP8266 Module").

#### 2. Install Blynk App and Project and get Token

- Install the Blynk app on your phone by finding the Blynk app in your app store ([Android](https://play.google.com/store/apps/details?id=cc.blynk&hl=en_US) [Apple](https://apps.apple.com/us/app/blynk-iot-for-arduino-esp32/id808760481) and installing it.
- Run the Blynk app, and create an account (if you don't already have one). 
- In the Blynk app find the menu option to scan a QR code, click it and then scan the QR code that’s located [here](https://github.com/OpenGarage/OpenGarage-Firmware/blob/master/OGBlynkApp/og_blynk_1.1.png) (OpenGarage Project for Blynk App). If you have just created a new Blynk account you will have enough energy credits to use the project in the app for free. If you already have other project(s) in your Blynk account, you may need to purchase further energy credits to use the OpenGarage project in the app.  
- Once the project is scanned, go to the Project Settings, and copy or email the Blynk authorisation token to yourself. This is the token you will need to put into the auth.h file in step 3 below.
-You can configure the project in the app, such as changing the name of the button, but DO NOT change the virtual pin numbers for any button or widget (V1, V2 etc).  

*NOTE: You can logon to the same Blynk account on more than one phone. If you do this on other family members phones, they can use the same project in the app to control the same GarDoor-Car. 

#### 3. Load the sketch in the Arduino IDE and modify the user parameters in auth.h

Download the files from this repo to your computer, and open the GarDoor-Car-Vx.xx.ino file from the folder. GarDoor-Car's configuration parameters are found in auth.h. Select the auth.h tab in the Arduino IDE. This section describes the configuration parameters and their permitted values.

*IMPORTANT: No modification of the sketch code in GarDoor-Car-Vx.xx.ino is necessary (or advised, unless you are confident you know what you are doing and are prepared for things to break unexpectedly).*

**Wifi Settings**

*NOTE: An available IPv4 network with a DHCP for IPv4 service is assumed* 

WIFI_HOSTNAME "your-device-name"

The wifi hostname GarDoor-Car will use. Should be unique among all the devices connected to your network. Must be placed within quotation marks. (Default: GarDoor)

WIFI_SSID "your-wifi-ssid"

The wifi ssid GarDoor-Car will connect to. Must be placed within quotation marks.

WIFI_PASSWORD "your-wifi-password"

The wifi ssid's password. Must be placed within quotation marks.

**MQTT Settings**

MQTT_SERVER "w.x.y.z"

The IPv4 address of your MQTT broker. Must be placed within quotation marks.

MQTT_USER "your-mqtt-username"

The username required to authenticate to your MQTT broker. Must be placed within quotation marks. Use "" (i.e. a pair of quotation marks) if your broker does not require authentication.

MQTT_PASSWORD "your-mqtt-password"

The password required to authenticate to your MQTT broker. Must be placed within quotation marks. Use "" (i.e. a pair of quotation marks) if your broker does not require authentication.

MQTT_PORT your-mqtt-server-port-number

The port on your MQTT server GarDoor-Car will use to connect to the MQTT service. Must be a number. (Default: 1883)

MQTT_AVAIL_TOPIC WIFI_HOSTNAME "mqtt-topic-for-gardoor-availability"

Name of the topic that the MQTT broker will host and HA will use to determine if GarDoor-Car is online or offline. Must be placed within quotation marks. (Default: /availability)

TimeReadSYS milliseconds

Number of milliseconds (1000 per second) between which system readings of ESP8266 data will be taken and published using MQTT. Must be a number. (Default: 100000)

topicESPtoMQTT WIFI_HOSTNAME "mqtt-topic-for-esp8266-data"

Name of the topic that the MQTT broker will host and HA will use to get running data about your ESP8266. Must be placed within quotation marks. (Default: /ESPtoMQTT)

**OTA Settings**

OTApassword "your-ota-password"

The password you will need to enter to upload remotely via the ArduinoIDE. Must be placed within quotation marks. (Default: ota password)

OTAport your-esp8266-device-port-number

The port you will need to enter into the ArduinoIDE to allow it to connect to your running ESP8266. Must be a number. (Default: 8266)

**Blynk App Settings**

BLYNK_ENABLED true

Set to true to allow you to use the Blynk app. Set to false to disable use of Blynk. (Default: true)

BlynkAuthToken "your-blynk-project-token"

The Blynk token from the project in the Blynk app (see step 2 above). Must be placed within quotation marks.  

BLYNK_NOTIFY true

Set to true to enable notifications via the Blynk app. Set to false to disable Blynk notifications. (Default: true)

BLYNK_DEFAULT_DOMAIN "blynk-server-address-or-name"

The name or IP address of the Blynk server the Blynk app/GarDoor-Car connects to. Only change this if you have setup a local Blynk server. Must be placed within quotation marks. (Default: blynk-cloud.com)  

BLYNK_DEFAULT_PORT blynk-server-port-number

The port the Blynk app/GarDoor-Car uses to connect to the Blynk server. Only change this if you have setup a local Blynk server. Must be a number. (Default: 80) 

BLYNK__AVAIL_TOPIC WIFI_HOSTNAME "mqtt-topic-for-blynk-availability"

Name of the topic that the MQTT broker will host and HA will use to determine if GarDoor-Car is connected to the Blynk server. Must be placed within quotation marks. (Default: /blynkavailability)

**Distance Parameters**

ULTRASONIC_MAX_DISTANCE number-in-cm

Maximum distance (in cm) to ping. If using HC-SR04P sensors should be 400. If using HC-SR04 sensors, should be 450. Must be a number. (Default: 400)  

ULTRASONIC_DIST_MAX_CLOSE number-in-cm

Maximum distance (in cm) to indicate that the garage door is closed (used by sensor 1). If the measured distance to the object is less than this, then the door is published as closed.If the measured distance to the object is more than this, then the door is published as open. Must be a number. (Default: 120)   

ULTRASONIC_DIST_MAX_CAR number-in-cm

Maximum distance (in cm) to indicate that the car is present (used by sensor 2). If the measured distance to the object is less than this, then the car is present. If the measured distance to the object is more than this, then the car is absent. Must be a number. (Default: 120)   

ULTRASONIC_SETTLE_TIMEOUT milliseconds 

Number of milliseconds (1000 per second) to wait between pings (as both sensors get triggered at the same time). Waiting between pings improves readings for each sensor. Must be a number. (Default: 500)

RELAY_ACTIVE_TIMEOUT milliseconds

Number of milliseconds (1000 per second) the relay will close to actuate the door opener.  While it is "closed" the NO connector on the relay will allow current to flow, while the NC connector will cut the flow of current. Must be a number. (Default: 500)

RELAY_ACTIVE_TYPE HIGH

Set to LOW if using an active-low relay module. Set to HIGH if using an active-high relay module. (Default: HIGH)

DOOR_TRIG_PIN 8266_pin_identifier

The ESP8266 GPIO pin that triggers the two ultrasonic sensors to determine the distance to an object and output the results on their Echo pins. Must be a number. (Default: 14)  (which is D5 on the NodeMCU)

**Door 1 Parameters**

DOOR1_ALIAS "name"

The alias to be used for Door 1 (garage door) in serial messages, MQTT topic and webpage. Must be placed within quotation marks. (Default: Door)

MQTT_DOOR1_ACTION_TOPIC WIFI_HOSTNAME "mqtt-topic"

The MQTT broker topic GarDoor-Car will subscribe to for action commands for Door 1 (garage door). Must be placed within quotation marks. (Default: /1/action)

MQTT_DOOR1_STATUS_TOPIC WIFI_HOSTNAME "mqtt-topic"

The Mqtt broker topic GarDoor-Car will publish Door 1's (garage door) status to. Must be placed within quotation marks. (Default: /1/status)






DOOR1_OPEN_PIN D2

The GPIO pin connected to the relay that is connected to Door 1's garage door opener's open terminals. (Default: NodeMCU D2 / Arduino 4)

DOOR1_CLOSE_PIN D2

The GPIO pin connected to the relay that is connected to Door 1's garage door opener's close terminals. If your garage door opener is like most (all?), the same terminals control open and close via a momentary connection of the terminals. In this case, set DOOR1_CLOSE_PIN and DOOR1_OPEN_PIN to the same pin. (Default: NodeMCU D2 / Arduino 4)

DOOR1_STATUS_PIN D5

The GPIO pin connected to the reed/magnetic switch attached to Door 1. (Default: NodeMCU D5 / Arduino 14)

DOOR1_STATUS_SWITCH_LOGIC "NO"

The type of reed/magnetic switch used for Door 1. Must be placed within quotation marks. Set to "NO for normally-open. Set to "NC" for normally-closed. (Default: NO)

**Car Parameters**

DOOR2_ENABLED false

Set to true to enable GarHAge to control/monitor Door 2. Set to false to disable Door 2. (Default: false)

DOOR2_ALIAS "Door 2"

The alias to be used for Door 2 in serial messages. Must be placed within quotation marks. (Default: Door 2)

MQTT_DOOR2_ACTION_TOPIC "garage/door/2/action"

The topic GarHAge will subscribe to for action commands for Door 2. Must be placed within quotation marks. (Default: garage/door/2/action)

MQTT_DOOR2_STATUS_TOPIC "garage/door/2/status"

The topic GarHAge will publish Door 2's status to. Must be placed within quotation marks. (Default: garage/door/2/status)

DOOR2_OPEN_PIN D1

The GPIO pin connected to the relay that is connected to Door 2's garage door opener's open terminals. (Default: NodeMCU D1 / Arduino 5)

DOOR2_CLOSE_PIN D1

The GPIO pin connected to the relay that is connected to Door 2's garage door opener's close terminals. If your garage door opener is like most (all?), the same terminals control open and close via a momentary connection of the terminals. In this case, set DOOR2_CLOSE_PIN and DOOR2_OPEN_PIN to the same pin. (Default: NodeMCU D1 / Arduino 5)

DOOR2_STATUS_PIN D6

The GPIO pin connected to the reed/magnetic switch attached to Door 2. (Default: NodeMCU D6 / Arduino 12)

DOOR2_STATUS_SWITCH_LOGIC "NO"

The type of reed/magnetic switch used for Door 2. Must be placed within quotation marks. Set to "NO for normally-open. Set to "NC" for normally-closed. (Default: NO)


**DHT Parameters**

 
 
 
 
 
  

#### 4. Upload the sketch to your NodeMCU / microcontroller

If using the NodeMCU, connect it to your computer via MicroUSB; press and hold the reset button on the NodeMCU, press and hold the Flash button on the NodeMCU, then release the Reset button. Select Sketch - Upload in the Arduino IDE.

If using a different ESP8266 microcontroller, follow that device's instructions for putting it into flashing/programming mode.

#### 5. Check the Arduino IDE Serial Monitor

Open the Serial Monitor via Tools - Serial Monitor. Reset your microcontroller. If all is working correctly, you should see something similar to the following messages:

Starting GarHAge...
Relay mode: Active-High

Connecting to your-wifi-ssid.. WiFi connected - IP address: 192.168.1.100
Attempting MQTT connection...Connected!
Publishing birth message "online" to GarHAge/availability...
Subscribing to garage/door/1/action...
Subscribing to garage/door/2/action...
Door 1 closed! Publishing to garage/door/1/status...
Door 2 closed! Publishing to garage/door/2/status...

If you receive these (or similar) messages, all appears to be working correctly. 

#### 6. Check Blynk App

Make sure the Blynk project is running (in the project there is a triangle-shaped Run button). You should get a distance reading in the app and be able to click the button to activate the relay. 

#### 7. Reserve an IPv4 address on your DHCP service

Logon onto your router (or device/server) that runs your DHCP service. Reserve the IP address of the GarDoor-Car device (also known as a DHCP static binding). This ensures that GarDoor-Car always uses the same IP address when it reboots. This makes it easier to open the webpage or upload a new sketch using Arduino OTA.  

Disconnect GarHAge from your computer and prepare to install in your garage.


#### OTA Uploading
This code also supports remote uploading to the ESP8266 using Arduino's OTA library. To utilize this, you'll need to first upload the sketch using the traditional USB method. However, if you need to update your code after that, your WIFI-connected ESP chip should show up as an option under Tools -> Port -> 'HostName'at your.ip.address.xxx. 

More information on OTA uploading can be found [here](http://esp8266.github.io/Arduino/versions/2.0.0/doc/ota_updates/ota_updates.html). 


#### Web page examples
![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus1.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus2.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus3.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus4.png?raw=true "Webpage Status")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/webpagestatus5.png?raw=true "Webpage Status")

#### Example Home Assistant Configuration 

[YAML Configuration](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/Example%20Home%20Assistant%20Configuration.yaml)

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/HA-entities1.png?raw=true "HA Example")

![alt text](https://github.com/SmbKiwi/GarDoor-Car/blob/v2.00R/HA-entities2.png?raw=true "HA Example")


#### Sample MQTT commands
Listen to MQTT commands
> mosquitto_sub -h 172.17.0.1 -t '#'

Open the garage door
> mosquitto_pub -h 172.17.0.1 -t GarDoor/1/action -m "OPEN"

