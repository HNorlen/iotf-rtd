Embedded C for Device Developers
================================

- See `iotf-embeddedc <https://github.com/ibm-messaging/iotf-embeddedc>`_ on GitHub


Dependencies
------------

- `Eclipse Paho Embedded C library <http://git.eclipse.org/c/paho/org.eclipse.paho.mqtt.embedded-c.git/>`__ - provides an MQTT C client library, check `here <http://www.eclipse.org/paho/clients/c/embedded/>`__ for more information.


Installation
--------------
To install the IoT Platform client library for Embedded C follow the instructions below.

1. To install the latest version of the library, enter the following code in your command line.

.. code::

  [root@localhost ~]# git clone https://github.com/ibm-messaging/iotf-embeddedc.git

2. Copy the Paho library .tar file that was downloaded in the previous step to the *lib* directory.

.. code::
    
    cd iotf-embeddedc
    cp ~/org.eclipse.paho.mqtt.embedded-c-1.0.0.tar.gz lib/

3. Extract the library file

.. code::
    
    cd lib
    tar xvzf org.eclipse.paho.mqtt.embedded-c-1.0.0.tar.gz


When downloaded, the client has the following file structure:

.. code::

 |-lib - contains all the dependent files
 |-samples - contains the helloWorld and sampleDevice samples
   |-sampleDevice.c - sample device implementation
   |-helloworld.c - quickstart application
   |-README.md
   |-Makefile
   |-build.sh
 |-iotfclient.c - Main client file
 |-iotfclient.h - Header file for the client
 
 
Initializing the Client Library
-------------------------------

After downloading the client library, it must be initialized and connected to the IoT Platform. There are 2 ways to initialize the IoT Platform Client Library for Embedded C:

Passing as Parameters
~~~~~~~~~~~~~~~~~~~~~

The 'initialize' function takes the following details to connect to the IoT Platform service:

-   client - Pointer to the *iotfclient*
-   org - Your organization ID
-   type - The type of your device
-   id - The device ID
-   authmethod - Method of authentication (the only value currently supported is "token")
-   authtoken - API key token (required if auth-method is "token")

.. code:: c

	#include "iotfclient.h"
	....
	....
	Iotfclient client;
	//quickstart
	rc = initialize(&client,"quickstart","iotsample","001122334455",NULL,NULL);
	//registered
	rc = initialize(&client,"orgid","type","id","token","authtoken");
	....


Using a Configuration File
~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also use a configuration file to initialize the Embedded C client library. The function 'initialize\_configfile' takes the configuration file path as a parameter.

.. code:: c

	#include "iotfclient.h"
	....
	....
	char *filePath = "./device.cfg";
	Iotfclient client;
	rc = initialize_configfile(&client, filePath);
	....


The configuration file must use the following format.

.. code::

	org=$orgId
	type=$myDeviceType
	id=$myDeviceId
	auth-method=token
	auth-token=$token


Connecting to the Service
-------------------------

After initializing the IoT Platform Embedded C client library, you can connect to the IoT Platform by calling the 'connectiotf' function.


.. code:: c

	#include "iotfclient.h"
	....
	....
	Iotfclient client;
	char *configFilePath = "./device.cfg";
	
	rc = initialize_configfile(&client, configFilePath);
	
	if(rc != SUCCESS){
		printf("initialize failed and returned rc = %d.\n Quitting..", rc);
		return 0;
	}
	
	rc = connectiotf(&client);
	
	if(rc != SUCCESS){
		printf("Connection failed and returned rc = %d.\n Quitting..", rc);
		return 0;
	}
	....


Handling commands
------------------------------------------

When the device client connects, it automatically subscribes to any command for this device. To process specific commands you need to register a command callback function by calling the function 'setCommandHandler'. The commands are returned as:

- commandName - name of the command invoked
- format - e.g json, xml
- payload


.. code:: c

	#include "iotfclient.h"
	
	void myCallback (char* commandName, char* format, void* payload)
	{
	printf("The command received :: %s\n", commandName);
	printf("format : %s\n", format);
	printf("Payload is : %s\n", (char *)payload);
	}
	...
	...
	char *filePath = "./device.cfg";
	rc = connectiotfConfig(filePath);
	setCommandHandler(myCallback);
	
	yield(1000);
	....

.. note:: The 'yield' function must be called periodically to receive commands.


Publishing events
-----------------------------------

Events can be published by using:

- eventType - Type of event to be published e.g status, gps
- eventFormat - Format of the event e.g json
- data - Payload of the event
- QoS - qos for the publish event. Supported values : QOS0, QOS1, QOS2

.. code:: c

	#include "iotfclient.h"
	....
	rc = connectiotf (org, type, id , authmethod, authtoken);
	char *payload = {\"d\" : {\"temp\" : 34 }};
	
	rc= publishEvent("status","json", "{\"d\" : {\"temp\" : 34 }}", QOS0); 
	....


Disconnect Client
-----------------

To disconnect the client and release the connections, run the following code snippet.

.. code:: c

	#include "iotfclient.h"
	....
	rc = connectiotf (org, type, id , authmethod, authtoken);
	char *payload = {\"d\" : {\"temp\" : 34 }};
	
	rc= publishEvent("status","json", payload , QOS0);
	...
	rc = disconnect();
	....


Samples
-------

Sample device and application code is provided in `GitHub <https://github.com/ibm-messaging/iotf-embeddedc/tree/master/samples>`_.
