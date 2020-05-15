### Lab - How to manage your Azure IoT Hub

@@@secondary
**Scenario**

Our asset tracking solution is getting bigger, and provisioning devices one by one (even through DPS) cannot scale. We want to use DPS to enroll many devices automatically and securely using x.509 certificate authentication. Within our solution, we will use sensors to track our assets being transported. Each time a sensor is added in a transportation box, it will auto provision through DPS. We want to have a metric for the warehouse manager of how many boxes were "tagged" and need to count the Device Connected events from IoT Hub.

In this lab, you will setup a Group Enrollment within Device Provisioning Service (DPS) using a Root CA x.509 certificate chain. You will configure the linked IoT Hub to using monitoring to track the number of connected devices and telemetry messages sent, as well as send connection events to a log. Additionally you will create an alert that will be triggered based upon the average number of devices connected. You will the configure 10 simulated IoT Devices that will authenticate with DPS using a Device CA Certificate generated on the Root CA Certificate chain. The IoT Devices will be configured to send telemetry to the the IoT Hub.

**In this lab, you will**:

* Verify Lab Prerequisites
* Enable diagnostic logs.
* Enable metrics.
* Set up alerts for those metrics.
* Download and run an app that simulates IoT devices connecting via X509 and sending messages to the hub.
* Run the app until the alerts begin to fire.
* View the metrics results and check the diagnostic logs.
@@@

#### Exercise 1: Set Up and Use Metrics and Diagnostic Logs with an IoT Hub

@@@secondary
If you have an IoT Hub solution running in production, you want to set up some metrics and enable diagnostic logs. Then if a problem occurs, you have data to look at that will help you diagnose the problem and fix it more quickly. In this lab, you'll see how to enable the diagnostic logs, and how to check them for errors. You'll also set up some metrics to watch, and alerts that fire when the metrics hit a certain boundary.

For example, you could have an e-mail sent to you when the number of connected devices exceed a certain threshold, or when the number of messages used gets close to the quota of messages allowed per day for the IoT Hub.
@@@

##### Setup Resources

In order to complete this lab, you will need to reuse a number of resources from a previous lab - **Automatic Enrollment of Devices in DPS** as well as a storage account.

1. [ ] Open a new tab on your browser and navigate to the Azure Cloud Shell **`https://shell.azure.com/`**

1. [ ] Login to you Azure Subscription (the same one you used for your IoT Central App) and if your account is a member of more than one directory, choose the directory you used for your IoT Central account.

1. [ ] Once the bash shell is open, create** a **monitoring** folder, and navigate to it by entering the following commands:

    ```bash
    mkdir ~/monitoring
    cd ~/monitoring
    ```

1. [ ] To create an empty file in which we will copy the setup script, enter the following commands:

    ```bash
    touch setup.sh
    chmod +x setup.sh
    ```

1. [ ] To edit the contents of the **setup.sh** file, use the **{ }** icon in Azure Cloud Shell to open the **Cloud Editor**.

    To open the **setup.sh** file, you will have to expand the **monitoring** node in the **Files** list to locate it.

1. [ ] Copy the following script into the cloud editor:

    ```bash
    #!/bin/bash

    YourID="{YOUR-ID}"
    RGName="AZ-220"
    Location="westus"
    IoTHubName="$RGName-HUB-$YourID"
    DPSName="$RGName-DPS-$YourID"
    DeviceName="asset-track"
    StorageAccountName="$RGName-STORAGE-$YourID"

    # Storage Account name must be in lowercase with no '-'
    ToLowerAlphaNum () {
        echo $1 | tr '[:upper:'] '[:lower:]' | tr -cd '[:alnum:]'
    }

    StorageAccountName=$( ToLowerAlphaNum $StorageAccountName )

    # create resource group
    az group create --name $RGName --location $Location -o Table

    # create IoT Hub
    az iot hub create --name $IoTHubName -g $RGName --sku S1 --location $Location -o Table

    # create DPS
    az iot dps create --name $DPSName -g $RGName --sku S1 --location $Location -o Table

    # Get IoT Hub Connection String so DPS can be linked
    IoTHubConnectionString=$(
        az iot hub show-connection-string --hub-name $IoTHubName --query connectionString --output tsv
    )

    # Link IoT Hub with DPS
    az iot dps linked-hub create --dps-name $DPSName -g $RGName --connection-string $IoTHubConnectionString --location $Location

    # Create a Storage Account
    az storage account create --name $StorageAccountName --resource-group $RGName --location=$Location --sku Standard_LRS -o Table 

    StorageConnectionString=$( az storage account show-connection-string --name $StorageAccountName -o tsv )
    ```
@@@warning
**Note**: Review this script. You can see that it perform the following actions (and create resources if they don't already exist):
     * Builds the resource names
     * Note that the storage account name is set to lowercase with no dashes to match the naming rules.
     * Create Resource Group
     * Create IoT Hub
     * Create DPS
     * Link IoT Hub and DPS
     * Create Storage Account
@@@

1. [ ] In order to specify the correct resource names and location, update the following variables at the top of the file:

    * YourID
    * RGName
    * Location
</br>    
@@@warning
**Note**: If you have existing resources you wish to reuse, ensure you set the **YourID** value to the same you used before, as well as the same **RGName** and **Location**.
@@@

1. [ ] To save the edited **setup.sh** file, press **CTRL-Q**. If prompted to save you changes before closing the editor, click **Save**.

1. [ ] To run the **setup.sh** script, run the following:

    ```bash
    ./setup.sh
    ```
@@@warning
**Note**: If the IoT Hub and DPS resources already exist, you will see red warnings stating the name is not available - you can ignore these errors.
@@@

You have now ensured the resources are available for this lab. Next, we shall setup monitoring and logging.





#### Exercise 2: Enable Logging

@@@secondary
Azure Resource logs are platform logs emitted by Azure resources that describe their internal operation. All resource logs share a common top-level schema with the flexibility for each service to emit unique properties for their own events.
@@@

1. [ ] Sign in to the **Azure portal** and navigate to your IoT hub.

1. [ ] In the left hand navigation, under **Monitoring**, select **Diagnostic settings**.
@@@warning
**Note**: Diagnostics are disabled by default.
@@@

1. [ ] At the top of the **Diagnostic settings** page, under **Subscription**, select the subscription you used to create the IoT Hub.

1. [ ] Under **Resource group**, select the resource group you used for this lab - "AZ-220-RG".

1. [ ] Under **Resource type**, select **IoT Hub**.

1. [ ] Under **Resource**, select the IoT Hub you are using for this lab - **AZ-220-HUB-\<INITIALS-DATE\>**.

    Once you select the resource, the page will update with the option to turn on diagnostics, as well as a list of available metrics to monitor.

1. [ ] To turn on diagnostics, click **Turn on diagnostics**.

    The **Diagnostic settings** detail pane will be shown.

1. [ ] Under **Name**, enter **diags-hub**.

    You can see that there are 3 options available for routing the metrics - you can learn more about each by following the links below:

    * [Archive Azure resource logs to storage account](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/resource-logs-collect-storage)
    
    * [Stream Azure monitoring data to an event hub](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/stream-monitoring-data-event-hubs)
    * [Collect Azure resource logs in Log Analytics workspace in Azure Monitor](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/resource-logs-collect-workspace)

    In this lab we will use the storage account option.

1. [ ] Check **Archive to a storage account** and the **Storage account** configuration section will appear.

1. [ ] To specify the storage account to use, click **Configure**.

    The **Select a storage account** pane will appear.
@@@warning
**Note**: In production, you should not use an existing storage account that has other, non-monitoring data stored in it so that you can better control access to monitoring data. If you are also archiving the Activity log to a storage account though, you may choose to use that same storage account to keep all monitoring data in a central location.
@@@

1. [ ] On the  **Select a storage account** pane, under **Subscription**, select the subscription you used to create the storage account earlier.

1. [ ] Under **Storage account**, select the storage account you created earlier.

1. [ ]  To complete the storage account selection, click **OK**.

    The **Select a storage account** pane will close and the specified storage account will be displayed under **Storage account**.

1. [ ] Under **log**, check **Connections** and **Device Telemetry** and then update the **Retention (days)** value for each to **7**. You can do this by either moving the slider or directly entering **7** into the value textbox.

1. [ ] Click **Save** to save the settings.

1. [ ]  Close the **Diagnostics settings** pane.

    The main **Diagnostics settings** page is displayed - you should see that the list of **Diagnostics settings** has now been updated to show the **diags-hub** setting you just created.

 Later, when you look at the diagnostic logs, you'll be able to see the connect and disconnect logging for the device.

##### Setup Metrics

Now set up some metrics to watch for when messages are sent to the hub.

1. [ ] In the left hand navigation area, under **Monitoring**, click **Metrics**.

    The **Metrics** pane is displayed showing a new, empty, chart.

1. [ ] To change the time range and granularity for the chart, at the top-right of the screen, click **Last 24 hours (Automatic)**.

1. [ ] In the dropdown that appears, select **Last 4 hours** for **Time Range**, and set **Time Granularity** to **1 minute**, and ensure **Show time as** is set to **local time**.

1. [ ] Click **Apply** to save these settings.

1. [ ]  Under the **Chart Title** and toolbar, you will see a default metric entry.

    We will now add a metric to monitor how many telemetry messages have been sent.

1. [ ] Note that the **SCOPE** is already set to the IoT Hub.

1. [ ] Under **METRIC NAMESPACE**, note that the **IoT Hub standard metrics** namespace is selected.
@@@warning
**Note**: By default, there is only one metric namespace available. Namespaces are a way to categorize or group similar metrics together. By using namespaces, you can achieve isolation between groups of metrics that might collect different insights or performance indicators. For example, you might have a namespace called **az220memorymetrics** that tracks memory-use metrics which profile your app. Another namespace called **az220apptransaction** might track all metrics about user transactions in your application. You can learn more about custom metrics and namespaces [here](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/metrics-custom-overview?toc=%2Fazure%2Fazure-monitor%2Ftoc.json#namespace).
@@@

1. [ ] In the **METRIC** dropdown list, select **Telemetry messages sent**. Notice how many metrics are available!

1. [ ] Under **AGGREGATION**, select **Sum**. Notice there are 4 aggregation operations available - *Avg*, *Min*, *Max* and *Sum*.

    We have completed the specification for the first metric. Notice that the chart title has updated to reflect the metric chosen. Now let's add another to monitor the total number of messages used.

1. [ ] Under the updated **Chart Title**, in the toolbar, click **Add metric**.

    A new metric will appear. Notice that, again, the **SCOPE** and **METRIC NAMESPACE** values are pre-populated and the **METRIC** dropdown is focused and open.

1. [ ] Under **METRIC**, select **Connected devices (preview)**.

1. [ ] Under **AGGREGATION**, select **Avg**.

    Your screen now shows the minimized metric for Telemetry messages sent, plus the new metric for avg connected devices. Notice that the chart title has updated to reflect both metrics.
@@@warning
**Note**: To edit the chart title, click the **pencil** to the right of the title. 
@@@

1. [ ] Under the updated **Chart Title**, in the toolbar, click **Pin to dashboard**. Note that you can choose to pin to the current dashboard or choose another. Select the dashboard you created in the first lab - "AZ-220-RG".
@@@warning
**Note**: In order to retain the chart you have just created, it **must** be pinned to a dashboard.
@@@

1. [ ]  Navigate to the "AZ-220-RG" dashboard and verify the chart is displayed.
@@@warning
**Note**: You can customize the size and position of the chart by using drag and drop operations.
@@@

Now that we have enable logging and setup a chart to monitor metrics, we will set up an alert.

#### Exercise 4: Configure an Alert

@@@secondary
Now let us create an alert. Alerts proactively notify you when important conditions are found in your monitoring data. They allow you to identify and address issues before the users of your system notice them. In our asset tracking scenario, we use sensors to track our assets being transported. Each time a sensor is added in a transportation box, it will auto provision through DPS. We want to have a metric for the warehouse manager of how many boxes were "tagged" and need to count the Device Connected events from IoT Hub.

In this task we are going to add an alert that will inform the warehouse manager when 5 or more devices have connected.
@@@

1. [ ] In the Azure Portal, navigate to the IoT Hub we are using for this lab.

1. [ ] In the left hand navigation area, under **Monitoring**, click **Alerts**.

    The empty **Alerts** page is displayed. Notice that the **Subscription**, **Resource group**, **Resource** and **Time range** fields are pre-populated.

1. [ ] Under **Time range**, select **Past hour**.

1. [ ] To add a new alert, click **+ New Alert Rule** (while the list is empty, you will see a **New Alert Rule** button in the center of the page - you can click this or the one in the toolbar).

    The **Create rule** pane is displayed.

1. [ ] At the top of the page, you will see two fields - **RESOURCE** and **HIERARCHY**. Notice they are pre-populated with the IoT Hub. To change the selected resource, you would click **Select**.

1. [ ] Under **Condition** you will see that no conditions have been defined. Click **Add** to add a new condition.

    The **Configure signal logic** pane is displayed. You will notice that there is a paginated table of available signals displayed. The fields above the table filter the table to assist in finding the signal types you want.

1. [ ] Under **Signal type**, you will note that **All** is selected. Click on the dropdown and note that there are 3 available options: *All*, *Metrics* and *Activity Log*. Leave the selection as **All** for now.
@@@warning
**Note** The signal types available for monitoring vary based on the selected target(s). The signal types may be metrics, log search queries or activity logs.
@@@

1. [ ] Under **Monitor service**, you will note that **All** is selected. Click on the dropdown and note that there are 3 available options: *All*, *Platform* and *Activity Log - Administrative*. Leave the selection as **All** for now.
@@@warning
**Note**: The platform service provides metrics on service utiization, where as the activity log tracks administrative activities.
@@@

1. [ ]  In the **Search by signal name** textbox, enter **connected** and this will immediately filter, then select **Connected devices (preview)** from the list below.

    The pane will update to display a chart similar to that you would create under **Metrics**, displaying the values associated with the selected signal (in this case *Connected devices (preview)*).

    Beneath the chart is the area that defines the **Alert logic**.

1. [ ] Under **Threshold** there are two possible selections - *Static* and *Dynamic*. You will notice that **Static** is selected and **Dynamic** is unavailable for this signal type.
@@@warning
**Note**: As the names suggest, *Static Thresholds* specify a constant expression for the threshold, whereas *Dynamic Thresholds* detection leverages advanced machine learning (ML) to learn metrics' historical behavior, identify patterns and anomalies that indicate possible service issues. You can learn more about *Dynamic Thresholds* [here](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-dynamic-thresholds).
@@@

    We are going to create a static threshold that raises and alert whenever the *connected devices (preview)* signal is equal to 5 or more.

1. [ ] Under **Operator**, click the dropdown list and note the available operators. Select **Greater than or equal to**.

1. [ ] Under **Aggregation type**, click the dropdown and note the available options - select **Average**.

1. [ ] Under **Threshold value**, enter **5**.
@@@warning
**Note**: The **Condition preview** refreshes to display the condition in an easier to read format.
@@@

    Below the **Condition preview** is the **Evaluation based on** area. The values herein determine the historical time period that is aggregated using the **Aggregation type** selected above and how often the condition is evaluated.

1. [ ] Under **Aggregation granularity (Period)**, select the dropdown and notice the available periods - select **5 minutes**.

1. [ ] Under **Frequency of evaluation**, select the dropdown and notice the available frequencies, select **Every 1 Minute**.
@@@warning
**Note**: As the **Frequency of evaluation** is shorter than **Aggregation granularity (Period)**, this results in a sliding window evaluation. What this means is every minute, the preceding 5 minutes of values will be aggregated (in this case, averaged), and then evaluated against the condition. In a minutes time, again the preceding 5 minutes of data will be aggregated - this will include one minute of new data and four minutes of data that was already evaluated. Thus we have a sliding window that moves forward a minute at a time, but is always including 4 minutes of earlier data.
@@@

1. [ ] To configure the alert condition, click **Done**.

    The **Configure signal logic** pane closes and the **Create rule** pane appears. Notice that the **CONDITION** is now populated and a **Monthly cost in USD** is displayed. At the time of writing, the estimated cost of the alert condition is $0.10.

    Next, we need to configure the action taken when the alert condition is met.

1. [ ] Under **ACTIONS**, notice that no action group is selected. There are two options available - **Select action group** and **Create action group**. As we do not have an action group created yet, click **Create action group**.

    The **Add action group** pane is displayed.
@@@warning
**Note**:  An action group is a collection of notification preferences defined by the owner of an Azure subscription. An action group name must be unique within the Resource Group is is associated with. Azure Monitor and Service Health alerts use action groups to notify users that an alert has been triggered. Various alerts may use the same action group or different action groups depending on the user's requirements. You may configure up to 2,000 action groups in a subscription. You can learn more about creating and managing Action Groups [here](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/action-groups).
@@@


1. [ ] Next to **Action group name**, enter **AZ-220 Email Action Group**.
@@@warning
**Note**: An action group name must be unique within the Resource Group is is associated with.
@@@

1. [ ] Next to **Short name**, enter **AZ220EmailAG**.
@@@warning
**Note**: The short name is used in place of a full action group name when notifications are sent using this group and is limited to a max of 12 characters.
@@@

1. [ ] Next to **Subscription**, select the subscription you have been using for this lab.

1. [ ] Next to **Resource group**, select the resource group you are using for this lab - "AZ-220-RG".
@@@warning
**Note**: Action Groups are usually shared across a subscription and would likely be centrally managed by the Azure subscription owner. As such they are more likely to be included in a common resource group rather than in a project specific resource group such as "AZ-220-RG". We are using "AZ-220-RG" to make it easier to clean up the resources after the lab.
@@@

1. [ ] In the next area, **Actions**, you can define a list of actions that will be performed whenever this action group is invoked.

1. [ ] Under **Action name**, enter **AZ220Notifications**.

1. [ ] Under **Action Type**, click the dropdown and notice the available options - select **Email/SMS/Push/Voice**.

    Immediately, the **Email/SMS/Push/Voice** action details pane is displayed. Notice that you can choose up to 4 methods for delivering the notification. For the purpose of this lab, we'll use **Email** and **SMS**.

1. [ ] Check **Email** and enter an email you wish to use to receive the alert.

1. [ ] Check **SMS**, enter your **Country code** and the **Phone number** you wish to receive the SMS alert.

1. [ ] Skip **Azure app Push Notifications** and **Voice**.

1. [ ] Finally, there is the option to **Enable the common alert schema** - select **Yes**.

@@@warning
**Note**: There are many benefits to using the Common Alert Schema. It standardizes the consumption experience for alert notifications in Azure today. Historically, the three alert types in Azure today (metric, log, and activity log) have had their own email templates, webhook schemas, etc. With the common alert schema, you can now receive alert notifications with a consistent schema. You can learn more about the Common ALert6 Schema [here](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-common-schema).
@@@
@@@danger
**Important:** Given the benefits, you may wonder why the common alert schema is not enabled by default - well, when you select **Yes** you will see a warning **Enabling the common alert schema might break any existing integrations.** Bear this in min
@@@

@@@success
**Congratulations**! you have now completed this lab!
@@@
