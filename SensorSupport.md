## Sensor Support (PoC Approach)

In Jan 2020 release, sensors (any type) are not supported out of the box by TBI. However following approach is provided to enable our business partners in providing sensor support, specifically desk level sensors

### Pre-req ###
 
This section assumes that user will have the following instances obtained and configured,
- TRIRIGA
- TRIRIGA Building Insights

***

### Functional Components required for Sensor Support ###

![Sensor Support](/2H_2019_Release/Hill1/images/SensorSupport.png)

### steps for supporting sensors ###

![Sensor Support steps ](/2H_2019_Release/Hill1/images/sensorSupportSteps.png)


### Steps for TRIRIGA System Administrator
#### 3. IOT Platform
- 3.1 Create device type
- 3.2 [Create devices, add metadata, connect devices](#define-sensor-devices-in-wiotp)
- 3.3 [Create physical and logical interface for the device typeW](#create-physical--logical-interfaces-for-sensor-devices-in-wiotp)
- 3.4 [Verify raw sensor data in PostgreSQL](#verify-raw-sensor-data-in-postgresql)
#### 4. IoT Sensor support component
- 4.1 [Deploy sensor support component(Node-RED) in Cloud](https://github.ibm.com/IoT4Buildings/Desk-Sensor-Support/blob/master/docs/Install_Nodes.md)
- 4.2 [Download and import flow from git](https://github.ibm.com/IoT4Buildings/Desk-Sensor-Support/blob/master/docs/Install_Nodes.md#import-flows)
- 4.3 [Modify the flow to support the required sensors](https://github.ibm.com/IoT4Buildings/Desk-Sensor-Support/blob/master/docs/configure_nodes.md)
- 4.4 [Validate the metadata](https://github.ibm.com/IoT4Buildings/Desk-Sensor-Support/blob/master/docs/validation-flow.md)
- 4.5 [Trigger realtime data aggreation flow](https://github.ibm.com/IoT4Buildings/Desk-Sensor-Support/blob/master/docs/realtime-aggregation-flow.md)
- 4.6 [Trigger historical data aggregation flow](https://github.ibm.com/IoT4Buildings/Desk-Sensor-Support/blob/master/docs/historical-flow.md)
- View results in TBI dashboard

***
#### Sensor schema ####

The reference implementation supports a desk level sensor with the following schema,

```
{
    "timestamp": "12-02-2020 12:52:44",
    "bat": 5,
    "event": 0
}

where,

bat represents the battery level of the sensor
event represents:
0 = Keep Alive
1 = Vacant
2 = Busy
3 = First Transmission
```

#### TBI schema
TBI expects the occupancy information for an organization and not individual desk. The details of data schema is as follows,

```
{
    "time": "12-02-2020T12:52:44.000Z",
    "occupancycount": 50,
    "orgoccupancycount": 30,
    "orgid": "145786",
    "deviceid": "854609"
}
```

where,

| Property          | Type              | Created out of the Box | Comments|
|-------------------|-------------------|------------------------|---------|
| time              | string(date-time) | No                     |  Time at which the occupancy is derived.   Time must be in UTC and in ISO8601 format yyyy-dd-mm'T'hh:mm:ss.sssZ.|
| occupancycount    | number            | No                     |  The derived occupancy count for the given floor   at the specified time.|
| orgoccupancycount | number            | No                     |  The derived occupancy count of an organization(BU) for a specific floor.|
| orgid             | string            | No                     |  The system record id of the organization in TRIRIGA for which the occupancy is calculated.|
| deviceid          | string            | Yes                    |  The system record id of the floor in TRIRIGA for which the occupancy is calculated. This is the deviceid against which the event needs be sent so that it gets added in the appropriate table. |

***


### Define sensor devices in WIoTP
In order to connect sensor devices to Watson IoT Platform(WIoTP), first it needs to be registered in WIoTP. [This tutorial](https://cloud.ibm.com/docs/services/IoT?topic=iot-platform-getting-started#step1) provides details about how one can register the device type and devices in WIoTP.
While registering each of the desk level sensor to Watson IoT Platform, the following information needs to be provided as metadata for each of the sensors

* **tririgaFloorId:** This id is the system record id of the floor in TRIRIGA under which the desklevel sensors are deployed
* **tririgaOrgId:** This id is thesystem record id of chargeback organization in TRIRIGA to which the seat is allocated

Following are the steps to add the same in metadata of the devices,
* Click **Edit Metadata** button while registering the deive
![edit-metadata](/2H_2019_Release/Hill1/images/edit-metadata.png)
* Add **tririgaFloorId** and **tririgaOrgId** obtained from TRIRIGA as shown below
![add-metadata](/2H_2019_Release/Hill1/images/add-metadata.png)

***

### Create physical & logical interfaces for sensor devices in WIoTP
The reference implementation of sensor support component expects the sensor data in PostgreSQL and in the following format. 
```
event - Number 
timestamp - data-time in ISO-8601 format
tririgaorgid - String
tririgafloorid - String
```

Example,
```
event - 1 
timestamp - "2020-02-27T17:27:21.000"
tririgaorgid - "546876"
tririgafloorid - "925410"
```
to persist the sensor data in PostgreSQL, physical & logical interface needs to be created in WIoTP. Following are the steps to create the same,

* Make sure that the sensor devices are sending the data so that the creation of interfaces are easy
* Click **Create Physical Interface** from device type page,
![physical-interface](/2H_2019_Release/Hill1/images/physical-interface.png)
* Derive properties from running devices as shown below, (**Note** that the schema may be different based on the sensor)
![pysical-derive-properties](/2H_2019_Release/Hill1/images/pysical-derive-properties.png)
* Click **Create Logical Interface** as shown below,
![logical-interface](/2H_2019_Release/Hill1/images/logical-interface.png)
* Map the sensor data properties as shown below,
![map-properties-logical](/2H_2019_Release/Hill1/images/map-properties-logical.png)
* Add the metadata properties as shown below,
![tririga-properties-logical](/2H_2019_Release/Hill1/images/tririga-properties-logical.png)
* Also, if required transform the timestamp to ISO-8601 format if the original timestamp is not in ISO-8601 format, For example, use the following transformation logic,
```
$join([ $substring($event.timestamp, 6, 4), '-', $substring($event.timestamp, 3, 2), '-', $substring($event.timestamp, 0, 2), 'T', $substring($event.timestamp, 11, 2), ":", $substring($event.timestamp, 14, 2), ":", $substring($event.timestamp, 17, 2), "+03:00"])
```
* Click activate to activate the interface. WIoTP will create a table with name iot_deviceType (Example, iot_SS10) in PostgreSQL and move the raw sensor data.

***

### Verify raw sensor data in PostgreSQL
* Connect to PostgreSQL and verify that table with name "iot_deviceType" exists
* Verify that it receives the latest data in the schema set by logical interface. For example,
![verify-raw-data](/2H_2019_Release/Hill1/images/verify-raw-data.PNG)

***
