OpenEnergyMonitor EmonTx Sensors
================================

.. seo:: :description: Instructions for setting up OpenEnergyMonitor EmonTx energy monitors with ESPHome. :image: emontx.jpg :keywords: EmonTx, OpenEnergyMonitor, energy monitor, power monitoring, CT clamp

The emontx sensor platform allows you to use your OpenEnergyMonitor EmonTx energy monitoring devices with ESPHome.
The EmonTx is a wireless energy monitoring device that can measure voltage, current, and power consumption. It is commonly used in home energy monitoring systems.
The EmonTx can be equipped with various sensors, such as CT clamps for current measurement, and can also measure temperature.

For further information about the EmonTx and similar devices, see the `OpenEnergyMonitor site <https://openenergymonitor.org/>`_.

.. figure:: images/emontx5.jpg
    :align: center
    :width: 50.0%
    
    OpenEnergyMonitor EmonTx5

.. code-block:: yaml

    # Example configuration entry
    emontx:
      id: myemontx

Configuration variables
-----------------------

In `emontx` platform:

- **id** (*Optional*, :ref:`config-id`): Manually specify the ID used for code generation or multiple hubs.

- **emoncms** (*Optional*): For forwarding data to an `emoncms` server.

  - **emoncms_server** (**Required**): The URL of the emoncms server.
  - **api_key** (**Required**): The API write key for the emoncms server.
  - **node** (**Required**): The node ID to use for the emoncms server.
  - **http_id** (**Required**, :ref:`config-id`): The ID of the HTTP component to use for communication with the emoncms server.

.. note::

    If you configure the ``emoncms`` option, you will also need to define the ``http_request`` component in your configuration and reference its ID in the ``http_id`` parameter. This is required for the component to communicate with the emoncms server.
    
    For example:
    
    .. code-block:: yaml
    
        http_request:
          id: http_client
          useragent: esphome/emontx
          timeout: 10s
        
        emontx:
          emoncms:
            emoncms_server: "https://emoncms.org"
            api_key: YOUR_API_KEY
            node: 1
            http_id: http_client

Sensors
*******


Hardware Setup
--------------

The EmonTx can be connected to your ESP device via the serial UART interface.

Make sure the EmonTx is configured to output data in the correct format. The default baud rate for communication is 115200.

See Also
--------
- :ref:`sensor-filters`
- `OpenEnergyMonitor <https://openenergymonitor.org/>`_
- :ghedit:`Edit`