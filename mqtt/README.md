# MQTT firmware for H801 LED dimmer

The is an alternative firmware for the H801 LED dimmer that uses MQTT as a control channel. This makes it easy to integrate into Home Assistant and other Home Automation applications.

## Flashing the H801

1. Open the case and solder 6 Jumers on the board. Four are needed for the serial connection (GND, 3.3V, RX und TX) and two are needed for the Jumper (J1 and J2) to enter the flash mode.
1. Download Arduino IDE (in my case 1.8.1) and prepare the IDE by opening the preferences window.
1. Enter ```http://arduino.esp8266.com/stable/package_esp8266com_index.json``` into additional board manager URLs field.
1. Open boards manager from tools > board menu and install esp8266 platform 
1. Install following new library using the library manager. ("Sketch" menu and then include library > Manage Libraries).
   1. WifiManager
   1. PubSubClient
   1. ArduinoOTA   
1. Connect the wires of your USB Serial adapter with the H801 (3.3V > 3.3V, GND > GND, RX > RX and TX > TX) and place a jumper or cable between J1 and J2
1. Connect the Serial USB adapter with your computer. The H801 will be powered with the 3.3V from the USB Serial adapter and enters the flash mode. 
1. Open the *.ino file 
1. Select following in the menu tools:
   1. Board: Generic ESP8266
   1. Flash Size: 1M (64K SPIFFS)
   1. Upload Speed: 115200
   1. Port: Select your COM Port of your Serial adapter e.g. COM5
1. Select Sketch > Upload
1. Once the the flash has finished unplug the Serial wires, remove the Jumper and attach a Power Supply on the right side labled with "VCC and GND".
1. The device will start and you should inf an ESP* Wifi, which you can connect to.
1. Open the page: 192.168.4.1 and enter all required values. (Wifi, password, MQTT Server user and password).
1. Please note here your ESP ID, you will need it later for. The below example ID is: 0085652A

Status information:
If you have entered the correct Wifi credentials and the H801 was able to connect to your wifi, it will blink with the green led 10 times
If the connection to the MQTT fails it will blink with the red LED 10 times. It will try to reconnect after 5sec.

## Channels

| Channel                          | Remark                                                                                         |
| -------------------------------- | ---------------------------------------------------------------------------------------------- |
| XXXXXXXX/rgb/light/status        | Dimmer send current status (ON/OFF)                                                            |
| XXXXXXXX/rgb/light/switch        | Set RGB channel ON or OFF                                                                      |
| XXXXXXXX/rgb/brightness/status   | Dimmer sends current brightness (this will be stored even when you switch the RGB channel off) |
| XXXXXXXX/rgb/brightness/set      | Set brigness (0-255)                                                                           |
| XXXXXXXX/rgb/rgb/status          | Dimmer reports the RGB value (3 values 0-255)                                                  |
| XXXXXXXX/rgb/rgb/set             | Set the RGB values (Format r,g,b) values 0-255 for each channel                                |
| XXXXXXXX/w[12]/light/status      | Status of W1/W2 channel                                                                        |
| XXXXXXXX/w[12]/light/switch      | Set W1/W2 ON or OFF                                                                            |
| XXXXXXXX/w[12]/brightness/status | Brightness of W1/W2 channel                                                                    |
| XXXXXXXX/w[12]/brightness/set    | Set brightness of W1/W2 channel                                                                |

## Home assistant example configuration

Replace 0085652A with your ESP ID.

```yaml
light:

- platform: mqtt
  name: "RGB"
  state_topic: "0085652A/rgb/light/status"
  command_topic: "0085652A/rgb/light/switch"
  brightness_state_topic: "0085652A/rgb/brightness/status"
  brightness_command_topic: "0085652A/rgb/brightness/set"
  rgb_state_topic: "0085652A/rgb/rgb/status"
  rgb_command_topic: "0085652A/rgb/rgb/set"
  state_value_template: "{{ value_json.state }}"
  brightness_value_template: "{{ value_json.brightness }}"
  rgb_value_template: "{{ value_json.rgb | join(',') }}"
  qos: 0
  payload_on: "ON"
  payload_off: "OFF"
  optimistic: false

- platform: mqtt
  name: "W1"
  state_topic: "0085652A/w1/light/status"
  command_topic: "0085652A/w1/light/switch"
  brightness_state_topic: "0085652A/w1/brightness/status"
  brightness_command_topic: "0085652A/w1/brightness/set"
  state_value_template: "{{ value_json.state }}"
  brightness_value_template: "{{ value_json.brightness }}"
  qos: 0
  payload_on: "ON"
  payload_off: "OFF"
  optimistic: false

- platform: mqtt
  name: "W2"
  state_topic: "0085652A/w2/light/status"
  command_topic: "0085652A/w2/light/switch"
  brightness_state_topic: "0085652A/w2/brightness/status"
  brightness_command_topic: "0085652A/w2/brightness/set"
  state_value_template: "{{ value_json.state }}"
  brightness_value_template: "{{ value_json.brightness }}"
  qos: 0
  payload_on: "ON"
  payload_off: "OFF"
  optimistic: false

```

