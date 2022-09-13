% Intro to ESPHome
% https://github.com/rtward/ESPHome-Talk
% ![Link to Talk](images/qr-link.png) 

# What is ESPHome?

::: notes

 - ESPHome is a configurable operating system for programming ESP based microcontrollers.
 - ESPHome allows for easy configuration of microcontrollers to perform many different automation tasks.
 - You can configure the controller to respond to inputs in many different ways.
 - Can work local only
 - My goal here is to convince you to use ESPHome for your next project instead of doing custom coding if you can avoid it.

:::

# What devices does it support?

::: notes

 - All sorts of things!
 - Switches, light/motion sensors, bluetooth inputs, air quality, temperature, motion...
 - If you can connect it with a wire, there's probably a way to attach it to an ESPHome controller

:::

# What can I do with it?

::: notes

 - Watch for bluetooth devices to come into range
 - Watch the current on a device
 - Read prox cards and unlock doors

:::

# Installation

::: notes

 - There are a few options for installing ESPHome on your main system, in order to program microcontrollers

:::

## Native

``` { .bash .numberLines }
# Ubuntu
apt install esphome

# Arch
pacman -S esphome

# OSX
brew install esphome
```

::: notes

 - The native option is probably the least used, but it is available and some people prefer it.
 - You'll install the esphome command line client from your operating system's repos

:::

## Docker

``` { .bash .numberLines }
docker pull esphome/esphome
```

::: notes

 - This is the most common option. 
 - It has the advantage of not having to worry about dependencies.
 - You will need to explicity pass through devices for local programming however.

:::

## HomeAssistant

Install the ESPHome Addon

::: notes

 - If you're going to be integrating ESPHome into a HomeAssistant setup, this can be an easy option.
 - We won't be covering this today, it's not as useful for initial programming, but can be an easy way to keep devices up to date.

:::

# Running

``` { .bash .numberLines }
esphome ...

docker run --rm -v "${PWD}":/config -it esphome/esphome ...
```

::: notes

 - Once you've installed esphome, you can run it on the command line.
 - If you installed it natively, just run the command esphome.
 - If you installed it using docker, you'll need a little bit extra, but otherwise the commands will be the same

:::

# Programming

## Wizard

``` { .bash .numberLines }
esphome wizard my-project.yaml
```

::: notes

 - The wizard is an easy way to interactively create a base config file.
 - It'll ask you questions on the command line and you'll type the answers.
 - It'll then write out a base config file you can use for further editing.

:::

## Config File

`my-project.yaml`

``` { .yaml .numberLines }
esphome:
  name: widget
  platform: ESP8266
  board: nodemcu

wifi:
  ssid: home-wifi
  password: home-wifi-password
  ap:
    ssid: fallback-wifi
    password: fallback-wifi-password
```

::: notes

 - Once you have your base config file, it'll look something like this.
 - The ESPHome section is the basic configuration of the controller, it's name, and the type of hardware
 - The WiFi section sets up both the wifi the controller will try to connectt to, as well as the fallback network it will create when it can't find the primary network.

:::

## Via Cable

``` { .bash .numberLines }
esphome run my-project.yaml

docker run --rm -it \
  -v "${PWD}":/config \
  --device=/dev/ttyUSB0 \
  esphome/esphome run my-project.yaml
```

::: notes

 - For your initial programming, you'll probably connect the ESP directly with a USB cable.
 - Once you run the above commands, it'll notice the ESP connected via USB and give you the option to program it.

:::

## OTA

``` { .bash .numberLines }
esphome run my-project.yaml
```

::: notes

 - After the initial programming, the easiest way to update your controllers is OTA (over the air)
 - When you run this command without an ESP connected, it'll ask you if you want to update over the air.
 - As long as the device you're working from and the controller are connected to the same network, it should find it and update it seamlessly.

:::

## Logs 

``` { .bash .numberLines }
esphome logs my-project.yaml
```

::: notes

 - If you're trying to debug one of your controllers, youc an run the logs command.
 - This will print out the logs from the device on your computer and allow you to see what's going on with the controller in real time.

:::

# Syntax

::: notes

 - So now you've got your controller where you can program it, and start working on adding functionality
 - I'm going to talk about some of the syntax next

:::

## Triggers

::: notes

 - Triggers are anything that causes an action to happen on the controller, whether it's a switch closing, a bluetooth device coming in range, a timer, boot up, whatever.
 - I'm only going to talk about a few of them, just to give you an idea.  There are many more available in the ESPHome docs

:::

---

``` { .yaml .numberLines }
esphome:
  on_boot:
    then:
      - logger.log: "Hello World"
```

::: notes

 - On boot is just like it says, the automation runs once when the controller starts up.
 - This can be very useful for setting default and resetting the system to some known good state.
 - There are also on_shutdown and other lifecycle events available.

:::

---

``` { .yaml .numberLines }
sensor:
  - platform: dht
    humidity:
    name: "Temperature"
    id: temperature_sensor
    on_value:
      then:
        - logger.log:
          format: "Temp %.1f"
          args: ['id(temperature_sensor).state']
```

::: notes

- The on_value trigger will execute whenever a sensor has a new value.
- See also the on_value_range trigger to execute an automation when a sensor gets within a certain range of values.

:::

---

``` { .yaml .numberLines }
binary_sensor:
  - platform: gpio
    on_press:
      then:
        - logger.log: "Hello World"
```

::: notes

 - The on_press trigger is very useful for buttons attached to the GPIO pins of the controller.  Can also be used for any type of mechanical switch that is on or off.
 - See also on_release, on_click, on_multiclick

:::

---

``` { .yaml .numberLines }
time:
  - platform: sntp
    on_time:
      - seconds: 0
        minutes: /5
        then:
          - logger.log: "Five Minute Timer"

      - seconds: 0
        hours: 7
        days_of_week: MON-FRI
        then:
          - logger.log: "Weekday Alarm"
```

::: notes

 - Timer triggers are useful for causing automations to happen at certain times.

:::

## Actions

::: notes

 - Actions are the thing you do in response to a trigger.
 - Sometimes they can be simple, and as we'll see later, sometime they can get very complicated.

:::

---

``` { .yaml .numberLines }
binary_sensor:
  - platform: gpio
    on_press:
      then:
        - logger.log: "Button Pressed"
```

::: notes

 - You've already seen the logger, this just writes out notes into the system log which can be viewed with the command we talked about earlier.
 - You'll mostly want to use this in combination with other actions

:::

---

``` { .yaml .numberLines }
output:
  - platform: gpio
    pin: D1
    id: relay_1

binary_sensor:
  - platform: gpio
    on_press:
      then:
        - logger.log: "Button Pressed"
        - output.turn_on: relay_1
```

::: notes

 - You can use the turn on / off actions to switch external devices on and off.

:::

---

``` { .yaml .numberLines }
binary_sensor:
  - platform: gpio
    on_press:
      then:
        - logger.log: "Button Pressed"
        - http_request.get:
          url: https://esphome.io
          verify_ssl: false
```

::: notes

 - You can use the HTTP request module to interact with external servers during actions

:::

---

``` { .yaml .numberLines }
binary_sensor:
  - platform: gpio
    on_press:
      then:
        - logger.log: "Button Pressed"
        - lambda: |-
            id(some_binary_sensor).publish_state(false);

```

::: notes

 - Lambdas are the most complicated action, but also the most powerful.
 - Not going to get too into the syntax, as in my experience it's not needed most of the time.
 - You can write C code which will be executed as your action, while having full access to the state and info about the trigger event.
 - I'd advise you to avoid making lambdas a go-to if you can avoid it, as they're inherently much harder to maintain and work on than a normal YAML action.

:::

## Conditions

::: notes

 - Conditions are where you can really take your actions to the next level.

:::

---

``` { .yaml .numberLines }
sensor:
  - platform: dht
    humidity:
    name: "Temperature"
    id: temperature_sensor
    on_value_range:
      - below: 70
        then:
          - if:
            condition:
              binary_sensor.is_on: motion_sensor
            then:
              - logger.log: "Turning on heater"
              - switch.turn_on: heater
```

::: notes

 - The If action allows you to use a condition to optionally execute one action or another depending on the condition.

:::

---

``` { .yaml .numberLines }
sensor:
  - platform: dht
    humidity:
    name: "Temperature"
    id: temperature_sensor
    on_value_range:
      - below: 70
        then:
          - switch.turn_on: heater
          - wait_until:
            condition:
              sensor.in_range:
                id: temperature_sensor
                above: 73
            then:
              - switch.turn_off: heater
```

::: notes

 - The wait_until action allows you to wait for a condition before continuing your action
 - You can see the sensor.in_range condition which allows you to execute an action only if another sensor is within some given values.

:::

---

``` { .yaml .numberLines }
binary_sensor:
  - platform: gpio
    on_press:
      then:
        - if:
          condition:
            switch.is_on: light
          then:
            - logger.log: "Button Pressed"
            - switch.turn_off: light
```

::: notes

 - The switch condition alows you to check the state of a switch before continuing with your actions

:::

---

``` { .yaml .numberLines }
binary_sensor:
  - platform: gpio
    on_press:
      then:
        - if:
          condition:
            lambda: |-
              return id(some_sensor).state < 30;
          then:
            - logger.log: "Button Pressed"
```

::: notes

 - Like the lambda action, the lambda condition is a do-anything option when the other conditions won't work.

:::

## Scripts

``` { .yaml .numberLines }
script:
  - id: blink_light
    then:
      - repeat:
        times: 5
        then: 
        - switch.turn_on: my_switch
        - delay: 1s
        - switch.turn_off: my_switch
        - delay: 1s

binary_sensor:
  - platform: gpio
    on_press:
      then:
        script.execute: blink_light
```

::: notes

 - Scripts are reusable actions which you can apply in multiple places.
 - Think of them like functions if you've used other programming languages.

:::

# Adding Functionality

::: notes

 - We've talked about how to program ESP controllers, but what can we actually connect to them?

:::

## Bluetooth Tracker

``` { .yaml .numberLines }
esp32_ble_tracker:

sensor:
  - platform: template
    name: "Trash Bin In Garage"
    id: trash_bin_in_garage
  - platform: ble_rssi
    mac_address: AA:BB:CC:DD:EE:FF
    name: "Trash Bin"
    id: trash_bin
    on_value_range:
      - above: 200
        then:
        - sensor.template.publish:
          id: trash_bin_in_garage
          state: 1
```

::: notes

 - This is a project I've been working on lately.  Using BT trackers to know if my garbage cans are in the garage.

:::

## Air Quality

``` { .yaml .numberLines }
uart:
  rx_pin: D2
  baud_rate: 9600

sensor:
  - platform: pm1006
    pm_2_5:
      name: "Kitchen Air Quality"
```

::: notes

 - This is an air quality sensor based on the IKEA VINDRIKTNING

:::

## Plant Sensor

``` { .yaml .numberLines }
sensor:
  - platform: xiaomi_hhccjcy01
    mac_address: AA:BB:CC:DD:EE:FF
    temperature:
      name: "Jade Plant Temperature"
    moisture:
      name: "Jade Plant Moisture"
    illuminance:
      name: "Jade Plant Light"
    conductivity:
      name: "Jade Plant Conductivity"
    battery_level:
      name: "Jade Plant Battery Level"
```

::: notes

 - This is an integration with a common bluetooth plant sensor available on Amazon and AliExpress called MiFlora

:::

## Current Clamp

``` { .yaml .numberLines }
sensor:
  - platform: adc
    pin: A2
    id: adc_sensor

  - platform: ct_clamp
    sensor: adc_sensor
    name: "Washing Machine Current"
    update_interval: 60s
```

::: notes

 - This is an integration with a current clamp that can be used to tell how much power a device is drawing. 
 - The top sensor is the analog to digital current input.
 - That input is converted into current by the bottom sensor

:::

## Temperature and Humidity

``` { .yaml .numberLines }
sensor:
  - platform: aht10
    temperature:
      name: "Attic Temperature"
    humidity:
      name: "Attic Humidity"
    update_interval: 60s
```

::: notes

 - There are several off the shelf temperature sensors you can use for measuring temperature and humidity.

:::

## GPIO

``` { .yaml .numberLines }
binary_sensor:
  - platform: gpio
    pin:
      number: D2
      mode:
        input: true
        pullup: true
    name: toggle_switch

switch:
  - platform: gpio
    pin: 25
    name: relay
```

::: notes

 - The GPIO ports are the swiss army knife of the ESP device.
 - Any sort of swtich can be used.

:::

## Display

``` { .yaml .numberLines }
i2c:
  sda: D1
  scl: D2

display:
  - platform: ssd1306_i2c
    model: SSD1306 128x64
    address: 60
    update_interval: 1s
    lambda: >-
      it.fill(COLOR_OFF);
      it.print(0, 0, id(hack), id(yellow), id(lockout_name).c_str());
```

::: notes

 - ESPHome supports a number of different LED and LCD displays.
 - It has a basic graphics toolkit for writing text to the screen.

:::

# Integrating

::: notes

 - While ESPHome is very powerful on its own, it's mostly used as part of larger systems.
 - Home Assistant is the most common one, although anything that speaks MQTT can be made to integrate.

:::

## MQTT

``` { .yaml .numberLines }
mqtt:
  broker: 192.168.1.201
  username: widget
  password: mqtt-password

mqtt:
  on_message:
    topic: my/custom/topic
    qos: 0
    then:
      - logger.log: "Hello World"

binary_sensor:
  - platform: gpio
    on_press:
      then:
        - mqtt.publish:
          topic: my/custom/topic
          payload: "Button Pressed"
```

::: notes

 - MQTT is a message passing protocol that's a lightway way for many different systems to communicate with each other.

:::

## Home Assistant

``` { .yaml .numberLines }
api:
```

::: notes

 - Home Assistant is the most common integration for ESPHome.
 - All you need to do is add this line to your config and it'll automatically get picked up in HA.
 - Sensors will be published to HA, and any switches or other outputs will be accessible as well.

:::

# Examples

## Trash Bin Sensor

``` { .yaml .numberLines }
esphome:
  name: kitchen-air-quality
  platform: ESP8266
  board: nodemcu
  on_boot:
    then:
      - script.execute: turn_off

api: 

wifi:
  ssid: ############
  password: ##############
  ap:
    ssid: kitchen-air-quality-fallback 
    password: ##############

esp32_ble_tracker:

sensor:
  - platform: template
    name: "Trash Bin In Garage"
    id: trash_bin_in_garage
  - platform: ble_rssi
    mac_address: AA:BB:CC:DD:EE:FF
    name: "Trash Bin"
    id: trash_bin
    on_value_range:
      - above: 200
        then:
        - sensor.template.publish:
          id: trash_bin_in_garage
          state: 1
```

## Air Quality Meter

``` { .yaml .numberLines }
esphome:
  name: kitchen-air-quality
  platform: ESP8266
  board: nodemcu
  on_boot:
    then:
      - script.execute: turn_off

api: 

wifi:
  ssid: ############
  password: ##############
  ap:
    ssid: kitchen-air-quality-fallback 
    password: ##############

uart:
  rx_pin: D2
  baud_rate: 9600

sensor:
  - platform: aht10
    temperature:
      name: "Kitchen Temperature"
    humidity:
      name: "Kitchen Humidity"
    update_interval: 60s
  - platform: pm1006
    pm_2_5:
      name: "Kitchen Air Quality"
```

## Arch Reactor Lockouts

``` { .yaml .numberLines }
esphome:
  name: lockout-backdoor
  platform: ESP8266
  board: nodemcu
  on_boot:
    then:
      - script.execute: turn_off

api: null

wifi:
  ssid: ARMembers
  password: ##############
  ap:
    ssid: lockout-backdoor-fallback
    password: ##############
  use_address: 1.1.1.1

globals:
  - id: lockout_name
    type: std::string
    restore_value: no
    initial_value: '"backdoor"'
  - id: allowed_card_ids
    type: std::vector<std::string>
    restore_value: no
    initial_value: '{ "aaaaa", "bbbbb", "cccccc" }'
  - id: allowed_names
    type: std::vector<std::string>
    restore_value: no
    initial_value: '{ "John", "Jane", "Joe" }'
  - id: active_time
    type: uint32_t
    restore_value: no
    initial_value: "5"
  - id: last_scanned_id
    type: std::string
    restore_value: no
    initial_value: '""'
  - id: last_scanned_name
    type: std::string
    restore_value: no
    initial_value: '""'
  - id: expiration_time
    type: uint32_t
    restore_value: no
    initial_value: "0"

script:
  - id: turn_on
    then:
      - lambda: |-
          id(expiration_time) = id(active_time);
          id(relay).turn_on();
  - id: turn_off
    then:
      - lambda: id(relay).turn_off();

interval:
  - interval: 1s
    then:
      - lambda: |-
          if (id(expiration_time) > 0) {
              id(expiration_time) -= 1;
              if (id(expiration_time) == 0) {
                  id(turn_off).execute();
              }
          }

captive_portal: null

logger: null

ota: null

font:
  - id: hack
    file: Hack-Regular.ttf
    size: 15
  - id: hack10
    file: Hack-Regular.ttf
    size: 10
color:
  - id: yellow
    red: 50%
    green: 50%
    blue: 0%
    white: 0%
  - id: red
    red: 100%
    green: 0%
    blue: 0%
    white: 0%
  - id: green
    red: 0%
    green: 100%
    blue: 0%
    white: 0%

switch:
  - platform: gpio
    pin: D0
    id: relay

i2c:
  sda: D2
  scl: D1

display:
  - platform: ssd1306_i2c
    model: SSD1306 128x64
    address: 60
    update_interval: 1s
    lambda: >-
      it.fill(COLOR_OFF);


      it.print(0, 0, id(hack), id(yellow), id(lockout_name).c_str());


      if (id(expiration_time) > 0) {
          it.print(0, 15, id(hack10), id(green), "Unlocked");
          it.print(0, 25, id(hack10), id(green), to_string(id(expiration_time)).c_str());
          it.print(25, 25, id(hack10), id(green), "Seconds Left");
      } else {
          it.print(0, 15, id(hack), id(red), "Locked");
      }


      it.print(0, 45, id(hack10), id(last_scanned_id).c_str());

      it.print(40, 45, id(hack10), id(last_scanned_name).c_str());

wiegand_reader:
  d0_pin: 12
  d1_pin: 14
  on_tag:
    lambda: |-
      ESP_LOGD("reader", "Read tag %s", x.c_str());

      id(last_scanned_id) = x;
      id(last_scanned_name) = "Not Allowed";

      bool found = false;
      for (int i = 0; i < id(allowed_card_ids).size(); i++) {
          if (id(allowed_card_ids)[i].compare(x) == 0) {
              id(last_scanned_name) = id(allowed_names)[i];
              id(turn_on).execute();
              found = true;
              ESP_LOGD(
                  "reader",
                  "allowed tag found matching %s: %s",
                  x.c_str(),
                  id(last_scanned_name).c_str()
              );
              break;
          }
      }

      if (!found) {
          ESP_LOGD("reader", "no allowed tags found matching %s", x.c_str());
      }

```

# Questions

![Link to Talk](images/qr-link.png)

https://github.com/rtward/ESPHome-Talk

https://esphome.io/index.html
