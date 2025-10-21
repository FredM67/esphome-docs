---
description: "Instructions for setting up French Teleinformation"
title: "Teleinformation from Linky electrical counter."
params:
  seo:
    description: Instructions for setting up French Teleinformation
    image: teleinfo.jpg
---



## Component/Hub

The `teleinfo`   component allows you to retrieve data from a
French electrical counter using Teleinformation ([datasheet](https://www.enedis.fr/media/2035/download)). It works with Linky electrical
counter but also legacy EDF electrical counter.

{{< img src="teleinfo-full.jpg" alt="Image" caption="Linky electrical counter" width="50.0%" class="align-center" >}}

..

A simple electronic assembly with an optocoupler and a resistor could
let you retrieve detailed power consumption or power production.
There is plenty of example on the web.

As the communication with the Teleinformation is done using UART, you need to
have an [UART bus](#uart) in your configuration with the `rx_pin`
connected to the output of the optocoupler component. Additionally, you need to
set the baud rate to 9600bps if counter is configured to work in standard
mode or 1200bps in historical mode.  To find out which mode you are using,
simply press -/+ buttons on the counter and look for `Standard mode` or
`Historical mode` as below.

{{< img src="teleinfo-standard.jpg" alt="Image" caption="Linky electrical counter configured in standard mode." width="50.0%" class="align-center" >}}

..

{{< img src="teleinfo-historical.jpg" alt="Image" caption="Linky electrical counter configured in historical mode." width="50.0%" class="align-center" >}}

..

```yaml
# Example configuration entry
teleinfo:
  id: myteleinfo

```

## Configuration variables

In teleinfo platform:

- **historical_mode** (*Optional*): Whether to use historical mode or standard mode.
  With historical mode, baudrate of 1200 must be used whereas 9600 must be used in
  standard mode. Defaults to `false`  .

- **update_interval** (*Optional*, [Time](#config-time)): The interval to check the
  sensor. Defaults to `60s`  .

- **uart_id** (*Optional*, [ID](#config-id)): Manually specify the ID of the [UART bus](#uart) if you want
  to use multiple UART buses.

- **id** (*Optional*, [ID](#config-id)): Manually specify the ID used for code generation or multiple hubs.

### Sensor

```yaml
sensor:
  - platform: teleinfo
    tag_name: "HCHC"
    name: "hchc"
    unit_of_measurement: "Wh"
    icon: mdi:flash
    teleinfo_id: myteleinfo
  - platform: teleinfo
    tag_name: "HCHP"
    name: "hchp"
    unit_of_measurement: "Wh"
    icon: mdi:flash
    teleinfo_id: myteleinfo
  - platform: teleinfo
    tag_name: "PAPP"
    name: "papp"
    unit_of_measurement: "VA"
    icon: mdi:flash
    teleinfo_id: myteleinfo

```

- **tag_name** (**Required**, string): Specify the tag you want to retrieve from the Teleinformation.
- **teleinfo_id** (*Optional*, [ID](#config-id)): Specify the ID of used hub.
- All other options from [Sensor](#config-sensor).

### Pre-defined Sensor Configurations

The teleinfo component includes pre-defined configurations for all standard/historical TIC (Télé-Information Client) tags. When you specify a `tag_name`, the component automatically applies the appropriate:

- unit_of_measurement
- device_class
- state_class
- accuracy (number of decimals)

For example, when using the "PAPP" tag (apparent power), the sensor will automatically use "VA" as the unit of measurement, "power" as the device class, "measurement" as the state class, and 0 decimals.

```yaml
# Example using pre-defined configuration
sensor:
  - platform: teleinfo
    tag_name: "PAPP"
    name: "Apparent Power"
    # No need to specify unit_of_measurement, device_class, etc.
    teleinfo_id: myteleinfo

```

You can override any pre-defined value by explicitly setting it in your configuration:

```yaml
sensor:
  - platform: teleinfo
    tag_name: "PAPP"
    name: "Apparent Power"
    unit_of_measurement: "volt-ampere"  # Override the pre-defined unit
    accuracy_decimals: 1                # Override the pre-defined accuracy
    teleinfo_id: myteleinfo

```

### Text Sensor

```yaml
text_sensor:
  - platform: teleinfo
    tag_name: "OPTARIF"
    name: "optarif"
    teleinfo_id: myteleinfo

```

- **tag_name** (**Required**, string): Specify the tag you want to retrieve from the Teleinformation.
- **teleinfo_id** (*Optional*, [ID](#config-id)): Specify the ID of used hub.
- All other options from [Text Sensor](#config-text_sensor).

## See Also

- {{< apiref "teleinfo/teleinfo.h" "teleinfo/teleinfo.h" >}}
