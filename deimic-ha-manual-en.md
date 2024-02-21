## Prerequisites
1. Deimic Master v3 module
1. Installed following applications (Windows operating system recommended)
   * [MQTT Explorer](https://github.com/thomasnordquist/MQTT-Explorer/releases)
   * [DEIMIC Configurator](https://www.deimic.pl/wsparcie,do-pobrania.html)
1. A configured instance of Home Assistant (HA OS version) on the local network (e.g. on a Raspberry Pi). Installation instructions can be found [here](https://www.home-assistant.io/installation/). 
    > We recommend installing HA OS, because the _container_ type installation does not allow installing add-ons (Studio Code Server).

## Introduction
Integration is possible via the MQTT protocol. The Deimic Master module announces each state change in a dedicated topic, which can be subscribed to by Home Assistant. The communication works both ways, so the Home Assistant can set the state of the device on the Master (turn on/off a light, open/close a curtain, etc.).

## Step 1 - add MQTT integration
1. Open the settings, go to the "Devices & services" tab, and then click "Add integration"
    ![Home Assistant - settings - add itegration 2](/assets/image.png)
    ![Home Assistant - settings - add itegration 2](/assets/image-1.png)
1. Then, search for "MQTT, select an integration from the list, and in the next window select "MQTT" again. 
    ![Home Assistant - settings - add MQTT](/assets/image-2.png)
    ![Home Assistant - settings - add MQTT](/assets/image-3.png)
1. In the next window, enter the IP address of the Deimic Master module (1), the port number, if different from the default (2), and confirm the options (3).
    ![Home Assistant - configure MQTT](/assets/image-5.png)

## Step 2 - installing Studio Code Server
This is an add-on to Home Assistant that will allow you to later edit configuration files directly from Home Assistant.

1. Go to settings (1) and select "Extras" (2)
    ![Home Assistant - addons](/assets/image-4.png)
1. In the next window, click "Search for add-on" in the lower right corner, then type "Studio Code Server" and select from the list.
    ![Home Assistant - add Studio Code Server](/assets/image-6.png)
1. In the next window, confirm the installation
    ![Home Assistant - add Studio Code Server](/assets/image-7.png)
1. After installing the add-on, a new icon will appear on the left side, which will allow you to edit the configuration.
    ![Home Assistant - Studio Code Server yaml preview](/assets/image-8.png)

## Step 3 - configure the devices
Devices are added by editing the configuration (`configuration.yaml` file). This file has a specific structure that must be maintained.

### Keyword description
* `mqtt` - must be above the entire device configuration, means that all lines below are for MQTT integration (see step 1)
* `binary_sensor` / `sensor` / `switch` / etc.  - device class type; more information [in Home Assistant documentation](https://www.home-assistant.io/docs/configuration/customizing-devices/#device-class)
* `unique_id` – A unique identifier for each device; the following naming standard may be used: `<serial number>_<socket type><number>`, ie.L "1a2c1953_o00", where "1a2c1953" is Master's serial number
* `name` – the name under which the device will be displayed
* `state_topic` - MQTT topic with current device status
* `payload_on`, `payload_off` - values that set the given state of the device: `ON` or `OFF`
* `device_class` – device type configuration (setting a specific sensor: humidity, temperature, motion) - defines how the device works and how it looks in Home Assistant; each class of devices has its own types - see Home Assistant documentation for more details, e.g. [here](https://www.home-assistant.io/integrations/sensor/#device-class)
  
### Where to get topic addresses from 
The **MQTT Explorer** application is used to preview MQTT communications. Every time the state of the device changes, the corresponding MQTT topic changes. In this way, you can find out what queues to set in the Home Assistant configuration.
![MQTT Explorer deimic example](/assets/image-9.png)

The **Deimic Configurator** application allows you to see what output/input a device is connected to. Knowing where to look, it's easier to find the right topic in **MQTT Explorer**. To check a particular action, expand the description field by clicking on the triangle in the MQTT Explorer application.

### Example code
```yaml
mqtt:
  switch:
    - unique_id: "1a2c1953_o00"
      name: "Output 00"
      state_topic: "system/1a2c1953/outputs/0/get"
      command_topic: "system/1a2c1953/outputs/0/set"
      payload_on: "ON"
      payload_off: "OFF"
      state_on: "ON"
      state_off: "OFF"
      optimistic: false
      qos: 0
      retain: true

    - unique_id: "1a2c1953_o01"
      name: "Output 01"
      state_topic: "system/1a2c1953/outputs/1/get"
      command_topic: "system/1a2c1953/outputs/1/set"
      payload_on: "ON"
      payload_off: "OFF"
      state_on: "ON"
      state_off: "OFF"
      optimistic: false
      qos: 0
      retain: true

  sensor:
    - unique_id: "Temp_sensor"
      name: "Temperature"
      state_topic: "system/1a2c1953/analog_smart_inputs/30/0/get"
      unit_of_measurement: "° C"
      device_class: "temperature"

    - unique_id: "Humidity_sensor"
      name: "Humidity"
      state_topic: "system/1a2c1953/analog_smart_inputs/24/3/get"
      unit_of_measurement: "%"
      device_class: "moisture"

  binary_sensor:
    - unique_id: "Motion_sensor"
      name: "Motion sensor"
      state_topic: "system/1a2c1953/smart_inputs/24/2/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"
      device_class: "motion"

    - unique_id: "smart_sensor01"
      name: "Smart sensor 01"
      state_topic: "system/1a2c1953/smart_inputs/22/0/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"

    - unique_id: "smart_sensor02"
      name: "Smart sensor 02"
      state_topic: "system/1a2c1953/smart_inputs/22/1/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"

    - unique_id: "smart_sensor01"
      name: "Smart sensor 01"
      state_topic: "system/<nr seryjny Master>/smart_inputs/22/0/get"
      payload_on: "SHORT:1"
      payload_off: "SHORT:0"
```

## Step 4
An extension of the installation can be Home Brdige integration on iOS.
[HomeKit Bridge](https://www.home-assistant.io/integrations/homekit/)
