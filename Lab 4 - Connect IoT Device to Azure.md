### Lab - Connect IoT Device to Azure


@@@secondary
**Scenario**

Contoso is known for producing high quality of cheeses. Due to the company rapidly growing in popularity and sales, they want to ensure that their cheeses stay at the same level of quality. At the moment, a worker temperature and humidity data is collected by floor workers every shift.

Contoso is exploring adding an IoT device to monitor the temperature and humidity of their batches of cheeses. For the asset monitoring solution, you will be connecting an IoT device with temperature and humidity sensors (temperature, humidity) to IoT Hub.

In this lab, you will complete the following activities:

* Verify Lab Prerequisites
* Register a Device ID in Azure IoT Hub using the Azure CLI.
* You will then configure and run a pre-built Simulated Device written in C# to connect to Azure IoT Hub and send Device-to-Cloud telemetry messages.
* Verify the device telemetry is being received by Azure IoT Hub using the Azure CLI.
@@@

#### Exercise 1: Verify Lab Prerequisites

@@@secondary
This lab assumes that the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |

If these resources are not available, you will need to run the **lab04-setup.azcli** script as instructed below before moving on to Exercise 2. The script file is included in the GitHub repository that you cloned locally as part of the dev environment configuration (lab 3).
@@@

@@@warning
**Note**: The **lab04-setup.azcli** script is written to run in a **bash** shell environment - the easiest way to execute this is in the Azure Cloud Shell.
@@@

1. [ ] If necessary, log in to Azure Portal using your Azure account credentials.

 ```cli
https://portal.azure.com
 ```

1. [ ] If you are prompted about setting up storage for Cloud Shell, accept the defaults.

1. [ ] Verify that the Azure Shell is using **Bash**.

    The dropdown in the top-left corner of the Azure Cloud Shell page is used to select the environment. Verify that the selected dropdown value is **Bash**.

1. [ ] On the Azure Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. [ ] In the dropdown, click **Upload**.

1. [ ] In the file selection dialog, navigate to the folder location of the GitHub lab files that you downloaded when you configured your development environment.

    In Lab 3 of this course, "Setup the Development Environment", you cloned the GitHub repository containing lab resources by downloading a ZIP file and extracting the contents locally. The extracted folder structure includes the following folder path:

    * Allfiles
      * Labs
          * 04-Connect an IoT Device to Azure
            * Setup
</br>

    The lab04-setup.azcli script file is located in the Setup folder for lab 4.

1. [ ] Select the **lab04-setup.azcli** file, and then click **Open**.

    A notification will appear when the file upload has completed.

1. [ ] To verify that the correct file has uploaded, enter the following command:

    ```bash
    ls
    ```

    The `ls` command lists the content of the current directory. You should see the lab04-setup.azcli file listed.

1. [ ] To create a directory for this lab that contains the setup script and then move into that directory, enter the following Bash commands:

    ```bash
    mkdir lab4
    ```
    ```bash
    mv lab04-setup.azcli lab4
    ```
    ```bash
    cd lab4
    ```

    These commands will create a directory for this lab, move the **lab04-setup.azcli** file into that directory, and then change directory to make the new directory the current working directory.

1. [ ] To ensure the **lab04-setup.azcli** has the execute permission, enter the following command:

    ```bash
    chmod +x lab04-setup.azcli
    ```

1. [ ] On the Cloud Shell toolbar, to edit the **lab04-setup.azcli** file, click **Open Editor** (second button from the right - **{ }**).

1. [ ] In the **Files** list, to expand the lab4 folder, click **lab4**, and then click **lab04-setup.azcli**.

    The editor will now show the contents of the **lab04-setup.azcli** file.

1. [ ] In the editor, update the values of the `{YOUR-ID}` and `{YOUR-LOCATION}` variables.

    In the sample below, you need to set `{YOUR-ID}` to the Unique ID you created at the start of this course - i.e. **CAH191211**, and set `{YOUR-LOCATION}` to the location that matches your resource group.

    ```bash
    #!/bin/bash

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

    Location="{YOUR-LOCATION}"
    ```
@@@warning
**Note**:  The `{YOUR-LOCATION}` variable should be set to the short name for the region where you are deploying all of your resources. You can see a list of the available locations and their short-names (the **Name** column) by entering this command:
@@@
    >
    > ```bash
    > az account list-locations -o Table
    >
    > DisplayName           Latitude    Longitude    Name
    > --------------------  ----------  -----------  ------------------
    > East Asia             22.267      114.188      eastasia
    > Southeast Asia        1.283       103.833      southeastasia
    > Central US            41.5908     -93.6208     centralus
    > East US               37.3719     -79.8164     eastus
    > East US 2             36.6681     -78.3889     eastus2
    > ```

1. [ ] In the top-right of the editor window, to save the changes made to the file and close the editor, click **...**, and then click **Close Editor**.

    If prompted to save, click **Save** and the editor will close.
@@@warning
**Note**:  You can use **CTRL+S** to save at any time and **CTRL+Q** to close the editor.
@@@

1. [ ] To create the resources required for this lab, enter the following command:

    ```bash
    ./lab04-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

Once the script has completed, you will be ready to continue with the lab.

#### Exercise 2: Create Azure IoT Hub Device ID using Azure CLI

@@@secondary
The `iot` Azure CLI modules includes several commands for managing IoT Devices within Azure IoT Hub under the `az iot hub device-identity` command group. These commands can be used to manage IoT Devices within scripts or directly from the command-line / terminal.
@@@

##### Task 1: Create the IoT Hub Device ID

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] At the top of the Azure Portal click on the **Cloud Shell** icon to open up the **Azure Cloud Shell** within the Azure Portal.

1. [ ] If you are prompted about setting up storage for Cloud Shell, accept the defaults.

1. [ ] When the pane opens, choose the option for the **Bash** terminal within the Cloud Shell.

1. [ ] Within the Cloud Shell, to ensure the Cloud Shell has the IoT extension installed, run the following command:

    ```cli
    az extension add --name azure-cli-iot-ext
    ```

1. [ ] Still within the Cloud Shell, run the following Azure CLI command to create **Device Identity** in Azure IoT Hub that will be used for a Simulated Device.

    ```sh
    az iot hub device-identity create --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```
@@@warning
**Note**:  Be sure to replace the _{IoTHubName}_ placeholder with the name of your Azure IoT Hub. If you have forgotten your IoT Hub name, you can enter the following command:
 ```
az iot hub list -o table
```
@@@

##### Task 2: Get the Device Connection String

1. [ ] Within the Cloud Shell, run the following Azure CLI command to get _device connection string_ for the Device ID that was just added to the IoT Hub. This connection string will be used to connect the Simulated Device to the Azure IoT Hub.

    ```cmd/sh
    az iot hub device-identity show-connection-string --hub-name {IoTHUbName} --device-id SimulatedDevice1 --output table
    ```

1. [ ] Make note of the **Device Connection String** that was output from the previous command. You will need to save this for use later.

    The connection string will be in the following format:

    ```cli
    HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}
    ```

#### Exercise 3: Configure and Test a Simulated Device (C#)

@@@secondary
In this exercise you will configure a simulated device written in C# to connect to Azure IoT Hub using the Device ID and Shared Access Key created in the previous exercise. You will then test the device and ensure that IoT Hub is receiving telemetry from the device as expected.
@@@

##### Task 1: Open the Lab 4 Starter Code Project

1. [ ] Open a new instance of Visual Studio Code.

1. [ ] On the left-side menu, click **Explorer**.

    The Explorer pane lists the file/folder hierarchy. Your new instance of Visual Studio Code will not have an open folder.

1. [ ] On the File menu, click **Open Folder**.

1. [ ] In the Open Folder dialog, navigate to the Lab 4 folder that contains the starter code project.

    The local path to the Lab 4 Starter project folder should be similar to the following:

    * AZ-220-Microsoft-Azure-IoT-Developer-master
    
      * Allfiles
        * Labs
        
          * 04-Connect an IoT Device to Azure
            * Starter
          <br/>
@@@warning
**Note**: You cloned the GitHub project when you set up the dev environment in Lab 3. Check with your course instructor if needed to locate your resource files.
@@@

1. [ ] To open the folder, click **Starter**, and then click **Select Folder**.

    The Explorer pane of Visual Studio Code should now list two C# project files:

    * SimulatedDevice.cs
    * SimulatedDevice.csproj

##### Task 2: Update the Device Connection String

1. [ ] In Visual Studio Code Explorer pane, to open the SimulatedDevice.cs file, click **SimulatedDevice.cs**.

1. [ ] In the Editor view, locate the code line containing the `s_connectionString` variable.

    ```C#
    private readonly static string s_connectionString = "{Your device connection string here}";
    ```

1. [ ] Replace the value placeholder `{Your device connection string here}` with the Device Connection String that you copied previously.

    This will enable the Simulated Device to authenticate, connect, and communicate with the Azure IoT Hub.

    Once configured, the variable will look similar to the following (with your specific connection information included):

    ```csharp
    private readonly static string s_connectionString = "HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}";
    ```

1. [ ] On the **View** menu, click **Terminal**.

    Verify that the selected terminal shell is the windows command prompt.

1. [ ] In the Terminal view, at the command prompt, enter the following command:

    ```cmd/sh
    dotnet run
    ```

    This command will build and run the Simulated Device application. Be sure the terminal location is set to the directory with the `SimulatedDevice.cs` file.
@@@warning
**Note**:  If the command outputs a `Malformed Token` or other error message, then make sure the **Device Connection String** is configured correctly as the value of the `s_connectionString` variable.
@@@

1. [ ] Once the Simulated Device application is running, it will be sending event messages to the Azure IoT Hub that include `temperature` and `humidity` values.

    The terminal output will look similar to the following:

    ```text
    IoT Hub C# Simulated Device. Ctrl-C to exit.

    10/25/2019 6:10:12 PM > Sending message: {"temperature":27.714212817472504,"humidity":63.88147743599558}
    10/25/2019 6:10:13 PM > Sending message: {"temperature":20.017463779085066,"humidity":64.53511070671263}
    10/25/2019 6:10:14 PM > Sending message: {"temperature":20.723927165718717,"humidity":74.07808918230147}
    10/25/2019 6:10:15 PM > Sending message: {"temperature":20.48506045736608,"humidity":71.47250854944461}
    10/25/2019 6:10:16 PM > Sending message: {"temperature":25.027703996760632,"humidity":69.21247714628115}
    10/25/2019 6:10:17 PM > Sending message: {"temperature":29.867399432634656,"humidity":78.19206098010395}
    10/25/2019 6:10:18 PM > Sending message: {"temperature":33.29597232085465,"humidity":62.8990878830194}
    10/25/2019 6:10:19 PM > Sending message: {"temperature":25.77350195766124,"humidity":67.27347029711747}
    ```
@@@warning
**Note**: Leave the simulated device app running for now. Your next task will be to verify that your IoT Hub is receiving the telemetry messages.
@@@

##### Task 3: Verify Telemetry Stream sent to Azure IoT Hub

@@@secondary
In this task, you will use the Azure CLI to verify telemetry sent by the simulated device is being received by Azure IoT Hub.
@@@

1. [ ] Using a browser, open the Azure Cloud Shell and login with the Azure subscription you are using for this course.

1. [ ] In the Azure Cloud Shell, enter the following command:

    ```cli
    az iot hub monitor-events --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```

    _Be sure to replace the **{IoTHubName}** placeholder with the name of your Azure IoT Hub._
@@@warning
**Note**:  If you receive a message stating _"Dependency update required for IoT extension version"_ when running the Azure CLI command, then press `y` to accept the update and press `Enter`. This will allow the command to continue as expected.
@@@

    The `--device-id` parameter is optional and allows you to monitor the events from a single device. If the parameters is omitted, the command will monitor all events sent to the specified Azure IoT Hub.

    The `monitor-events` command within the `az iot hub` Azure CLI module offers the capability to monitor device telemetry & messages sent to an Azure IoT Hub from within the command-line / terminal.

1. [ ] Notice that the `az iot hub monitor-events` Azure CLI command outputs a JSON representation of the events that are arriving at your specified Azure IoT Hub. 

    This command enables you to monitor the events being sent to IoT hub. You are also verifying that the device is able to connect to and communicate with the your IoT hub.

    You should see messages displayed that are similar to the following:

    ```cmd/sh
    Starting event monitor, filtering on device: SimulatedDevice1, use ctrl-c to stop...
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":25.058683971901743,\"humidity\":67.54816981383979}"
        }
    }
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":29.202181296051563,\"humidity\":69.13840303623043}"
        }
    }
    ```

1. [ ] Once you have verified that IoT hub is receiving the telemetry, press **Ctrl-C** in the Azure Cloud Shell and Visual Studio Code windows.

    Ctrl-C is used to stop the running apps. Always remember to shut down unneeded apps and jobs.

@@@success
**Congratulations**! you have now completed this lab!
@@@
