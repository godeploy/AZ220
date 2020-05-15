### Lab - Build with IoT Central

@@@secondary
**Scenario**

Azure IoT Central enables the easy monitoring and management of a fleet of remote devices.

Azure IoT Central encompasses a range of underlying technologies that work great, but can be complicated to implement when many technologies are needed. These technologies include Azure IoT Hub, the Azure Device Provisioning System (DPS), Azure Maps, Azure Time Series Insights, Azure IoT Edge, and others. It's only necessary to use these technologies directly, if more granularity is needed than available through IoT Central.

One of the purposes of this lab is to help you decide if there's enough features in IoT Central to support the scenarios you are likely to need. So, let's investigate what IoT Central can do with a fun and involved scenario.

Contoso operates a fleet of refrigerated trucks. You've a number of customers within a city, and a base that you operate from. You command each truck to take its contents and deliver it to any one customer. However, the cooling system may fail on any one of your trucks, and if the contents does start to melt, you'll need the option of instructing the truck to return to base, and then dump the contents. Alternatively, you can deliver the contents to another customer who might be nearer to the truck when you become aware the contents are melting.

In order to make these decisions, you'll need an up-to-date picture of all that is going on with your trucks. You'll need to know the location of each truck on a map, the state of the cooling system, and the state of the contents.

IoT Central provides all you need to handle this scenario. 

In this lab you will:

* Create an Azure IoT Central custom app, using the IoT Central * portal
* Create a device template for a custom device, using the IoT * Central portal
* Create a programming project to simulate a refrigerated truck, with routes selected by Azure Maps, using Visual Studio Code, or Visual Studio
* Monitor and command the simulated device, from an IoT Central dashboard
@@@

#### Exercise 1: Create a Custom IoT Central app

1. [ ] Navigate to **Azure IoT Central** `https://apps.azureiotcentral.com/?azure-portal=true`. It's a good idea to bookmark this URL, as it's the home for all your IoT Central apps.

1. [ ] Click on **Build**, then **Custom apps**.

1. Your **Application name** can be any friendly name, such as "Refrigerated Trucks". However, the **URL** _must_ be unique, which is why you'll add a unique ID to the end of the URL for the app. For example, `refrigerated-trucks-<your id>`, replacing `<your id>` with some unique ID.

1. [ ] Leave the **Application template** as **Preview application**.

1. [ ] Select the free **7 day free trial** option. Seven days is plenty of time to complete the scenario.

1. [ ] Fill out your contact info, and click **Create**. Wait a few seconds whilst the app resource is built.

1. [ ] You should now see a **Dashboard** with a few default links.

 The next time you visit your Azure central home page, select **My apps** in the left-hand menu, and an icon for your  **Refrigerated Trucks** app should appear.

 You've now created the app. The next step is to specify a _device template_.

#### Exercise 2: Create Device Template

@@@secondary
The data communicated between a remote device, and IoT Central, is specified in a _device template_. The device template encapsulates all the details of the data, so that both the device and IoT Central have all they need to make sense of the communication.

In this Lab, you'll create a device template for a refrigerated truck.
@@@

##### Task 1 - Create a device template

1. [ ] Within the **Azure IoT Central** `https://apps.azureiotcentral.com/?azure-portal=true` portal (which you may still have open), select **Device Templates** from the menu on the left-hand side.

1. [ ] Click **+ New** to create a new template.

1. [ ] You'll next see a range  of template options, select **IoT device**. We are going to build the template from scratch.
@@@info
**Tip!** Take note of the other options. You may want to use those prebuilt template options in a future project!
@@@

1. [ ] Click **Next: Customize**, then **Next: Review**. Do not select the **Gateway device** box. Then click **Create**.

1. [ ] Enter the name for your device template: "RefrigeratedTruck", and click Enter.

1. [ ] For **Create a capability model**, click **Custom**. You should now see a screen similar to the following image.
@@@warning
**Note**: Take note of a few important elements that you have created. Including that the template is in Draft form, and the locations of the **+ Add interface**, **Views**, and **Publish** controls.
@@@

1. [ ]  You are now ready to add the specifics of the device template. Click **Add interface**, then **Custom**, to start building from a blank interface.

An interface defines a set of _capabilities_. We have quite a few to create, to define a refrigerated truck.

##### Task 2 - Add sensor telemetry

@@@secondary
Telemetry is the data values transmitted by sensors. The most important sensor in our refrigerated truck, monitors the temperature of the contents.
@@@

1. [ ] Click **+ Add capability**, and enter the following values:

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Contents temperature |
    | Name | ContentsTemperature |
    | Capability Type | Telemetry |
    | Semantic type | Temperature |
    | Schema | Double |
    | Unit | <sup>o</sup>C |

1. [ ] Your screen should now look like the following image.
@@@danger
**Important!** The names entered for the interface must be entered _exactly_ as shown in this unit. This is because an exact match is needed between these names, and entries in the code you'll be adding later in this lab.
@@@

Let's add the rest of the template.

##### Task 3 - Add state telemetry

@@@secondary
States are important, they let the operator know what is going on. A state in IoT Central is a name associated with a range of values. In addition, you later get to choose a color to associate with each value.
@@@

1. [ ] Use the **+ Add capability** control to add a state for the truck's refrigerated contents: one of _empty_, _full_, or _melting_.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Contents state |
    | Name | ContentsState |
    | Capability Type | Telemetry |
    | Semantic type | State |
    | Value schema | String |

1. [ ] Now, click **+**, and enter "empty" for the **Display name**, and **Value**. The **Name** field should automatically be populated with "empty". So, all three fields are identical, containing "empty".

1. [ ] Add two more state values: "full" and "melting". Again, the same text should appear in the **Display name**, **Name**, and **Value**.

1. [ ] Carefully check each capability before moving on. 

1. [ ] Now, to add some uncertainty to our simulation, let's add a failure state for the cooling system. If the cooling system fails, as you'll see in the following units, the chances of the contents melting increase considerably! Add _on_, _off_ and _failed_ entries for a cooling system. Start by clicking **+ Add capability**, and add another state.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Cooling system state |
    | Name | CoolingSystemState |
    | Capability Type | Telemetry |
    | Semantic type | State |
    | Value schema | String |

1. [ ] Now add three values: on, off, and failed. Make sure that each word appears in the **Display name**, **Name**, and **Value** fields.

1. [ ] A more complex state is the state of the truck itself. If all goes well, a truck's normal routing might be: _ready_, _enroute_, _delivering_, _returning_, _loading_, and back to _ready_ again.  However, you should add the _dumping_ state to cater for when melted contents need to be disposed of! Using the same process as for the last two steps, create this new state.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Truck state |
    | Name | TruckState |
    | Capability Type | Telemetry |
    | Semantic type | State |
    | Value schema | String |

##### Task 4 - Add event telemetry

@@@secondary
Events are issues triggered by the device, and communicated to the IoT Central app. Events can be one of three types: _Error_, _Warning_, or _Informational_.

One possible event a device might trigger is a conflicting command. An example might be a truck is returning empty from a customer, but receives a command to deliver its contents to another customer. If a conflict occurs, it's a good idea for the device to trigger an event to warn the operator of the IoT Central app.

Another event might be just to acknowledge, and record, the customer ID that a truck is to deliver to.
@@@

1. [ ] Use **+ Add capability**, then create an event as follows.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Event |
    | Name | Event |
    | Capability Type | Telemetry |
    | Semantic type | Event |
    | Schema | String |

##### Task 6 - Add location telemetry

@@@secondary
A location is probably the most important, and yet one of the easiest measurements to add to a device template. Under the hood, it consists of a latitude, longitude, and an optional altitude, for the device.
@@@

1. [ ] Use **+ Add capability**, and add a location for our truck as follows.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Location |
    | Name | Location |
    | Capability Type | Telemetry |
    | Semantic type | Location |
    | Schema | Geopoint |

##### Task 7 - Add properties

@@@secondary
A property of a device is typically a constant value, that is communicated to the IoT Central app when communication is first initiated. In our refrigerated truck scenario, a good example of a property is the license plate of the truck, or some similar unique truck ID.

Properties can also be device configuration data. We will define an _optimal temperature_ for the truck contents as a property. This optimal temperature might change with different types of content, different weather conditions, or whatever might be appropriate. A setting has an initial default value, which may not need to be changed, but the ability to change it easily and quickly is there, if needed. This kind of property is called a _writable property_.

A property is a single value. If more complex sets of data need to be transmitted to a device, a Command (see below) is the more appropriate way of handling it.
@@@

1. [ ] Use **+ Add capability**, and add the truck ID property.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Truck ID |
    | Name | TruckID |
    | Capability Type | Property |
    | Semantic type | None |
    | Schema | String |
    | Writable | Off |
    | Unit | None |

1. [ ] Next, add the optimal temperature property.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Optimal Temperature |
    | Name | OptimalTemperature |
    | Capability Type | Property |
    | Semantic type | None |
    | Schema | Double |
    | Writable | On |
    | Unit |  <sup>o</sup>C  |

##### Task 8 - Add commands

@@@secondary
Commands are sent by the operator of the IoT Central app to the remote devices. Commands are similar to writable properties, but a command can contain any number of input fields, whereas a writable property is limited to a single value.

For refrigerated trucks, there are two commands you should add: a command to deliver the contents to a customer, and a command to recall the truck to base.
@@@

1. [ ] Use **+ Add capability**, and add the first command.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Go to customer |
    | Name | GoToCustomer |
    | Capability Type | Command |
    | Command | Synchronous |

1. [ ] When you turn on the **Request** option, you'll be able to enter more details of the command.

    | Entry summary | Value |
    | --- | --- |
    | Request | On |
    | Display name | Customer ID |
    | Name | CustomerID |
    | Schema | Integer |
    | Unit | None |

1. [ ] Enter another new command, for recalling the truck.

    | Entry summary | Value |
    | --- | --- |
    | Display Name | Recall |
    | Name | Recall |
    | Capability Type | Command |
    | Command | Synchronous |

1. [ ] This time there are no additional parameters for the command, so leave **Request** off.

1. [ ] Click **Save**.

 Before going any further carefully double check your interface. After an interface has been published, there are very limited editing options. It's important to get it right before publishing. If you click on the name of the device template, in the menu that ends with the **Views** option, you'll get a summary of the capabilities.

##### Task 9 - Publish the template

1. [ ] Click **Save** again, if you've made any changes since the last time you saved.

1. [ ] Click **Publish**. You should see that the annotation changes from **Draft** to **Published**.

 Preparing a device template does take some care and some time.

 In the next task, you use the capabilities of the device template to prepare a controllers dashboard. Preparing views can be done before, or after, a device template is published.

#### Exercise 3: Monitor Simulated Device

@@@secondary
You'll first create a dashboard showing all the capabilities of the device template. Next, you'll create a real device, and record the connection settings needed for the remote device app.
@@@

##### Task 1 - Create a rich dashboard

1. [ ] Click on the **Views** menu option, then on **Visualizing the device**.

1. [ ] You should now see a list of all the **Telemetry**, **Properties**, and **Commands** you created, each with a check box.

1. [ ] Click the **Location** check box, then **Add tile**. Dashboards are made up of tiles. The reason we choose the location tile first, is that we want to expand it from its default size. Drag the lower right-hard corner of the tile, so that the tile is at least twice the default size. This tile is the most fun, it will show the location of the truck on a map of the world.

1. [ ] Before adding more tiles, change the **View name** to something more specific, "Truck view", or something similar.

1. [ ]  Now, click each of the rest of the telemetry and properties capabilities in turn, starting at the top, and **Add tile**. We are going for function over form here, we can prettify the dashboard later. For now, we just want a dashboard that will confirm all the telemetry being sent from our remote device. There's no need to add the commands to the dashboard, though that option does exist.

1. [ ] When you've added all the tiles, scroll around a bit on your dashboard, and check out the wording in the tiles.

1. [ ] You can drag tiles around, and the portal will try to rearrange them neatly.

1. [ ] When you are satisfied with your dashboard, click **Save**, then click **Publish**. You'll now notice that in the dialog that appears, that the **Views** entry is **Yes**. Click **Publish** in the dialog.

You can create as many views as you want to, giving each a friendly name. For this lab though, one dashboard will work well.

The next step is to create a device.

##### Task 2 - Create a real device

@@@secondary
By "real" IoT Central understands that there's a remote app running. The app can be in a real device, taking input from real sensors, or running a simulation. Both options are treated as a connection to a _real_ device.
@@@

1. [ ] Click **Devices** in the left-hand menu.

1. [ ] Click **RefrigeratedTruck** in the **Devices** menu, to ensure the device we create uses this device template. The device template you select will be shown in bold text.

1. [ ] Click **+ New**. Verify in the dialog that the device name includes the **RefrigeratedTruck** text. If it doesn't, you've not selected the right device template.

1. [ ] Change the **Device ID** to a friendlier name, say "RefrigeratedTruck1".

1. [ ] Change the **Device name** to a friendlier name, say "RefrigeratedTruck - 1".

1. Leave the **Simulated** setting at **Off**. We are going to be building a real truck here. Well, a simulated _real_ truck! Setting this value to **On** instructs IoT Central to pump out random values for our telemetry. These random values can be useful in validating a device template.

1. Click **Create**. Wait a few seconds, then your device list should be populated with a single entry. Note the **Device status** is **Registered**. Not until the device status is **Provisioned** will the IoT Central app accept a connection to the device. The coding task that follows shows how to provision a device.

1. [ ] Click on the **RefrigeratedTruck - 1** name, and you'll see the live dashboard, with lots of **Waiting for data** messages.

1. [ ] Click on the **Commands** entry in the bar that includes **Truck view**. Notice that the two commands you entered are ready to be run.

The next step is to create the keys that will allow a remote device to communicate with this app.

##### Task 3 - Record the connection keys

1. [ ] Click **Connect** in the top-right menu. Do _not_ click **Connect to gateway**.

1. [ ] In the **Device connection** dialog that follows, carefully copy the **ID scope**, **Device ID**, and **Primary key** to a text file. Typically, use a tool like Notepad, and save the file with a meaningful name, say "Truck connections.txt".

1. [ ] Leave the **Connect method** as **Shared access signature (SAS)**.

1. [ ] When you've saved off the IDs and key, click **Close** on the dialog.

Leave the IoT portal open in your browser, waiting as it is.

#### Exercise 4: Create a free Azure Maps account

@@@secondary
If you do not already have an Azure Maps account, you'll need to create one.
@@@

1. [ ] Navigate to **Azure Maps** `https://azure.microsoft.com/services/azure-maps/?azure-portal=true`

1. [ ] Follow the prompts to create a free account. When your account is set up, you'll need the **Subscription Key** for the account. Copy and paste this key into your text document, with a note that it applies to Azure Maps.

1. [ ] You can (optionally) verify your Azure Maps subscription key works. Save the following HTML to an .html file. Replace the **subscriptionKey** entry with your own key. Then, load the file into a web browser. Do you see a map of the world?

 ```html
<!DOCTYPE html>
<html>
<head>
    <title>Map</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>
    <!-- Add a reference to the Azure Maps Services lab JavaScript file. -->
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas-service.min.js"></script>
    <script>
        function GetMap() {
            //Instantiate a map object
            var map = new atlas.Map("myMap", {
                //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<your Azure Maps subscription key>'
                }
            });
        }
    </script>
    <style>
        html,
        body {
            width: 100%;
            height: 100%;
            padding: 0;
            margin: 0;
        }
        #myMap {
            width: 100%;
            height: 100%;
        }
    </style>
</head>
<body onload="GetMap()">
    <div id="myMap"></div>
</body>
</html>
 ```

#### Exercise 5: Create a Programming Project for a Real Device

@@@secondary
In this task, you are going to create a programming project to simulate a sensor device in a refrigerated truck. This simulation enables you to test the code long before requiring a real truck!

IoT Central treats this simulation as "real" because the communication code between the device app and the IoT Central app is the same for a real truck. In other words, if you do run a refrigerated truck company, you would start with simulated code similar to the code in this task. After this code works to your satisfaction, the simulation-specific code would be replaced with code that receives sensor data. This limited update makes writing the following code a valuable experience.
@@@

##### Task 1 - Create the device app

@@@secondary
Using Visual Studio Code, build the device sensor app.
@@@

1. [ ] Open a terminal in Visual Studio Code, and create a folder called "RefrigeratedTruck" (enter `mkdir RefrigeratedTruck`). Navigate to the RefrigeratedTruck folder.

1. [ ] Enter the following command in the terminal: `dotnet new console`. This command creates a Program.cs file in your folder, along with a project file.

1. [ ] Enter `dotnet restore` in the terminal. This command gives your app access to the required .NET packages.

1. [ ] In the terminal, install the required libraries:

    ```CLI
    dotnet add package AzureMapsRestToolkit
    dotnet add package Microsoft.Azure.Devices.Client
    dotnet add package Microsoft.Azure.Devices.Provisioning.Client
    dotnet add package Microsoft.Azure.Devices.Provisioning.Transport.Mqtt
    dotnet add package System.Text.Json
    ```

1. [ ] From the **File** menu, open up the Program.cs file, and delete the default contents.

1. [ ] After you've entered the code below into the Program.cs file, you can run the app with the command `dotnet run`. This command will run the Program.cs file in the current folder, so ensure you are in the `RefrigeratedTruck` folder.

1. [ ] Open Visual Studio, and create a new **Visual C#/Windows Desktop** project. Select **Console App (.NET Framework)**.

1. [ ] Give the project a friendly name, such as "RefrigeratedTruck".

1. [ ] Under **Tools/NuGet Package Manager**, select **Manage NuGet Packages for Solution**. Install the following libraries:

    * **AzureMapsRestToolkit**
    
    * **Microsoft.Azure.Devices.Client**
    * **Microsoft.Azure.Devices.Provisioning.Client**
    * **Microsoft.Azure.Devices.Provisioning.Transport.Mqtt**
    * **System.Text.Json**

1. [ ] Delete the default contents of the Program.cs file.

1. [ ] Add all the code that follows to the Program.cs file.

##### Task 2 - Write the device app

In the blank Program.cs file, insert the following code. Each additional section of code should be appended to the end of the file, in the order listed here.
@@@warning
**Note**: If you would like to skip this task, and load all of the code into your app, then download and copy all of the contents of Program.cs from [MicrosoftDocs/mslearn-your-first-iot-central-app](https://github.com/MicrosoftDocs/mslearn-your-first-iot-central-app) into the Program.cs file of your project. If you copy this code (and replace the connection and subscription strings) then go straight to the next task, and start testing!
@@@

1. [ ] Add the `using` statements, including for Azure IoT Central and Azure Maps.

   ```cs
    using System;
    using System.Text.Json;
    using System.Text;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Microsoft.Azure.Devices.Provisioning.Client;
    using Microsoft.Azure.Devices.Provisioning.Client.Transport;
    using AzureMapsToolkit;
    using AzureMapsToolkit.Common;
    ```

1. [ ] Add the namespace, class, and global variables.

 ```cs
    namespace refrigerated_truck
    {
        class Program
        {
            enum StateEnum
            {
                ready,
                enroute,
                delivering,
                returning,
                loading,
                dumping
            };
            enum ContentsEnum
            {
                full,
                melting,
                empty
            }
            enum FanEnum
            {
                on,
                off,
                failed
            }

            // Azure maps service globals.
            static AzureMapsServices azureMapsServices;

            // Telemetry globals.
            const int intervalInMilliseconds = 5000;        // Time interval required by wait function.

            // Refrigerated truck globals.
            static int truckNum = 1;
            static string truckIdentification = "Truck number " + truckNum;

            const double deliverTime = 600;                 // Time to complete delivery, in seconds.
            const double loadingTime = 800;                 // Time to load contents.
            const double dumpingTime = 400;                 // Time to dump melted contents.
            const double tooWarmThreshold = 2;              // Degrees C that is too warm for contents.
            const double tooWarmtooLong = 60;               // Time in seconds for contents to start melting if temps are above threshold.


            static double timeOnCurrentTask = 0;            // Time on current task in seconds.
            static double interval = 60;                    // Simulated time interval in seconds.
            static double tooWarmPeriod = 0;                // Time that contents are too warm in seconds.
            static double tempContents = -2;                // Current temp of contents in degrees C.
            static double baseLat = 47.644702;              // Base position latitude.
            static double baseLon = -122.130137;            // Base position longitude.
            static double currentLat;                       // Current position latitude.
            static double currentLon;                       // Current position longitude.
            static double destinationLat;                   // Destination position latitude.
            static double destinationLon;                   // Destination position longitude.

            static FanEnum fan = FanEnum.on;                // Cooling fan state.
            static ContentsEnum contents = ContentsEnum.full;    // Truck contents state.
            static StateEnum state = StateEnum.ready;       // Truck is full and ready to go!
            static double optimalTemperature = -5;         // Setting - can be changed by the operator from IoT Central.

            const string noEvent = "none";
            static string eventText = noEvent;              // Event text sent to IoT Central.

            static double[,] customer = new double[,]
            {
                // Lat/lon position of customers.
                // Gasworks Park
                {47.645892, -122.336954},

                // Golden Gardens Park
                {47.688741, -122.402965},

                // Seward Park
                {47.551093, -122.249266},

                // Lake Sammamish Park
                {47.555698, -122.065996},

                // Marymoor Park
                {47.663747, -122.120879},

                // Meadowdale Beach Park
                {47.857295, -122.316355},

                // Lincoln Park
                {47.530250, -122.393055},

                // Gene Coulon Park
                {47.503266, -122.200194},

                // Luther Bank Park
                {47.591094, -122.226833},

                // Pioneer Park
                {47.544120, -122.221673 }
            };

            static double[,] path;                          // Lat/lon steps for the route.
            static double[] timeOnPath;                     // Time in seconds for each section of the route.
            static int truckOnSection;                      // The current path section the truck is on.
            static double truckSectionsCompletedTime;       // The time the truck has spent on previous completed sections.
            static Random rand;

            // IoT Central global variables.
            static DeviceClient s_deviceClient;
            static CancellationTokenSource cts;
            static string GlobalDeviceEndpoint = "global.azure-devices-provisioning.net";
            static TwinCollection reportedProperties = new TwinCollection();

            // User IDs.
            static string ScopeID = "<your Scope ID>";
            static string DeviceID = "<your Device ID>";
            static string PrimaryKey = "<your device Primary Key>";
            static string AzureMapsKey = "<your Azure Maps Subscription Key>";
    ```

1. [ ] Add the methods to get a route via Azure Maps.


 ```cs
static double Degrees2Radians(double deg)
            {
                return deg * Math.PI / 180;
            }
            // Returns the distance in meters between two locations on Earth.
            static double DistanceInMeters(double lat1, double lon1, double lat2, double lon2)
            {
                var dlon = Degrees2Radians(lon2 - lon1);
                var dlat = Degrees2Radians(lat2 - lat1);
                var a = (Math.Sin(dlat / 2) * Math.Sin(dlat / 2)) + Math.Cos(Degrees2Radians(lat1)) * Math.Cos(Degrees2Radians(lat2)) * (Math.Sin(dlon / 2) * Math.Sin(dlon / 2));
                var angle = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
                var meters = angle * 6371000;
                return meters;
            }
            static bool Arrived()
            {
                // If the truck is within 10 meters of the destination, call it good.
                if (DistanceInMeters(currentLat, currentLon, destinationLat, destinationLon) < 10)
                    return true;
                return false;
            }
            static void UpdatePosition()
            {
                while ((truckSectionsCompletedTime + timeOnPath[truckOnSection] < timeOnCurrentTask) && (truckOnSection < timeOnPath.Length - 1))
                {
                    // Truck has moved onto the next section.
                    truckSectionsCompletedTime += timeOnPath[truckOnSection];
                    ++truckOnSection;
                }
                // Ensure remainder is 0 to 1, as interval may take count over what is needed.
                var remainderFraction = Math.Min(1, (timeOnCurrentTask - truckSectionsCompletedTime) / timeOnPath[truckOnSection]);
                // The path should be one entry longer than the timeOnPath array.
                // Find how far along the section the truck has moved.
                currentLat = path[truckOnSection, 0] + remainderFraction * (path[truckOnSection + 1, 0] - path[truckOnSection, 0]);
                currentLon = path[truckOnSection, 1] + remainderFraction * (path[truckOnSection + 1, 1] - path[truckOnSection, 1]);
            }
            static void GetRoute(StateEnum newState)
            {
                // Set the state to ready, until the new route arrives.
                state = StateEnum.ready;
                var req = new RouteRequestDirections
                {
                    Query = $"{currentLat},{currentLon}:{destinationLat},{destinationLon}"
                };
                var directions = azureMapsServices.GetRouteDirections(req).Result;
                if (directions.Error != null || directions.Result == null)
                {
                    // Handle any error.
                    redMessage("Failed to find map route");
                }
                else
                {
                    int nPoints = directions.Result.Routes[0].Legs[0].Points.Length;
                    greenMessage($"Route found. Number of points = {nPoints}");
                    // Clear the path. Add two points for the start point and destination.
                    path = new double[nPoints + 2, 2];
                    int c = 0;
                    // Start with the current location.
                    path[c, 0] = currentLat;
                    path[c, 1] = currentLon;
                    ++c;
                    // Retrieve the route and push the points onto the array.
                    for (var n = 0; n < nPoints; n++)
                    {
                        var x = directions.Result.Routes[0].Legs[0].Points[n].Latitude;
                        var y = directions.Result.Routes[0].Legs[0].Points[n].Longitude;
                        path[c, 0] = x;
                        path[c, 1] = y;
                        ++c;
                    }
                    // Finish with the destination.
                    path[c, 0] = destinationLat;
                    path[c, 1] = destinationLon;
                    // Store the path length and time taken, to calculate the average speed.
                    var meters = directions.Result.Routes[0].Summary.LengthInMeters;
                    var seconds = directions.Result.Routes[0].Summary.TravelTimeInSeconds;
                    var pathSpeed = meters / seconds;
                    double distanceApartInMeters;
                    double timeForOneSection;
                    // Clear the time on path array. The path array is 1 less than the points array.
                    timeOnPath = new double[nPoints + 1];
                    // Calculate how much time is required for each section of the path.
                    for (var t = 0; t < nPoints + 1; t++)
                    {
                        // Calculate distance between the two path points, in meters.
                        distanceApartInMeters = DistanceInMeters(path[t, 0], path[t, 1], path[t + 1, 0], path[t + 1, 1]);
                        // Calculate the time for each section of the path.
                        timeForOneSection = distanceApartInMeters / pathSpeed;
                        timeOnPath[t] = timeForOneSection;
                    }
                    truckOnSection = 0;
                    truckSectionsCompletedTime = 0;
                    timeOnCurrentTask = 0;
                    // Update the state now the route has arrived. One of: enroute or returning.
                    state = newState;
                }
            }
```

@@@warning
**Note**: The key call here is **var directions = azureMapsServices.GetRouteDirections(req).Result;**. The **directions** structure is complex. Consider setting a breakpoint in this method, and examining the contents of **directions**.
@@@

1. [ ] Add the direct method to deliver to a customer.

   ```cs
        static Task<MethodResponse> CmdGoToCustomer(MethodRequest methodRequest, object userContext)
        {
            try
            {
                // Pick up variables from the request payload, with the name specified in IoT Central.
                var payloadString = Encoding.UTF8.GetString(methodRequest.Data);
                int customerNumber = Int32.Parse(payloadString);

                // Check for a valid key and customer ID.
                if (customerNumber >= 0 && customerNumber < customer.Length)
                {
                    switch (state)
                    {
                        case StateEnum.dumping:
                        case StateEnum.loading:
                        case StateEnum.delivering:
                            eventText = "Unable to act - " + state;
                            break;

                        case StateEnum.ready:
                        case StateEnum.enroute:
                        case StateEnum.returning:
                            if (contents == ContentsEnum.empty)
                            {
                                eventText = "Unable to act - empty";
                            }
                            else
                            {
                                // Set event only when all is good.
                                eventText = "New customer: " + customerNumber.ToString();

                                destinationLat = customer[customerNumber, 0];
                                destinationLon = customer[customerNumber, 1];

                                // Find route from current position to destination, storing route.
                                GetRoute(StateEnum.enroute);
                            }
                            break;
                    }

                    // Acknowledge the direct method call with a 200 success message.
                    string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
                    return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
                }
                else
                {
                    eventText = $"Invalid customer: {customerNumber}";

                    // Acknowledge the direct method call with a 400 error message.
                    string result = "{\"result\":\"Invalid customer\"}";
                    return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
                }
            }
            catch
            {
                // Acknowledge the direct method call with a 400 error message.
                string result = "{\"result\":\"Invalid call\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
            }
        }
    ```
@@@warning
**Note**: The device responds with a conflict, if the device isn't in the correct state. The command itself is acknowledged at the end of the method. The recall command that follows in the next step handles things similarly.
@@@

1. [ ] Add the recall direct method.

   ```cs
        static void ReturnToBase()
        {
            destinationLat = baseLat;
            destinationLon = baseLon;

            // Find route from current position to base, storing route.
            GetRoute(StateEnum.returning);
        }
        static Task<MethodResponse> CmdRecall(MethodRequest methodRequest, object userContext)
        {
            switch (state)
            {
                case StateEnum.ready:
                case StateEnum.loading:
                case StateEnum.dumping:
                    eventText = "Already at base";
                    break;

                case StateEnum.returning:
                    eventText = "Already returning";
                    break;

                case StateEnum.delivering:
                    eventText = "Unable to recall - " + state;
                    break;

                case StateEnum.enroute:
                    ReturnToBase();
                    break;
            }

            // Acknowledge the command.
            if (eventText == noEvent)
            {
                // Acknowledge the direct method call with a 200 success message.
                string result = "{\"result\":\"Executed direct method: " + methodRequest.Name + "\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
            }
            else
            {
                // Acknowledge the direct method call with a 400 error message.
                string result = "{\"result\":\"Invalid call\"}";
                return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 400));
            }
        }
    ```

1. [ ] Add the method that updates the truck simulation at each time interval.

   ```cs
        static double DieRoll(double max)
        {
            return rand.NextDouble() * max;
        }

        static void UpdateTruck()
        {
            if (contents == ContentsEnum.empty)
            {
                // Turn the cooling system off, if possible, when the contents are empty.
                if (fan == FanEnum.on)
                {
                    fan = FanEnum.off;
                }
                tempContents += -2.9 + DieRoll(6);
            }
            else
            {
                // Contents are full or melting.
                if (fan != FanEnum.failed)
                {
                    if (tempContents < optimalTemperature - 5)
                    {
                        // Turn the cooling system off, as contents are getting too cold.
                        fan = FanEnum.off;
                    }
                    else
                    {
                        if (tempContents > optimalTemperature)
                        {
                            // Temp getting higher, turn cooling system back on.
                            fan = FanEnum.on;
                        }
                    }

                    // Randomly fail the cooling system.
                    if (DieRoll(100) < 1)
                    {
                        fan = FanEnum.failed;
                    }
                }

                // Set the contents temperature. Maintaining a cooler temperature if the cooling system is on.
                if (fan == FanEnum.on)
                {
                    tempContents += -3 + DieRoll(5);
                }
                else
                {
                    tempContents += -2.9 + DieRoll(6);
                }

                // If the temperature is above a threshold, count the seconds this is occurring, and melt the contents if it goes on too long.
                if (tempContents >= tooWarmThreshold)
                {
                    // Contents are warming.
                    tooWarmPeriod += interval;

                    if (tooWarmPeriod >= tooWarmtooLong)
                    {
                        // Contents are melting.
                        contents = ContentsEnum.melting;
                    }
                }
                else
                {
                    // Contents are cooling.
                    tooWarmPeriod = Math.Max(0, tooWarmPeriod - interval);
                }
            }

            timeOnCurrentTask += interval;

            switch (state)
            {
                case StateEnum.loading:
                    if (timeOnCurrentTask >= loadingTime)
                    {
                        // Finished loading.
                        state = StateEnum.ready;
                        contents = ContentsEnum.full;
                        timeOnCurrentTask = 0;

                        // Turn on the cooling fan.
                        // If the fan is in a failed state, assume it has been fixed, as it is at the base.
                        fan = FanEnum.on;
                        tempContents = -2;
                    }
                    break;

                case StateEnum.ready:
                    timeOnCurrentTask = 0;
                    break;

                case StateEnum.delivering:
                    if (timeOnCurrentTask >= deliverTime)
                    {
                        // Finished delivering.
                        contents = ContentsEnum.empty;
                        ReturnToBase();
                    }
                    break;

                case StateEnum.returning:

                    // Update the truck position.
                    UpdatePosition();

                    // Check to see if the truck has arrived back at base.
                    if (Arrived())
                    {
                        switch (contents)
                        {
                            case ContentsEnum.empty:
                                state = StateEnum.loading;
                                break;

                            case ContentsEnum.full:
                                state = StateEnum.ready;
                                break;

                            case ContentsEnum.melting:
                                state = StateEnum.dumping;
                                break;
                        }
                        timeOnCurrentTask = 0;
                    }
                    break;

                case StateEnum.enroute:

                    // Move the truck.
                    UpdatePosition();

                    // Check to see if the truck has arrived at the customer.
                    if (Arrived())
                    {
                        state = StateEnum.delivering;
                        timeOnCurrentTask = 0;
                    }
                    break;

                case StateEnum.dumping:
                    if (timeOnCurrentTask >= dumpingTime)
                    {
                        // Finished dumping.
                        state = StateEnum.loading;
                        contents = ContentsEnum.empty;
                        timeOnCurrentTask = 0;
                    }
                    break;
            }
        }
    ```
@@@warning
**Note**: This function is called every time interval. The actual time interval is set at 5 seconds, though the _simulated time_ (the number of simulated seconds you specify that has passed each time this function is called) is set by the global `static double interval = 60`. Setting this value at 60 means the simulation runs at a rate of 60 divided by 5, or 12 times real time. To lower the simulated time, reduce `interval` to, say, 30 (for a simulation that runs at six times real-time). Setting `interval` at 5 would run the simulation in real-time. Though realistic, this speed would be a bit slow, given the real driving times to the customer destinations.
@@@

1. [ ] Add the methods to send truck telemetry. Send events too, if any have occurred.

   ```cs
        static void colorMessage(string text, ConsoleColor clr)
        {
            Console.ForegroundColor = clr;
            Console.WriteLine(text);
            Console.ResetColor();
        }
        static void greenMessage(string text)
        {
            colorMessage(text, ConsoleColor.Green);
        }

        static void redMessage(string text)
        {
            colorMessage(text, ConsoleColor.Red);
        }

        static async void SendTruckTelemetryAsync(Random rand, CancellationToken token)
        {
            while (true)
            {
                UpdateTruck();

                // Create the telemetry JSON message.
                var telemetryDataPoint = new
                {
                    ContentsTemperature = Math.Round(tempContents, 2),
                    TruckState = state.ToString(),
                    CoolingSystemState = fan.ToString(),
                    ContentsState = contents.ToString(),
                    Location = new { lon = currentLon, lat = currentLat },
                    Event = eventText,
                };
                var telemetryMessageString = JsonSerializer.Serialize(telemetryDataPoint);
                var telemetryMessage = new Message(Encoding.ASCII.GetBytes(telemetryMessageString));

                // Clear the events, as the message has been sent.
                eventText = noEvent;

                Console.WriteLine($"\nTelemetry data: {telemetryMessageString}");

                // Bail if requested.
                token.ThrowIfCancellationRequested();

                // Send the telemetry message.
                await s_deviceClient.SendEventAsync(telemetryMessage);
                greenMessage($"Telemetry sent {DateTime.Now.ToShortTimeString()}");

                await Task.Delay(intervalInMilliseconds);
            }
        }
    ```
@@@warning
**Note**: The `SendTruckTelemetryAsync` is an important function, handling the sending of telemetry, states, and events to IoT Central. Note the use of JSON strings to send the data.
@@@

1. [ ] Add the code to handle settings and properties. You only have one setting and one property in our app, though if there are more, they are easily added.

   ```cs
        static async Task SendDevicePropertiesAsync()
        {
            reportedProperties["TruckID"] = truckIdentification;
            await s_deviceClient.UpdateReportedPropertiesAsync(reportedProperties);
            greenMessage($"Sent device properties: {JsonSerializer.Serialize(reportedProperties)}");
        }
        static async Task HandleSettingChanged(TwinCollection desiredProperties, object userContext)
        {
            string setting = "OptimalTemperature";
            if (desiredProperties.Contains(setting))
            {
                BuildAcknowledgement(desiredProperties, setting);
                optimalTemperature = (int) desiredProperties[setting]["value"];
                greenMessage($"Optimal temperature updated: {optimalTemperature}");
            }
            await s_deviceClient.UpdateReportedPropertiesAsync(reportedProperties);
        }

        static void BuildAcknowledgement(TwinCollection desiredProperties, string setting)
        {
            reportedProperties[setting] = new
            {
                value = desiredProperties[setting]["value"],
                status = "completed",
                desiredVersion = desiredProperties["$version"],
                message = "Processed"
            };
        }
    ```
@@@warning
**Note**: This section of code is generic to most C# apps that communicate with IoT Central. To add additional properties or settings, add to `reportedProperties`, or create a new setting string, and check on `desiredProperties`, respectively. No other code changes are usually needed.
@@@

1. [ ] Add the `Main` function.

   ```cs
            static void Main(string[] args)
            {

                rand = new Random();
                colorMessage($"Starting {truckIdentification}", ConsoleColor.Yellow);
                currentLat = baseLat;
                currentLon = baseLon;

                // Connect to Azure Maps.
                azureMapsServices = new AzureMapsServices(AzureMapsKey);

                try
                {
                    using (var security = new SecurityProviderSymmetricKey(DeviceID, PrimaryKey, null))
                    {
                        DeviceRegistrationResult result = RegisterDeviceAsync(security).GetAwaiter().GetResult();
                        if (result.Status != ProvisioningRegistrationStatusType.Assigned)
                        {
                            Console.WriteLine("Failed to register device");
                            return;
                        }
                        IAuthenticationMethod auth = new DeviceAuthenticationWithRegistrySymmetricKey(result.DeviceId, (security as SecurityProviderSymmetricKey).GetPrimaryKey());
                        s_deviceClient = DeviceClient.Create(result.AssignedHub, auth, TransportType.Mqtt);
                    }
                    greenMessage("Device successfully connected to Azure IoT Central");

                    SendDevicePropertiesAsync().GetAwaiter().GetResult();

                    Console.Write("Register settings changed handler...");
                    s_deviceClient.SetDesiredPropertyUpdateCallbackAsync(HandleSettingChanged, null).GetAwaiter().GetResult();
                    Console.WriteLine("Done");

                    cts = new CancellationTokenSource();

                    // Create a handler for the direct method calls.
                    s_deviceClient.SetMethodHandlerAsync("GoToCustomer", CmdGoToCustomer, null).Wait();
                    s_deviceClient.SetMethodHandlerAsync("Recall", CmdRecall, null).Wait();

                    SendTruckTelemetryAsync(rand, cts.Token);

                    Console.WriteLine("Press any key to exit...");
                    Console.ReadKey();
                    cts.Cancel();
                }
                catch (Exception ex)
                {
                    Console.WriteLine();
                    Console.WriteLine(ex.Message);
                }
            }


            public static async Task<DeviceRegistrationResult> RegisterDeviceAsync(SecurityProviderSymmetricKey security)
            {
                Console.WriteLine("Register device...");

                using (var transport = new ProvisioningTransportHandlerMqtt(TransportFallbackType.TcpOnly))
                {
                    ProvisioningDeviceClient provClient =
                              ProvisioningDeviceClient.Create(GlobalDeviceEndpoint, ScopeID, security, transport);

                    Console.WriteLine($"RegistrationID = {security.GetRegistrationID()}");

                    Console.Write("ProvisioningClient RegisterAsync...");
                    DeviceRegistrationResult result = await provClient.RegisterAsync();

                    Console.WriteLine($"{result.Status}");

                    return result;
                }
            }
        }
    }
    ```
@@@warning
**Note**: Direct methods are set in the client using statements such as `s_deviceClient.SetMethodHandlerAsync("cmdGoTo", CmdGoToCustomer, null).Wait();`.
@@@

Fantastic! You are now ready to test your code.

#### Exercise 6: Test Your IoT Central Device

Working through this task is an exciting time in IoT Central development! Finally, you get to check whether all the moving parts you've created work together.

##### Test the device app and IoT Central app together

To fully test the refrigerated truck device, it helps to break down the testing into a number of discreet checks:

1. [ ] The device app connects to Azure IoT Central.

1. [ ] The telemetry functions send data on the specified interval.

1. [ ] The data is picked up correctly by IoT Central.

1. [ ] The command to send the truck to a specified customer works as expected.

1. [ ] The command to recall the truck works as expected.

1. [ ] Check customer and conflict events are transmitted correctly.

1. [ ] Check the truck properties, and change the optimal temperature.

In addition to this list, there are edge-cases you could also investigate. One such case is what happens when the truck's contents start to melt? This state is left up to chance in our simulation, with the use of random numbers in our code in the previous task.

To begin the testing, with your [Azure IoT Central](https://apps.azureiotcentral.com/?azure-portal=true) app open in a browser, run the device app.

1. [ ] In Visual Studio, select **Debug/Start without Debugging**.

1. [ ] In the terminal, enter `dotnet run`.

A console screen should open, with the text: **Starting Truck number 1**.

##### Confirm the device app connects to Azure IoT Central

1. [ ] If one of the next lines on the console is **Device successfully connected to Azure IoT Central** you've made the connection. If you do not get this message, it usually means either the IoT Central app isn't running, or the connection key strings aren't correct.

1. [ ] The "connected" line should be followed by some text verifying the settings and properties were sent successfully.

If all goes well, go straight into the second test.

##### Confirm the telemetry functions send data on the specified interval

1. [ ] A console message should appear every five seconds, with the contents temperature.

1. [ ] Watch the telemetry for a short while, and mentally prepare for the main test of this lab!

##### Confirm the data is picked up correctly by IoT Central

1. [ ] To verify the data is being received at IoT Central, make sure your IoT Central pp is open, and the device selected. If not, select the **Devices** entry in the left-hand menu. Double-click the real device (**RefrigeratedTruck - 1**), in the list of devices.

1. [ ] Locate the **Contents temperature** tile, and verify approximately that the temperatures being sent by the device app, in the console window, match the data being shown in the telemetry view of the IoT Central app.

1. [ ] Check the state tiles: **Truck state**, **Cooling system state**, and **Contents state** in the IoT Central app, to verify the truck and its contents are in the expected state.

1. [ ] Check the **Location** map view for the device. A blue circle near Seattle, USA shows our truck ready to go. You may have to zoom out a bit.

If all is well, this is great progress. The truck is at its base, in the correct state, and waiting for a command.

##### Continue the tests of your IoT Central device

In this task, we complete the app testing.

#####. Confirm the command to send the truck to a specified customer works as expected

Now for the best fun of all.

1. [ ] Click the **Commands** title for the device. This control should be under the truck name, and right of the **Truck view** control.

1. [ ] Enter a customer ID, say "1" ("0" through "9" are valid customer IDs), and click **Run**.

1. [ ]  In the console for the device app, you should both see  **New customer** event, and a **Route found** message.
@@@warning
**Note**: If you see a message including the text **Access denied due to invalid subscription key**, then check your subscription key to Azure Maps.
@@@

1. [ ] In the dashboard **Location** tile, is your truck on its way? You might have to wait a short time for the apps to sync up.

1. [ ] Verify the event text in the dashboard tile.

Great progress! Take a moment to just watch the map update, and your truck deliver its contents.

##### Confirm the command to recall the truck works as expected

1. [ ] When the truck returns to base, and is reloaded with contents, it's state will be **ready**. Try issuing another delivery command. Choose another customer ID.

1. [ ] Issue a recall command before the truck reaches its customer, to check the truck responds to this command.

##### Check customer and conflict events are transmitted correctly

To test a conflict event, send a command that you know doesn't make sense.

1. [ ] With your truck at the base, issue a Recall command. The truck should respond with the "already at base" event.

##### Check the truck properties, and change the optimal temperature

1. [ ] The simplest test is to check the **Truck ID** tile. This tile should have picked up the **Truck number 1** message when the apps were started.

1. [ ] A more complex test is to check the writable property, **OptimalTemperature**. To change this value, click on **Jobs** in the left-hand menu.

1. [ ] Click **+ New**, top-right.

1. [ ] Give the job a friendly name, "Set optimal temperature to -10".

1. [ ] For **Device group**, select **RefrigeratedTruck - All devices**. For **Job type**, select **Writable properties**. For **Writable properties**, select **Optimal Temperature**.

1. [ ] Finally, set the value as **-10**.

1. [ ]  Running this job should set the optimal temperature for all trucks in the device group, just one in our case. Click **Run**. Wait for the **Status** of the job to change from **Pending** to **Completed**. This change should only take a few seconds.

1. [ ] Navigate back, via **Devices** to your dashboard. Verify the **Optimal temperature** has been set to -10, in the tile on the dashboard.

##### Next steps

With the testing for one truck complete, it is time to consider expanding our IoT Central system.

#### Exercise 7: Create multiple devices

In this task, we consider what steps would be necessary to add multiple trucks to our system.

##### Add multiple devices to the IoT Central app

1. [ ] To add multiple devices, start in the [Azure IoT Central](https://apps.azureiotcentral.com/?azure-portal=true) app, clicking **Devices** in the left-hand menu.

1. [ ] Click **RefrigeratedTruck** in the **Devices** menu, to ensure the device we create uses this device template. The device template you select will be shown in bold text.

1. [ ] Click **+ New**. Verify in the dialog that the device name includes the **RefrigeratedTruck** text. If it doesn't, you've not selected the right device template.

1. [ ] Change the **Device ID** to a friendlier name, say "RefrigeratedTruck2".

1. [ ] Change the **Device name** to a friendlier name, say "RefrigeratedTruck - 2".

1. [ ] Leave the **Simulated** setting at **Off**.

1. [ ] Click **Create**.

Repeat this process to create as many devices as you need.

##### Provision the new devices

1. [ ] Double-click on **RefrigeratedTruck - 2**, and then click **Connect** (top right of your IoT Central screen).

1. [ ] In the **Device Connection** screen, copy the **Device ID** and the **Primary Key** to your text file, noting that they are for the second truck. There is no need to copy the **Scope ID**, as this value is identical to the value for the first truck (it identifies your app, not an individual device).

1. [ ] Click **Close**.

1. [ ] Back on the **Devices** page, repeat the process for any other devices you created, copying their **Device ID** and **Primary Key** to your text file.

1. [ ] When you have completed connecting all new trucks, notice that the **Provisioning Status** is still **Registered**. Not until you make the connection will this change.

##### Create new apps for each new device

Each truck is simulated by one running copy of the device app. So, you need multiple versions of this app running simultaneously.

1. [ ]  Create multiple projects by repeating the steps in the **Create a programming project for a real device** for each new device. Copy and paste the entire app from your current working project, replacing the **Device ID** and **Primary Key** with new values. No need to change the **Scope ID** or the **Azure Maps subscription Key**, as these are identical for all devices.

1. [ ] Remember to add the necessary libraries to each new project.

1. [ ] Change the `truckNum` in each project to a different value.

1. [ ] Set each project app running.

##### Verify the telemetry from all the devices

1. [ ] Verify that the one dashboard you created works for all trucks.

1. [ ] Using the dashboard for each truck, try ordering the trucks to different customers. Using the **Location** map on each dashboard, verify the trucks are heading in the right direction!

Congratulations on completing the lab! We suggest that you clean up your resources.

@@@Success
Congratulations, you have now completed this lab!
@@@
