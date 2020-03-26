## 4.3 Configure Nodes ##
You need to configure the following nodes in the Flows
#### Configuration & validation Flow ####
1. In Configure TBI credentials function node

  ![Git](/images/configure_tbi_credentials.png)

 you need to change the following credentials  
```
credentials = {
    "username":"abc@in.ibm.com", // TBI Username
    "password":"*******************", // TBI Password
    "url":"https://TBI-AGG.mybluemix.net", // TBI URL
    "deviceType":"PIR",   // Sensor device type from WIoTP
    "aggregationInterval":15,  // Aggregation Interval in minutes
    "stopOnValidationfailure": true,
    "offset": "-05:00" // Building Offset
}
```
2. In Get devices from WIoTP Node you need to add your WIoTP credentials,

 ![Git](/images/configure_device_manager.png)

3. In that node you will find API key click on the small pen button to add your credentials

 ![Git](/images/edit_cred_button.png)

4. Now add your API Key and API token in the given Fields
 ![Git](/images/add_cred.png)

5. Then click on update or add and then done.

#### Realtime aggregation FLow ####
1. In Publish to WIoTP node you need to add your WIoTP credentials
![Git](/images/configure_wiotp_node.png)

2. In that node you will find API key click on the small pen button to add your credentials
![Git](/images/wiotp_edit_button.png)

3. Then you need to add your WIoTP credentials in the given **API key** and **API Token** Fields and add **Application ID** as any name
![Git](/images/wiotp_new_cred.png)

4. Then click on update or add and then click on done to add your WIoTP credentials to sensor_support.

#### Historical data aggregation ####
1. In the Read dataa node you need to configure with PostgreSQL credentials
![Git](/images/postgres_cred.png)

2. In this node click on the edit button to add your postgreSQL credentials
![Git](/images/postgres_edit_button.png)

3. Now you need to add the following credentials to this node
```
  1. Host
  2. Port
  3. Database
  4. Username
  5. Password
  6. Add mark Use SSL
```
![Git](/images/postgres_add.png)

4. Click on update or add and then Done
