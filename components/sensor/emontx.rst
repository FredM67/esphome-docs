OpenEnergyMonitor EmonTx Sensors
================================

.. seo:: :description: Instructions for setting up OpenEnergyMonitor EmonTx energy monitors with ESPHome. :image: emontx.jpg :keywords: EmonTx, OpenEnergyMonitor, energy monitor, power monitoring, CT clamp

.. _emontx-component:

Component/Hub
-------------

The ``emontx`` component allows you to use ESPHome to create a connection to an OpenEnergyMonitor emonTX via a supported device (ESP32 recommended).
This component is a global hub that establishes the connection to the EmonTx via :ref:`UART <uart>` and translates the received data.
Using the :ref:`emontx sensors <emontx-sensors>`, you can then create sensors for Home Assistant that track voltage, current, power as measured by CTs (up to 12), pulse data, and temperature depending on the configuration of the emonTX.

The component can send data to an MQTT Broker either as JSON or as individual topics to be consumed by any system.

This component can also be used to send data to a remote emoncms instance such as `emoncms <https://emoncms.org/>`_ via HTTP or a locally hosted system via HTTP or MQTT. Working directly with emoncms seamlessly is a key benefit of this component, allowing you to integrate your energy monitoring data with powerful visualization and analysis tools.

.. figure:: images/emontx5.jpg
    :alt: OpenEnergyMonitor EmonTx5
    :align: center
    :width: 50.0%

    OpenEnergyMonitor EmonTx.

As the communication with the EmonTx is done using UART, you need to have an :ref:`UART bus <uart>` in your configuration with the ``rx_pin`` connected to the data pin of the EmonTx and with the baud rate set to 115200.

.. code-block:: yaml

    # Example configuration entry
    uart:
      id: emontx_uart # using UART2
      rx_pin: GPIO16
      tx_pin: GPIO17
      baud_rate: 115200

    emontx:
      id: myemontx

Configuration variables
-----------------------

In `emontx` platform:

- **id** (*Optional*, :ref:`config-id`): Manually specify the ID used for code generation or multiple hubs.
- **uart_id** (*Optional*, :ref:`config-id`): Manually specify the ID of the UART Component if you want to use multiple UART buses.
- **emoncms** (*Optional*): For forwarding data to an `emoncms` server.

  - **http** (*Optional*): For forwarding data to an HTTP server.

    - **server** (**Required**): The URL of the emoncms server.
    - **apikey** (**Required**, string): The API Read/Write key for the emoncms server.
    - **node** (**Required**, string): The node ID to use for the emoncms server.

  - **mqtt** (*Optional*): For forwarding data to an MQTT broker, including emoncms via MQTT.

    - **publish_mode** (*Optional*, enum): The MQTT publish mode. Defaults to ``json``.

      - ``json`` - Publish the raw json as receive from EmonTx
      - ``individual`` - Publish each value individually

    - **base_prefix** (*Optional*, string): The MQTT base prefix to use for publishing data. Defaults to ``emon``.
    - **node** (*Optional*, string): The MQTT node to use for publishing data. Defaults to ``${device_name}``.

Emoncms Integration
-------------------

This integration is typically intended to forward data to an emoncms server, such as `emoncms.org <https://emoncms.org/>`_ or a locally hosted instance.

.. warning::

    If you enable ``emoncms`` and you do *not* use the :doc:`/components/api`, ie the module is exclusively used for forwarding data to Emoncms and it's *not* connected to any Home Assistant instance, you must
    remove the ``api:`` configuration or set ``reboot_timeout: 0s``, otherwise the ESP will
    reboot every 15 minutes because no client connected to the native API.

There's two ways to send data to emoncms: via HTTP or MQTT.

If you configure the ``emoncms`` option, you must either define the :doc:`/components/http_request` component or the :doc:`/components/mqtt` component in your configuration.
This is required for the component to communicate with the emoncms server.

The http method
***************
The component will forward data to the emoncms server using HTTP POST requests.
The data will be forwarded in JSON format, which is the expected format for emoncms.

This method is suitable for sending data to a remote emoncms server or a locally hosted instance.

Example:

.. code-block:: yaml

    http_request:
      useragent: esphome/emontx
      timeout: 10s
    
    emontx:
      emoncms:
        http:
          server: "https://emoncms.org"
          apikey: YOUR_API_KEY
          node: 1

.. note::

    The ``node`` parameter must be compliant with what emoncms expects. Depending on your emoncms server configuration, this could be a numeric ID (like "1") or a string identifier. Check your emoncms server documentation to ensure you're using the correct node format.

The MQTT method
***************
This method is suitable for sending data to a locally hosted emoncms instance or any other MQTT broker that supports the emoncms MQTT protocol.
As of now `emoncms.org` does not support MQTT, so this method is primarily for local installations or other compatible systems.

You'll have two possibilities to forward data to emoncms via MQTT:

1. **Publish the raw JSON**: This method publishes the raw JSON data received from the EmonTx to the MQTT broker. This is the default way to publish.

2. **Publish individual values**: This method publishes each sensor value individually to separate MQTT topics.

The component will publish all sensor data to topics following this structure:

- ``<base_prefix>/<node>/{json_data}`` for raw JSON data
- ``<base_prefix>/<node>/<sensor_name>`` for individual sensor values

Example:

.. code-block:: yaml

    mqtt:
      broker: 192.168.1.10
      port: 1883           # Optional
      username: mqtt_user  # Optional
      password: mqtt_pass  # Optional
      id: mqtt_client      # Optional
    
    emontx:
      emoncms:
        mqtt:
          publish_mode: individual
          base_prefix: "emon"
          node: "emontx"

With this configuration, data will be published to topics such as:

- ``emon/emontx/V1`` for voltage on phase 1
- ``emon/emontx/P1`` for power on CT1
- ``emon/emontx/E1`` for energy on CT1

For integration with emoncms via MQTT, use the base prefix that includes any node identification required by your emoncms instance.

MQTT Integration
----------------

This integration is typically intended to forward data to a non-Home Assistant system, such as Jeedom, Domoticz, or a custom MQTT consumer.

.. warning::

    If you enable ``mqtt`` forwarding and you do *not* use the :doc:`/components/api`, ie the module is exclusively used for forwarding data via MQTT and it's *not* connected to any Home Assistant instance, you must
    remove the ``api:`` configuration or set ``reboot_timeout: 0s``, otherwise the ESP will
    reboot every 15 minutes because no client connected to the native API.

If you configure the ``mqtt`` option, you will need to define the :doc:`/components/mqtt` component in your configuration.
This is required for the component to publish data to the MQTT broker.

The component will publish all sensor data to topics following this structure:
``${device_name}/sensor/<sensor_name>``

Only sensor(s) defined in the configuration will be published (see :ref:`emontx-sensors`).

Example:

.. code-block:: yaml

    mqtt:
      broker: 192.168.1.10
      port: 1883           # Optional
      username: mqtt_user  # Optional
      password: mqtt_pass  # Optional
      id: mqtt_client      # Optional
    
    emontx:

With this configuration, data will be published to topics such as:

- ``${device_name}/sensor/V1`` for voltage on phase 1
- ``${device_name}/sensor/P1`` for power on CT1
- ``${device_name}/sensor/E1`` for energy on CT1

You can customize the MQTT topic structure by modifying the `topic_prefix` parameters in the `mqtt` configuration.
See the :doc:`/components/mqtt` documentation for more details on how to configure MQTT topics.

.. _emontx-sensors:

Sensors
-------

The EmonTx component provides several sensors that can be used to monitor various parameters:

- **Power**: Calculates the power consumption based on the voltage and current readings.
- **Energy**: Accumulates the energy consumption over time.
- **Voltage**: Measures the voltage of the mains supply.
- **Current**: Measures the current flowing through the connected CT clamps.
- **Power Factor**: Calculates the power factor based on the voltage and current readings.
- **Pulse**: Measures the number of pulses from the connected pulse sensor (interface S0 for example).
- **Temperature**: Reports temperatures of connected Dallas DS18B20 sensors.

Predefined Sensor Configuration
*******************************
Each type of sensor in the EmonTx component has predefined configuration parameters:

Power Sensors (P)
^^^^^^^^^^^^^^^^^
Power sensors have the following default configuration:

- Unit of Measurement: W (Watt)
- Device Class: power
- State Class: measurement
- Accuracy: 0 decimal place

Energy Sensors (E)
^^^^^^^^^^^^^^^^^^
Energy sensors have the following default configuration:

- Unit of Measurement: Wh (Watt-hours)
- Device Class: energy
- State Class: total_increasing
- Accuracy: 0 decimal places

Voltage Sensors (V)
^^^^^^^^^^^^^^^^^^^
Voltage sensors have the following default configuration:

- Unit of Measurement: V (Volt)
- Device Class: voltage
- State Class: measurement
- Accuracy: 2 decimal places

Current Sensors (I)
^^^^^^^^^^^^^^^^^^^
Current sensors have the following default configuration:

- Unit of Measurement: A (Ampere)
- Device Class: current
- State Class: measurement
- Accuracy: 2 decimal places

Power Factor Sensors (PF)
^^^^^^^^^^^^^^^^^^^^^^^^^
Power factor sensors have the following default configuration:

- Unit of Measurement: (dimensionless)
- Device Class: power_factor
- State Class: measurement
- Accuracy: 2 decimal places

Temperature Sensors (T)
^^^^^^^^^^^^^^^^^^^^^^^
Temperature sensors have the following default configuration:

- Unit of Measurement: °C (Celsius)
- Device Class: temperature
- State Class: measurement
- Accuracy: 2 decimal place

Pulse Sensors (PULSE)
^^^^^^^^^^^^^^^^^^^^^
Pulse sensors have the following default configuration:

- Unit of Measurement: pulses
- Accuracy: 0 decimal places (whole numbers)

These predefined configurations can be overridden in your YAML configuration if needed.

Sensor Indexing
***************

The EmonTx sensors use a specific indexing scheme that depends on the physical configuration of your EmonTx device:

Voltage Sensors (V1-V3)
^^^^^^^^^^^^^^^^^^^^^^^
Voltage sensors are indexed based on your power system configuration:

- **Vrms**: Voltage reading for single-phase systems
- **V1**: Voltage reading for phase 1 in multi-phase systems
- **V2**: Voltage reading for phase 2 in multi-phase systems
- **V3**: Voltage reading for phase 3 in three-phase systems

Power Sensors (P1-P12)
^^^^^^^^^^^^^^^^^^^^^^
Power sensors are indexed based on the CT clamp connections:

- **P1-P6**: Power readings for CT1-CT6 on the standard EmonTx
- **P7-P12**: Power readings for CT7-CT12 when an expansion board is present

Energy Sensors (E1-E12)
^^^^^^^^^^^^^^^^^^^^^^^
Energy sensors follow the same indexing scheme as power sensors:

- **E1-E6**: Energy accumulation for CT1-CT6 on the standard EmonTx
- **E7-E12**: Energy accumulation for CT7-CT12 when an expansion board is present

Current Sensors (I1-I12)
^^^^^^^^^^^^^^^^^^^^^^^^
Current sensors are indexed according to the CT inputs:

- **I1-I6**: Current readings from CT1-CT6 on the standard EmonTx
- **I7-I12**: Current readings from CT7-CT12 when an expansion board is present

Power Factor Sensors (PF1-PF12)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Power factor sensors follow the same indexing as the CT inputs:

- **PF1-PF6**: Power factor for CT1-CT6 on the standard EmonTx
- **PF7-PF12**: Power factor for CT7-CT12 when an expansion board is present

Temperature Sensors (T1-T3)
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Temperature sensors are indexed according to the connected temperature probes:

- **T1-T3**: Readings from up to 3 temperature sensors (usually DS18B20)

Pulse Sensor (PULSE, DIGPULSE, ANAPULSE)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The pulse sensor is a single counter input and doesn't use indexing.

.. note::

    The actual availability of sensors depends on your specific EmonTx configuration and firmware.
    Not all sensor indexes may be active or report values in your setup.
    For example, in a single-phase system, only V1 will provide readings, while V2 and V3 won't be available.

    To check what sensors are available in your EmonTx, you can refer to the EmonTx documentation or the firmware configuration.
    You can also use the ESPHome logs to see which sensors are reporting data.
    
    For example:

    .. code-block:: text

        [14:43:36][I][emontx:099]: Received data: {"MSG":54378,"V1":234.16,"V2":234.13,"V3":234.22,"P1":0,"P2":0,"P3":0,"P4":0,"P5":0,"P6":0,"E1":74,"E2":-9,"E3":-12,"E4":7,"E5":-4,"E6":-6,"pulse":0}


Example of Sensor Configuration
*******************************
Here is an example of how to configure the EmonTx sensors in your ESPHome YAML configuration:

.. code-block:: yaml

    sensor:
      - platform: emontx
        tag_name: "V1"
        name: "Voltage L1"
      - platform: emontx
        tag_name: "V2"
        name: "Voltage L2"
      - platform: emontx
        tag_name: "V3"
        name: "Voltage L3"
      - platform: emontx
        tag_name: "P1"
        name: "Power CT1"
      - platform: emontx
        tag_name: "E2"
        name: "Energy CT2"
      - platform: emontx
        tag_name: "I3"
        name: "Current CT3"
      - platform: emontx
        tag_name: "pf1"
        name: "Power factor CT1"
      - platform: emontx
        tag_name: "T1"
        name: "Temp 1"
      - platform: emontx
        tag_name: "pulse"
        name: "Pulse"

Hardware Setup
--------------

The EmonTx can be connected to your ESP device via the serial UART interface.

Make sure the EmonTx is configured to output data in JSON format. The default baud rate for communication is 115200.

See Also
--------
- :ref:`sensor-filters`
- `OpenEnergyMonitor <https://openenergymonitor.org/>`_
- :ghedit:`Edit`
