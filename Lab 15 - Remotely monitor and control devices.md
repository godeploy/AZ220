### Lab - Remotely monitor and control devices with Azure IoT Hub

@@@secondary
**Scenario**

Suppose you manage a gourmet cheese making company in a southern location. The company is proud of its cheese, and is careful to maintain the perfect temperature and humidity of a natural cave that is used to age the cheese. There are sensors in the cave that report on the temperature and humidity. A remote operator can set a fan to new settings if needed, to maintain the perfect environment for the aging cheese. The fan can heat and cool, and humidify and de-humidify.

Caves are used to mature cheese, their constant temperature, humidity, and air flow make them nearly ideal for the process. Not to mention the cachet of having your cheese products mature in a natural cave, instead of a constructed cellar. Something to put on your product labels!

The accepted ideal temperature for aging cheese is 50 degrees fahrenheit (10 degrees centigrade), with up to 5 degrees (2.78 degrees C) either side of this being acceptable. Humidity is also important. Measured in percentage of maximum saturation, a humidity of between 75 and 95 percent is considered fine. We'll set 85 percent as the ideal, with a 10 percent variation as acceptable. These values apply to most cheeses. To achieve specific results, such as a certain condition of the rind, cheese makers will adjust these values for some of the time during aging.

In a southern location, a natural cave near the surface might have an ambient temperature of around 70 degrees. The cave might also have a relative humidity of close to 100 percent, because of water seeping through the roof. These high numbers aren't perfect conditions for aging cheese. At a more northerly location, the ambient temperature of a natural cave can be the ideal of 50 degrees. Because of our location, we need some Azure IoT intervention!

**Sending and Receiving Telemetry**

The frequency of telemetry output is an important factor. A temperature sensor in a refrigeration unit may only have to report every minute, or less. An acceleration sensor on an aircraft may have to report at least every second.

An IoT device may contain one or more sensors, and have some computational power. There may be LED lights, and even a small screen, on the IoT device. However, the device isn't intended for direct use by a human operator. An IoT device is designed to receive its instructions from the cloud.

**Control a Cheese Cave Device**

In this module, we assume the IoT cheese cave monitoring device has temperature and humidity sensors. The device has a fan capable of both cooling or heating, and humidifying or de-humidifying. Every few seconds, the device sends current temperature and humidity values to the IoT Hub. This rapid frequency is unrealistic for a cheese cave (maybe every 15 minutes, or less, would be granular enough), except during code development when we want rapid activity!

For this lab, we assume that the fan can be in one of three states: on, off, and failed. The fan is initialized to the off state. In a later unit, the fan is turned on by use of a direct method.

Another feature of our IoT device is that it can accept desired values from the IoT Hub. The device can then adjust its fan to target these desired values. These values are coded in this module using a feature called device twins. Desired values will override any default settings for the device.

**Coding the Sample**

The coding in this module is broken down into three parts: sending and receiving telemetry, sending and receiving a direct method, and managing device twins.

Let's start by writing two apps: one for the device to send telemetry, and one back-end service to run in the cloud, to receive the telemetry. You'll be able to select your preferred language (Node.js or C#), and development environment (Visual Studio Code, or Visual Studio).

In this lab you will:

* Create a custom Azure IoT Hub, using the IoT Hub portal
* Create an IoT Hub device ID, using the IoT Hub portal
* Create an app to send device telemetry to the custom IoT Hub, in C# or Node.js
* Create a back-end service app to listen for the telemetry
* Implement a direct method, to communicate settings to the remote device
* Implement device twins, to maintain remote device properties
@@@


#### Exercise 1: Create a custom Azure IoT Hub, using the IoT Hub portal

This lab assumes the following resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-{YOUR-ID} |
| IoT Device | CheeseCaveID |

To create these resources, please update and execute the **lab-setup.azcli** script before starting the lab.

1. [ ] Using a browser, open the **Azure Shell** `https://shell.azure.com` and login with the Azure subscription you are using for this course.

1. [ ] To ensure the Azure Shell is using **Bash**, ensure the dropdown selected value in the top-left is **Bash**.

1. [ ] To upload the setup script, in the Azure Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. [ ] In the dropdown, select **Upload** and in the file selection dialog, navigate to the **lab-setup.azcli** file for this lab. Select the file and click **Open** to upload it.

    A notification will appear when the file upload has completed.

1. [ ] You can verify that the file has uploaded by listing the content of the current directory by entering the `ls` command.

1. [ ] To create a directory for this lab, move **lab-setup.azcli** into that directory, and make that the current working directory, enter the following commands:

    ```bash
    mkdir lab15
    mv lab-setup.azcli lab15
    cd lab15
    ```

1. [ ] To ensure the **lab-setup.azcli** has the execute permission, enter the following commands:

    ```bash
    chmod +x lab-setup.azcli
    ```

1. [ ] To edit the **lab-setup.azcli** file, click **{ }** (Open Editor) in the toolbar (second button from the right). In the **Files** list, select **lab15** to expand it and then select **lab-setup.azcli**.

    The editor will now show the contents of the **lab-setup.azcli** file.

1. [ ] In the editor, update the values of the `YourID` and `Location` variables. Set `YourID` to your initials and todays date - i.e. **CAH121119**, and set `Location` to the location that makes sense for your resources.

@@@warning
**Note**: The `Location` variable should be set to the short name for the location. You can see a list of the available locations and their short-names (the **Name** column) by entering this command:
    
     az account list-locations -o Table
    
     ```text
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
**Note**: You can use **CTRL+S** to save at any time and **CTRL+Q** to close the editor.
@@@

1. [ ] To create a resource group named **AZ-220-RG**, create an IoT Hub named **AZ-220-HUB-{YourID}**, add a device with an ID of **CheeseCaveID**, and display the device connection string, enter the following command:

    ```bash
    ./lab-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

1. [ ] Once complete, the script will be display data similar to:

    ```text
    Configuration Data:
    ------------------------------------------------
    AZ-220-HUB-DM121119 hub connectionstring:
    HostName=AZ-220-HUB-DM121119.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=nV9WdF3Xk0jYY2Da/pz2i63/3lSeu9tkW831J4aKV2o=

    CheeseCaveID device connection string:
    HostName=AZ-220-HUB-DM121119.azure-devices.net;DeviceId=CheeseCaveID;SharedAccessKey=TzAzgTYbEkLW4nWo51jtgvlKK7CUaAV+YBrc0qj9rD8=

    AZ-220-HUB-DM121119 eventhub endpoint:
    sb://iothub-ns-az-220-hub-2610348-5a463f1b56.servicebus.windows.net/

    AZ-220-HUB-DM121119 eventhub path:
    az-220-hub-dm121119

    AZ-220-HUB-DM121119 eventhub SaS primarykey:
    tGEwDqI+kWoZroH6lKuIFOI7XqyetQHf7xmoSf1t+zQ=
    ```

    Copy these values to a local text file - you will need them for the coding portion of this lab.
    
You've now completed the preparatory work for this module, the next steps are all coding and testing. Before we advance though, a quick knowledge check!

#### Exercise 2: Write Code to Send and Receive Telemetry

@@@secondary
At the end of this unit, you'll be sending and receiving telemetry.
@@@

##### Create an app to send telemetry

1. [ ] To use C# in Visual Studio Code, ensure both [.NET Core](https://dotnet.microsoft.com/download), and the [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) are installed.

1. [ ]  To open a terminal in Visual Studio Code, open the **Terminal** menu and click **New Terminal**.

1. [ ] In the terminal, to create a directory called "cheesecavedevice" and change the current directory to that directory, enter the following commands:

   ```bash
   mkdir cheesecavedevice
   cd cheesecavedevice
   ```

1. [ ] To create a new .NET console application. enter the following command in the terminal:

    ```bash
    dotnet new console
    ```

    This command creates a **Program.cs** file in your folder, along with a project file.

1. [ ] In the terminal, to install the required libraries. Enter the following commands:

    ```bash
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Microsoft.Azure.Devices.Shared
    dotnet add package Newtonsoft.Json
    ```

1. [ ] From the **File** menu, open up the **Program.cs** file, and delete the default contents.
@@@warning
**Note**: If you are unsure where the **Program.cs** file is located, enter the command `pwd` in the console to see the current directory.
@@@

1. [ ] After you've entered the code below into the **Program.cs** file, you can run the app with the command `dotnet run`. This command will run the **Program.cs** file in the current folder.

##### Add Code to Send Telemetry

This section adds code to send telemetry from a simulated device. The device sends temperature (in degrees fahrenheit) and humidity (in percentages), regardless of whether any back-end app is listening or not.

1. [ ] If it isn't already open in Visual Studio Code, open the **Program.cs** file for the device app.

1. [ ] Copy and paste the following code:

    ```csharp
    // Copyright (c) Microsoft. All rights reserved.
    // Licensed under the MIT license. See LICENSE file in the project root for full license information.

    using System;
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    using System.Text;
    using System.Threading.Tasks;
    using Newtonsoft.Json.Linq;

    namespace simulated_device
    {
        class SimulatedDevice
        {
        // Global constants.
            const float ambientTemperature = 70;                    // Ambient temperature of a southern cave, in degrees F.
            const double ambientHumidity = 99;                      // Ambient humidity in relative percentage of air saturation.
            const double desiredTempLimit = 5;                      // Acceptable range above or below the desired temp, in degrees F.
            const double desiredHumidityLimit = 10;                 // Acceptable range above or below the desired humidity, in percentages.
            const int intervalInMilliseconds = 5000;                // Interval at which telemetry is sent to the cloud.

            // Global variables.
            private static DeviceClient s_deviceClient;
            private static stateEnum fanState = stateEnum.off;                      // Initial setting of the fan.
            private static double desiredTemperature = ambientTemperature - 10;     // Initial desired temperature, in degrees F.
            private static double desiredHumidity = ambientHumidity - 20;           // Initial desired humidity in relative percentage of air saturation.

            // Enum for the state of the fan for cooling/heating, and humidifying/de-humidifying.
            enum stateEnum
            {
                off,
                on,
                failed
            }

            // The device connection string to authenticate the device with your IoT hub.
            private readonly static string s_deviceConnectionString = "<your device connection string>";

            private static void colorMessage(string text, ConsoleColor clr)
            {
                Console.ForegroundColor = clr;
                Console.WriteLine(text);
                Console.ResetColor();
            }
            private static void greenMessage(string text)
            {
                colorMessage(text, ConsoleColor.Green);
            }

            private static void redMessage(string text)
            {
                colorMessage(text, ConsoleColor.Red);
            }

            // Async method to send simulated telemetry.
            private static async void SendDeviceToCloudMessagesAsync()
            {
                double currentTemperature = ambientTemperature;         // Initial setting of temperature.
                double currentHumidity = ambientHumidity;               // Initial setting of humidity.

                Random rand = new Random();

                while (true)
                {
                    // Simulate telemetry.
                    double deltaTemperature = Math.Sign(desiredTemperature - currentTemperature);
                    double deltaHumidity = Math.Sign(desiredHumidity - currentHumidity);

                    if (fanState == stateEnum.on)
                    {
                        // If the fan is on the temperature and humidity will be nudged towards the desired values most of the time.
                        currentTemperature += (deltaTemperature * rand.NextDouble()) + rand.NextDouble() - 0.5;
                        currentHumidity += (deltaHumidity * rand.NextDouble()) + rand.NextDouble() - 0.5;

                        // Randomly fail the fan.
                        if (rand.NextDouble() < 0.01)
                        {
                            fanState = stateEnum.failed;
                            redMessage("Fan has failed");
                        }
                    }
                    else
                    {
                        // If the fan is off, or has failed, the temperature and humidity will creep up until they reaches ambient values, thereafter fluctuate randomly.
                        if (currentTemperature < ambientTemperature - 1)
                        {
                            currentTemperature += rand.NextDouble() / 10;
                        }
                        else
                        {
                            currentTemperature += rand.NextDouble() - 0.5;
                        }
                        if (currentHumidity < ambientHumidity - 1)
                        {
                            currentHumidity += rand.NextDouble() / 10;
                        }
                        else
                        {
                            currentHumidity += rand.NextDouble() - 0.5;
                        }
                    }

                    // Check: humidity can never exceed 100%.
                    currentHumidity = Math.Min(100, currentHumidity);

                    // Create JSON message.
                    var telemetryDataPoint = new
                    {
                        temperature = Math.Round(currentTemperature, 2),
                        humidity = Math.Round(currentHumidity, 2)
                    };
                    var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
                    var message = new Message(Encoding.ASCII.GetBytes(messageString));

                    // Add custom application properties to the message.
                    message.Properties.Add("sensorID", "S1");
                    message.Properties.Add("fanAlert", (fanState == stateEnum.failed) ? "true" : "false");

                    // Send temperature or humidity alerts, only if they occur.
                    if ((currentTemperature > desiredTemperature + desiredTempLimit) || (currentTemperature < desiredTemperature - desiredTempLimit))
                    {
                        message.Properties.Add("temperatureAlert", "true");
                    }
                    if ((currentHumidity > desiredHumidity + desiredHumidityLimit) || (currentHumidity < desiredHumidity - desiredHumidityLimit))
                    {
                        message.Properties.Add("humidityAlert", "true");
                    }

                    Console.WriteLine("Message data: {0}", messageString);

                    // Send the telemetry message.
                    await s_deviceClient.SendEventAsync(message);
                    greenMessage("Message sent\n");

                    await Task.Delay(intervalInMilliseconds);
                }
            }
            private static void Main(string[] args)
            {
                colorMessage("Cheese Cave device app.\n", ConsoleColor.Yellow);

                // Connect to the IoT hub using the MQTT protocol.
                s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

                SendDeviceToCloudMessagesAsync();
                Console.ReadLine();
            }
        }
    }
    ```
@@@danger
**Important:** Read through the comments in the code, noting how the temperature and humidity settings from the description of the scenario in the introduction have worked their way into the code.
@@@

1. [ ] Replace the `<your device connection string>` with the device connection string you saved off in the previous unit. No other lines of code need to be changed.

1. [ ] Save the **Program.cs** file.

##### Test your Code to Send Telemetry

1. [ ] To run the app in the terminal, enter the following command:

    ```bash
    dotnet run
    ```

   This command will run the **Program.cs** file in the current folder.

1. [ ] You should quickly see console output, similar to the following:
@@@warning
**Note**: Green text is used to show things are working as they should and red text when bad stuff is happening. If you don't get a screen similar to this image, start by checking your device connection string.
@@@

1. [ ] Watch the telemetry for a short while, checking that it is giving vibrations in the expected ranges.

1. [ ] You can leave this app running, as it's needed for the next section.

##### Create a Second App to Receive Telemetry

Now we have a device pumping out telemetry, we need to listen for that telemetry with a back-end app, also connected to our IoT Hub.

1. [ ]  As the device app is running in a copy of Visual Studio Code, you will need to open a new instance of Visual Studio Code.

1. [ ] To open a terminal in Visual Studio Code, open the **Terminal** menu and click **New Terminal**.

1. [ ] In the terminal, to create a directory called "cheesecaveoperator" and change the current directory to that directory, enter the following commands:

   ```bash
   mkdir cheesecaveoperator
   cd cheesecaveoperator
   ```

1. [ ] To create a new .NET console application. enter the following command in the terminal:

    ```bash
    dotnet new console
    ```

    This command creates a **Program.cs** file in your folder, along with a project file.

1. [ ] In the terminal, to install the required libraries. Enter the following commands:

    ```bash
    dotnet add package Microsoft.Azure.EventHubs
    dotnet add package Microsoft.Azure.Devices
    dotnet add package Newtonsoft.Json
    ```

1. [ ] From the **File** menu, open up the **Program.cs** file, and delete the default contents.
@@@warning
**Note**: If you are unsure where the **Program.cs** file is located, enter the command `pwd` in the console to see the current directory.
@@@

1. [ ] After you've entered the code below into the **Program.cs** file, you can run the app with the command `dotnet run`. This command will run the **Program.cs** file in the current folder.

##### Add Code to Receive Telemetry

This section adds code to receive telemetry from the IoT Hub Event Hub endpoint. 

1. [ ] If it isn't already open in Visual Studio Code, open the **Program.cs** file for the device app.

1. [ ]  Copy and paste the following code:

    ```csharp
    // Copyright (c) Microsoft. All rights reserved.
    // Licensed under the MIT license. See LICENSE file in the project root for full license information.

    using System;
    using System.Threading.Tasks;
    using System.Text;
    using System.Collections.Generic;
    using System.Linq;

    using Microsoft.Azure.EventHubs;
    using Microsoft.Azure.Devices;
    using Newtonsoft.Json;

    namespace cheesecave_operator
    {
        class ReadDeviceToCloudMessages
        {
            // Global variables.
            // The Event Hub-compatible endpoint.
            private readonly static string s_eventHubsCompatibleEndpoint = "<your event hub endpoint>";

            // The Event Hub-compatible name.
            private readonly static string s_eventHubsCompatiblePath = "<your event hub path>";
            private readonly static string s_iotHubSasKey = "<your event hub Sas key>";
            private readonly static string s_iotHubSasKeyName = "service";
            private static EventHubClient s_eventHubClient;

            // Connection string for your IoT Hub.
            private readonly static string s_serviceConnectionString = "<your service connection string>";

            // Asynchronously create a PartitionReceiver for a partition and then start reading any messages sent from the simulated client.
            private static async Task ReceiveMessagesFromDeviceAsync(string partition)
            {
                // Create the receiver using the default consumer group.
                var eventHubReceiver = s_eventHubClient.CreateReceiver("$Default", partition, EventPosition.FromEnqueuedTime(DateTime.Now));
                Console.WriteLine("Created receiver on partition: " + partition);

                while (true)
                {
                    // Check for EventData - this methods times out if there is nothing to retrieve.
                    var events = await eventHubReceiver.ReceiveAsync(100);

                    // If there is data in the batch, process it.
                    if (events == null) continue;

                    foreach (EventData eventData in events)
                    {
                        string data = Encoding.UTF8.GetString(eventData.Body.Array);

                        greenMessage("Telemetry received: " + data);

                        foreach (var prop in eventData.Properties)
                        {
                            if (prop.Value.ToString() == "true")
                            {
                                redMessage(prop.Key);
                            }
                        }
                        Console.WriteLine();
                    }
                }
            }

            public static void Main(string[] args)
            {
                colorMessage("Cheese Cave Operator\n", ConsoleColor.Yellow);

                // Create an EventHubClient instance to connect to the IoT Hub Event Hubs-compatible endpoint.
                var connectionString = new EventHubsConnectionStringBuilder(new Uri(s_eventHubsCompatibleEndpoint), s_eventHubsCompatiblePath, s_iotHubSasKeyName, s_iotHubSasKey);
                s_eventHubClient = EventHubClient.CreateFromConnectionString(connectionString.ToString());

                // Create a PartitionReceiver for each partition on the hub.
                var runtimeInfo = s_eventHubClient.GetRuntimeInformationAsync().GetAwaiter().GetResult();
                var d2cPartitions = runtimeInfo.PartitionIds;

                // Create receivers to listen for messages.
                var tasks = new List<Task>();
                foreach (string partition in d2cPartitions)
                {
                    tasks.Add(ReceiveMessagesFromDeviceAsync(partition));
                }

                // Wait for all the PartitionReceivers to finish.
                Task.WaitAll(tasks.ToArray());
            }

            private static void colorMessage(string text, ConsoleColor clr)
            {
                Console.ForegroundColor = clr;
                Console.WriteLine(text);
                Console.ResetColor();
            }
            private static void greenMessage(string text)
            {
                colorMessage(text, ConsoleColor.Green);
            }

            private static void redMessage(string text)
            {
                colorMessage(text, ConsoleColor.Red);
            }
        }
    }
    ```
@@@danger
**Important:** Read through the comments in the code. Our implementation only reads messages after the back-end app has been started. Any telemetry sent prior to this isn't handled.
@@@

1. [ ] Replace the `<your device connection string>` with the device connection string you saved off in the previous unit. No other lines of code need to be changed.

1. [ ]  Replace the `<your event hub endpoint>`, `<your event hub path>`, and the `<your event hub Sas key>` with the strings you saved off to your text file.

1. [ ] Save the **Program.cs** file.

##### Test your Code to Receive Telemetry

This test is important, checking whether your back-end app is picking up the telemetry being sent out by your simulated device. Remember your device app is still running, and sending telemetry.

1. [ ] To run the app in the terminal, enter the following command:

    ```bash
    dotnet run
    ```

   This command will run the **Program.cs** file in the current folder.
@@@warning
**Note**: You can ignore the warning about the unused variable `s_serviceConnectionString` - we will be using that variable shortly.
@@@

1. [ ] You should quickly see console output, and immediately respond if it successfully connects to IoT Hub. If not, carefully check your IoT Hub service connection string, noting that this string should be the service connection string, and not any other.:
@@@warning
**Note**: Green text is used to show things are working as they should and red text when bad stuff is happening. If you don't get a screen similar to this image, start by checking your device connection string.
@@@

1. [ ] Watch the telemetry for a short while, checking that it is giving vibrations in the expected ranges.

1. [ ] You can leave this app running, as it's needed for the next section.

1. [ ] Visually compare the telemetry sent and received. Is there an exact match? Is there much of a delay? If it looks good, close both the console windows for now.

 Completing this unit is great progress. you've an app sending telemetry from a device, and a back-end app acknowledging receipt of the data. This unit covers the monitoring side of our scenario. The next step handles the control side - what to do when issues arise with the data. Clearly, there are issues, we're getting temperature and humidity alerts!

#### Exercise 3: Write Code to Invoke a Direct Method

@@@secondary
In this unit, we'll add code to the device app for a direct method to turn on the fan. Next, we add code to the back-end service app to invoke this direct method.

Calls from the back-end app to invoke direct methods can include multiple parameters as part of the payload. Direct methods are typically used to turn features of the device off and on, or specify settings for the device.

**Handle Error Conditions**

There are several error conditions that need to be checked for when a device receives instructions to run a direct method. One of these checks is simply to respond with an error if the fan is in a failed state. Another error condition to report is when an invalid parameter is received. Clear error reporting is important, given the potential remoteness of the device.

**Invoke a Direct Method**

Direct methods require that the back-end app prepares the parameters, then makes a call specifying a single device to invoke the method. The back-end app will then wait for, and report, a response.

The device app contains the functional code for the direct method. The function name is registered with the IoT client for the device. This process ensures the client knows what function to run when the call comes from the IoT Hub (there could be many direct methods).
@@@

##### Add Code to Define a Direct Method in the Device App

1. [ ]  Return to the Visual Studio Code instance that is running the **cheesecavedevice** app.

1. [ ] If the app is still running, place input focus on the terminal and press **CTRL+C** to exit the app.

1. [ ] In the editor, ensure **Program.cs** is open.

1. [ ] To define the direct method, add the following code at the end of the **SimulatedDevice** class:

    ```csharp
    // Handle the direct method call
    private static Task<MethodResponse> SetFanState(MethodRequest methodRequest, object userContext)
    {
        if (fanState == stateEnum.failed)
        {
            // Acknowledge the direct method call with a 400 error message.
            string result = "{\"result\":\"Fan failed\"}";
            redMessage("Direct method failed: " + result);
            return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
        }
        else
        {
            try
            {
                var data = Encoding.UTF8.GetString(methodRequest.Data);

                // Remove quotes from data.
                data = data.Replace("\"", "");

                // Parse the payload, and trigger an exception if it's not valid.
                fanState = (stateEnum)Enum.Parse(typeof(stateEnum), data);
                greenMessage("Fan set to: " + data);

                // Acknowledge the direct method call with a 200 success message.
                string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
            }
            catch
            {
                // Acknowledge the direct method call with a 400 error message.
                string result = "{\"result\":\"Invalid parameter\"}";
                redMessage("Direct method failed: " + result);
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
            }
        }
    }
    ```
@@@warning
**Note**: This code defines the implementation of the direct method and is executed when the direct method is invoked. The fan has three states: **on**, **off**, and **failed**. The method above sets the fan to either of the first two of these states. If the payload text doesn't match one of these two, or the fan is in a failed state, an error is returned.
@@@

1. [ ] To register the direct method, add the following lines of code to the Main method, after creating the device client.

    ```csharp
    // Create a handler for the direct method call
    s_deviceClient.SetMethodHandlerAsync("SetFanState", SetFanState, null).Wait();
    ```

    The modified **Main** method should look like:

    ```csharp
    private static void Main(string[] args)
    {
        colorMessage("Cheese Cave device app.\n", ConsoleColor.Yellow);

        // Connect to the IoT hub using the MQTT protocol.
        s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

        // Create a handler for the direct method call
        s_deviceClient.SetMethodHandlerAsync("SetFanState", SetFanState, null).Wait();

        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();
    }
    ```

1. [ ] Save the **Program.cs** file.

You've completed what is needed at the device end of things. Next, we need to add code to the back-end service.

##### Add Code to Call a Direct Method in the Back End App

1. [ ] Return to the Visual Studio Code instance that is running the **cheesecaveoperator** app.

1. [ ] If the app is still running, place input focus on the terminal and press **CTRL+C** to exit the app.

1. [ ] In the editor, ensure **Program.cs** is open.

1. [ ] Add the following line to the global variables at the top of the **ReadDeviceToCloudMessages** class:

    ```csharp
    private static ServiceClient s_serviceClient;
    ```

1. [ ] Add the following task to the **ReadDeviceToCloudMessages** class, after the **Main** method:

    ```csharp
    // Handle invoking a direct method.
    private static async Task InvokeMethod()
    {
        try
        {
            var methodInvocation = new CloudToDeviceMethod("SetFanState") { ResponseTimeout = TimeSpan.FromSeconds(30) };
            string payload = JsonConvert.SerializeObject("on");

            methodInvocation.SetPayloadJson(payload);

            // Invoke the direct method asynchronously and get the response from the simulated device.
            var response = await s_serviceClient.InvokeDeviceMethodAsync("CheeseCaveID", methodInvocation);

            if (response.Status == 200)
            {
                greenMessage("Direct method invoked: " + response.GetPayloadAsJson());
            }
            else
            {
                redMessage("Direct method failed: " + response.GetPayloadAsJson());
            }
        }
        catch
        {
            redMessage("Direct method failed: timed-out");
        }
    }
    ```
@@@warning
**Note**: This code is used to invoke the **SetFanState** direct method on the device app.
@@@

1. [ ] Add the following code to the **Main** method, before creating the receivers to listen for messages:

    ```csharp
    // Create a ServiceClient to communicate with service-facing endpoint on your hub.
    s_serviceClient = ServiceClient.CreateFromConnectionString(s_serviceConnectionString);
    InvokeMethod().GetAwaiter().GetResult();
    ```
@@@warning
**Note**: This code creates the client we used to connect to the IoT Hub so we can invoke the direct method on the device.
@@@

1. [ ] Save the **Program.cs** file.

You have now completed the code changes to support the **SetFanState** direct method.

##### Test the direct method

To test the method, start the apps in the correct order. We can't invoke a direct method that hasn't been registered!

1. [ ] Start the **cheesecavedevice** device app. It will begin writing to the terminal, and telemetry will appear.

1. [ ] Start the **cheesecaveoperator** back-end app. This app immediately calls the direct method. Do you notice it's handled by the back-end app, with output similar to the following?
@@@warning
**Note**: If you see the message `Direct method failed: timed-out` then double check you have saved the changes in the **cheesecavedevice** and started the app.
@@@

1. [ ] Now check the console output for the **cheesecavedevice** device app, you should see that the fan has been turned on.

You are now successfully monitoring and controlling a remote device. We have turned on the fan, which will slowly move the environment in the cave to our initial desired settings. However, we might like to remotely specify those desired settings. We could specify desired settings with a direct method (which is a valid approach). Or we could use another feature of IoT Hub, called device twins. Let's look into the technology of device twins.





#### Exercise 4: Write Code for Device Twins

@@@secondary
In this exercise, we'll add some code to both the device app and back-end service app, to show device twin synchronization in operation.

As a reminder, a device twin contains four types of information:

* **Tags**: information on the device that isn't visible to the device.
* **Desired properties**: the desired settings specified by the back-end app.
* **Reported properties**: the reported values of the settings on the device.
* **Device identity properties**: read-only information identifying the device.

Device twins are designed for querying, and automatically synchronizing, with the real IoT Hub device. The device twin can be queried, at any time, by the back-end app. This query can return the current state information for the device. Getting this data doesn't involve a call to the device, as the device and twin will have synchronized automatically. Much of the functionality of device twins is provided by Azure IoT, so not much code needs to be written to make use of them.

There is some overlap between the functionality of device twins and direct methods. We could set desired properties using direct methods, which might seem an intuitive way of doing things. However, using direct methods would require the back-end app to record those settings explicitly, if they ever needed to be accessed. Using device twins, this information is stored and maintained by default.
@@@

##### Add Code To Use Device Twins To Synchronize Device Properties

1. [ ] Return to the Visual Studio Code instance that is running the **cheesecaveoperator** app.

1. [ ] If the app is still running, place input focus on the terminal and press **CTRL+C** to exit the app.

1. [ ] In the editor, ensure **Program.cs** is open.

1. [ ] Add the following code to the end of the **ReadDeviceToCloudMessages** class:

    ```csharp
    // Device twins section.
    private static RegistryManager registryManager;

    private static async Task SetTwinProperties()
    {
        var twin = await registryManager.GetTwinAsync("CheeseCaveID");
        var patch =
            @"{
                tags: {
                    customerID: 'Customer1',
                    cellar: 'Cellar1'
                },
                properties: {
                    desired: {
                        patchId: 'set values',
                        temperature: '50',
                        humidity: '85'
                    }
                }
        }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);

        var query = registryManager.CreateQuery(
          "SELECT * FROM devices WHERE tags.cellar = 'Cellar1'", 100);
        var twinsInCellar1 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Cellar1: {0}",
          string.Join(", ", twinsInCellar1.Select(t => t.DeviceId)));

    }
    ```
@@@warning
**Note**: The **SetTwinProperties** method creates a piece of JSON that defines tags and properties that will be added to the device twin, and then updates thew twin. The next part of the method demonstrates how a query can be performed to list the devices where the **cellar** tag is set to "Cellar1".
@@@

1. [ ]  Now, add the following lines to the **Main** method, before the lines creating a service client.

    ```csharp
    // A registry manager is used to access the digital twins.
    registryManager = RegistryManager.CreateFromConnectionString(s_serviceConnectionString);
    SetTwinProperties().Wait();
    ```
@@@warning
**Note**: Read the comments in this section of code.
@@@

1. [ ] Save the **Program.cs** file.

##### Add Code to Synchronize Device Twin Settings for the Device

1. [ ] Return to the Visual Studio Code instance that is running the **cheesecavedevice** app.

1. [ ] If the app is still running, place input focus on the terminal and press **CTRL+C** to exit the app.

1. [ ] In the editor, ensure **Program.cs** is open.

1. [ ] Add the following code at the end of the **SimulatedDevice** class:

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        try
        {
            desiredHumidity = desiredProperties["humidity"];
            desiredTemperature = desiredProperties["temperature"];
            greenMessage("Setting desired humidity to " + desiredProperties["humidity"]);
            greenMessage("Setting desired temperature to " + desiredProperties["temperature"]);

            // Report the properties back to the IoT Hub.
            var reportedProperties = new TwinCollection();
            reportedProperties["fanstate"] = fanState.ToString();
            reportedProperties["humidity"] = desiredHumidity;
            reportedProperties["temperature"] = desiredTemperature;
            await s_deviceClient.UpdateReportedPropertiesAsync(reportedProperties);

            greenMessage("\nTwin state reported: " + reportedProperties.ToJson());
        }
        catch
        {
            redMessage("Failed to update device twin");
        }
    }
    ```
@@@warning
**Note**: This code defines handler invoked when a desired property changes in the device twin. Notice that new values are then reported back to the IoT Hub to confirm the change.
@@@

1. [ ] To register the desired property changed handler, add the following lines after the statements creating a handler for the direct method:

    ```csharp
    // Get the device twin to report the initial desired properties.
    Twin deviceTwin = s_deviceClient.GetTwinAsync().GetAwaiter().GetResult();
    greenMessage("Initial twin desired properties: " + deviceTwin.Properties.Desired.ToJson());

    // Set the device twin update callback.
    s_deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();
    ```

1. [ ] Save the **Program.cs** file.
@@@warning
**Note**: Now you have added device twins to your app, you can reconsider having explicit variables such as **desiredHumidity**. Instead, you can use the variables in the device twin object.
@@@

##### Test the Device Twins

To test the method, start the apps in the correct order. 

1. [ ] Start the **cheesecavedevice** device app. It will begin writing to the terminal, and telemetry will appear.

1. [ ] Start the **cheesecaveoperator** back-end app. 

1. [ ] Now check the console output for the **cheesecavedevice** device app, confirming the device twin synchronized correctly.

1. [ ]  If we let the fan do its work, we should eventually get rid of those red alerts!

The code given in this module isn't industrial quality. It does show how to use direct methods, and device twins. However, the messages are sent only when the back-end service app is first run. Typically, a back-end service app would require a browser interface, for an operator to send direct methods, or set device twin properties, when required.
@@@warning
**Note**: Before you go, don't forget to close both instances of Visual Studio Code - this will exit the apps if they are still running.
@@@

@@@success
**Congratulations**! you have now completed this lab!
@@@
