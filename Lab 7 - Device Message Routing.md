### Lab - Device Message Routing

@@@secondary
**Scenario**

Suppose you manage a packaging facility. Packages are assembled for shipping, then placed on a conveyor belt that takes the packages and drops them off in mailing bins. Your metric for success is the number of packages leaving the conveyor belt.

The conveyor belt is a critical link in your process, and is monitored for vibration. The conveyor belt has three speeds: stopped, slow, and fast. The number of packages being delivered at slow speed is less than at the faster speed, though the vibration is also less at the slower speed. If the vibration becomes excessive, the conveyor belt has to be stopped and inspected.

There are a number of different types of vibration. Forced vibration is vibration caused by an external force. Such a force as the broken wheel example, or a weighty package placed improperly on the conveyor belt. There's also increasing vibration, which might happen if a design limit is exceeded.

Vibration is typically measured as an acceleration (meters per second squared, m/s2).

The ultimate goal here is preventive maintenance. Detect that something is wrong, before any damage is caused.

It's not always easy to detect abnormal vibration levels. For this reason, you are looking to Azure IoT Hub to detect data anomalies. You plan to have a vibration detection sensor on the conveyor belt, sending continuous telemetry to an IoT Hub. The IoT Hub will use Azure Stream Analytics, and a built-in ML model, to give you advance warning of vibration anomalies. You also plan to archive all the telemetry data, just in case it's ever needed.

You decide to build a prototype of the planned system, initially using simulated telemetry.

In this lab, you will 

This lab includes:

* Verify Lab Prerequisites
* Create an Azure IoT Hub, and a device ID using Azure CLI
* Create a C# app to send device telemetry to the IoT Hub, using Visual Studio code
* Create a message route, through to blob storage, using the Azure portal
* Create a second message route, through to an Azure Analytics job, using the Azure portal
* Create an Azure Function to identify anomalies.
@@@

#### Exercise 1: Verify Lab Prerequisites

This lab assumes the following resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |
| Device ID | VibrationSensorId |

If the resources are unavailable, please execute the **lab-setup.azcli** script before starting the lab.

The **lab-setup.azcli** script is written to run in a **bash** shell environment - the easiest way to execute this is in the Azure Cloud Shell.

1. [ ] Using a browser, open the `https://shell.azure.com` and login with the Azure subscription you are using for this course.

1. [ ] To ensure the Azure Shell is using **Bash**, ensure the dropdown selected value in the top-left is **Bash**.

1. [ ] To upload the setup script, in the Azure Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. [ ] In the dropdown, select **Upload** and in the file selection dialog, navigate to the **lab-setup.azcli** file for this lab. Select the file and click **Open** to upload it.

    A notification will appear when the file upload has completed.

1. [ ] You can verify that the file has uploaded by listing the content of the current directory by entering the `ls` command.

1. [ ] To create a directory for this lab, move **lab-setup.azcli** into that directory, and make that the current working directory, enter the following commands:

    ```bash
    mkdir lab7
    mv lab-setup.azcli lab7
    cd lab7
    ```

1. [ ] To ensure that **lab07-setup.azcli** has the execute permission, enter the following command:

    ```bash
    chmod +x lab-setup.azcli
    ```

1. [ ] To edit the **lab-setup.azcli** file, click **{ }** (Open Editor) in the toolbar (second button from the right). In the **Files** list, select **lab7** to expand it and then select **lab-setup.azcli**.

    The editor will now show the contents of the **lab-setup.azcli** file.

1. [ ] In the editor, update the values of the `YourID` and `Location` variables. Set `YourID` to your initials and todays date - i.e. **CAH121119**, and set `Location` to the location that makes sense for your resources.

@@@warning
**Note**: The `Location` variable should be set to the short name for the location. You can see a list of the available locations and their short-names (the **Name** column) by entering this command:
  
    
    az account list-locations -o Table
    
    text
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

1. [ ] To create a resource group named **AZ-220-RG**, create an IoT Hub named **AZ-220-HUB-{YourID}**, add a device with an ID of **VibrationSensorId**, and display the device connection string, enter the following command:

    ```bash
    ./lab-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

1. [ ] Notice that, once the script has completed, the connection string for the device is displayed.

    The connection string starts with "HostName="

1. [ ] Copy the connection string into a text document, and note that it is for the **VibrationSensorId** device.

    The next step is to code the sending of telemetry messages.

#### Exercise 2: Write Code for Vibration Telemetry

@@@secondary
The key to monitoring our conveyor belt is the output of vibration telemetry. Vibration is usually measured as an acceleration (m/s^2), although sometimes it's measured in g-forces, where 1 g = 9.81 m/s^2. There are three types of vibration.

* Natural vibration, which is just the frequency a structure tends to oscillate.
* Free vibration, which occurs when the structure is impacted, but then left to oscillate without interference.
* Forced vibration, which occurs when the structure is under some stress.

Forced vibration is the dangerous one for our conveyor belt. Even if it starts at a low level this vibration can build so that the structure fails prematurely. There's less of a case for free vibration in conveyor belt operation. Most machines, as we all know, have a natural vibration.

The code sample that you will build simulates a conveyor belt running at a range of speeds (stopped, slow, fast). The faster the belt is running, the more packages are delivered, but the greater the effects of vibration. We'll add natural vibration, based on a sine wave with some randomization. It's possible our anomaly detection system will falsely identify a spike or dip in this sine wave as an anomaly. We'll then add two forms of forced vibration. The first has the effect of a cyclic increase in vibration (see the images below). And secondly, an increasing vibration, where an additional sine wave is added, starting small but growing.

We assume that our conveyor belt has just one sensor device (our simulated IoT Device). In addition to communicating vibration data, the sensor also pumps out some other data (packages delivered, ambient temperature, and similar metrics). For this lab, the additional values will be sent to a storage archive.

Almost all the coding in this lab will be completed during this exercise. You will be using Visual Studio Code to build the simulator code in C#.

In this exercise, you will:

* build the conveyor belt simulator
* send telemetry messages to the IoT Hub created in the previous unit

Later in this lab you will complete a small amount of SQL coding.
@@@

##### Task 1: Create an App to Send Telemetry

1. [ ] To use C# in Visual Studio Code, ensure both [.NET Core](https://dotnet.microsoft.com/download), and the [C# extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) are installed.

1. [ ] To open a terminal in Visual Studio Code, open the **Terminal** menu and click **New Terminal**.

1. [ ] In the terminal, to create a directory called "vibrationdevice" and change the current directory to that directory, enter the following commands:

   ```bash
   mkdir vibrationdevice
   cd vibrationdevice
   ```

1. [ ] To create a new .NET console application. enter the following command in the terminal:

    ```bash
    dotnet new console
    ```

    This command creates a **Program.cs** file in your folder, along with a project file.

1. [ ] In the terminal, to install the required libraries. Enter the following commands:

    ```bash
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Newtonsoft.Json
    ```

1. From the **File** menu, open up the **Program.cs** file, and delete the default contents.
@@@warning
**Note**: If you are unsure where the **Program.cs** file is located, enter the command `pwd` in the console to see the current directory.
@@@

1. [ ] After you've entered the code below into the **Program.cs** file, you can run the app with the command `dotnet run`. This command will run the **Program.cs** file in the current folder.

##### Task 2: Add Code to Send Telemetry

The following app simulates a conveyor belt, and reports vibration sensor data every two seconds.

1. [ ] If it isn't already open in Visual Studio Code, open the **Program.cs** file for the device app.

1. [ ] Copy and paste the following code:

    ```csharp
    // Copyright (c) Microsoft. All rights reserved.
    // Licensed under the MIT license. See LICENSE file in the project root for full license information.

    using System;
    using Microsoft.Azure.Devices.Client;
    using Newtonsoft.Json;
    using System.Text;
    using System.Threading.Tasks;

    namespace vibration_device
    {
        class SimulatedDevice
        {
            // Telemetry globals.
            private const int intervalInMilliseconds = 2000;                                // Time interval required by wait function.
            private static readonly int intervalInSeconds = intervalInMilliseconds / 1000;  // Time interval in seconds.

            // Conveyor belt globals.
            enum SpeedEnum
            {
                stopped,
                slow,
                fast
            }
            private static int packageCount = 0;                                        // Count of packages leaving the conveyor belt.
            private static SpeedEnum beltSpeed = SpeedEnum.stopped;                     // Initial state of the conveyor belt.
            private static readonly double slowPackagesPerSecond = 1;                   // Packages completed at slow speed/ per second
            private static readonly double fastPackagesPerSecond = 2;                   // Packages completed at fast speed/ per second
            private static double beltStoppedSeconds = 0;                               // Time the belt has been stopped.
            private static double temperature = 60;                                     // Ambient temperature of the facility.
            private static double seconds = 0;                                          // Time conveyor belt is running.

            // Vibration globals.
            private static double forcedSeconds = 0;                                    // Time since forced vibration started.
            private static double increasingSeconds = 0;                                // Time since increasing vibration started.
            private static double naturalConstant;                                      // Constant identifying the severity of natural vibration.
            private static double forcedConstant = 0;                                   // Constant identifying the severity of forced vibration.
            private static double increasingConstant = 0;                               // Constant identifying the severity of increasing vibration.

            // IoT Hub global variables.
            private static DeviceClient s_deviceClient;

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
            private static async void SendDeviceToCloudMessagesAsync(Random rand)
            {
                // Simulate the vibration telemetry of a conveyor belt.
                double vibration;

                while (true)
                {
                    // Randomly adjust belt speed.
                    switch (beltSpeed)
                    {
                        case SpeedEnum.fast:
                            if (rand.NextDouble() < 0.01)
                            {
                                beltSpeed = SpeedEnum.stopped;
                            }
                            if (rand.NextDouble() > 0.95)
                            {
                                beltSpeed = SpeedEnum.slow;
                            }
                            break;

                        case SpeedEnum.slow:
                            if (rand.NextDouble() < 0.01)
                            {
                                beltSpeed = SpeedEnum.stopped;
                            }
                            if (rand.NextDouble() > 0.95)
                            {
                                beltSpeed = SpeedEnum.fast;
                            }
                            break;

                        case SpeedEnum.stopped:
                            if (rand.NextDouble() > 0.75)
                            {
                                beltSpeed = SpeedEnum.slow;
                            }
                            break;
                    }

                    // Set vibration levels.
                    if (beltSpeed == SpeedEnum.stopped)
                    {
                        // If the belt is stopped, all vibration comes to a halt.
                        forcedConstant = 0;
                        increasingConstant = 0;
                        vibration = 0;

                        // Record how much time the belt is stopped, in case we need to send an alert.
                        beltStoppedSeconds += intervalInSeconds;
                    }
                    else
                    {
                        // Conveyor belt is running.
                        beltStoppedSeconds = 0;

                        // Check for random starts in unwanted vibrations.

                        // Check forced vibration.
                        if (forcedConstant == 0)
                        {
                            if (rand.NextDouble() < 0.1)
                            {
                                // Forced vibration starts.
                                forcedConstant = 1 + 6 * rand.NextDouble();             // A number between 1 and 7.
                                if (beltSpeed == SpeedEnum.slow)
                                    forcedConstant /= 2;                                // Lesser vibration if slower speeds.
                                forcedSeconds = 0;
                                redMessage($"Forced vibration starting with severity: {Math.Round(forcedConstant, 2)}");
                            }
                        }
                        else
                        {
                            if (rand.NextDouble() > 0.99)
                            {
                                forcedConstant = 0;
                                greenMessage("Forced vibration stopped");
                            }
                            else
                            {
                                redMessage($"Forced vibration: {Math.Round(forcedConstant, 1)} started at: {DateTime.Now.ToShortTimeString()}");
                            }
                        }

                        // Check increasing vibration.
                        if (increasingConstant == 0)
                        {
                            if (rand.NextDouble() < 0.05)
                            {
                                // Increasing vibration starts.
                                increasingConstant = 100 + 100 * rand.NextDouble();     // A number between 100 and 200.
                                if (beltSpeed == SpeedEnum.slow)
                                    increasingConstant *= 2;                            // Longer period if slower speeds.
                                increasingSeconds = 0;
                                redMessage($"Increasing vibration starting with severity: {Math.Round(increasingConstant, 2)}");
                            }
                        }
                        else
                        {
                            if (rand.NextDouble() > 0.99)
                            {
                                increasingConstant = 0;
                                greenMessage("Increasing vibration stopped");
                            }
                            else
                            {
                                redMessage($"Increasing vibration: {Math.Round(increasingConstant, 1)} started at: {DateTime.Now.ToShortTimeString()}");
                            }
                        }

                        // Apply the vibrations, starting with natural vibration.
                        vibration = naturalConstant * Math.Sin(seconds);

                        if (forcedConstant > 0)
                        {
                            // Add forced vibration.
                            vibration += forcedConstant * Math.Sin(0.75 * forcedSeconds) * Math.Sin(10 * forcedSeconds);
                            forcedSeconds += intervalInSeconds;
                        }

                        if (increasingConstant > 0)
                        {
                            // Add increasing vibration.
                            vibration += (increasingSeconds / increasingConstant) * Math.Sin(increasingSeconds);
                            increasingSeconds += intervalInSeconds;
                        }
                    }

                    // Increment the time since the conveyor belt app started.
                    seconds += intervalInSeconds;

                    // Count the packages that have completed their journey.
                    switch (beltSpeed)
                    {
                        case SpeedEnum.fast:
                            packageCount += (int)(fastPackagesPerSecond * intervalInSeconds);
                            break;

                        case SpeedEnum.slow:
                            packageCount += (int)(slowPackagesPerSecond * intervalInSeconds);
                            break;

                        case SpeedEnum.stopped:
                            // No packages!
                            break;
                    }

                    // Randomly vary ambient temperature.
                    temperature += rand.NextDouble() - 0.5d;

                    // Create two messages:
                    // 1. Vibration telemetry only, that is routed to Azure Stream Analytics.
                    // 2. Logging information, that is routed to an Azure storage account.

                    // Create the telemetry JSON message.
                    var telemetryDataPoint = new
                    {
                        vibration = Math.Round(vibration, 2),
                    };
                    var telemetryMessageString = JsonConvert.SerializeObject(telemetryDataPoint);
                    var telemetryMessage = new Message(Encoding.ASCII.GetBytes(telemetryMessageString));

                    // Add a custom application property to the message. This is used to route the message.
                    telemetryMessage.Properties.Add("sensorID", "VSTel");

                    // Send an alert if the belt has been stopped for more than five seconds.
                    telemetryMessage.Properties.Add("beltAlert", (beltStoppedSeconds > 5) ? "true" : "false");

                    Console.WriteLine($"Telemetry data: {telemetryMessageString}");

                    // Send the telemetry message.
                    await s_deviceClient.SendEventAsync(telemetryMessage);
                    greenMessage($"Telemetry sent {DateTime.Now.ToShortTimeString()}");

                    // Create the logging JSON message.
                    var loggingDataPoint = new
                    {
                        vibration = Math.Round(vibration, 2),
                        packages = packageCount,
                        speed = beltSpeed.ToString(),
                        temp = Math.Round(temperature, 2),
                    };
                    var loggingMessageString = JsonConvert.SerializeObject(loggingDataPoint);
                    var loggingMessage = new Message(Encoding.ASCII.GetBytes(loggingMessageString));

                    // Add a custom application property to the message. This is used to route the message.
                    loggingMessage.Properties.Add("sensorID", "VSLog");

                    // Send an alert if the belt has been stopped for more than five seconds.
                    loggingMessage.Properties.Add("beltAlert", (beltStoppedSeconds > 5) ? "true" : "false");

                    Console.WriteLine($"Log data: {loggingMessageString}");

                    // Send the logging message.
                    await s_deviceClient.SendEventAsync(loggingMessage);
                    greenMessage("Log data sent\n");

                    await Task.Delay(intervalInMilliseconds);
                }
            }

            private static void Main(string[] args)
            {
                Random rand = new Random();
                colorMessage("Vibration sensor device app.\n", ConsoleColor.Yellow);

                // Connect to the IoT hub using the MQTT protocol.
                s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt);

                // Create a number between 2 and 4, as a constant for normal vibration levels.
                naturalConstant = 2 + 2 * rand.NextDouble();

                SendDeviceToCloudMessagesAsync(rand);
                Console.ReadLine();
            }
        }
    }
    ```
@@@danger
**Important:** Take a few minutes, and read through the comments in the code. Notice how the vibration math from the description of the scenario in the introduction has worked its way into the code. The most important section of code for learning about IoT messages, starts with the "Create two messages:" comment.

@@@

1. [ ] Replace the `<your device connection string>` (line 44) with the device connection string you saved off in the previous unit. No other lines of code need to be changed.

1. [ ] Save the **Program.cs** file.
@@@warning
**Note:** The code is also available in the `/labFiles` folder - remember to replace the `<your device connection string>`.
@@@

##### Task 3: Test your Code to Send Telemetry

1. [ ] To run the app in the terminal, enter the following command:

    ```bash
    dotnet run
    ```

   This command will run the **Program.cs** file in the current folder.

1. [ ] You should quickly see console output, similar to the following:

    ![Console Output](./Media/M99-L07-vibration-telemetry.png)
@@@warning
**Note**: Green text is used to show things are working as they should and red text when bad stuff is happening. If you don't get a screen similar to this image, start by checking your device connection string.

@@@
1. [ ] Watch the telemetry for a short while, checking that it is giving vibrations in the expected ranges.

1. [ ] You can leave this app running, as it's needed for the next section.

##### Task 4: Verify the IoT Hub is Receiving Telemetry

1. [ ] To verify that your IoT Hub is receiving the telemetry, open the **Azure Portal** `https://portal.azure.com` and navigate to the Azure IoT Hub **AZ-220-HUB-_{YourID}_** **Overview** pane.

1. [ ] On the **Overview** page, scroll down to the bottom of the page where the metrics tiles are displayed.

1. [ ] Adjacent to **Show data for last**, change the time range to one hour. The **Device to cloud messages** plot should show some activity.

    If no activity is shown, wait a short while, as there's some latency.

    With your device pumping out telemetry, and your hub receiving it, the next step is to route the messages to their correct endpoints.

#### Exercise 3: Create a Message Route to Azure Blob Storage

@@@secondary
The architecture of our vibration monitoring system requires data be sent to two destinations: storage and analysis. Azure IoT provides a great method of directing data to the right service, through *message routing*.

In our scenario, we need create two routes:

* the first route will be to storage for achiving data
* the second route will to an Event Hub for anomoly detection

Since message routes are best built and tested one at a time, this exercise will focus on the storage route. We'll call this route the "logging" route, and it involves digging a few levels deep into the creation of Azure resources. All the features required to build this route are available in the Azure portal.

We will keep the storage route simple, and use Azure Blob storage (though Data Lake storage is also available). The key feature of message routing is the filtering of incoming data. The filter, written in SQL, streams output down the route only when certain conditions are met.
@@@

@@@secondary
- One of the easiest ways to filter data is on a message property, which is why we added these two lines to our code:
 
 ```csharp
...
telemetryMessage.Properties.Add("sensorID", "VSTel");
...
loggingMessage.Properties.Add("sensorID", "VSLog");
 ```

 - An SQL query embedded into our message route can test the `sensorID` value.

In this exercise, you will create and test the logging route.
@@@

##### Task 1: Route the logging message to Azure storage

1. [ ] In the **Azure Portal** `https://portal.azure.com` ensure the **Overview** page for the IoT Hub you created (**AZ-220-HUB-_{YourID}_**) is open.

1. [ ] In the left-hand menu, under **Messaging**, select **Message routing**.

1. [ ] On the **Message routing** page, ensure that **Routes** is selected.

1. [ ] Click **+ Add** to add the first route.

    The **Add a route** blade is displayed.

1. [ ] One the **Add a route** blade, under **Name**, enter `vibrationLoggingRoute`.

1. [ ] To the right of **Endpoint**, click **+ Add endpoint**, and select **Storage** from the drop-down list.

    The **Add a storage endpoint** pane is displayed.

1. [ ] Under **Endpoint name**, enter `vibrationLogEndpoint`.

1. [ ] To create storage and select a container, click **Pick a container**.

    A list of the storage accounts already present in the Azure Subscription is listed. At this point you could select an existing storage account and container, however for this lab we will create new ones.

1. [ ] To create a new storage account, click **+ Storage account**.

    The **Create storage** pane is displayed.

1. [ ] On the **Create Storage** pane, under **Name**, enter **vibrationstore** and add your initials and today's date - **vibrationstorecah191211**. 
@@@warning
**Note**: This field can only contain lower-case letters and numbers, must be between 3 and 24 characters, and must be unique.
@@@

1. [ ] Under **Account kind**, select **StorageV2 (general purpose V2)**.

1. [ ] Under **Performance**, select **Standard** if it is not selected.

    This keeps costs down at the expense of overall performance.

1. [ ] Under **Replication**, select **Locally-redundant storage (LRS)** if it is not already selected.

    This keeps costs down at the expense of risk mitigation for disaster recovery. In production your solution may require a more robust replication strategy.

1. [ ] Under **Location**, choose the region you are using for all of your lab work.

1. [ ] To create the storage account, click **OK**, then wait until the request is validated, then completed. Validation and creation can take a minute or two.

    Once complete, the **Create storage account** pane will close. The **Storage accounts** screen will now appear. It should have updated and show the storage account that was just created.

1. [ ] Search for **vibrationstore**, and select the storage account you just created. 

   The **Containers** blade should appear. As this is a new storage account, there are no containers to list.

1. [ ] To create a container, click **+ Container**.

    The **New container** popup is displayed.

1. [ ] In the **New container** popup, under **Name**, enter **vibrationcontainer**

   Again, only lower-case letters and numbers are accepted.

1. [ ] Under **Public access level**, ensure **Private (no anonymous access)** is selected.

1. [ ] To create the container, click **OK**, then wait for your container to be available. 

1. [ ] To choose the container for the solution, highlight the container in the list, and click **Select** at the bottom of the page.

    You will return to the **Add a storage endpoint** pane. Note that the **Azure Storage container** has been set to the URL for the storage account and container you just created.

1. [ ] Leave the **Batch frequency** and **Chunk size window** to the default values of **100**.

1. [ ] Under **Encoding**, note there are two options and that **AVRO** is selected.
@@@warning
**Note**: By default IoT Hub writes the content in Avro format, which has both a message body property and a message property. The Avro format is not used for any other endpoints. Although the Avro format is great for data and message preservation, it's a challenge to use it to query data. In comparison, JSON or CSV format is much easier for querying data. IoT Hub now supports writing data to Blob storage in JSON as well as AVRO.
@@@

1. [ ] The final field **File name format** specifies the pattern used to write the data to files in storage. The various tokens are replace with values as the file is created.

1. [ ] To create the endpoint, click **Create** at the bottom of the pane. Validation and creation will take a few moments.

    You should now be back at the **Add a route** blade. 

1. [ ] Under **Data source**, ensure **Device Telemetry Messages** is selected.

1. [ ] Under **Enable route**, ensure **Enable** is selected.

1. [ ] Under **Routing query**, replace **true** with the query below:

    ```sql
    sensorID = "VSLog"
    ```

    This ensures that messages only follow this route if the `sensorID = "VSLog"`.

1. [ ] To save this route, click **Save**. Wait for the success message.

    Once completed, the route should be listed on the **Message routing** blade.

The next step will be to verify that the logging route is working.

#### Exercise 4: Logging Route Azure Stream Analytics Job

@@@secondary
To verify that the logging route is working as expected, we will create a Stream Analytics job that routes logging messages to Blob storage. 

This will enable us to verify that our route includes the following settings:

* **Name** - vibrationLoggingRoute
* **Data Source** - DeviceMessages
* **Routing query** - sensorID = "VSLog"
* **Endpoint** - vibrationLogEndpoint
* **Enabled** - true
@@@

##### Task 1: Create the Stream Analytics Job

1. [ ] In the **Azure portal**, select **+ Create a resource**. 

2. [ ] Search for and select `Stream Analytics job`. Click **Create**.

    The **New Stream Analytics job** pane is displayed.

3. [ ] On the **New Stream Analytics job** pane, under **Name**, enter `vibrationJob`.

4. [ ] Under **Subscription**, choose the subscription you are using for the lab.

5. [ ] Under **Resource group**, select **AZ-220-RG**.

6. [ ] Under **Location**, select the region you are using for all of your lab work.

7. [ ] Under **Hosting environment**, select **Cloud**.

    Edge hosting will be discussed later in the course.

8. [ ] Under **Streaming units**, reduce the number from **3** to **1**.

    This lab does not require 3 units and this will reduce costs.

9. [ ]  To create the streaming analytics job, click **Create**.

10. [ ] Wait for the **Deployment succeeded** message, then open the new resource.
@@@info
**Tip:** If you miss the message to go to the new resource, or need to find a resource at any time, select **Home/All resources**. Enter enough of the resource name for it to appear in the list of resources.
@@@

    You'll now see the empty job, showing no inputs or outputs, and a skeleton query. The next step is to populate these entries.

11. [ ] To create an input, in the left hand navigation, under **Job topology**, click **Inputs**.

    The **Inputs** pane is displayed.

12. [ ] On the **Inputs** pane, click **+ Add stream input**, and select **IoT Hub** from the dropdown list.

    The **New Input** pane will be displayed.

13. [ ] On the **New Input** pane, under **Input alias**, enter `vibrationInput`.

14. [ ] Ensure **Select IoT Hub from your subscriptions** is selected.

15. [ ] Under **Subscription**, ensure the subscription you used to create the IoT Hub earlier is selected.

16. [ ] Under **IoT Hub**, select the IoT Hub you created at the beginning of the course labs, **AZ-220-HUB-_{YourID}_**.

17. [ ] Under **Endpoint**, ensure **Messaging** is selected.

18. [ ] Under **Shared access policy name**, ensure **iothubowner** is selected.
@@@warning
**Note**: The **Shared access policy key** is populated and read-only.
@@@

19. [ ] Under **Consumer group**, ensure **$Default** is selected.

20. [ ] Under **Event serialization format**, ensure **JSON** is selected.

21. [ ] Under **Encoding**, ensure **UTF-8** is selected.

22. [ ] Under **Event compression type**, ensure **None** is selected.

23. [ ] To save the new input, click **Save**, then wait for the input to be created.

    The **Inputs** list should be updated to show the new input.

24. [ ] To create an output, in the left hand navigation, under **Job topology**, click **Outputs**.

    The **Outputs** pane is displayed.

25. [ ] On the **Outputs** pane, click **+ Add**, and select **Blob storage/Data Lake Storage Gen2** from the dropdown list.

    The **New output** pane is displayed.

26. [ ] On the **New output** pane, under **Output alias**, enter `vibrationOutput`.

27. [ ] Ensure **Select storage from your subscriptions** is selected.

28. [ ] Under **Subscription**, choose the subscription you are using for this lab.

29. [ ] Under **Storage account**, choose the storage account you created earlier - **vibrationstore** plus your initials and date.
@@@warning
**Note**: The **Storage account key** is automatically populated and read-only.
@@@

30. [ ] Under **Container**, ensure **Use existing** is selected and select **vibrationcontainer** from the dropdown list.

31. [ ] Leave the **Path pattern** blank.

32. [ ] Leave the **Date format** and **Time format** at their defaults.

32. [ ] Under **Event serialization format**, ensure **JSON** is selected.

33. [ ] Under **Encoding**, ensure **UTF-8** is selected.

34. [ ] Under **Format**, ensure **Line separated**.
@@@warning
**Note**: This setting stores each record as a JSON object on each line and, taken as a whole, results in a file that is an invalid JSON record. The other option, **Array**, ensures that the entire document is formatted as a JSON array where each record is an item in the array. This allows the entire file to be parsed as valid JSON.
@@@

35. [ ] Leave **Minimum rows** blank.

36. [ ] Leave **Minimum time Hours** and **Minutes** blank.

37. [ ] Under **Authentication mode**, ensure **Connection string** is selected.

38. [ ] To create the output, click **Save**, then wait for the output to be created.

    The **Outputs** list will be updated with the new output.

39. [ ] To edit the query, in the left hand navigation, under **Job topology**, click **Query**.

40. [ ] In the query edit pane, replace the existing query with the query below:

    ```sql
    SELECT
        *
    INTO
        vibrationOutput
    FROM
        vibrationInput
    ```

41. [ ] Above the edit pane, in the toolbar, click **Save Query**.

42. [ ] In the left hand navigation, click **Overview**.

##### Task 2: Test the Logging Route

@@@secondary
Now for the fun part. Does the telemetry your device app is pumping out work its way along the route, and into the storage container?
@@@

1. [ ] Ensure the device app you create in Visual Studio Code is still running. If not, run it in the Visual Studio Code terminal using `dotnet run`.

1. [ ] In the **vibrationJob** blade's **Overview** page, click **Start**.
   
2. [ ] In the **Start job** pane, leave the **Job output start time** at **Now**, and click **Start**.

    It will take a few moments for the job to start.

3. [ ] Return to the **Azure Portal** `https://portal.azure.com/#home`

4. [ ] Select the **vibrationstore** (plus your initials and date) resource from your resource group tile.  (If it's not visible, use the **Refresh** button at the top of the resource group tile.)

6. [ ] On the **Overview** page, scroll down until you can see the **Monitoring** section.

7. [ ] Under **Monitoring**, adjacent to **Show data for last**, change the time range to **1 hour**. You should see activity in the charts.

8. [ ] For added reassurance that all the data is getting to the account, open the storage in **Storage Explorer (preview)**. You can find links to **Storage Explorer (preview)** in multiple locations; the easiest to find is probably in the left hand navigation area.
@@@warning
**Note**: The Storage Explorer is currently in preview mode, so its exact mode of operation may change.
@@@

9. [ ]  In **Storage Explorer (preview)**, under **BLOB CONTAINERS**, select **vibrationcontainer**.

10. [ ] To view the data, you will need to navigate down a hierarchy of folders. The first folder will be named for the IoT Hub, the next will be a partition, then year, month, day and finally hour. Within the hour folder, you will see files named for the minute they were generated.

11. [ ] To stop the Azure Streaming Analytics job, return to your portal dashboard and select **vibrationJob**.

12. [ ] On the **Stream Analytics Job** page, click **Stop** and click **Yes** in the confirmation popup.

You've traced the activity from the device app, to the hub, down the route, and to the storage container. Great progress!

@@@info
**Next Steps**

The final part of this scenario requires that the telemetry data is sent to an EventHub for real-time analysis in PowerBI. We will cover this second part in the next lab, after you have been introduced to data visualization.

You may wish to exit the device simulator app by pressing **CTRL-C** in the Visual Studio Code Terminal.
@@@
@@@danger
**Important**: Do not remove these resources until you have completed the Data Visualization module of this course.
@@@

@@@success
**Congratulations**! you have now completed this lab!
@@@
