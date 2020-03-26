## 4.6 Historical data aggregation

Historical data aggregation aims to work on the historical data if any and provide the occupancy by the aggregation interval specified. The flow needs to be triggered manually and it works in the following way,

![historical flow](../images/historical-flow.PNG)

* Triggers the flow if its not running already
* Forms a PostgreSQL query to get the least time where the first entry is present,
```
select timestamp from iot_deviceType order by timestamp asc limit 1
```
* Executes the query and retrieves the time from PostgreSQL
* Forms the start time and end time based on the time retrieved and aggregation interval
* Forms another PostgreSQL query to get the data between the start time and end time
```
SELECT * FROM (SELECT event, deviceid, tririgaorgid, tririgafloorid, timestamp, row_number() OVER (PARTITION BY deviceid ORDER BY timestamp DESC) AS rn  FROM (select * from iot_deviceType" where timestamp > start_time and timestamp <= end_time as tmp where event = 1 or event = 2) t2 WHERE rn = 1
```
* Query filters only those sensors for which occupancy data (either 1 or 2) is present
* Executes the query
* Aggregates occupancy data (vacant or busy) at org level based on metadata "tririgafloorid" and "tririgaorgid"

```
    floorOrgs = global.get("hist_floorOrgs")
    sensors_state = global.get("hist_sensors_state")
    occdata = global.get("hist_occdata")
    floors = global.get("hist_floors")
    var occupancy = 0
    for(i=0;i<payload.length;i++){
        prev_state = 1
        if(payload[i]['deviceid'] in sensors_state){
            prev_state = sensors_state[payload[i]['deviceid']]
        }
        curr_state = payload[i]["event"]
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
        if(curr_state == 2 && prev_state == 1){
            occupancy = 1
        }
        else if(curr_state == 1 && prev_state == 2){
            occupancy = -1
        }
        else{
            occupancy = 0
        }
        occdata[floorId] = occdata[floorId] + occupancy
        occdata[orgKey] = occdata[orgKey] + occupancy
        sensors_state[payload[i]['deviceid']] = curr_state
    }
```

* Maintains the state of each sensor such that the occupancy count is calculated from the previous state when the data comes in for the next batch
* Forms a next batch request where the start time and end time is adjusted to next cycle based on aggregation interval specified
```
    start_time = new Date(global.get("hist_start_time"))
    end_time = new Date(global.get("hist_start_time"))
    end_time.setMinutes(new Date(end_time).getMinutes() + credentials.aggregationInterval)
```
* The flow also parallely formats the aggregated data into the form required by TBI and uses the IBM IoT node to send the aggregated occupancy data into WIoTP platform
* Flow loops untill the current time is reached.
* **Note:** If your sensor data requires different way of aggregation, then modify the **Form SQL Query" and **Compute occupany** node in the flow with required aggregation logic.
