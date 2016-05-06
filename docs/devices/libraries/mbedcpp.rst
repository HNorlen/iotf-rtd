mBed C++ for Device Developers
==============================

- See `ibmiotf <https://developer.mbed.org/teams/IBM_IoT/code/IBMIoTF/>`_ on `developer.mbed.org <https://developer.mbed.org/>`_

The `mBed C++ client library <https://developer.mbed.org/teams/IBM_IoT/code/IBMIoTF/>`_ can be used to connect `mBed devices <https://www.mbed.com/en/>`__ like `LPC1768 <https://developer.mbed.org/platforms/mbed-LPC1768/>`__, `FRDM-K64F <https://developer.mbed.org/platforms/FRDM-K64F/>`__ and etc.. to the IoT Platform Cloud service with ease. Although the library uses C++, it still avoids dynamic memory allocations and use of STL functions as the mBed devices sometimes have idiosyncratic memory models which make porting difficult. In any case, the library allows one to make memory use as predictable as possible. 

Dependencies
------------

- `Eclipse Paho MQTT library <https://developer.mbed.org/teams/mqtt/code/MQTT/>`__ - Provides a MQTT client library for mBed devices, check `here <http://www.eclipse.org/paho/clients/c/embedded/>`__ for more information.
- `EthernetInterface library <https://developer.mbed.org/users/mbed_official/code/EthernetInterface/>`__ - A mBed IP library over Ethernet.

How to use the library
-------------------------------------------------------------------------------
Use the `mBed Compiler <https://developer.mbed.org/compiler/>`__ to create your applications using this mBed C++ IBMIoTF Client Library. The mBed Compiler provides a lightweight online C/C++ IDE that is pre-configured to let you quickly write programs, compile and download them to run on your mbed Microcontroller. In fact, you don't have to install or set up anything to get running with mbed.


Refer to the step by step `mBed C++ Client Library for IBM IoT Platform Recipe <https://developer.ibm.com/recipes/tutorials/mbed-c-client-library-for-ibm-iot-foundation/>`__ that shows how one can use this library to connect an ARM mBed NXP LPC 1768 microcontroller to the IoT Platform.

Constructor
-------------------------------------------------------------------------------

The constructor builds the client instance, and accepts the following parameters:

* org - Your organization ID. (This is a required field. In case of quickstart flow, provide org as quickstart.)
* type - The type of your device. (This is a required field.)
* id - The ID of your device. (This is a required field.
* auth-method - Method of authentication (This is an optional field, needed only for registered flow and the only value currently supported is "token"). 
* auth-token - API key token (This is an optional field, needed only for registered flow).

These arguments create definitions which are used to interact with the IoT Platform service. 

The following code block shows how to create a DeviceClient instance to interact with the IoT Platform quickstart service.

.. code:: c++

  #include "DeviceClient.h"
  ....
  ....
  
  // Set IoT Platform connection parameters
  char organization[11] = "quickstart";     // For a registered connection, replace with your org
  char deviceType[8] = "LPC1768";           // For a registered connection, replace with your device type
  char deviceId[3] = "01";                  // For a registered connection, replace with your device id

  // Create DeviceClient
  IoTF::DeviceClient client(organization, deviceType, deviceId);
  
  // Get the DeviceID(MAC Address) if we are in quickstart mode and device id is not specified
  if((strcmp(organization, QUICKSTART) == 0) && (strcmp("", deviceId) == 0)) 
  {
  	char tmpBuf[50];
  	client.getDeviceId(tmpBuf, sizeof(tmpBuf));
  }
  ....

As shown above, if the device id is not specified, the DeviceClient uses the MAC address of the device as device id and connects to the IoT Platform. The device code can use getDeviceId() method to retrieve the device id from the DeviceClient instance.

The following code block shows how to create a DeviceClient instance to interact with the IoT Platform Registered organization.

.. code:: c++

  #include "DeviceClient.h"
  ....
  ....
  
  // Set IoT Platform connection parameters
  char organization[11] = "hrcl78";
  char deviceType[8] = "LPC1768";
  char deviceId[3] = "LPC176801";
  char method[6] = "token";
  char token[9] = "password";
  
  // Create DeviceClient
  IoTF::DeviceClient client(organization, deviceType, deviceId, method, token);
  ....

----

Connecting to the IoT Platform
------------------------------------------------

The device can connect to the IoT Platform by calling the connect function on the DeviceClient instance.

.. code:: c++

  #include "DeviceClient.h"
  ....
  ....
  
  // Create DeviceClient
  IoTF::DeviceClient client(organization, deviceType, deviceId, method, token);
  
  bool status = client.connect();
  

After the successful connection, the device can publish events to the IoT Platform and listen for commands.

Also, the device can query the status of the connection using the isConnected() method as follows,

.. code:: c++

  #include "DeviceClient.h"
  ....
  ....
  
  client.isConnected();


----

Publishing events
-------------------------------------------------------------------------------
Events are the mechanism by which devices publish data to the IoT Platform. The device controls the content of the event and assigns a name for each event it sends.

When an event is received by the IBM IoT Platform the credentials of the connection on which the event was received are used to determine from which device the event was sent. With this architecture it is impossible for a device to impersonate another device.

Events can be published at any of the three `quality of service levels <../messaging/mqtt.html#/>`__ defined by the MQTT protocol.  By default events will be published as qos level 0.

Publish event using default quality of service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The below sample shows how to publish various data points of LPC1768 like x,y & z axis, joystick position, current temperature reading and etc.. to IoT Platform in JSON format.

.. code:: c++

	boolean status = client.connect();
	
	// Create buffer to hold the event
	char buf[250];
	
	// Construct an event message with desired datapoints in JSON format
	sprintf(buf,
            "{\"d\":{\"myName\":\"IoT mbed\",\"accelX\":%0.4f,\"accelY\":%0.4f,\"accelZ\":%0.4f,
            \"temp\":%0.4f,\"joystick\":\"%s\",\"potentiometer1\":%0.4f,\"potentiometer2\":%0.4f}}",
            MMA.x(), MMA.y(), MMA.z(), sensor.temp(), joystickPos, ain1.read(), ain2.read());
        
        status = client.publishEvent("blink", buf);
	....

The complete sample can be found `here <https://developer.mbed.org/teams/IBM_IoT/code/IBMIoTClientLibrarySample/file/e58533b6bc6b/src/Main.cpp>`__.

Publish event using user-defined quality of service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Events can be published at higher MQTT quality of service levels, but these events may take slower than QoS level 0, because of the extra confirmation of receipt. Also quickstart flow allows only Qos of 0.

.. code:: c

	#include "MQTTClient.h"
	
	boolean status = client.connect();
	
	// Create buffer to hold the event
	char buf[250];
	
	// Construct an event message with desired datapoints in JSON format
	sprintf(buf,
            "{\"d\":{\"myName\":\"IoT mbed\",\"accelX\":%0.4f,\"accelY\":%0.4f,\"accelZ\":%0.4f,
            \"temp\":%0.4f,\"joystick\":\"%s\",\"potentiometer1\":%0.4f,\"potentiometer2\":%0.4f}}",
            MMA.x(), MMA.y(), MMA.z(), sensor.temp(), joystickPos, ain1.read(), ain2.read());
        
        status = client.publishEvent("blink", buf, MQTT::QOS2);
	....

	
Handling the connection lost error during the event publish
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When the publishEvent() method returns false, one can check the status of the connection and call reConnect() if the connection is lost,

.. code:: c

	#include "MQTTClient.h"
	
	status = client.publishEvent("blink", buf, MQTT::QOS2);
	
	if(status == false) {
	    // Check if connection is lost and retry
	    while(!client.isConnected())
	    {
	        client.reConnect();
	        wait(5.0);
	    }
	}
	....

The library does not store the events published during the unconnected state, and hence, the device needs to call the publishEvent() method again to send those events once the connection is reestablished.


----

Handling commands
-------------------------------------------------------------------------------
When the device client connects, it automatically subscribes to any commands for this device. To process specific commands you need to register a command callback method. 
The messages are returned as an instance of the Command class which has the following properties:

- command - name of the command invoked
- format - e.g json, xml
- payload

Following code defines a sample command callback function that processes the LED blink interval command from the application and adds the same to the DeviceClient instance.

.. code:: c++

    #include "DeviceClient.h"
    #include "Command.h"
    
    // Process the command and set the LED blink interval
    void processCommand(IoTF::Command &cmd)
    {
        if (strcmp(cmd.getCommand(), "blink") == 0) 
    	{
    	    char *payload = cmd.getPayload();
    	    char* pos = strchr(payload, '}');
    	    if (pos != NULL) {
    	        *pos = '\0';
    	        char* ratepos = strstr(payload, "rate");
    	        if(ratepos == NULL)
    	            return;
    	        if ((pos = strchr(ratepos, ':')) != NULL)
    	        {
    	            int blink_rate = atoi(pos + 1);
    	            blink_interval = (blink_rate <= 0) ? 0 : (blink_rate > 50 ? 1 : 50/blink_rate);
    	        }
    	    }
    	} else {
            WARN("Unsupported command: %s\n", cmd.getCommand());
        }
    }

    client.setCommandCallback(processCommand); 
    
    client.yield(10);  // allow the MQTT client to receive messages
    ....
    
The complete sample can be found `here <https://developer.mbed.org/teams/IBM_IoT/code/IBMIoTClientLibrarySample/file/e58533b6bc6b/src/Main.cpp>`__.

.. note:: The 'client.yield()' function must be called periodically to receive commands.

----

Disconnect Client
-----------------

To disconnect the client and release the connections, run the following code snippet.

.. code:: c++

	...
	client.disconnect();
	....

----

Samples
-------

`IBMIoTClientLibrarySample <https://developer.mbed.org/teams/IBM_IoT/code/IBMIoTClientLibrarySample/>`__ - A Sample code that showcases how to use IBMIoTF client library to connect the mbed LPC1768 or FRDM-K64F devices to the IBM Internet of Things Cloud service.
