### Lab - Automate IoT devices management with Azure IoT Hub

@@@secondary
**Scenario**

Suppose you manage a company that offers a solution to maintain and monitor cheese caves' temperature and humidity at optimal levels. You have been working with gourmet cheese making companies for a long time and established long term trust with these customers who value the quality of your product.

Your solution consists in sensors and a climate system installed in the cave that report in real time on the temperature and humidity and an online portal customers can use to monitor and remotely operate their devices to adapt the temperature and humidity to the type of cheese they stored in their cave or to fine tune the environment for perfectly aging their cheese.

Your company is always enhancing the software running on the devices to better adapt to your customers different cheeses and diverse types of rooms they use to store their cheese. In addition to the features updates, you also want to make sure the devices deployed at customers locations have the latest security patches to ensure privacy and prevent hackers to take control of the system. In order to do this, you need to keep the devices up to date by remotely updating their firmware.

In this lab, you will you'll learn how automate device management with IoT Hub to configure and manage IoT devices remotely at scale.

This lab includes:

* Create an Azure IoT Hub and a Device ID
* Setup an Azure IoT environment: and Azure IoT Hub instance and a device Id
* Write code for simulating the device that will implement the firmware update
* Test the firmware update process on a single device using Azure IoT Hub automatic device management
@@@


#### Exercise 1: Create an Azure IoT Hub and a Device ID

This lab assumes the following resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-{YOUR-ID} |
| IoT Device | SimulatedSolutionThermostat |

To create these resources, please update and execute the **lab-setup.azcli** script before starting the lab.

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Open the Azure Cloud Shell by clicking the **Terminal** icon within the top header bar of the Azure portal, and select the **Bash** shell option.

1. [ ] Before the Azure CLI can be used with commands for working with Azure IoT Hub, the **Azure IoT Extensions** need to be installed. To install the extension, run the following command:

    ```sh
    az extension add --name azure-cli-iot-ext
    ```

1. [ ] To upload the setup script, in the Azure Cloud Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. [ ] In the dropdown, select **Upload** and in the file selection dialog, navigate to the **lab-setup.azcli** file for this lab. Select the file and click **Open** to upload it.

    A notification will appear when the file upload has completed.

1. [ ] You can verify that the file has uploaded by listing the content of the current directory by entering the `ls` command.

1. [ ] To create a directory for this lab, move **lab-setup.azcli** into that directory, and make that the current working directory, enter the following commands:

    ```bash
    mkdir lab14
    mv lab-setup.azcli lab14
    cd lab14
    ```

1. [ ] To ensure the **lab-setup.azcli** has the execute permission, enter the following commands:

    ```bash
    chmod +x lab-setup.azcli
    ```

1. [ ]  To edit the **lab-setup.azcli** file, click **{ }** (Open Editor) in the toolbar (second button from the right). In the **Files** list, select **lab14** to expand it and then select **lab-setup.azcli**.

    The editor will now show the contents of the **lab-setup.azcli** file.

1. [ ] In the editor, update the values of the `YourID` and `Location` variables. Set `YourID` to your initials and todays date - i.e. **CP123019**, and set `Location` to the location that makes sense for your resources.

@@@warning
**Note**: The `Location` variable should be set to the short name for the location. You can see a list of the available locations and their short-names (the **Name** column) by entering this command:
    
    
     az account list-locations -o Table
    
    
     DisplayName           Latitude    Longitude    Name
     --------------------  ----------  -----------  ------------------
     East Asia             22.267      114.188      eastasia
     Southeast Asia        1.283       103.833      southeastasia
     Central US            41.5908     -93.6208     centralus
     East US               37.3719     -79.8164     eastus
     East US 2             36.6681     -78.3889     eastus2
   
@@@

11. [ ] To save the changes made to the file and close the editor, click **...** in the top-right of the editor window and select **Close Editor**.

    If prompted to save, click **Save** and the editor will close.
@@@warning
**Note**: You can use **CTRL+S** to save at any time and **CTRL+Q** to close the editor.
@@@

1. [ ] To create a resource group named **AZ-220-RG**, create an IoT Hub named **AZ-220-HUB-{YourID}**, add a device with a Device ID of **SimulatedSolutionThermostat**, and display the device connection string, enter the following command:

    ```bash
    ./lab-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

1. [ ]  Once complete, the connection string for the device, starting with "HostName=", is displayed. Copy this connection string into a text document and note that it is for the **SimulatedSolutionThermostat** device.

#### Exercise 2: Write code to simulate device that implements firmware update

@@@secondary
At the end of this task, you'll have a device simulator awaiting for a firmware update request from IoT Hub.

Before getting started with your first firmware update on an IoT device, take a minute to review what it actually means to implement such an operation and how Azure IoT Hub helps making the process.
@@@

##### What does updating an IoT device's firmware imply?

@@@secondary
IoT devices most often are powered by optimized operating systems or even sometimes running code directly on the silicon (without the need for an actual operating system). In order to update the software running on this kind of devices the most common method is to flash a new version of the entire software package, including the OS as well as the apps running on it (called firmware).

Because each device has a specific purpose, its firmware is also very specific and optimized for the purpose of the device as well as the constrained resources available.

The process for updating a firmware is also something that can be very specific to the hardware itself and to the way the hardware manufacturer does things. This means that a part of the firmware update process is not generic and you will need to work with your device manufacturer to get the details of the firmware update process (unless you are developing your own hardware which means you probably know what the firmware update process).

While firmware updates can be and used to applied manually on devices, this is no longer possible considering the rapid growth in scale of IoT solutions. Firmware updates are now more commonly done over-the-air (OTA) with deployments of new firmware managed remotely from the cloud.

There is a set of common denominators to all over-the-air firmware updates for IoT devices:

1. Firmware versions are uniquely identified
1. Firmware comes in a binary file format that the device will need to acquire from an online source
1. Firmware is locally stored is some form of physical storage (ROM memory, hard drive,...)
1. Device manufacturer provide a description of the required operations on the device to update the firmware.
@@@

##### Azure IoT Hub Automatic Device Management

@@@secondary
Azure IoT Hub offers advanced support for implementing device management operations on a single and on collections of devices. The [Automatic Device Management](https://docs.microsoft.com/azure/iot-hub/iot-hub-auto-device-config) feature allows to simply configure a set of operations, trigger them and then monitor their execution.

In this exercise, you will create a simple simulator that will manage the device twin desired properties changes and will trigger a local process simulating a firmware update. The overall process would be exactly the same for a real device with the exception of the actual steps for the local firmware update. You will then use the Azure Portal to configure and execute a firmware update for a single device. IoT Hub will use the device twin properties to transfer the configuration change request to the device and monitor the progress
@@@

##### Task 1 - Create the device simulator app

1. [ ]  To use C# in Visual Studio Code, ensure both [.NET Core](https://dotnet.microsoft.com/download), and the [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) are installed

1. [ ]  Open a terminal in Visual Studio Code. Create a folder called **fwupdatedevice** and Navigate to the **fwupdatedevice** folder by running the following commands in the terminal:

    ```cmd/sh
    mkdir fwupdatedevice
    cd fwupdatedevice
    ```

1. [ ] Enter the following command in the terminal to create a **Program.cs** file in your folder, along with a project file.

    ```cmd/sh
    dotnet new console
    ```

1. [ ] Enter `dotnet restore` in the terminal. This command gives your app access to the required .NET packages.

    ```cmd/sh
    dotnet restore
    ```


1. [ ] In the terminal, install the required libraries. Enter the following commands and make sure all three libraries are installed:

    ```cmd/sh
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Microsoft.Azure.Devices.Shared
    dotnet add package Newtonsoft.Json
    ```

1. [ ] From the **File** menu, open up the **Program.cs** file, and delete the default contents.

##### Task 2 - Add code to your app

1. [ ] Open the **Program.cs** file for the device app.

1. [ ] Copy and paste the following code.

    ```cs
    // Copyright (c) Microsoft. All rights reserved.
    // Licensed under the MIT license. See LICENSE file in the project root for full license information.

    using Microsoft.Azure.Devices.Shared;
    using Microsoft.Azure.Devices.Client;
    using System;
    using System.Threading.Tasks;
    
    namespace fwupdatedevice
    {
        class SimulatedDevice
        {
            // The device connection string to authenticate the device with your IoT hub.
            static string s_deviceConnectionString = "";
    
            // Device ID variable
            static string DeviceID="unknown";
    
            // Firmware version variable
            static string DeviceFWVersion = "1.0.0";
    
            // Simple console log function
            static void LogToConsole(string text)
            {
                // we prefix the logs with the device ID
                Console.WriteLine(DeviceID + ": " + text);
            }
    
            // Function to retreive firmware version from the OS/HW
            static string GetFirmwareVersion()
            {
                // In here you would get the actual firmware version from the hardware. For the simulation purposes we will just send back the FWVersion variable value
                return DeviceFWVersion;
            }
    
            // Function for updating a device twin reported property to report on the current Firmware (update) status
            // Here are the values expected in the "firmware" update property by the firmware update configuration in IoT Hub
            //  currentFwVersion: The firmware version currently running on the device.
            //  pendingFwVersion: The next version to update to, should match what's
            //                    specified in the desired properties. Blank if there
            //                    is no pending update (fwUpdateStatus is 'current').
            //  fwUpdateStatus:   Defines the progress of the update so that it can be
            //                    categorized from a summary view. One of:
            //         - current:     There is no pending firmware update. currentFwVersion should
            //                    match fwVersion from desired properties.
            //         - downloading: Firmware update image is downloading.
            //         - verifying:   Verifying image file checksum and any other validations.
            //         - applying:    Update to the new image file is in progress.
            //         - rebooting:   Device is rebooting as part of update process.
            //         - error:       An error occurred during the update process. Additional details
            //                    should be specified in fwUpdateSubstatus.
            //         - rolledback:  Update rolled back to the previous version due to an error.
            //  fwUpdateSubstatus: Any additional detail for the fwUpdateStatus . May include
            //                     reasons for error or rollback states, or download %.
            //
            // reported: {
            //       firmware: {
            //         currentFwVersion: '1.0.0',
            //         pendingFwVersion: '',
            //         fwUpdateStatus: 'current',
            //         fwUpdateSubstatus: '',
            //         lastFwUpdateStartTime: '',
            //         lastFwUpdateEndTime: ''
            //   }
            // }

            static async Task UpdateFWUpdateStatus(DeviceClient client, string currentFwVersion, string pendingFwVersion, string fwUpdateStatus, string fwUpdateSubstatus, string lastFwUpdateStartTime, string lastFwUpdateEndTime)
            {
                TwinCollection properties = new TwinCollection();
                if (currentFwVersion!=null)
                    properties["currentFwVersion"] = currentFwVersion;
                if (pendingFwVersion!=null)
                    properties["pendingFwVersion"] = pendingFwVersion;
                if (fwUpdateStatus!=null)
                    properties["fwUpdateStatus"] = fwUpdateStatus;
                if (fwUpdateSubstatus!=null)
                    properties["fwUpdateSubstatus"] = fwUpdateSubstatus;
                if (lastFwUpdateStartTime!=null)
                    properties["lastFwUpdateStartTime"] = lastFwUpdateStartTime;
                if (lastFwUpdateEndTime!=null)
                    properties["lastFwUpdateEndTime"] = lastFwUpdateEndTime;
    
                TwinCollection reportedProperties = new TwinCollection();
                reportedProperties["firmware"] = properties;
    
                await client.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
            }
            
            // Execute firmware update on the device
            static async Task UpdateFirmware(DeviceClient client, string fwVersion, string fwPackageURI, string fwPackageCheckValue)
            {
                LogToConsole("A firmware update was requested from version " + GetFirmwareVersion() + " to version " + fwVersion);
                await UpdateFWUpdateStatus(client, null, fwVersion, null, null, DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"), null);
    
                // Get new firmware binary. Here you would download the binary or retreive it from the source as instructed for your device, then double check with a hash the integrity of the binary you downloaded 
                LogToConsole("Downloading new firmware package from " + fwPackageURI);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "0", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "25", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "50", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "75", null, null);
                await Task.Delay(2 * 1000);
                await UpdateFWUpdateStatus(client, null, null, "downloading", "100", null, null);
                // report the binary has been downloaded
                LogToConsole("The new firmware package has been successfully downloaded.");
                
                // Check binary integrity
                LogToConsole("Verifying firmware package with checksum " + fwPackageCheckValue);
                await UpdateFWUpdateStatus(client, null, null, "verifying", null, null, null);
                await Task.Delay(5 * 1000);
                // report the binary has been downloaded
                LogToConsole("The new firmware binary package has been successfully verified");
    
                // Apply new firmware
                LogToConsole("Applying new firmware");
                await UpdateFWUpdateStatus(client, null, null, "applying", null, null, null);
                await Task.Delay(5 * 1000);
    
                // On a real device you would reboot at the end of the process and the device at boot time would report the actual firmware version, which if successfull should be the new version.
                // For the sake of the simulation, we will simply wait some time and report the new firmware version
                LogToConsole("Rebooting");
                await UpdateFWUpdateStatus(client, null, null, "rebooting", null, null, DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ"));
                await Task.Delay(5 * 1000);
    
                // On a real device you would issue a command to reboot the device. Here we are simply runing the init function
                DeviceFWVersion = fwVersion;
                await InitDevice(client);
    
            }
    
            // Callback for responding to desired property changes 
            static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
            {
                LogToConsole("Desired property changed:");
                LogToConsole($"{desiredProperties.ToJson()}");
    
                // Execute firmware update
                if (desiredProperties.Contains("firmware") && (desiredProperties["firmware"]!=null))
                {
                    // In the desired properties, we will find the following information:
                    // fwVersion: the version number of the new firmware to flash
                    // fwPackageURI: URI from where to download the new firmware binary
                    // fwPackageCheckValue: Hash for validating the integrity of the binary  downloaded
                    // We will assume the version of the firmware is a new one
                    TwinCollection fwProperties = new TwinCollection(desiredProperties["firmware"].ToString());
                    await UpdateFirmware((DeviceClient)userContext, fwProperties["fwVersion"].ToString(), fwProperties["fwPackageURI"].ToString(), fwProperties["fwPackageCheckValue"].ToString());
    
                }
            }
    
            static async Task InitDevice(DeviceClient client)
            {
                LogToConsole("Device booted");
                LogToConsole("Current firmware version: " + GetFirmwareVersion());
                await UpdateFWUpdateStatus(client, GetFirmwareVersion(), "", "current", "", "", "");
            }
    
            static async Task Main(string[] args)
            {
                // Get the device connection string from the command line
                if (string.IsNullOrEmpty(s_deviceConnectionString) && args.Length > 0)
                {
                    s_deviceConnectionString = args[0];
                } else
                {
                    Console.WriteLine("Please enter the connection string as argument.");
                    return;
                }
    
                DeviceClient deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);
    
                if (deviceClient == null)
                {
                    Console.WriteLine("Failed to create DeviceClient!");
                    return;
                }
    
                // Get the device ID 
                string[] elements = s_deviceConnectionString.Split('=',';');
    
                for(int i=0;i<elements.Length; i+=2)
                {
                    if (elements[i]=="DeviceId") DeviceID = elements[i+1];
                }
    
                // Run device init routine
                await InitDevice(deviceClient);
    
                // Attach callback for Desired Properties changes
                await deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, deviceClient).ConfigureAwait(false);
    
                // Wait for keystroke to end app
                // TODO
                while (true)
                {
                    Console.ReadLine();
                    return;
                }
            }
        }
    }
        
    ```
@@@warning
**Note**: Read through the comments in the code, noting how the device reacts to device twin changes to execute a firmware update based on the configuration shared in the desired Property "firmware". You can also note the function that will report the current firmware update status through the reported properties of the device twin.
@@@

1. [ ] After you've entered the code below into the **Program.cs** file, you can run the app with the command `dotnet run`. This command will run the **Program.cs** file in the current folder, so ensure you are in the fwupdatedevice folder. 


    ```cmd/sh
    dotnet run
    ```


1. [ ] Save the **Program.cs** file.

 At this point your device is ready to be manage from IoT Hub. Next, we will test that the firmware update process works as expected for this simulated device.

#### Exercise 3: Test firmware update on a single device

@@@secondary
In this exercise, we will use the Azure portal to create a new device management configuration and apply it to our single simulated device.

@@@
##### Task 1 - Start device simulator

In the same terminal you setup the application for the simulated device, start the simulator typing the following command (replacing \<device connection string> with the device connection string you got at the end of task 2):

   ```
   dotnet run "<device connection string>" 
   ```

You should see the following output in the terminal (where "mydevice" is the device ID you used when creating the device identity):

``` 
    mydevice: Device booted
    mydevice: Current firmware version: 1.0.0
```
@@@warning
**Note**:  Make sure to put "" around your connection string. For example: "HostName=AZ-220-HUB-{YourID}.azure-devices.net;DeviceId=SimulatedSolutionThermostat;SharedAccessKey={}="
@@@

##### Create the device management configuration

1. [ ] Sign into the **Azure portal** `https://portal.azure.com/learn.docs.microsoft.com?azure-portal=true`

1. [ ] Go to the IoT Hub blade. You can find your IoT Hub by typing in the search bar (on top) the name you used when creating it in task 2.

1. [ ] In the IoT Hub, find the **Automatic Device Management** blade and select **IoT Device Configuration**, then select **Add Device Configuration**

1. [ ]  Enter an ID for the configuration such as **firmwareupdate** then click on **Next: Twins settings >** on the bottom.
    
1. [ ] For the **Device Twin Property** field, enter the following:

    ```
    properties.desired.firmware
    ```
    
1. [ ] In the **Device Twin Property Content** field type the following. Then click on **Next: Metrics >**

    ``` json
    {
        "fwVersion":"1.0.1",
        "fwPackageURI":"https://MyPackage.uri",
        "fwPackageCheckValue":"1234"
    }
    ```
   
1. [ ] In the **Metrics** blade we will define a custom metric to track the firmware update was effective. Create a new custom metric called **"fwupdated"** and type in the below criteria, then click on **Next: Target devices >**

    ``` SQL
        SELECT deviceId FROM devices
            WHERE properties.reported.firmware.currentFwVersion='1.0.1'
    ```
    
1. [ ] In the **Priority** field, type **"10"** and in the **Target Condition** field, type in the following query, replacing "\<your device id\>" with the device Id you used to create the device in task 2, then click on **Next: Review + Create >**

    ``` SQL
        deviceId='<your device id>'
    ```
    
1. [ ] On the next blade you should see the validation succeed for your new configuration. Click on **Create**.

1. [ ] Once the configuration has been created you will see it in the **Automatic Device Management** blade.

 At this point IoT Hub will look for devices matching the configuration's target devices criteria, and will apply the firmware update configuration automatically.
    
You have validated that the firmware update process on your simulated device works. You can stop the device simulator by simply pressing the "Enter" key in the terminal.

@@@success
**Congratulations**! you have now completed this lab!
@@@
