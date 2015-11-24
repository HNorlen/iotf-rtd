Java Client Library - Introduction
============================================


This `Java Client Library <https://github.com/ibm-messaging/iot-java/releases/latest>`__ can be used to simplify interactions with the `IBM Internet of Things Foundation <https://internetofthings.ibmcloud.com>`__. The documentation is divided into three sections:  

- The `Devices section <../java/java_cli_devices.html>`__ contains information on how devices publish events and handle commands using the Java ibmiotf Client Library. 
- The `Managed Device section <../java/java_deviceManagement.html>`__ contains information on how devices can connect to the Internet of Things Foundation Device Management service using Java ibmiotf Client Library and perform device management operations like firmware update, location update, and diagnostics update.
- The `Applications section <../java/java_cli_app.html>`__ details how applications can use the Java ibmiotf Client Library to interact with devices.

Download
-------------------------------------------------------------------------------
The latest version of the client library is available `here <https://github.com/ibm-messaging/iot-java/releases/latest>`__ to download.

The zip module contains the following files,

* com.ibm.iotf.client-<version>.jar - Client library that enables one to talk to Internet of Things Foundation Connect.
* com.ibm.iotf.samples-<version>.jar - Contains samples.
* DMDeviceSample.properties - Properties file used to configure the list of options required to connect to Internet of Things Foundation Connect and perform device management operations.
* device.prop - Properties file to specify connectivity information when DeviceClient is used
* lib - Contains all dependent libraries
* javadoc - Contains the client library documentation

This Java Client Library requires Oracle Java 7 or later,

*  `Java 7 <http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html>`__
*  `Java 8 <https://java.com/en/download/>`__

Dependencies
-------------------------------------------------------------------------------

-  `Paho MQTT Java Client <http://git.eclipse.org/c/paho/org.eclipse.paho.mqtt.java.git/>`__   - provides a client class which enable applications to connect to an MQTT broker
-  `google-gson <https://code.google.com/p/google-gson/>`__   - library for interacting with JSON objects
-  `Apache Commons Logging <http://commons.apache.org/proper/commons-logging/download_logging.cgi>`__   - library for logging various informations
-  `Apache Commons Codec <https://commons.apache.org/proper/commons-codec/download_codec.cgi>`__  - provides common encoder and decoder functionalities such as Base64
-  `Apache Commons Lang <https://commons.apache.org/proper/commons-lang/download_lang.cgi>`__ - provides methods for manipulating core Java classes
-  `Apache Ant <http://ant.apache.org/>`__   - build tool for automated builds
-  `Apache HttpClient <https://hc.apache.org/downloads.cgi>`__   - A HTTP Client library
-  `Apache HttpCore <https://hc.apache.org/downloads.cgi>`__   - A HTTP Core library
-  `Joda-Time <http://www.joda.org/joda-time/download.html>`__ - The date and time library for Java 

----
