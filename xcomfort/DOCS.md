# Home Assistant Add-on: xComfort

xComfort is a wireless European home automation system, using the 868,3MHz band. The system is closed source. This code was reverse engineered (by Karl Oygard - link below) from a variety of sources, without documentation from Eaton, and may not follow their specifications.

This code supports both extended and regular status messages. Older devices only send the latter, which are not routed and have no delivery guarantees. Careful placement of the CI is important, so that it can see these messages, or you will need to use more than one CI to improve coverage.

To view the source code of this daemon, go to [github.com/karloygard/xcomfortd-go](https://github.com/karloygard/xcomfortd-go).

## How to use

By default, datapoints are read out from the eprom of the CI, and *must be kept updated if and when devices are added*.  Consult the [MRF manual](http://www.eaton.com/ecm/groups/public/@pub/@eatonnl/@electrical/documents/content/pct_325435.pdf) (paragraph USB-RF-Communication Stick) for documentation on how to do this.  For testing purposes, both TXT and DPL file formats are supported, but the latter format is generally superior.

Switching actuators are by default configured as switches in Home Assistant.  This may not be the desired behaviour if the actuator is controlling a light source.  You can force datapoints to be configured as light sources by prepending the datapoint name in MRF with `LI_`, e.g. a switching actuator named `Bathroom actuator` will be discovered as a switch, while the same actuator named `LI_Bathroom actuator` will be discovered as a light source.

Heating actuators are supported as climate devices.  However, MQTT discovery doesn't specify a way of sending current temperature to the device, only reading it.  To work around this, current temperature is exposed in a separate `number.{name of the device}_current` entity on the device.  Writing a value to this entity will send it to the device.  The heating actuator will go into emergency mode if the current or desired temperature isn't updated once every hour.  The following automation copies the temperature from a sensor to the climate entity when it changes or every 30 minutes:

```- alias: Copy temperature to heating actuator
  description: Update current temp on change or every 30 min
  trigger:
  - platform: state
    entity_id: sensor.{name of the sensor}
  - platform: time_pattern
    minutes: '/30'
  condition: []
  action:
  - service: number.set_value
    target:
      entity_id: number.{name of the heating actuator}_current
    data_template:
      value: '{{ states(''sensor.{name of the sensor}'') }}'
```

## Configuration

Add-on configuration:

```yaml
mqtt_client_id: xcomfort
mqtt_host: core_mosquitto
mqtt_port: 1883
mqtt_user: ""
mqtt_passwd: ""
eprom: true
datapoints_file: ""
verbose: false
eci_hosts: ["eci-host-1", "eci-host-2"]
use_hidapi: false
ha_discovery: true
ha_discovery_prefix: homeassistant
ha_discovery_remove: false
```

### Option: `mqtt_client_id`

ID of the MQTT client the daemon connects with. 

### Option: `mqtt_host`

ID (intenal name; IP or DNS name) of the MQTT host to send the MQTT messages to. 
Default = core_mosquitto (this is the standard internal host name Hassio assign to the default Mosquitto plugin)

### Option: `mqtt_port`

The port that the host is listening on.
Default = 1883; this is the non-ssl port.

### Option: `mqtt_user`

The username used to connect to the host with. 

### Option: `mqtt_passwd`

The password for the user to connect to the host with.

### Option: `eprom`

Read datapoint list from eprom.

### Option: `datapoints_file`

Name of the datapoints file in the Home Assistant configuration directory.  Only one file can be specified, so if you are using more than one device, the same file will be used for all devices.  Recommend the `eprom` option instead.

### Option: `verbose`

Enable verbose logging.

### Option: `eci_hosts`

Host addresses (array) of ECI devices.

### Option: `ha_discovery`

Make the add-on send MQTT device discovery messages to Home Assistant.

### Option: `ha_discovery_prefix`

The Home Assistant discovery prefix.

### Option: `ha_discovery_remove`

When set to true, the add-on will send MQTT device discovery messages to clear out devices on configuration changes.  While convenient, this has the unfortunate side effect of also wiping any device configuration that the user may have made in Home Assistant.  Leaving this off makes devices and their associated configuration persistent, but if you remove devices, you will have to manually remove them from Home Assistant.

### Option: `use_hidapi`

Talk to USB sticks with hidapi.  Hidapi appears to have intermittent issues, and is only included for testing purposes, leaving this to false is recommended.

## License

MIT License

Original Copyright (c) 2021 Karl Anders Øygard and Guðmundur Björn Birkisson
Additional Copyright (C) 2023 Davy Geurts (Kooda)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
