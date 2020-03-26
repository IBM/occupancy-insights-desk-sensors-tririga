## 4.5 Realtime data aggregation

Realtime aggregation flow starts whenever the TBI credentials are set and there are no validation failures. It works in the following way,

![realtime flow](../images/realtime-flow.PNG)

* Forms a PostgreSQL query to get the latest events of all the sensor devices in the last 24 hours from the raw sensor table 

```
SELECT * FROM (SELECT event, deviceid, tririgaorgid, tririgafloorid, timestamp, row_number() OVER (PARTITION BY deviceid ORDER BY timestamp DESC) AS rn  FROM iot_deviceType where tririgafloorid IS NOT NULL and tririgaorgid IS NOT NULL and (event = 1 or event = 2) and timestamp >= (NOW() - '1 day'::interval)) t2 WHERE rn = 1
```
* Query filters only those sensors for which occupancy data (either 1 or 2) is present
* Executes the query
* Aggregates occupancy data (vacant or busy) at org level based on metadata "tririgafloorid" and "tririgaorgid"
```
for(i=0;i<payload.length;i++){
    floorId = payload[i]['tririgafloorid']
    orgId = payload[i]['tririgaorgid']
    orgKey = floorId+":"+orgId
    if(!floors.includes(floorId)){
        floors.push(floorId)
        occdata[floorId] = 0
    }
    if(!floorOrgs.includes(orgKey)){
        floorOrgs.push(orgKey)
        occdata[orgKey] = 0
    }
    occupancy = 0
    if(payload[i]["event"] == 2){
        occupancy = 1
    }
    occdata[floorId] = occdata[floorId] + occupancy
    occdata[orgKey] = occdata[orgKey] + occupancy
}
```
* Formats the aggregated data into the form required by Watson IoT Platform(WIoTP) and TBI
* Uses the IBM IoT node to send the aggregated occupancy data into WIoTP platform
* Flow loops for every aggregation interval specified. Incase if one wants a more accurate data, then set the aggregation interval to lower value in configuration flow.
* **Note:** If your sensor data requires different way of aggregation, then modify the **PostgreSQL Query" and **Compute occupany** node in the flow with required aggregation logic.
