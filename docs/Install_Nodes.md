This section describes how to install Node-RED with required packages in Cloud.

### 4. Deploy Node-RED on IBM Cloud
Refer to [IBM Developer tutorial](https://developer.ibm.com/tutorials/how-to-create-a-node-red-starter-application/) for how to create a new Node-RED Starter Application in IBM Cloud.

***

### 4.1 Add extra nodes to your Node-RED palette
- On your application’s details page, click **View toolchain**.
- In the Continuous Delivery box, click **git**. This step takes you to a git repository where you can edit the application source code from your browser.
![Git](/images/toolchain-git.PNG)
- Scroll down the list of files and click on **package.json**. This file lists the module dependencies of your application.
![Package.json](/images/package.json)
- Click the **Edit** button.
![Edit package](/images/edit-package.PNG)
- Add the following entries to the top of the dependencies section (1):
```
"node-red-dashboard": "2.19.4",
"node-red-contrib-scx-ibmiotapp": "0.0.49",
"node-red-contrib-postgres-multi": "0.2.0",
"node-red-contrib-loop-processing": "0.3.1",
"node-red-contrib-ibm-wiotp-device-ops": "0.1.3"
```

**Note:** Do not forget the comma (,) at the end of the line to separate it from the next entry.
- Add a Commit message (2) and click Commit changes (3)
![Add dependencies](/images/add-npm-packages.PNG)
- At this point, the Continuous Delivery pipeline will automatically run to build and deploy that change into your application. If you view the Delivery Pipeline you can watch its progress. The Build section shows you the last commit made (1) and the Deploy section shows the progress of redeploying the application (2).
![Build and Deploy](/images/buid-and-deploy.PNG)
- After the Deploy stage completes, your application will have restarted and now have the node-red-dashboard nodes preinstalled.

***

### 4.2 Import Flows ###
After the Node-RED is installed, import the Desk level sensor support flows. Complete the following steps. 
1. Go to he Node-RED user-interface,  in the top right corner click on 3 horizontal dashes. A drop-down menu displays.
 ![Git](/images/import.png)
2. Click on **Import** and then **clipboard**. A pop up window displays.

 ![Git](/images/import_Flow.png)
3. Copy the [desk_sensor_support_flow.json](../desk_sensor_support_flow.json) to this window  and click on **import**.

***
