C# for Device Developers
========================

- See `iot-csharp <https://github.com/IoT-Analytics/iot-csharp>`_ in GitHub


Constructor
-----------
The constructor builds the client instance, and accepts arguments containing the following definitions:

- ``orgId`` - Your organization ID.
- ``deviceType`` - The type of your device.
- ``deviceId`` - The ID of your device.
- ``auth-method`` - Method of authentication (the only value currently supported is "token").
- ``auth-token`` - API key token (required if auth-method is "token").


If deviceId and deviceType are provided, the client will connect to the IoT Platform Quickstart, and default to an unregistered device. The argument lists creates definitions which are used to interact with the IoT Platform module.

.. code:: C#

	namespace com.ibm.iotf.client

	public DeviceClient(string orgId, string deviceType, string deviceID, string authmethod, string authtoken)
            : base(orgId, "d" + CLIENT_ID_DELIMITER + orgId + CLIENT_ID_DELIMITER + deviceType + CLIENT_ID_DELIMITER + deviceID, "use-token-auth", authtoken)
        {

        }


Publishing events
------------------
Events are the mechanism by which devices publish data to the IoT Platform. The device controls the content of the event and assigns a name for each event it sends.

When an event is received by the IoT Platform the credentials of the connection on which the event was received are used to determine which device sent the event. With this architecture it is impossible for a device to impersonate another device.

Events can be published at any of the three `quality of service (QoS) levels <../mqtt.html#/qoslevels>`_, defined by the MQTT protocol. By default events will be published as QoS level 0.

Publish event using default quality of service
----------------------------------------------
.. code:: C#


	deviceClient.connect();
    deviceClient.publishEvent("event", "json", "{temp:23}");


Publish event using user-defined quality of service
-----------------------------------------------------
Events can be published at higher MQTT quality of service levels, but events published at QoS levels above 0 may be slower then QoS 0 events, because of the extra confirmation of receipt used in QoS levels above 0.

.. code:: C#

	deviceClient.connect();
    deviceClient.publishEvent("event", "json", "{temp:23}", 2);


Handling commands
-------------------
When the device client connects, it automatically subscribes to any commands for this device. To process specific commands, you must register a command callback method. 


.. code:: C#

	public static void processCommand(string cmdName, string format, string data) {
    ...
  }

.. code:: C#

	deviceClient.connect();
    deviceClient.commandCallback += processCommand;
    
