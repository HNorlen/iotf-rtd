Python for Device Developers
============================

- See `iot-python <https://github.com/ibm-messaging/iot-python>`_ in GitHub
- See `ibmiotf <https://pypi.python.org/pypi/ibmiotf>`_ on PyPi



Constructor
-------------------------------------------------------------------------------

The constructor builds the client instance, and accepts an options dict containing the following definitions:

- ``org`` - Your organization ID.
- ``type`` - The type of your device.
- ``id`` - The ID of your device.
- ``auth-method`` - Method of authentication (the only value currently supported is ``token``).
- ``auth-token`` - API key token (required if auth-method is ``token``).

If no options dict is provided, the client will connect to the IoT Platform Quickstart, and default to an unregistered device. The options dict creates definitions which are used to interact with the IoT Platform module.

.. code:: python

  import ibmiotf.device
  try:
    options = {
      "org": organization, 
      "type": deviceType, 
      "id": deviceId, 
      "auth-method": authMethod, 
      "auth-token": authToken
    }
    client = ibmiotf.device.Client(options)
  except ibmiotf.ConnectionException  as e:
    ...


Using a configuration file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of including an options dict directly, you can use a configuration file containing an options dict. If you are using a configuration file containing an options dict, use the following code format.

.. code:: python

    import ibmiotf.device
    try:
      options = ibmiotf.device.ParseConfigFile(configFilePath)
      client = ibmiotf.device.Client(options)
    except ibmiotf.ConnectionException  as e:
      ...

The content of the configuration file must be in the following format(need not contain **$** sign):

::

    [device]
    org=$orgId
    type=$myDeviceType
    id=$myDeviceId
    auth-method=token
    auth-token=$token


----


Publishing events
-------------------------------------------------------------------------------
Events are the mechanism by which devices publish data to the IoT Platform. The device controls the content of the event and assigns a name for each event it sends.

When an event is received by the IoT Platform the credentials of the connection on which the event was received are used to determine from which device the event was sent. With this architecture it is impossible for a device to impersonate another device.

Events can be published at any of the three :ref:`quality of service levels <qoslevels>` defined by the MQTT protocol.  By default events will be published as qos level 0.

Publish event using default quality of service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
.. code:: python

    client.connect()
    myData={'name' : 'foo', 'cpu' : 60, 'mem' : 50}
    client.publishEvent("status", "json", myData)


Publish event using user-defined quality of service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Events can be published at higher MQTT quality of servive levels, but these events may take slower then QoS level 0, because of the extra confirmation of receipt.

.. code:: python

    client.connect()
    myQosLevel=2
    myData={'name' : 'foo', 'cpu' : 60, 'mem' : 50}
    client.publishEvent("status", "json", myData, myQosLevel)

----

Handling commands
-------------------------------------------------------------------------------
When the device client connects it automatically subscribes to any command 
for this device.  To process specific commands you need to register a command 
callback method. The messages are returned as an instance of the Command class 
which has the following properties:

* command - string
* format - string
* data - dict 
* timestamp - datetime

.. code:: python

    def myCommandCallback(cmd):
      print("Command received: %s" % cmd.data)
      if cmd.command == "setInterval":
        if 'interval' not in cmd.data:
          print("Error - command is missing required information: 'interval'")
        else:
          interval = cmd.data['interval']
      elif cmd.command == "print":
        if 'message' not in cmd.data:
          print("Error - command is missing required information: 'message'")
        else:
          print(cmd.data['message'])

    ...
    client.connect()
    client.commandCallback = myCommandCallback


----




Custom message format support
-------------------------------------------------------------------------------
By default the library supports the encoding and decoding of Python dictionary objects as JSON when you use msgFormat "json".  When you use msgFormat "json-iotf" it will encode the message in accordance with the IoT Platform JSON Payload Specification.  To add support for your own custom message formats see the `sample in GitHub <https://github.com/ibm-messaging/iot-python/tree/master/samples/customMessageFormat>`__

Once you have created your custom encoder module it needs to be registered in the device client.  If you attempt to use an unknown message format when sending an event or the device receives a command send in a format that it does not know how to decode then the library will throw a MissingMessageDecoderException.

.. code:: python

    import myCustomCodec
    
    client.setMessageEncoderModule("custom", myCustomCodec)
    client.publishEvent("status", "custom", myData)

