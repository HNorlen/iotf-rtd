===============================================================================
Device Model
===============================================================================

The device model describes the metadata and management characteristics of a 
device. The device database in the Watson IoT platform is the master 
source of device information. Applications and managed devices are able 
to send updates to the database such as a location or the progress of a firmware 
update. Once these updates are received by the Watson IoT platform, the 
device database is updated, making the information available to applications.

.. note:: With the exception of the management extension, the entire device model 
   is available for both managed and unmanaged devices. However, an unmanaged 
   device can not directly update its device model in the database.

--------


Device Identification
-------------------------------------------------------------------------------

Every device has a typeId and deviceId attribute. Typically, the 
typeId represents the model of your device, whilst the deviceId can represent its serial number. Within
your Watson IoT platform organization, the combination of typeId and deviceId must be unique for each device.

In addition, the Watson IoT platform constructs a further identifier for each device based on your organizationId, the device's typeId and the device's deviceId. This identifier - the clientId - allows you to identify unequivocally an individual device.  

The characters which can be used in these identifiers are restricted so they can be used straightforwardly in 
communication protocols and REST APIs. There are a number of optional device 
identifiers for use with the device management protocol. These identifiers are:

- deviceInfo.manufacturer
- deviceInfo.serialNumber
- deviceInfo.model
- deviceInfo.deviceClass
- deviceInfo.description

For more information on the identifiers and descriptions of their comparative 
identifiers in other device management standards, see Device Model Attributes Reference.


--------


Identifiers and Device Type
-------------------------------------------------------------------------------

Each device connected to the Watson IoT platform is associated with a device 
type. Device types are intended to be groups of devices which share 
characteristics or behaviour. 

A device type has a set of attributes. When a device is added to the Watson IoT platform, the attributes in its device type are used as a template overridden 
by device-specific attributes. For example, the device type could have a value for 
the ``deviceInfo.fwVersion`` attribute reflecting the firmware version at the time of 
manufacture, and this value would be copied from the device type into the devices as 
they are added. However, if a device was added which already had a value for 
``deviceInfo.fwVersion``, it would not be overridden by the device type.

When a device type is updated, future registered devices will reflect the modified device
type template. However, existing devices of this device type are unaffected and remain
unchanged.


--------


Attributes
-------------------------------------------------------------------------------

The table below shows the list of attributes which can apply to devices in the Watson IoT platform. Additionally, italicized attributes can also apply to device types.

- API/DMA: Can be updated by API/Device Management Agent

  - R: Read only
  - W: Write and read
  - A: Append

+----------------------------------+------------+---------------------------------------------------+-----+-----+
| Attribute                        | Type       | Description                                       | API | DMA |
+==================================+============+===================================================+=====+=====+
| clientId                         | string     | The client ID used with MQTT connections          |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *typeId*                         | string     | Device type                                       |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| deviceId                         | string     | Device ID                                         |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| classId                          | string     | Class of device ("Device" or "Gateway")           |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| gatewayTypeId                    | string     | Type ID of the gateway this device is using       |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| gatewayId                        | string     | Device ID of the gateway this device is using     |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| status.alert                     | boolean    | Whether the device has an alert                   |  W  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.serialNumber*        | string     | The serial number of the device                   |  W  |  W  |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.manufacturer*        | string     | The manufacturer of the device                    |  W  |  W  |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.model*               | string     | The model of the device                           |  W  |  W  |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.deviceClass*         | string     | The class of the device                           |  W  |  W  |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.description*         | string     | The descriptive name of the device                |  W  |  W  |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.fwVersion*           | string     | The firmware version currently known to be on     |  W  |  W  |
|                                  |            | the device                                        |     |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.hwVersion*           | string     | The hardware version of the device                |  W  |  W  |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *deviceInfo.descriptiveLocation* | string     | The descriptive location, such as a room or       |  W  |  W  |
|                                  |            | building number, or a geopgraphical region        |     |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| *metadata*                       | complex    | Free-form metadata                                |  W  |  W  |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| added.auth.id                    | string     | ID that added the device                          |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| added.dateTime                   | string     | ISO8601 date-time: Date and time the device was   |  R  |     |
|                                  |            | added                                             |     |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| refs.diag.errorCodes             | string     | URI of diag extension for error codes, if present |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| refs.diag.logs                   | string     | URI of diag extension for logs, if present        |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| refs.location                    | string     | URI of location extension, if present             |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+
| refs.mgmt                        | string     | URI of mgmt extension, is present                 |  R  |     |
+----------------------------------+------------+---------------------------------------------------+-----+-----+

--------

Extended Attributes
-------------------------------------------------------------------------------

In addition to core attributes listed above, there are additional attributes which 
are treated as extensions to the core device model. Simple queries about the device 
return information from the core device model, but not the extensions. Information 
from the extensions must be specifically requested.

+-------------------+----------------------+-------------------------------------------------------+
| Extension Name    | Prefix of attributes | Purpose                                               |
+===================+======================+=======================================================+
| Diagnostics       | diag                 | Error logs and diagnostic information                 |
+-------------------+----------------------+-------------------------------------------------------+
| Location          | location             | Location of the device, potentially updated regularly |
+-------------------+----------------------+-------------------------------------------------------+
| Device management | mgmt                 | Device management actions, e.g. firmware update       |
+-------------------+----------------------+-------------------------------------------------------+


Diagnostics Extension
~~~~~~~~~~~~~~~~~~~~~~

The diagnostics attributes are optional, and only present for devices with error log information. These attributes are intended for diagnosing device problems, not troubleshooting connectivity to the Watson IoT platform. In order to retrieve the information in these attributes, it must be queried separately, because the information stored in these attributes could potentially be very large. 

Diagnostic log information is an array of entries which can have entries appended using an API, however, this can cause earlier entries to be lost, to keep the size of diagnostic logs manageable. Each entry consists of a message, an indication of severity, a timestamp and an optional byte-array of data.

+----------------------+------------+-------------------------------------------------------------+-----+-----+
| Attribute            | Type       | Description                                                 | API | DMA |
+======================+============+=============================================================+=====+=====+
| diag.errorCodes[]    | array of   | Array of error codes                                        |  A  |  A  |
|                      | integer(s) |                                                             |     |     |
+----------------------+------------+-------------------------------------------------------------+-----+-----+
| diag.log[]           | array      | Array of diagnostic data                                    |  A  |  A  |
+----------------------+------------+-------------------------------------------------------------+-----+-----+
| diag.log[].message   | string     | Diagnostic message                                          |     |     |
+----------------------+------------+-------------------------------------------------------------+-----+-----+
| diag.log[].timestamp | string     | ISO8601 date-time: Date and time of log entry               |     |     |
+----------------------+------------+-------------------------------------------------------------+-----+-----+
| diag.log[].logData   | string     | byte: Diagnostic data, base-64 encoded                      |     |     |
+----------------------+------------+-------------------------------------------------------------+-----+-----+
| diag.log[].severity  | number     | Severity of message, 0: informational, 1: warning, 2: error |     |     |
+----------------------+------------+-------------------------------------------------------------+-----+-----+


Location Extension
~~~~~~~~~~~~~~~~~~~

These attributes are optional and only present for devices with location information. The location information is stored separately in order to allow the use of storage mechanisms better suited to dynamic information in the event of frequently updated information, for example, in the case of a mobile device.

For solutions which place significant importance on frequent location updates, it is expected that the location would be treated as part of the device's event payload, enabling higher update rates, simple historical storage, and analytics. 

+---------------------------+--------+---------------------------------------------------------+-----+-----+
| Attribute                 | Type   | Description                                             | API | DMA |
+===========================+========+=========================================================+=====+=====+
| location.longitude        | number | Longitude in decimal degrees using WGS84                |  W  |  W  |
+---------------------------+--------+---------------------------------------------------------+-----+-----+
| location.latitude         | number | Latitude in decimal degrees using WGS84                 |  W  |  W  |
+---------------------------+--------+---------------------------------------------------------+-----+-----+
| location.elevation        | number | Elevation in metres using WGS84                         |  W  |  W  |
+---------------------------+--------+---------------------------------------------------------+-----+-----+
| location.measuredDateTime | string |ISO8601 date-time: Date and time of location measurement |  W  |  W  |
+---------------------------+--------+---------------------------------------------------------+-----+-----+
| location.updatedDateTime  | string | ISO8601 date-time: Date and time                        |  R  |     |
+---------------------------+--------+---------------------------------------------------------+-----+-----+
| location.accuracy         | number | Accuracy of the position in metres                      |  W  |  W  |
+---------------------------+--------+---------------------------------------------------------+-----+-----+


Device Management Extension
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``mgmt.`` attributes are only present for managed devices. When a managed device becomes dormant, it becomes unmanaged and the ``mgmt.`` attributes are deleted. The ``mgmt.`` attributes are set by the Watson IoT platform as a result of processing device management requests. These attributes cannot be directly written using the API.

Devices have a management lifecycle, defined by their status as managed devices. The device management agent on the device is responsible for sending a Manage Device request using the device management protocol. To deal with defunct devices in large device populations, a managed device can be set to send a Manage Device request regularly, allowing the Watson IoT platform to notice when a device has become dormant. To facilitate this functionality, the Manage Device request has an optional lifetime parameter, When the Watson IoT platform receives a Manage Device request with a lifetime, it calculates the time before which another Manage Device request is required and stores it in the  "mgmt.dormantDateTime" attribute.

+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| Attribute                      | Type    | Description                                            | API | DMA |
+================================+=========+========================================================+=====+=====+
| mgmt.dormant                   | boolean | Whether the device has become dormant                  |  R  |     |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.dormantDateTime           | string  | ISO8601 date-time: Date and time at which the managed  |  R  |     |
|                                |         | device will become dormant                             |     |     |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.lastActivityDateTime      | string  | ISO8601 date-time: Date and time of last activity,     |  R  |     |
|                                |         | updated periodically                                   |     |     |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.supports.deviceActions    | boolean | Whether the device supports Reboot and Factory Reset   |  R  |     |
|                                |         | actions                                                |     |     |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.supports.firmwareActions  | boolean | Whether the device supports Firmware Download and      |  R  |     |
|                                |         | Firmware Update actions                                |     |     |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.firmware.version          | string  | The version of the firmware on the device              |  R  |  W  |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.firmware.name             | string  | The name of the firmware to be used on the device      |  R  |  W  |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.firmware.uri              | string  |The URI from which the firmware image can be downloaded |  R  |  W  |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.firmware.verifier         | string  | The verifier such as a checksum for the firmware image |  R  |  W  |
|                                |         | to validate its integrity                              |     |     |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.firmware.state            | number  | Indicates the state of firmware download               |  R  |  W  |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.firmware.updateStatus     | number  | Indicates the status of the update                     |  R  |  W  |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
| mgmt.firmware.updatedDateTime  | string  | ISO8601 date-time: Date of last update                 |  R  |     |
+--------------------------------+---------+--------------------------------------------------------+-----+-----+
