# rpi-enviro-mqtt
Send air quality data from a Pimoroni RPi [Enviro+](https://shop.pimoroni.com/products/enviro) over [MQTT](http://mqtt.org/).

The `mqtt-all.py` script is a fork of the official [luftdaten.py](https://github.com/pimoroni/enviroplus-python/blob/master/examples/luftdaten.py) script. The main difference being that this script uses MQTT to publish data over the local network (vs internet to Luftdaten) so there no dependency on internet connection. Also since you don't need to poll the Luftdaten website for data, latency is almost eliminated and you can visualise data in real time, using a tool like [mqtt-explorer](https://mqtt-explorer.com/) (screenshot below).

<p align="center">
<img src="https://github.com/robmarkcole/rpi-enviro-mqtt/blob/master/assets/mqtt-explorer-usage.png" width="800">
</p>

## Installation & setup
You should first install the Pimoroni [enviroplus-python](https://github.com/pimoroni/enviroplus-python) library. There is one additional dependency required for this script, which is [paho-mqtt](http://www.eclipse.org/paho/). This can be installed with `pip3 install paho-mqtt`. You need an MQTT broker to receive the data, if you haven't already set one up, a broker can be installed on the rpi with `sudo apt-get install mosquitto mosquitto-clients`. Note this broker will run at startup automatically, which is very convenient. Note also that this broker is unsecured by default, and that this code is only for personal (not professional) use on a secure local network. You should publish some MQTT data from the command line to check the broker is functioning, and verify receipt of this data using the tool of your choice (e.g. mqtt-explorer). Clone this repo to your rpi, making note of the path to `mqtt-all.py` 

## Run the `mqtt-all.py` script
From the terminal you can run the script with:
```
python3 /home/pi/yourdir/mqtt-all.py --broker localhost --port 1883 --topic enviroplus
```

Note that the arguments passed here are the defaults, and just shown as an example. If you are issuing this command over SSH you probably want it to continue after you disconnect. This [can be achieved](https://raspberrypi.stackexchange.com/questions/29348/keep-process-running-after-close-session) by using `nohup`, so for example you would issue command:
```
nohup python3 /home/pi/yourdir/mqtt-all.py --broker 192.168.1.164 --topic enviro &
```

TODO add service file.

## Home Assistant integration
I personally am using [home-assistant](https://www.home-assistant.io/) to receive and log the enviro mqtt data. The benefits are: logging to `.db` sqlite file & graphing. Simply configure an mqtt-sensor on the `enviro` topic (or whatever you selected) and break out the individual readings using a template. Example below:

```yaml
sensor:
  - platform: mqtt
    name: "enviro"
    state_topic: "enviro"

  - platform: mqtt
    state_topic: "enviro"
    value_template: "{{ value_json.temperature }}"
    name: "enviro_temperature"
    unit_of_measurement: '°C'
    icon: "mdi:thermometer"

  - platform: mqtt
    state_topic: "enviro"
    value_template: "{{ value_json.humidity }}"
    name: "enviro_humidity"
    unit_of_measurement: '%'
    icon: "mdi:water-percent"

  - platform: mqtt
    state_topic: "enviro"
    value_template: "{{ value_json.pressure }}"
    name: "enviro_pressure"
    unit_of_measurement: 'Pa'
    icon: "mdi:arrow-down-bold"

  - platform: mqtt
    state_topic: "enviro"
    value_template: "{{ value_json.P2 }}"
    name: "enviro_pm2"
    unit_of_measurement: ''
    icon: "mdi:thought-bubble"

  - platform: mqtt
    state_topic: "enviro"
    value_template: "{{ value_json.P1 }}"
    name: "enviro_pm10"
    unit_of_measurement: ''
    icon: "mdi:thought-bubble-outline"
```

Using the created entities you can add an entity card like the following:

<p align="center">
<img src="https://github.com/robmarkcole/rpi-enviro-mqtt/blob/master/assets/ha_card.png" width="400">
</p>