======================================
Java Client Library - Managed Devices
======================================

Introduction
-------------

This client library describes how to use devices with the Java ibmiotf client library. For help with getting started with this module, see `Java Client Library - Introduction <../java/javaintro.html>`__. 

This section contains information on how devices can connect to the Internet of Things Foundation Device Management service using Java and perform device management operations like firmware update, location update, and diagnostics update.

The Device section contains information on how devices can publish events and handle commands using the Java ibmiotf Client Library. 

The Applications section contains information on how applications can use the Java ibmiotf Client Library to interact with devices. 


Device Management
-------------------------------------------------------------------------------
The `device management <../reference/device_mgmt.html>`__ feature enhances the Internet of Things Foundation Connect service with new capabilities for managing devices. Device management makes a distinction between managed and unmanaged devices:

* **Managed Devices** are defined as devices which have a management agent installed. The management agent sends and receives device metadata and responds to device management commands from the Internet of Things Foundation Connect. 
* **Unmanaged Devices** are any devices which do not have a device management agent. All devices begin their lifecycle as unmanaged devices, and can transition to managed devices by sending a message from a device management agent to the Internet of Things Foundation Connect. 


---------------------------------------------------------------------------
Connecting to the Internet of Things Foundation Device Management Service
---------------------------------------------------------------------------

Create DeviceData
------------------------------------------------------------------------
The `device model <../reference/device_model.html>`__ describes the metadata and management characteristics of a device. The device database in the Internet of Things Foundation Connect is the master source of device information. Applications and managed devices are able to send updates to the database such as a location or the progress of a firmware update. Once these updates are received by the Internet of Things Foundation Connect, the device database is updated, making the information available to applications.

The device model in the ibmiotf client library is represented as DeviceData and to create a DeviceData one needs to create the following objects,

* DeviceInfo (mandatory)
* DeviceLocation (required if the device supports location update)
* DiagnosticErrorCode (required if the device wants to update the ErrorCode)
* DiagnosticLog (required if the device wants to update Log information)
* DeviceFirmware (required if the device supports Firmware Actions)
* DeviceMetadata (optional)

The following code snippet shows how to create the mandatory object DeviceInfo along with an optional object DeviceMetadata with sample data:

.. code:: java

     DeviceInfo deviceInfo = new DeviceInfo.Builder().
				serialNumber("10087").
				manufacturer("IBM").
				model("7865").
				deviceClass("A").
				description("My RasPi Device").
				fwVersion("1.0.0").
				hwVersion("1.0").
				descriptiveLocation("EGL C").
				build();
	
     /**
       * Create a DeviceMetadata object 
      **/
     JsonObject data = new JsonObject();
     data.addProperty("customField", "customValue");
     DeviceMetadata metadata = new DeviceMetadata(data);

The following code snippet shows how to create the DeviceData object with the above created DeviceInfo and DeviceMetadata objects:

.. code:: java

	DeviceData deviceData = new DeviceData.Builder().
				 deviceInfo(deviceInfo).
				 metadata(metadata).
				 build();
Construct ManagedDevice
-------------------------------------------------------------------------------
ManagedDevice - A device class that connects the device as managed device to Internet of Things Foundation Connect and enables the device to perform one or more Device Management operations. Also the ManagedDevice instance can be used to do normal device operations like publishing device events and listening for commands from application.

ManagedDevice exposes 2 different constructors to support different user patterns, 

**Constructor One**

Constructs a ManagedDevice instance by accepting the DeviceData and the following properties,

* Organization-ID - Your organization ID.
* Device-Type - The type of your device.
* Device-ID - The ID of your device.
* Authentication-Method - Method of authentication (The only value currently supported is "token"). 
* Authentication-Token - API key token

All these properties are required to interact with the Internet of Things Foundation Connect. 

The following code shows how to create a ManagedDevice instance:

.. code:: java

	Properties options = new Properties();
	options.setProperty("Organization-ID", "uguhsp");
	options.setProperty("Device-Type", "iotsample-arduino");
	options.setProperty("Device-ID", "00aabbccde03");
	options.setProperty("Authentication-Method", "token");
	options.setProperty("Authentication-Token", "AUTH TOKEN FOR DEVICE");
	
	ManagedDevice managedDevice = new ManagedDevice(options, deviceData);
 
The existing users of DeviceClient might observe that the names of these properties have changed slightly. These names have been changed to mirror the names in the Internet of Things Foundation Connect Dashboard, but the existing users who want to migrate from the DeviceClient to the ManagedDevice can still use the old format and construct the ManagedDevice instance as follows:

.. code:: java

	Properties options = new Properties();
	options.setProperty("org", "uguhsp");
	options.setProperty("type", "iotsample-arduino");
	options.setProperty("id", "00aabbccde03");
	options.setProperty("auth-method", "token");
	options.setProperty("auth-token", "AUTH TOKEN FOR DEVICE");
	ManagedDevice managedDevice = new ManagedDevice(options, deviceData);

**Constructor Two**

Construct a ManagedDevice instance by accepting the DeviceData and the MqttClient instance. This constructor requires the DeviceData to be created with additional device attributes like Device Type and Device Id as follows:

.. code:: java
	
	// Code that constructs the MqttClient (either Synchronous or Asynchronous MqttClient)
	.....
	
	// Code that constructs the DeviceData
	DeviceData deviceData = new DeviceData.Builder().
				 typeId("Device-Type").
				 deviceId("Device-ID").
				 deviceInfo(deviceInfo).
				 metadata(metadata).
				 build();
	
	....
	ManagedDevice managedDevice = new ManagedDevice(mqttClient, deviceData);
	
Note this constructor helps the custom device users to create a ManagedDevice instance with the already created and connected MqttClient instance to take advantage of device management operations. But we recommend the users to use the library for all the device functionalities.

Manage	
------------------------------------------------------------------
The device can invoke manage() method to participate in device management activities. The manage request will initiate a connect request internally if the device is not connected to the Internet of Things Foundation Connect already:

.. code:: java

	managedDevice.manage();
	
The device can use overloaded manage (lifetime) method to register the device for a given timeframe. The timeframe specifies the length of time within which the device must send another **Manage device** request in order to avoid being reverted to an unmanaged device and marked as dormant.

.. code:: java

    managedDevice.manage(3600);

Refer to the `documentation <../device_mgmt/operations/manage.html>`__ for more information about the manage operation.

Unmanage
-----------------------------------------------------

A device can invoke unmanage() method when it no longer needs to be managed. The Internet of Things Foundation Connect will no longer send new device management requests to this device and all device management requests from this device will be rejected other than a **Manage device** request.

.. code:: java

	managedDevice.unmanage();

Refer to the `documentation <../device_mgmt/operations/manage.html>`__ for more information about the Unmanage operation.

Location Update
-----------------------------------------------------

Devices that can determine their location can choose to notify the Internet of Things Foundation Connect about location changes. In order to update the location, the device needs to create DeviceData instance with the DeviceLocation object first.

.. code:: java

    // Construct the location object with latitude, longitude and elevation
    DeviceLocation deviceLocation = new DeviceLocation.Builder(30.28565, -97.73921).
								elevation(10).
								build();
    DeviceData deviceData = new DeviceData.Builder().
				 deviceInfo(deviceInfo).
				 deviceLocation(deviceLocation).
				 metadata(metadata).
				 build();
	
    
Once the device is connected to Internet of Things Foundation Connect, the location can be updated by invoking the following method:

.. code:: java

	int rc = deviceLocation.sendLocation();
	if(rc == 200) {
	    	System.out.println("Current location (" + deviceLocation.toString() + ")");
	} else {
            	System.err.println("Failed to update the location");
	}

Later, any new location can be updated by changing the properties of the DeviceLocation object:

.. code:: java

	int rc = deviceLocation.update(40.28, -98.33, 11);
	if(rc == 200) {
		System.out.println("Current location (" + deviceLocation.toString() + ")");
	} else {
		System.err.println("Failed to update the location");
	}

The update() method informs the Internet of Things Foundation Connect about the new location.

Refer to the `documentation <../device_mgmt/operations/update.html>`__ for more information about the Location update.

Append/Clear ErrorCodes
-----------------------------------------------

Devices can choose to notify the Internet of Things Foundation Connect about changes in their error status. In order to send the ErrorCodes the device needs to construct a DiagnosticErrorCode object as follows:

.. code:: java

	DiagnosticErrorCode errorCode = new DiagnosticErrorCode(0);
	
	DeviceData deviceData = new DeviceData.Builder().
				 deviceInfo(deviceInfo).
				 deviceErrorCode(errorCode).
				 metadata(metadata).
				 build();

Once the device is connected to Internet of Things Foundation Connect, the ErrorCode can be sent by calling the send() method as follows:

.. code:: java

	errorCode.send();

Later, any new ErrorCodes can be easily added to the Internet of Things Foundation Connect by calling the append method as follows:

.. code:: java

	int rc = errorCode.append(500);
	if(rc == 200) {
		System.out.println("Current Errorcode (" + errorCode + ")");
	} else {
		System.out.println("Errorcode addition failed!");
	}

Also, the ErrorCodes can be cleared from Internet of Things Foundation Connect by calling the clear() method as follows:

.. code:: java

	int rc = errorCode.clear();
	if(rc == 200) {
		System.out.println("ErrorCodes are cleared successfully!");
	} else {
		System.out.println("Failed to clear the ErrorCodes!");
	}

Append/Clear Log messages
-----------------------------
Devices can choose to notify the Internet of Things Foundation Connect about changes by adding a new log entry. Log entry includes a log messages, its timestamp and severity, as well as an optional base64-encoded binary diagnostic data. In order to send log messages, the device needs to construct a DiagnosticLog object as follows:

.. code:: java

	DiagnosticLog log = new DiagnosticLog(
				"Simple Log Message", 
				new Date(),
				DiagnosticLog.LogSeverity.informational);
		
	DeviceData deviceData = new DeviceData.Builder().
				 deviceInfo(deviceInfo).
				 deviceLog(log).
				 metadata(metadata).
				 build();

Once the device is connected to Internet of Things Foundation Connect, the log message can be sent by calling the send() method as follows:

.. code:: java

	log.send();

Later, any new log messages can be easily added to the Internet of Things Foundation Connect by calling the append method as follows:

.. code:: java

	int rc = log.append("sample log", new Date(), DiagnosticLog.LogSeverity.informational);
			
	if(rc == 200) {
		System.out.println("Current Log (" + log + ")");
	} else {
		System.out.println("Log Addition failed");
	}

Also, the log messages can be cleared from Internet of Things Foundation Connect by calling the clear method as follows:

.. code:: java

	rc = log.clear();
	if(rc == 200) {
		System.out.println("Logs are cleared successfully");
	} else {
		System.out.println("Failed to clear the Logs")
	}	

The device diagnostics operations are intended to provide information on device errors, and does not provide diagnostic information relating to the devices connection to the Internet of Things Foundation Connect.

Refer to the `documentation <../device_mgmt/operations/diagnostics.html>`__ for more information about the Diagnostics operation.

Firmware Actions
-------------------------------------------------------------
The firmware update process is separated into two distinct actions:

* Downloading Firmware 
* Updating Firmware. 

The device needs to do the following activities to support Firmware Actions:

**1. Construct DeviceFirmware Object**

In order to perform Firmware actions the device needs to construct the DeviceFirmware object and add it to DeviceData as follows:

.. code:: java

	DeviceFirmware firmware = new DeviceFirmware.Builder().
				version("Firmware.version").
				name("Firmware.name").
				url("Firmware.url").
				verifier("Firmware.verifier").
				state(FirmwareState.IDLE).				
				build();
				
	DeviceData deviceData = new DeviceData.Builder().
				deviceInfo(deviceInfo).
				deviceFirmware(firmware).
				metadata(metadata).
				build();
	
	ManagedDevice managedDevice = new ManagedDevice(options, deviceData);
	managedDevice.connect();
		

The DeviceFirmware object represents the current firmware of the device and will be used to report the status of the Firmware Download and Firmware Update actions to Internet of Things Foundation Connect.

**2. Inform the server about the Firmware action support**

The device needs to set the firmware action flag to true in order for the server to initiate the firmware request. This can be achieved by invoking a following method with a boolean value:

.. code:: java

    	managedDevice.supportsFirmwareActions(true);
    	managedDevice.manage();
	
As the manage request informs the Internet of Things Foundation Connect about the firmware action support, manage() method needs to be called right after setting the firmware action support.

**3. Create the Firmware Action Handler**

In order to support the Firmware action, the device needs to create a handler and add it to ManagedDevice. The handler must extend a DeviceFirmwareHandler class and implement the following methods:

.. code:: java

	public abstract void downloadFirmware(DeviceFirmware deviceFirmware);
	public abstract void updateFirmware(DeviceFirmware deviceFirmware);

**3.1 Sample implementation of downloadFirmware**

The implementation must add logic to download the firmware and report the status of the download via DeviceFirmware object. If the Firmware Download operation is successful, then the state of the firmware to be set to DOWNLOADED and UpdateStatus should be set to SUCCESS.

If an error occurs during Firmware Download the state should be set to IDLE and updateStatus should be set to one of the error status values:

* OUT_OF_MEMORY
* CONNECTION_LOST
* INVALID_URI

A sample Firmware Download implementation for a Raspberry Pi device is shown below:

.. code:: java

	public void downloadFirmware(DeviceFirmware deviceFirmware) {
		boolean success = false;
		URL firmwareURL = null;
		URLConnection urlConnection = null;
		
		try {
			firmwareURL = new URL(deviceFirmware.getUrl());
			urlConnection = firmwareURL.openConnection();
			if(deviceFirmware.getName() != null) {
				downloadedFirmwareName = deviceFirmware.getName();
			} else {
				// use the timestamp as the name
				downloadedFirmwareName = "firmware_" +new Date().getTime()+".deb";
			}
			
			File file = new File(downloadedFirmwareName);
			BufferedInputStream bis = new BufferedInputStream(urlConnection.getInputStream());
			BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(file.getName()));
			
			int data = bis.read();
			if(data != -1) {
				bos.write(data);
				byte[] block = new byte[1024];
				while (true) {
					int len = bis.read(block, 0, block.length);
					if(len != -1) {
						bos.write(block, 0, len);
					} else {
						break;
					}
				}
				bos.close();
				bis.close();
				success = true;
			} else {
				//There is no data to read, so set an error
				deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.INVALID_URI);
			}
		} catch(MalformedURLException me) {
			// Invalid URL, so set the status to reflect the same,
			deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.INVALID_URI);
		} catch (IOException e) {
			deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.CONNECTION_LOST);
		} catch (OutOfMemoryError oom) {
			deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.OUT_OF_MEMORY);
		}
		
		if(success == true) {
			deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.SUCCESS);
			deviceFirmware.setState(FirmwareState.DOWNLOADED);
		} else {
			deviceFirmware.setState(FirmwareState.IDLE);
		}
	}

Device can check the integrity of the downloaded firmware image using the verifier and report the status back to Internet of Things Foundation Connect. The verifier can be set by the device during the startup (while creating the DeviceFirmware Object) or as part of the Download Firmware request by the application. A sample code to verify the same is below:

.. code:: java

	private boolean verifyFirmware(File file, String verifier) throws IOException {
		FileInputStream fis = null;
		String md5 = null;
		try {
			fis = new FileInputStream(file);
			md5 = org.apache.commons.codec.digest.DigestUtils.md5Hex(fis);
			System.out.println("Downloaded Firmware MD5 sum:: "+ md5);
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			fis.close();
		}
		if(verifier.equals(md5)) {
			System.out.println("Firmware verification successful");
			return true;
		}
		System.out.println("Download firmware checksum verification failed.. "
				+ "Expected "+verifier + " found "+md5);
		return false;
	}

The complete code can be found in the device management sample `RasPiFirmwareHandlerSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/RasPiFirmwareHandlerSample.java>`__.

**3.2 Sample implementation of updateFirmware**

The implementation must add logic to install the downloaded firmware and report the status of the update via DeviceFirmware object. If the Firmware Update operation is successful, then the state of the firmware should to be set to IDLE and UpdateStatus should be set to SUCCESS. 

If an error occurs during Firmware Update, updateStatus should be set to one of the error status values:

* OUT_OF_MEMORY
* UNSUPPORTED_IMAGE
			
A sample Firmware Update implementation for a Raspberry Pi device is shown below:

.. code:: java
	
	public void updateFirmware(DeviceFirmware deviceFirmware) {
		try {
			ProcessBuilder pkgInstaller = null;
			Process p = null;
			pkgInstaller = new ProcessBuilder("sudo", "dpkg", "-i", downloadedFirmwareName);
			boolean success = false;
			try {
				p = pkgInstaller.start();
				boolean status = waitForCompletion(p, 5);
				if(status == false) {
					p.destroy();
					deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.UNSUPPORTED_IMAGE);
					return;
				}
				System.out.println("Firmware Update command "+status);
				deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.SUCCESS);
				deviceFirmware.setState(FirmwareState.IDLE);
			} catch (IOException e) {
				e.printStackTrace();
				deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.UNSUPPORTED_IMAGE);
			} catch (InterruptedException e) {
				e.printStackTrace();
				deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.UNSUPPORTED_IMAGE);
			}
		} catch (OutOfMemoryError oom) {
			deviceFirmware.setUpdateStatus(FirmwareUpdateStatus.OUT_OF_MEMORY);
		}
	}

The complete code can be found in the device management sample `RasPiFirmwareHandlerSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/RasPiFirmwareHandlerSample.java>`__.

**4. Add the handler to ManagedDevice**

The created handler needs to be added to the ManagedDevice instance so that the ibmiotf client library invokes the corresponding method when there is a Firmware action request from Internet of Things Foundation Connect.

.. code:: java

	DeviceFirmwareHandlerSample fwHandler = new DeviceFirmwareHandlerSample();
	deviceData.addFirmwareHandler(fwHandler);

Refer to `this page <../device_mgmt/operations/firmware_actions.html>`__ for more information about the Firmware action.

Device Actions
------------------------------------
The Internet of Things Foundation Connect supports the following device actions:

* Reboot
* Factory Reset

The device needs to do the following activities to support Device Actions:

**1. Inform server about the Device Actions support**

In order to perform Reboot and Factory Reset, the device needs to inform the Internet of Things Foundation Connect about its support first. This can achieved by invoking a following method with a boolean value:

.. code:: java
	
	managedDevice.supportsDeviceActions(true);
    	managedDevice.manage();
	
As the manage request informs the Internet of Things Foundation Connect about the device action support, manage() method needs to be called right after setting the device action support.
	
**2. Create the Device Action Handler**

In order to support the device action, the device needs to create a handler and add it to ManagedDevice. The handler must extend a DeviceActionHandler class and provide implementation for the following methods:

.. code:: java

	public abstract void handleReboot(DeviceAction action);
	public abstract void handleFactoryReset(DeviceAction action);

**2.1 Sample implementation of handleReboot**

The implementation must add a logic to reboot the device and report the status of the reboot via DeviceAction object. The device needs to update the status along with a optional message only when there is a failure (because the successful operation reboots the device and the device code will not have a control to update the Internet of Things Foundation Connect). A sample reboot implementation for a Raspberry Pi device is shown below:

.. code:: java

	public void handleReboot(DeviceAction action) {
		ProcessBuilder processBuilder = null;
		Process p = null;
		processBuilder = new ProcessBuilder("sudo", "shutdown", "-r", "now");
		boolean status = false;
		try {
			p = processBuilder.start();
			// wait for say 2 minutes before giving it up
			status = waitForCompletion(p, 2);
		} catch (IOException e) {
			action.setMessage(e.getMessage());
		} catch (InterruptedException e) {
			action.setMessage(e.getMessage());
		}
		if(status == false) {
			action.setStatus(DeviceAction.Status.FAILED);
		}
	}

The complete code can be found in the device management sample `DeviceActionHandlerSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/DeviceActionHandlerSample.java>`__.

**2.2 Sample implementation of handleFactoryReset**

The implementation must add a logic to reset the device to factory settings and report the status via DeviceAction object. The device needs to update the status along with a optional message only when there is a failure (because as part of this process, the device reboots and the device will not have a control to update status to Internet of Things Foundation Connect). The skeleton of the Factory Reset implementation is shown below:

.. code:: java
	
	public void handleFactoryReset(DeviceAction action) {
		try {
			// code to perform Factory reset
		} catch (IOException e) {
			action.setMessage(e.getMessage());
		}
		if(status == false) {
			action.setStatus(DeviceAction.Status.FAILED);
		}
	}

**3. Add the handler to ManagedDevice**

The created handler needs to be added to the ManagedDevice instance so that the ibmiotf client library invokes the corresponding method when there is a device action request from Internet of Things Foundation Connect.

.. code:: java

	DeviceActionHandlerSample actionHandler = new DeviceActionHandlerSample();
	deviceData.addDeviceActionHandler(actionHandler);

Refer to `this page <../device_mgmt/operations/device_actions.html>`__ for more information about the Device Action.

Listen for Device attribute changes
-----------------------------------------------------------------

This ibmiotf client library updates the corresponding objects whenever there is an update request from the Internet of Things Foundation Connect, these update requests are initiated by the application either directly or indirectly (Firmware Update) via the Internet of Things Foundation Connect ReST API. Apart from updating these attributes, the library provides a mechanism where the device can be notified whenever a device attribute is updated.

Attributes that can be updated by this operation are location, metadata, device information and firmware.

In order to get notified, the device needs to add a property change listener on those objects that it is interested.

.. code:: java

	deviceLocation.addPropertyChangeListener(listener);
	firmware.addPropertyChangeListener(listener);
	deviceInfo.addPropertyChangeListener(listener);
	metadata.addPropertyChangeListener(listener);
	
Also, the device needs to implement the propertyChange() method where it receives the notification. A sample implementation is as follows:

.. code:: java

	public void propertyChange(PropertyChangeEvent evt) {
		if(evt.getNewValue() == null) {
			return;
		}
		Object value = (Object) evt.getNewValue();
		
		switch(evt.getPropertyName()) {
			case "metadata":
				DeviceMetadata metadata = (DeviceMetadata) value;
				System.out.println("Received an updated metadata -- "+ metadata);
				break;
			
			case "location":
				DeviceLocation location = (DeviceLocation) value;
				System.out.println("Received an updated location -- "+ location);
				break;
			
			case "deviceInfo":
				DeviceInfo info = (DeviceInfo) value;
				System.out.println("Received an updated device info -- "+ info);
				break;
				
			case "mgmt.firmware":
				DeviceFirmware firmware = (DeviceFirmware) value;
				System.out.println("Received an updated device firmware -- "+ firmware);
				break;		
		}
	}

Refer to `this page <../device_mgmt/operations/update.html>`__ for more information about updating the device attributes.

Examples
-------------
* `SampleRasPiDMAgent <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/SampleRasPiDMAgent.java>`__ - A sample agent code that shows how to perform various device management operations on Raspberry Pi.
* `SampleRasPiManagedDevice <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/SampleRasPiManagedDevice.java>`__ - A sample code that shows how one can perform both device operations and management operations.
* `SampleRasPiDMAgentWithCustomMqttAsyncClient <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/SampleRasPiDMAgentWithCustomMqttAsyncClient.java>`__ - A sample agent code with custom MqttAsyncClient.
* `SampleRasPiDMAgentWithCustomMqttClient <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/SampleRasPiDMAgentWithCustomMqttClient.java>`__ - A sample agent code with custom MqttClient.
* `RasPiFirmwareHandlerSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/RasPiFirmwareHandlerSample.java>`__ - A sample implementation of FirmwareHandler for Raspberry Pi.
* `DeviceActionHandlerSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/DeviceActionHandlerSample.java>`__ - A sample implementation of DeviceActionHandler
* `ManagedDeviceWithLifetimeSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/ManagedDeviceWithLifetimeSample.java>`__ - A sample that shows how to send regular manage request with lifetime specified.
* `DeviceAttributesUpdateListenerSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/DeviceAttributesUpdateListenerSample.java>`__ - A sample listener code that shows how to listen for a various device attribute changes .
* `NonBlockingDiagnosticsErrorCodeUpdateSample <https://github.com/ibm-messaging/iot-java/blob/master/samples/iotfdevicemanagement/src/com/ibm/iotf/sample/devicemgmt/device/NonBlockingDiagnosticsErrorCodeUpdateSample.java>`__ - A sample that shows how to add ErrorCode without waiting for response from the server.

Recipe
----------

Refer to `the recipe <https://developer.ibm.com/recipes/tutorials/connect-raspberry-pi-as-managed-device-to-ibm-iot-foundation/>`__ that shows how to connect the Raspberry Pi device as managed device to Internet of Things Foundation Connect to perform various device management operations in step by step using this client library.
