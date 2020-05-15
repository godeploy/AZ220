### Lab - Individual Enrollment of a Device in DPS

@@@secondary
**Scenario**

Contoso management is pushing for an an update to their Asset Monitoring and Tracking Solution that will use IoT devices to reduce the manual data entry work that is required under the current system and provide more advanced monitoring during the shipping process. The solution relies on the ability to provision and de-provision IoT devices. The best option for managing the provisioning requirements appears to be DSP.

The proposed system will use IoT devices with integrated sensors for tracking the location, temperature, pressure of shipping containers during transit. The devices will be placed within the existing shipping containers that Contoso uses to transport their cheese, and will connect to Azure IoT Hub using vehicle provided WiFi. The new system will provide continuous monitoring of the product environment and enable a variety of notification scenarios when issues are detected.

In Contoso's cheese packaging facility, when an empty container enters the system it will be equipped with the new IoT device and then loaded with packaged cheese products. The IoT device needs to be auto-provisioned to IoT hub using Device Provisioning Service. When the container arrives at the destination, the IoT device will be retrieved and then "decommissioned" through DPS. The device will be re-used for future shipments.

You have been tasked with validating the device provisioning and de-provisioning process using DPS. For the initial phase of the process you will use an Individual Enrollment approach.

In this lab, you will, create an individual enrollment within Azure Device Provisioning Service (DPS) to automatically connect a pre-built simulated device to Azure IoT Hub. You will also fully retire the device by removing it from both DPS and IoT Hub.

This lab includes:

* Verify Lab Prerequisites
* Create New individual enrollment in DPS
* Configure Simulated Device
* Test Simulated Device
* Retire the Device
@@@

#### Exercise 1: Verify Lab Prerequisites

@@@secondary
This lab assumes the following resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |
| Device Provisioning Service | AZ-220-DPS-_{YOUR-ID}_ |

If the resources are unavailable, please execute the **lab-setup.azcli** script before starting the lab.

The **lab-setup.azcli** script is written to run in a **bash** shell environment - the easiest way to execute this is in the Azure Cloud Shell.
@@@

1. [ ] Using a browser, open the Azure Shell and login with the Azure subscription you are using for this course.

1. [ ] To ensure the Azure Shell is using **Bash**, ensure the dropdown selected value in the top-left is **Bash**.

1. [ ] To upload the setup script, in the Azure Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. [ ] In the dropdown, select **Upload** and in the file selection dialog, navigate to the **lab-setup.azcli** file for this lab. Select the file and click **Open** to upload it.

    A notification will appear when the file upload has completed.

1. [ ] You can verify that the file has uploaded by listing the content of the current directory by entering the `ls` command.

1. [ ] To create a directory for this lab, move **lab-setup.azcli** into that directory, and make that the current working directory, enter the following commands:

    ```bash
    mkdir lab5
    mv lab-setup.azcli lab5
    cd lab5
    ```

1. [ ] To ensure the **lab-setup.azcli** has the execute permission, enter the following commands:

    ```bash
    chmod +x lab-setup.azcli
    ```

1. [ ] To edit the **lab-setup.azcli** file, click **{ }** (Open Editor) in the toolbar (second button from the right). In the **Files** list, select **lab5** to expand it and then select **lab-setup.azcli**.

    The editor will now show the contents of the **lab-setup.azcli** file.

1. [ ] In the editor, update the values of the `{YOUR-ID}` and `{YOUR-LOCATION}` variables. Set `{YOUR-ID}` to the Unique ID you created at the start of this - i.e. **CAH191211**, and set `{YOUR-LOCATION}` to the location that makes sense for your resources.

    ```bash
    #!/bin/bash

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

    Location="{YOUR-LOCATION}"
    ```
    
@@@warning
**Note:** The `{YOUR-LOCATION}` variable should be set to the short name for the region. You can see a list of the available regions and their short-names (the **Name** column) by entering this command:

     
     az account list-locations -o Table
     
     DisplayName           Latitude    Longitude    Name
     --------------------  ----------  -----------  ------------------
     East Asia             22.267      114.188      eastasia
     Southeast Asia        1.283       103.833      southeastasia
     Central US            41.5908     -93.6208     centralus
     East US               37.3719     -79.8164     eastus
     East US 2             36.6681     -78.3889     eastus2
     
@@@

10. [ ] To save the changes made to the file and close the editor, click **...** in the top-right of the editor window and select **Close Editor**.

    If prompted to save, click **Save** and the editor will close.
@@@warning
**NOTE** You can use **CTRL+S** to save at any time and **CTRL+Q** to close the editor.
@@@

10. [ ] To create a resources required for this lab, enter the following command:

    ```bash
    ./lab-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

    Once the script has completed, you will be ready to continue with the lab.

#### Exercise 2: Create new individual enrollment (Symmetric keys) in DPS

@@@secondary
In this exercise, you will create a new individual enrollment for a device within the Device Provisioning Service (DPS) using **symmetric key attestation**.
@@@

##### Task 1: Create the enrollment

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Notice that the **AZ-220** dashboard that you created in the previous task has been loaded.

    You should see both your IoT Hub and DPS resources listed.

1. [ ] On your Resource group tile, click **AZ-220-DPS-_{YOUR-ID}_**.

1. [ ] On the Device Provisioning Service **Settings** pane on the left side, click **Manage enrollments**.

1. [ ] At the top of the pane, click **+ Add individual enrollment**.

1. [ ] On the **Add Enrollment** blade, in the **Mechanism** dropdown, click **Symmetric Key**. This sets the attestation method to use Symmetric key authentication.

1. [ ] Notice the **Auto-generate keys** option is checked. This sets DPS to automatically generate both the **Primary Key** and **Secondary Key** values for the device enrollment when it's created.

    Optionally, un-checking this option enables custom keys to be manually entered.

1. [ ] In the **Registration ID** field, enter `DPSSimulatedDevice1` as the Registration ID to use for the device enrollment within DPS.

    By default, the Registration ID will be used as the IoT Hub Device ID when the device is provisioned from the enrollment. If these values need to be different, then enter the required IoT Hub Device ID in that field.

1. [ ] Leave the **IoT Hub Device ID** field blank.  This will cause the IoT Hub to use the Registration ID.

1. [ ] Leave **IoT Edge device** as `False`.
   
   The new device will not be an edge device.  That concept will be discussed later in the course.

2. [ ] Leave **Select how you want to assign devices to hubs** as **Evenly weighted distribution**.
   
   As you only have one IoT Hub associated with the enrollment, this setting is somewhat unimportant.  In larger environments where you have multiple distributed hubs, this setting will control how to choose what IoT Hub should receive this device enrollment.

3. [ ] Notice that the **AZ-220-HUB-_{YOUR-ID}_** IoT Hub is selected within the **Select the IoT hubs this device can be assigned to** dropdown.
   
   This field specifies the IoT Hub(s) this device can be assigned to.

4. [ ] Leave **Select how you want device data to be handled on re-provisioning** as the default value of **Re-provision and migrate data**.

    This field gives you high-level control over the re-provisioning behavior, where the same device (as indicated through the same Registration ID) submits a later provisioning request after already being provisioned successfully at least once.

5. [ ] In the **Initial Device Twin State** field, modify the `properties.desired` JSON object to include a property named `telemetryDelay` with the value of `"2"`. This will be used by the Device to set the time delay for reading sensor telemetry and sending events to IoT Hub.

    The final JSON will be like the following:

    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "telemetryDelay": "2"
            }
        }
    }
    ```

    This field contains JSON data that represents the initial configuration of desired properties for the device.

1. [ ] Leave **Enable entry** set to **Enable**.

    Generally, you'll want to enable new enrollment entries and keep them enabled.

2. [ ] At the top of the **Add Enrollment** blade, click **Save**.

##### Task 2: Validate the enrollment

3. [ ] In the **Manage enrollments** pane, click on the **individual enrollments** tab to view the list of individual device enrollments.

4. [ ] In the list, click on the **DPSSimulatedDevice1** individual enrollment that was just created to view the enrollment details.

5. [ ] Locate the **Authentication Type** section, and notice the **Mechanism** is set to **Symmetric Key**.

6. [ ]  Copy the **Primary Key** and **Secondary Key** values for this device enrollment (there is a button to the right of each textbox for this purpose), and save them for reference later.

    These are the authentication keys for the device to authenticate with the service.

7.  [ ] Locate the **Initial device twin State**, and notice the JSON for the device twin Desired State contains the `telemetryDelay` property set to the value of `"2"`.

8. [ ] Close the **DPSSimulatedDevice1** view to return to the **AZ-220-DPS-_{YOUR_ID}_** blade.

#### Exercise 3: Configure Simulated Device

@@@secondary
In this exercise, you will configure a Simulated Device written in C# to connect to Azure IoT using the individual enrollment created in the previous unit. You will also add code to the Simulated Device that will read and update device configuration based on the device twin within Azure IoT Hub.

The simulated device created in this unit is for a an asset tracking solution that will have Iot Device with sensors located within a transport box to track shipments in transit. The sensor telemetry from the device sent to Azure IoT Hub includes Temperature, Humidity, Pressure, and Latitude/Longitude coordinates of the transport box.

This is different than the earlier lab where a simulated device connected to Azure because in that lab, you used a shared access key to authenticate, which does not require device provisioning, but also does not give the provisioning management benefits (such as device twins), and requires fairly large distribution and management of a shared key.  In this lab, you are provisioning a unique device through the Device Provisioning Serivce.
@@@

##### Task 1: Create the Simulated Device

1. [ ] On the **AZ-220-DPS-_{YOUR_ID}_** blade, navigate to the **Overview** pane.

1. [ ] Within the **Overview** pane, copy the **ID Scope** for the Device Provisioning Service, and save it for reference later.  (There is a copy button to the right of the value that will appear when you hover over the value.)

    The **ID Scope** will be similar to this value: `0ne0004E52G`

1. [ ] Using **Visual Studio Code**, open the `/LabFiles` folder.

1. [ ] Open the `Program.cs` file.

1. [ ] Locate the `dpsIdScope` variable, and replace the value with the **ID Scope** of the Device Provisioning Service.
@@@warning
**Note**: The **ID Scope** for the **Device Provisioning Service** can be retrieved from within the Azure portal, by navigating to the DPS resource, then copying the **ID Scope** value on the **Overview** pane.
@@@

1. [ ] Locate the `registrationId` variable, and replace the value with `DPSSimulatedDevice1`.

    Remember that this was the the **Registration ID** of for the individual enrollment that was created in the Device Provisioning Service.

2. [ ] Locate the `individualEnrollmentPrimaryKey` and `individualEnrollmentSecondaryKey` variables, and replace their values with the **Primary Key** and **Secondary Key** values that were copied from the Enrollment Details for the individual enrollment for the device that was created in the Device Enrollment Service.

3. [ ] Review the source code for the simulated device, and take notice of the following items:

    - The `ProvisioningDeviceLogic` class contains the logic for reading from the simulated device sensors.
    - The `ProvisioningDeviceLogic.SendDeviceToCloudMessagesAsync` method contains the logic for generating the simulated sensor readings for Temperature, Humidity, Pressure, Latitude, and Longitude. This method also sends the telemetry as Device-to-Cloud messages to Azure IoT Hub.
</br>

4. [ ] Notice at the bottom of the `ProvisioningDeviceLogic.SendDeviceToCloudMessagesAsync` method, there is a `Task.Delay` call to pause the `while` loop for a period of time before reading the simulated sensors again and sending the telemetry. This code uses the `_telemetryDelay` variable that defines how many seconds to wait before sending telemetry again.

5. [ ] Locate the `_telemetryDelay` variable declaration towards the top of the `ProvisioningDeviceLogic` class. Notice the delay is defaulted to `1` second in the code.

6. [ ] To get started configuring the simulated device to set the `_telemetryDelay` based on configuration of the **device twin** within Azure IoT Hub, you need to add code to read the device twin desired state, and report back the current state.

7. [ ] Locate the `// TODO 1: Setup OnDesiredPropertyChanged Event Handling` comment. To setup the simulated device to be notified of device twin state changes, you need to use the `DeviceClient.SetDesiredPropertyUpdateCallbackAsync` method to wire up an event handler for the `OnDesiredPropertyChanged` event.

    Replace the `// TODO 1` comment with the following code that sets up the event handler on the DeviceClient:

    ```csharp
    Console.WriteLine("Connecting SetDesiredPropertyUpdateCallbackAsync event handler...");
    await iotClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).ConfigureAwait(false);
    ```

8. [ ] To complete setting up the event handler, the `OnDesiredPropertyChanged` method needs to be added to the `ProvisioningDeviceLogic` class. Add the following method code to the class, that also includes code to read the device twin Desired Properties, configures the `_telemetryDelay` variable, and then reports back the Reported Properties back to the device twin to tell Azure IoT Hub what the current state of the simulated device is configured to.

    ```csharp
    private async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        Console.WriteLine("Desired Twin Property Changed:");
        Console.WriteLine($"{desiredProperties.ToJson()}");

        // Read the desired Twin Properties
        if (desiredProperties.Contains("telemetryDelay"))
        {
            string desiredTelemetryDelay = desiredProperties["telemetryDelay"];
            if (desiredTelemetryDelay != null)
            {
                this._telemetryDelay = int.Parse(desiredTelemetryDelay);
            }
            // if desired telemetryDelay is null or unspecified, don't change it
        }


        // Report Twin Properties
        var reportedProperties = new TwinCollection();
        reportedProperties["telemetryDelay"] = this._telemetryDelay;
        await iotClient.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        Console.WriteLine("Reported Twin Properties:");
        Console.WriteLine($"{reportedProperties.ToJson()}");
    }
    ```

9. [ ]  Locate the `//TODO 2: Load device twin Properties` comment. To setup the simulated device to read the current device twin property desired state, and configure the device to match on device startup, add the following code in place of this comment:

    ```csharp
    Console.WriteLine("Loading device twin Properties...");
    var twin = await iotClient.GetTwinAsync().ConfigureAwait(false);
    await OnDesiredPropertyChanged(twin.Properties.Desired, null);
    ```

    This code calls the `DeviceTwin.GetTwinAsync` method to retrieve the device twin for the simulated device. It then accesses the `Properties.Desired` property object to retrieve the current Desired State for the device, and passes that to the `OnDesiredPropertyChanged` method that will configure the simulated devices `_telemetryDelay` variable.

    Notice, this code reuses the `OnDesiredPropertyChanged` method that was already created for handling _OnDesiredPropertyChanged_ events. This helps keep the code that reads the device twin desired state properties and configures the device at startup in a single place. The result is the code is simpler and easier to maintain.

10. Now the simulated device is all setup to be configured by the device twin within Azure IoT Hub.
@@@warning
**Note**: If you need help with pasting code in the `Program.cs` file, please refer to the `/LabFiles-Completed` folder for the full source code for the Simulated Device with the device twin configuration code. When using this completed code sample, be sure to configure the ID Scope, Registration ID, and individual enrollment Keys.
@@@

#### Exercise 4: Test the Simulated Device

@@@secondary
In this exercise, you will run the Simulated Device and verify it's sending sensor telemetry to Azure IoT Hub. You will also update the delay at which telemetry is sent to Azure IoT Hub by updating the device twin for the simulated device within Azure IoT Hub.
@@@

##### Task 1: Build and run the device

1. [ ] In Visual Studio Code, click on the **View** menu, then click **Terminal** to open the _Terminal_ pane.

1. [ ] Run the following command within the **Terminal** to build and run the Simulated Device application. Be sure the terminal location is set to the `/LabFiles` directory with the `Program.cs` file.

    ```cmd/sh
    dotnet run
    ```

1. [ ] When the Simulated Device application runs, it will first output some details about it's status. Notice the JSON output that follows the `Desired Twin Property Changed:` line contains the desired value for the `telemetryDelay` for the device.

    ```text
    RegistrationID = DPSSimulatedDevice1
    ProvisioningClient RegisterAsync . . . Device Registration Status: Assigned
    ProvisioningClient AssignedHub: AZ-220-HUB-CP1019.azure-devices.net; DeviceID: DPSSimulatedDevice1
    Creating Symmetric Key DeviceClient authentication
    Simulated Device. Ctrl-C to exit.
    DeviceClient OpenAsync.
    Connecting SetDesiredPropertyUpdateCallbackAsync event handler...
    Loading device twin Properties...
    Desired Twin Property Changed:
    {"telemetryDelay":"2","$version":1}
    Reported Twin Properties:
    {"telemetryDelay":2}
    Start reading and sending device telemetry...
    ```

1. [ ] The Simulated Device application will be sending telemetry events to the Azure IoT Hub that includes the `temperature`, `humidity`, `pressure`, `latitude`, and `longitude` values.

    The terminal output will look similar to the following:

    ```text
    11/6/2019 6:38:55 PM > Sending message: {"temperature":25.59094770373355,"humidity":71.17629229611545,"pressure":1019.9274696347665,"latitude":39.82133964767944,"longitude":-98.18181981142438}
    11/6/2019 6:38:57 PM > Sending message: {"temperature":24.68789062681044,"humidity":71.52098010830628,"pressure":1022.6521258267584,"latitude":40.05846882452387,"longitude":-98.08765031156229}
    11/6/2019 6:38:59 PM > Sending message: {"temperature":28.087463226675737,"humidity":74.76071353757787,"pressure":1017.614206096327,"latitude":40.269273772972454,"longitude":-98.28354453319591}
    11/6/2019 6:39:01 PM > Sending message: {"temperature":23.575667940813894,"humidity":77.66409506912534,"pressure":1017.0118147748344,"latitude":40.21020096551372,"longitude":-98.48636739129239}
    ```

    Notice the timestamp differences between telemetry readings. The telemetry delay the simulated device is running at should be `2` seconds as configured through the device twin; instead of the default of `1` second in the source code.

1. [ ] Verify the simulated device telemetry is being sent to Azure IoT Hub by running the following Azure CLI command in the Azure Cloud Shell (or a different command-line window).

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} --device-id DPSSimulatedDevice1
    ```

    _Be sure to replace the **{IoTHubName}** placeholder with the name of your Azure IoT Hub._

    Keep the simulated device running for the next task.

##### Task 2: Change the device configuration through its twin

With the simulated device running, the `telemetryDelay` configuration can be updated by editing the device twin Desired State within Azure IoT Hub. This can be done by configuring the Device in the Azure IoT Hub within the Azure portal.

1. [ ] Open the **Azure Portal** if it is not already open, and navigate to your **Azure IoT Hub** service.

2. [ ] On the IoT Hub blade, on the left side of the blade, under the **Explorers** section, click on **IoT devices**.

3. Within the list of IoT devices, click on the **Device ID** (likely **DPSSimulatedDevice1**) for the Simulated Device.
@@@danger
Important! Make sure you select the device from this lab.
@@@

1. [ ] On the device blade, click the **Device Twin** button at the top of the blade.

    Within the **Device twin** blade, there is an editor with the full JSON for the device twin. This enables you to view and/or edit the device twin state directly within the Azure portal.

3. [ ] Locate the JSON for the `properties.desired` object.
   
    This contains the Desired State for the device twin. Notice the `telemetryDelay` property already exists, and is set to `"2"`, as was configured when the device was provisioned based on the Individual enrollment in DPS.

4. [ ] Modify the `telemetryDelay` value to `"5"` to configure the device twin to set the Desired State to have the simulated device wait 5 seconds between telemetry readings.

5. [ ] At the top of the blade, click **Save**

    The `OnDesiredPropertyChanged` event will be triggered automatically within the code for the Simulated Device, and the device will update its configuration to reflect the changes to the device twin Desired state.

8. [ ]  In Visual Studio Code, view the Terminal output for the Simulated Device application.
    The output will show a message that the `Desired Twin Property Changed` along with the JSON for the new desired`telemetryDelay` property value. Once the device picks up the new configuration of device twin desired state, it will automatically update to start sending sensor telemetry every 5 seconds as now configured.

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":"5","$version":2}
    Reported Twin Properties:
    {"telemetryDelay":5}
    11/6/2019 7:29:55 PM > Sending message: {"temperature":33.01780830277959,"humidity":68.52464504936927,"pressure":1023.0929576073974,"latitude":39.97641877038439,"longitude":-98.49544472071804}
    11/6/2019 7:30:00 PM > Sending message: {"temperature":33.95490410689027,"humidity":71.57070464062072,"pressure":1013.3468084112261,"latitude":40.01604868659767,"longitude":-98.51051877869526}
    11/6/2019 7:30:05 PM > Sending message: {"temperature":22.055266337494956,"humidity":67.50505594886144,"pressure":1018.1765662249767,"latitude":40.22292566031555,"longitude":-98.4367936214764}
    ```

9. [ ]  Locate the command-line window with the `az iot hub monitor-events` Azure CLI command running.
    This will also display the telemetry events sent to Azure IoT Hub being received at the new interval of 5 seconds.

10. [ ] Use **Ctrl-C** to stop both the `az` command and the Simulated Device application.

6. [ ] In the Azure Portal, close the **Device twin** blade. 

1. [ ] Still in the Azure Portal, on the Simulated Device blade, again click the **Device Twin** button.

1. [ ] This time, locate the JSON for the `properties.reported` object.
   
    This contains the state reported by the device. Notice the `telemetryDelay` property exists here as well, and is also set to `5`.  There is also a `$metadata` value that shows you when the value was reported data was last updated and when the specific reported value was last updated.

1. [ ] Again close the **Device twin** blade.

1. [ ] Close the simulated device blade to return back to the IoT Hub blade.

#### Exercise 5: Retire the Device

@@@secondary
In this unit you will perform the necessary tasks to retire the device from both the Device Provisioning Service (DPS) and Azure IoT Hub. To fully retire an IoT Device from an Azure IoT solution it must be removed from both of these services. When the transport box arrives at it's final destination, then sensor will be removed from the box, and needs to be "decommissioned". Complete device retirement is an important step in the life cycle of IoT devices within an IoT solution.
@@@

##### Task 1: Retire the device from the DPS

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Notice that the **AZ-220** dashboard that you created in the previous task has been loaded.

    You should see both your IoT Hub and DPS resources listed.

1. [ ] On your Resource group tile, click **AZ-220-DPS-_{YOUR-ID}_** to navigate to the Device Provisioning Service.

1. [ ] On the Device Provisioning Service settings pane on the left side, click **Manage enrollments**.

1. [ ] In the Manage enrollments pane, click on the **individual enrollments** link to view the list of individual device enrollments.

1. [ ] Select the `DPSSimulatedDevice1` individual device enrollment by checking the box next to it in the list, then click **Delete** from the top of the blade.
@@@warning
**Note**: Deleting the individual enrollment from DPS will permanently remove the enrollment. To temporarily disable the enrollment, you can set the **Enable entry** setting to **Disable** within the **Enrollment Details** for the individual enrollment.
@@@

1. [ ] On the **Remove enrollment** prompt, click **Yes** to confirm that you want to delete this device enrollment from the Device Provisioning Service.

    The individual enrollment is now removed from the Device Provisioning Service (DPS). To complete the device retirement, the **Device ID** for the Simulated Device also must be removed from the **Azure IoT Hub** service.

##### Task 2: Retire the device from the IoT Hub

1. [ ] Navigate back to your Dashboard.
   
2. [ ] On your resource group tile, click **AZ-220-HUB-_{YOUR-ID}_** to navigate to the Azure IoT Hub.

2. [ ] On the IoT Hub blade, on the left side of the blade, under the **Explorers** section, click on **IoT devices**.

3. [ ] Within the list of IoT devices, click on checkbox to the left of the **Device ID** (likely **DPSSimulatedDevice1**) for the Simulated Device.
@@@danger
**Important**! Make sure you select the device from this lab.

@@@
4. [ ] At the top of the blade, click **Delete**.

5. [ ] On the **Are you certain you wish to delete selected device(s)** prompt, click **Yes** to confirm that you want to delete this device from Azure IoT Hub.
@@@warning
**Note**: Deleting the device ID from IoT Hub will permanently remove the device registration. To temporarily disable the device from connecting to IoT Hub, you can set the **Enable connection to IoT Hub** to **Disable** within the properties for the device.
@@@

 Now that the Device Enrollment has been removed from the Device Provisioning Service, and the matching Device ID has been removed from the Azure IoT Hub, the simulated device has been fully retired from the solution.
 
@@@success
**Congratulations**! you have now completed this lab!
@@@
