### Lab 13: Develop, Deploy and debug a custom module on Azure IoT Edge with VS Code

@@@secondary
**Lab Scenario**

To help manage fluctuations in consumer demand, Contoso maintains a small inventory of ripened cheese wheels in a warehouse at each cheese making facility. These ripened wheels are sealed in wax and the storage environment is carefully controlled to ensure that the cheese remains in perfect condition. Contoso uses a conveyor system to move the large wax-sealed cheese wheels from the warehouse to the packaging facilities.

In the past, Contoso has run their packaging process at full capacity, processing all of the cheese that is placed in the system. Any excess volume of packaged cheese was not an issue because it could be used for promotional offers, and additional cheese could be pulled from inventory as needed. However, with the significant growth that Contoso is experiencing, and with growing fluctuations due to worldwide demand, the company needs to automate the system in a way that helps to manage the volume of cheese being packaged.

Since you have already implemented the IoT solution that monitors the conveyor belt system in the packaging and shipping area, you have been tasked with developing a solution that helps to manage/control packaging volumes.

To ensure that the correct number of packages have been processed, you decide to create (and deploy to an IoT Edge device) a simple module that counts the number of packages detected on the conveyor belt system. You already have another module available that can be used to detect the packages (both modules be deployed to the same IoT Edge device).

You need to create and deploy a custom IoT Edge module that counts the number of packages detected by the other module.

This lab includes the following prerequisites for the development machine (lab host environment - VM or PC):

* Visual Studio Code with the following extensions installed:

  * [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) by Microsoft
  
  * [C#](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp) by Microsoft
  
  * [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)
  <br/>
* Docker Community Edition installed on development machine, with Docker Client version 18.03.0 or later
  * [Download Docker Desktop for Mac and Windows](https://www.docker.com/products/docker-desktop)
@@@

@@@danger
**Important**: Due to the January 13, 2020 removal of Azure Container Registry support for any TLS versions before TLS version 1.2, you must be running Docker Client 18.03.0 or later.
@@@


@@@secondary
In this lab, you will complete the following activities:

* Verify Lab Prerequisites
* Create Container Registry
* Create and customize an Edge module
* Deploy modules to Edge device
@@@



#### Exercise 1: Verify Lab Prerequisites

This lab assumes that the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |

If these resources are not available, you will need to run the **lab13-setup.azcli** script as instructed below before moving on to Exercise 2. The script file is included in the GitHub repository that you cloned locally as part of the dev environment configuration (lab 3).
@@@warning
**Note**:  The **lab13-setup.azcli** script is written to run in a **bash** shell environment - the easiest way to execute this is in the Azure Cloud Shell.
@@@

1. [ ] Using a browser, open the **Azure Cloud Shell** `https://shell.azure.com` and login with the Azure subscription you are using for this course.

1. [ ] If you are prompted about setting up storage for Cloud Shell, accept the defaults.

1. [ ] Verify that the Azure Shell is using **Bash**.

    The dropdown in the top-left corner of the Azure Cloud Shell page is used to select the environment. Verify that the selected dropdown value is **Bash**.

1. [ ] On the Azure Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. [ ] In the dropdown, click **Upload**.

1. [ ] In the file selection dialog, navigate to the folder location of the GitHub lab files that you downloaded when you configured your development environment.

    In Lab 3 of this course, "Setup the Development Environment", you cloned the GitHub repository containing lab resources by downloading a ZIP file and extracting the contents locally. The extracted folder structure includes the following folder path:

    * Allfiles
    
      * Labs
      
          * 13-Develop, Deploy and debug a custom module on Azure IoT Edge with VS Code
            * Setup

    The lab13-setup.azcli script file is located in the Setup folder for lab 13.

1. [ ] Select the **lab13-setup.azcli** file, and then click **Open**.

    A notification will appear when the file upload has completed.

1. [ ] To verify that the correct file has uploaded, enter the following command:

    ```bash
    ls
    ```

    The `ls` command lists the content of the current directory. You should see the lab13-setup.azcli file listed.

1. [ ] To create a directory for this lab that contains the setup script and then move into that directory, enter the following Bash commands:

    ```bash
    mkdir lab13
    mv lab13-setup.azcli lab13
    cd lab13
    ```

    These commands will create a directory for this lab, move the **lab13-setup.azcli** file into that directory, and then change directory to make the new directory the current working directory.

1. [ ] To ensure the **lab13-setup.azcli** has the execute permission, enter the following command:

    ```bash
    chmod +x lab13-setup.azcli
    ```

1. [ ] On the Cloud Shell toolbar, to edit the **lab13-setup.azcli** file, click **Open Editor** (second button from the right - **{ }**).

1. [ ] In the **Files** list, to expand the lab4 folder, click **lab13**, and then click **lab13-setup.azcli**.

    The editor will now show the contents of the **lab13-setup.azcli** file.

1. [ ] In the editor, update the values of the `{YOUR-ID}` and `{YOUR-LOCATION}` variables.

    Referencing the sample below as an example, you need to set `{YOUR-ID}` to the Unique ID you created at the start of this course - i.e. **CAH191211**, and set `{YOUR-LOCATION}` to the location that matches your resource group.

    ```bash
    #!/bin/bash

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

    Location="{YOUR-LOCATION}"
    ```
@@@warning
**Note**:  The `{YOUR-LOCATION}` variable should be set to the short name for the region where you are deploying all of your resources. You can see a list of the available locations and their short-names (the **Name** column) by entering the command below.
@@@
    >
    > ```bash
    > az account list-locations -o Table
    > ```
    > DisplayName           Latitude    Longitude    Name
    > --------------------  ----------  -----------  ------------------
    > East Asia             22.267      114.188      eastasia
    > Southeast Asia        1.283       103.833      southeastasia
    > Central US            41.5908     -93.6208     centralus
    > East US               37.3719     -79.8164     eastus
    > East US 2             36.6681     -78.3889     eastus2
    

1. [ ] In the top-right of the editor window, to save the changes made to the file and close the editor, click **...**, and then click **Close Editor**.

    If prompted to save, click **Save** and the editor will close.
@@@warning
**Note**:  You can use **CTRL+S** to save at any time and **CTRL+Q** to close the editor.
@@@

1. [ ] To create the resources required for this lab, enter the following command:

    ```bash
    ./lab13-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

 Once the script has completed, you will be ready to continue with the lab.

#### Exercise 2: Install Azure IoT EdgeHub Dev Tool

@@@secondary
In this exercise, you will will install the Azure IoT EdgeHub Dev Tool.
@@@

1. [ ] Verify that you have Python 3.7 installed in your development environment.

    Lab 3 of this course has you prepare the lab environment, including the installation Python 3.7. If Python is not installed, refer back to the instructions in Lab 3.
@@@warning
**Note**:  Currently, the Azure IoT EdgeHub Dev Tool uses a docker-py library that is not compatible with Python 3.8.
@@@

1. [ ] With Python installed, open Windows Command Prompt.

1. [ ] At the command prompt, to install the package manager for Python (Pip), enter the following commands:

    ```cmd/sh
    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
    python get-pip.py
    ```

    Pip is required to install the Azure IoT EdgeHub Dev Tool on your development machine. 
@@@danger
**Important**: When downloading code like this, you should consider reviewing the code before running it.
@@@

    If you have issues installing Pip, please reference the official Pip **installation instructions** `https://pip.pypa.io/en/stable/installing`
@@@warning
**Note**: On Windows, Python and/or Pip are sometimes installed but are not in the `PATH`. Check with your instructor if you have Python installed but it does not seem to be available.
@@@

1. [ ] To install the Azure IoT EdgeHub Dev Tool, enter the following command: 

    ```cmd/sh
    pip install iotedgehubdev
    ```
@@@warning
**Note**:  If you have multiple versions of Python installed in your development environment, including pre-installed Python 2.7 (for example, on Ubuntu or macOS), make sure you are using the correct `pip` or `pip3` to install `iotedgehubdev`.
@@@

    You can read more about the Azure IoT EdgeHub Dev Tool here: **Azure IoT EdgeHub Dev Tool** `https://pypi.org/project/iotedgehubdev/`

 Now that you have configured the Python environment and installed these tools, you are ready to create an Azure Container Registry which will be used to store our custom IoT Edge module.

#### Exercise 3: Create Azure Container Registry

@@@secondary
Azure Container Registry provides storage of private Docker images for container deployments. The service is a managed, private Docker registry service based on the open-source Docker Registry 2.0. Azure Container Registry is used to store and manage your private Docker container images.

In this exercise, you will use the Azure portal to create a new Azure Container Registry resource.
@@@

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] On the Azure portal menu, click **+ Create a resource**.

1. [ ] On the **New** blade, in the **Search the Marketplace** textbox, type **container registry** and then press **Enter**.

1. [ ] On the **Marketplace** blade, click **Container Registry**.

1. [ ] On the **Container Registry** blade, click **Create**.

1. [ ] On the **Create container registry** blade, under **Registry name**, enter a globally unique name.

    To provide a globally unique name, enter **AZ220ACR{YOUR-ID}**.

    For example: **AZ220ACRCAH191204**

    The name of your Azure Container Registry must be globally unique because it is a publicly accessible resource that you must be able to access from any IP connected device.

    Consider the following when you specify a unique name for your new Azure Container Registry:

    * As mentioned above, the name of the registry must be unique across all of Azure. This is true because the value assigned to the name will be used in the domain name assigned to the service. Since Azure enables you to connect from anywhere in the world to your registry, it makes sense that all container registries must be accessible from the Internet using the resulting domain name.

    * The registry name cannot be changed once the Azure Container Registry has been created. If you do need to change the name, you'll need to create a new Container Registry, re-deploy your container images, and delete your old Container Registry.
@@@warning
**Note**:  Azure will ensure that the name you enter is unique. If the name that you enter is not unique, Azure will display an asterisk at the end of the name field as a warning. You can append the name suggested above with `01` or `02` as necessary to achieve a globally unique name.
@@@

1. [ ] Under **Subscription**, ensure that the subscription you are using for this course is selected.

1. [ ] In the **Resource group** dropdown, click **AZ-220-RG**.

1. [ ] In the **Location** dropdown, choose the same Azure region that was used for the resource group.

1. [ ] Under **Admin user**, click **Enable**.

    This option will enable you to Docker login to the Azure Container Registry service using the registry name as the username and admin user access key as the password.

1. [ ] In the **SKU** dropdown, ensure that **Standard** is selected.

    Azure Container Registry is available in multiple service tiers, known as SKUs. These SKUs provide predictable pricing and several options for aligning to the capacity and usage patterns of your private Docker registry in Azure.

1. [ ] At the bottom of the blade, click **Review + create**.

    The settings you have entered will be validated.

1. [ ] To complete the creation of the Container Registry, at the bottom of the blade, click **Create**.

1. [ ] On your dashboard, refresh your Resources tile, and then click **AZ220ACR{YOUR-ID}**.

1. [ ] On the left side navigation menu, under **Settings**, click **Access keys**.

    This is where you will find the username and password for the **Admin user**  

1. [ ] Make a record of the following values:

    * **Login server**
    * **Username**
    * **password**
</br>

    By default, the admin Username will match the ACR name - **AZ220ACR{YOUR-ID}**

    This information will enable you to authenticate to the new registry, which is required to perform Docker operations in the upcoming steps.  

1. [ ] Open a command prompt, and then enter the following command:

    ```cmd/sh
    docker login --username <username> --password-stdin <loginserver>
    ```

    Replace the placeholders with the information you recorded, and enter the password you recorded when prompted.  For example:

    ```cmd/sh
    docker login --username az220acrcah191204 --password-stdin  az220acrcah191204.azurecr.io
    ```

    This command will record your credentials in the local Docker client configuration file (`$HOME/.docker/config.json`) or your operating system's secure credential storage mechanism (depending on the Docker configuration) for future use by the Docker toolset.
@@@warning
**Note**: You will not be prompted to enter your password - just type it after you enter the above command. You will see your password as you enter it. Press **Enter** to submit the password. It may take a few moments before the **Login Succeeded** message is shown.
@@@

 Now that you have created the Azure Container Registry and authenticated your local machine against it, you can create a custom IoT Edge Module container that will be stored in the registry.

#### Exercise 4: Create Custom Edge Module in C#.

@@@secondary
In this exercise, you will create an Azure IoT Edge Solution that contains a custom Azure IoT Edge Module written in C#.
@@@

1. [ ] Open Visual Studio Code.

1. [ ] On the **View** menu, to open the Visual Studio Command Palette, click **Command Palette**.

1. [ ] At the command prompt, type **Azure IoT Edge: New** and then click **Azure IoT Edge: New IoT Edge Solution**.

1. [ ] Browse to the folder where you want to create the new solutions, and then click **Select folder**.

1. [ ] When prompted for a solution name, enter **EdgeSolution**

    This name will be used as the directory name for the new **IoT Edge Solution** that will be created.

1. [ ] When prompted to select a module template, click **C# Module**.

    This will define `C#` as the development language for the custom IoT Edge Module added to the solution.

1. [ ] When prompted for the name of the custom IoT Edge Module, enter **ObjectCountingModule**

    This will be the name of the new IoT Edge Module that is being created.

1. [ ] When prompted for name of the Docker image repository for the module, update the placeholder value as follows:

    Replace the `localhost:5000` part of the default `localhost:5000/objectcountingmodule` repository location with the name of the Azure Container Registry server - similar to `az220acr{your-id}.azurecr.io`

    This will be the Docker repository where the IoT Edge Module docker image will be published. 

    The Docker image repository location follows the format shown below:

    ```text
    <acr-name>.azurecr.io/<module-name>
    ```

    Be sure to replace the placeholders with the appropriate values:

    * `<acr-name>`: Replace with the name of the Azure Container Registry service.
    * `<module-name>`: Replace with the name of the custom Azure IoT Edge Module that's being created.
</br>
@@@warning
**Note**:  The default Docker image repository in Visual Studio Code is set to `localhost:5000/<your module name>`. If you were to use a local Docker registry for testing, then **localhost** is fine.
@@@
@@@danger
**Important**: Make sure to remove any reference to port `5000` from your ACR references!  That port is used for a local Docker repository but it not used in the ACR case.
@@@

1. [ ] Wait for Visual Studio Code to create the solution.

    Once the new **IoT Edge Solution** has been created, Visual Studio Code will open the solution.

    If Visual Studio Code prompts you to load required resources or C# extension, click **Yes**

1. [ ] Take a moment to review the contents of the **Explorer** pane.

    Notice the files and directories that were created as part of the new IoT Edge Solution.

1. [ ] In the **Explorer** pane, to open the `.env` file, click **.env**.

    The .env file is located in the root directory of the IoT Edge Solution. This is where the username and password are configured for accessing your Docker registry.

    The username and password are stored in this file using the following format:

    ```text
    CONTAINER_REGISTRY_USERNAME_<registry-name>=<registry-username>
    CONTAINER_REGISTRY_PASSWORD_<registry-name>=<registry-password>
    ```

    The placeholders above are defined as follows:

    * `<registry-name>`: The name of your Docker registry.
    * `<registry-username>`: The username to use for accessing your Docker registry.
    * `<registry-password>`: The password to use for accessing your Docker registry.
</br>

    Within your version of `.env` file, notice that the `<registry-name>` has already been added to the configuration values. The value that has been added should match the name of the Docker registry that you specified when creating the IoT Edge Solution.
@@@warning
**Note**: You may wonder why you ran `docker login` before when you're supplying the same credentials here.  At the time when this lab was written, the Visual Studio Code tools do not automatically perform the `docker login` step with these credentials; they are only used to supply the credentials to the Edge Agent later as part of the deployment template.
@@@

1. [ ] Within the `.env` file, replace the placeholder values with the username and password values that you save earlier. 

    Replace the `<registry-username>` placeholder with the **Registry name** (_aka Username_) of the Azure Container Registry that was previously created.
    Replace the `<registry-password>` placeholder with the **password** for the Azure Container Registry.
@@@warning
**Note**:  The Azure Container Registry **Username** and **password** values can be found by accessing the **Access keys** pane for the **Azure Container Registry** service within the Azure portal, if you did not record them earlier.
@@@

1. [ ] In the **Explorer** pane, to open the `deployment.template.json` file, click **deployment.template.json**.

    The `deployment.template.json` file is located in the root IoT Edge Solution directory. This file is the _deployment manifest_ for the IoT Edge Solution. The deployment manifest tells an IoT Edge device (or a group of devices) which modules to install and how to configure them. The deployment manifest includes the _desired properties_ for each module twin. IoT edge devices report back the _reported properties_ for each module.

    Two modules are required in every deployment manifest; `$edgeAgent` and `$edgeHub`. These modules are part of the IoT Edge runtime that manages the IoT Edge devices and the modules running on it.

1. [ ] Scroll through the `deployment.template.json` deployment manifest file, and notice the following within the `properties.desired` section of the `$edgeAgent` element:

    * `systemModules` - This defines Docker images to use for the `$edgeAgent` and `$edgeHub` system modules that are part of the IoT Edge runtime.

    * `modules` - This defines the various modules that will be deployed and run on the IoT Edge device (or a group of devices).

1. [ ] Notice that within the `modules` section for the `$edgeAgent`, there are two modules defined:

    * `ObjectCountingModule`: This is the custom IoT Edge Module that is being created as part of this new IoT Edge Solution.

    * `SimulatedTemperatureSensor`: This defines the Simulated Temperature Sensor module to be deployed to the IoT Edge device.

1. [ ] Notice the `$edgeHub` section of the deployment manifest. 

    This section defines the desired properties (via `properties.desired` element) that includes the message routes for communicating messages between the IoT Edge Modules and finally to Azure IoT Hub service.

    ```json
        "$edgeHub": {
          "properties.desired": {
            "schemaVersion": "1.0",
            "routes": {
              "ObjectCountingModuleToIoTHub": "FROM /messages/modules/ObjectCountingModule/outputs/* INTO $upstream",
              "sensorToObjectCountingModule": "FROM /messages/modules/SimulatedTemperatureSensor/outputs/temperatureOutput INTO BrokeredEndpoint(\"/modules/ObjectCountingModule/inputs/input1\")"
            },
            ...
          }
        }
    ```

    The `sensorToObjectCountingModule` route is configured to route messages from the `SimulatedTemperatureSensor` (via `/messages/modules/SimulatedTemplaratureSensor/outputs/temperatureOutput`) module to the custom `OutputCounterModule` module (via `BrokeredEndpoint(\"/modules/ObjectCountingModule/inputs/input1\")"`).

    The `ObjectCountingModuleToIoTHub` route is configured to route messages that are sent out from the custom `OutputCounterModule` module (via `/messages/modules/SimulatedTemperatureSensor/outputs/temperatureOutput`) to the Azure IoT Hub service (via `$upstream`).

1. [ ] In Visual Studio Code, on the **View** menu, click **Command Palette**

1. [ ] At the command prompt, type **Azure IoT Edge: Set Default** and then click **Azure IoT Edge: Set Default Target Platform for Edge Solution**.

1. [ ] To select the target platform, click **amd64**.

    This target platform needs to be set to the hardware platform architecture of the IoT Edge Device.
@@@warning
**Note**: Since you are using the **IoT Edge on Ubuntu** Linux VM, the `amd64` option is the appropriate choice. For a Windows VM, use `windows-amd64`, and for modules that will be running on an ARM CPU architecture, use choose the `arm32v7` option.
@@@

1. [ ] In the **Explorer** pane, to expand the `/modules/ObjectCountingModule` directory, click **modules**.

    Notice that this directory contains the source code files for the new IoT Edge Module being developed.

1. [ ] In the **Explorer** pane, to open the `/modules/ObjectCountingModule/Program.cs` file, click **Program.cs**.

    This file contains the template source code for the newly created custom IoT Edge Module. This code provides a starting point for creating custom IoT Edge Modules.

1. [ ] In the Program.cs file, locate the `static async Task Init()` method, and then take a minute to review the code.

    This method initializes the `ModuleClient` for handling messages sent to the module, and sets up the callback to receive messages. Read the code comments within the code for this method and notice what each section of code does.

1. [ ] Locate the `static async Task<MessageResponse> PipeMessage(` method, and then take a minute to review the code.

    This method is called whenever the module is sent a message from the EdgeHub. The current state of the source code within this method receives messages sent to this module and pipes them out to the module output, without any change. Read through the code within this method and notice what it does.

    Also, within the `PipeMessage` method, notice the following lines of code and what they do:

    The following line of code within the method increments a counter that counts the number of messages sent to the module:

    ```csharp
    int counterValue = Interlocked.Increment(ref counter);
    ```

    The following lines of code within the method write out to the Module's `Console` a message that contains the total number of messages received by the Module, along with the current message's body as JSON.

    ```csharp
    byte[] messageBytes = message.GetBytes();
    string messageString = Encoding.UTF8.GetString(messageBytes);
    Console.WriteLine($"Received message: {counterValue}, Body: [{messageString}]");
    ```

 We have now created and configured a sample custom module. Next, we will debug it in the IoT Edge Simulator.

#### Exercise 5: Debug in Attach Mode with IoT Edge Simulator

@@@secondary
In this exercise, you will build and run a custom IoT Edge Module solution using the IoT Edge Simulator from within Visual Studio Code.
@@@

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] On your Resource group tile, click **AZ-220-HUB-_{YOUR-ID}_**.

1. [ ] On the left hand navigation menu, under **Settings**, click **Shared access policies**.

1. [ ] In the list of policies, click **iothubowner**.
@@@danger
**Important**: The Edge Simulator requires a privileged role for configuration. You would not use such a privileged role for normal use cases.
@@@

1. [ ] In the **iothubowner** pane, copy the value for **Connection string--primary key**.

    Record this value, as you will need it below.

1. [ ] On the left hand navigation menu, under **Automatic Device Management**, click **IoT Edge**. 

    This pane allows you to manage the IoT Edge devices connected to the IoT Hub.

1. [ ] At the top of the pane, click **Add an IoT Edge device**.

1. [ ] On the **Create a device** blade, under **Device ID**,  enter **SimulatedDevice**

    This is the device identity used for authentication and access control.

1. [ ] Under **Authentication type**, ensure that **Symmetric key** is selected.

1. [ ] Leave the **Auto-generate keys** box checked.

    This will have IoT Hub automatically generate the Symmetric keys for authenticating the device.

1. [ ] Leave the other settings at their defaults, and then click **Save**.

1. [ ] Switch to the **Visual Studio Code** instance containing your IoT Edge solution.

1. [ ] In the **Explorer** pane, right-click **deployment.debug.template.json**, and then click **Build and Run IoT Edge Solution in Simulator**. 

    This file is the debugging deployment manifest file. It is located in the root directory of the IoT Edge Solution.

    When the process begins, you will see a dialog open in the lower right corner of the windows that says, **Please setup iotedgehubdev first before starting simulator**.

1. [ ] When you see the prompt to **setup iotedgehubdev**, click **Setup**. 

1. [ ] When prompted for the **IoT Hub Connection String**, enter the **Connection string--primary key** you noted earlier.

1. [ ] When prompted to **Select an IoT Edge Device**, click **SimulatedDevice**.
@@@warning
**Note**: If you get an **Unauthorized** error in the lower-right-hand corner, run the `Azure IoT Edge: Set IoT Hub Connection String` command from the Command Palette to reconfigure your simulator connection string, then run `Azure IoT Edge: Setup IoT Edge Simulator` from the command palette and try to select your device again.
@@@
@@@warning
**Note**: It is possible that you will be prompted for your Admin password on your local machine (in the Visual Studio Code **TERMINAL** window), particularly on Linux or macOS. Enter your password at the prompt and press **Enter**. The reason it might ask for your password is that the setup command for `iotedgehubdev` is being run using `sudo` as it requires elevated privileges.
@@@

    Once the IoT Edge Simulator is set up successfully, a **Setup IoT Edge Simulator successfully** message will be displayed in the Visual Studio Code TERMINAL.

    Now when you build and run the module in the IoT Edge Simulator, it will run as expected.

1. [ ] In the **Explorer** pane, right-click **deployment.debug.template.json**, and then click **Build and Run IoT Edge Solution in Simulator**.
@@@warning
**Note**: If you are on Windows and see a message in in **TERMINAL** that reads, in part, `open //./pipe/docker_engine: The system cannot find the file specified.`, Docker is likely not started, or running correctly.  A Docker restart or even a full computer restart might be necessary.
@@@
@@@warning
**Note**: If you see a message that reads, in part, `image operating system "linux" cannot be used on this platform`, change your Docker configuration to support Linux containers.  (Ask your instructor for assistance if necessary.)
@@@
@@@warning
**Note**: A build may take some time depending on what Docker images you have on your machine already and the speed of your Internet connection.  The build includes downloading Docker images if not present and updating container instances if necessary.
@@@

1. [ ] Observe the build process reporting in your TERMINAL window.

    It can take quite a few minutes to download and build everything that is required to simulate the IoT Edge device and run your modules, so be patient.

    Notice that once the **IoT Edge Simulator** is running, the Modules that you built will begin sending message output that is reported to the TERMINAL window:

    ```text
    SimulatedTemperatureSensor    |         12/09/2019 15:05:08> Sending message: 4, Body: [{"machine":{"temperature":23.023276334173641,"pressure":1.2304998355387693},"ambient":{"temperature":20.56235126408858,"humidity":24},"timeCreated":"2019-12-09T15:05:08.4596891Z"}]
    ObjectCountingModule           | Received message: 4, Body: [{"machine":{"temperature":23.023276334173641,"pressure":1.2304998355387693},"ambient":{"temperature":20.56235126408858,"humidity":24},"timeCreated":"2019-12-09T15:05:08.4596891Z"}]
    ObjectCountingModule           | Received message sent
    SimulatedTemperatureSensor    |         12/09/2019 15:05:13> Sending message: 5, Body: [{"machine":{"temperature":23.925331861560853,"pressure":1.3332656551145274},"ambient":{"temperature":20.69443827876562,"humidity":24},"timeCreated":"2019-12-09T15:05:13.4856557Z"}]
    ObjectCountingModule           | Received message: 5, Body: [{"machine":{"temperature":23.925331861560853,"pressure":1.3332656551145274},"ambient":{"temperature":20.69443827876562,"humidity":24},"timeCreated":"2019-12-09T15:05:13.4856557Z"}]
    ObjectCountingModule           | Received message sent
    ```

    Notice the output from the **ObjectCountingModule** contains the text `Received message: #` where `#` is the total message count that has been received by the custom **ObjectCountingModule** IoT Edge Module that was created.

1. [ ] With the IoT Edge Simulator still running, open the Azure portal, and then open the Cloud Shell.

1. [ ] At the Cloud Shell command prompt, to monitor the messages being sent to Azure IoT Hub from the `SimulatedDevice` running in the IoT Edge Simulator on your local machine, enter the following command:

    ```cmd/sh
    az iot hub monitor-events --hub-name "AZ-220-HUB-{YOUR-ID}"
    ```

    Be sure to replace the `AZ-220-HUB-{YOUR-ID}` value in the above command with the name of your Azure IoT Hub service.

1. [ ] Observe the output displayed in the Cloud Shell.

    With everything still running, notice the output of the previous command in the Cloud Shell will display a JSON representation of the messages being received by the Azure IoT Hub.

    The output should look similar to the following:

    ```json
    {
        "event": {
            "origin": "SimulatedDevice",
            "payload": "{\"machine\":{\"temperature\":88.003809452058647,\"pressure\":8.6333453806142764},\"ambient\":{\"temperature\":21.090260561364826,\"humidity\":24},\"timeCreated\":\"2019-12-09T15:16:32.402965Z\"}"
        }
    }
    {
        "event": {
            "origin": "SimulatedDevice",
            "payload": "{\"machine\":{\"temperature\":88.564600328362815,\"pressure\":8.6972329488008278},\"ambient\":{\"temperature\":20.942187817041848,\"humidity\":25},\"timeCreated\":\"2019-12-09T15:16:37.4355705Z\"}"
        }
    }
    ```

1. [ ] Compare the output in the Visual Studio Code TERMINAL with the output in the Cloud Shell.

    Notice that even though there are 2 IoT Edge Modules running on the IoT Edge Device that generate messages, there is still only a single copy of each message getting sent to Azure IoT Hub. The IoT Edge Device has a message pipeline defined where messages from the `SimulatedTemperatureSensor` are piped to the `ObjectCountingModule` which then sends messages out to the Azure IoT Hub.

1. [ ] To stop monitoring Azure IoT Hub events, press **Ctrl + C** within the Azure Cloud Shell.

1. [ ] Switch to your Visual Studio Code window.

1. [ ] On the left side toolbar, to open the Visual Studio Code debugger view, click **Run**.

    The Run button is the forth button down from the top, and includes a bug-like shape on the icon.

1. [ ] At the top of the **Run** pane, in the dropdown, ensure that **ObjectCountingModule Remote Debug (.NET Core)** is selected.

1. [ ] To the left of the dropdown, click **Start Debugging**.

    You could also press **F5** to start debugging

1. [ ] When prompted to **Select the process to attach to**, click **dotnet ObjectCountingModule.dll**.

1. [ ] On the left side toolbar, to change to the file Explorer view, click **Explorer**

1. [ ] In the EXPLORER pane, to open the `/modules/ObjectCountingModule/Program.cs` source code file, click **Program.cs**.

1. [ ] In the code editor, locate the `static async Task<MessageResponse> PipeMessage(` method.

1. [ ] Select the `static async Task<MessageResponse> PipeMessage(` code line, and then, to set a breakpoint, press **F9**.

    Visual Studio Code enables you to set a breakpoint within your code by clicking on the line and pressing the **F9** key.

1. [ ] Notice that execution stops at the breakpoint that is set, and the editor highlights that specific line of code.

1. [ ] To open the Visual Studio Code debugger view, on the left toolbar, click **Run**.

1. [ ] Notice the variables listed in the left panel.

1. [ ] To resume execution, click **Continue**.

    You can also press **F5** to resume.

1. [ ] Notice that each time the breakpoint it hit, execution stops.

1. [ ] To stop debugging, click the **Disconnect** button, or press **Shift + F5**.

1. [ ] To stop the IoT Edge Simulator, open the **Command Palette**, and then select the **Azure IoT Edge: Stop IoT Edge Simulator** option.

 Now that the module has been created and tested in the IoT Edge simulator, it is time to deploy it to the cloud.

#### Exercise 6: Deploy IoT Edge Solution

@@@secondary
In this exercise, you will build and publish the custom IoT Edge Module into the Azure Container Registry (ACR) service. Once published to ACR, the custom module will then be made available to be deployed to any IoT Edge Device.
@@@

1. [ ] Open the Visual Studio Code window containing your EdgeSolution project.

1. [ ] In the **Explorer** view, to open the `.env` file, click **.env**.

    The `.env` file is located in the root directory of the IoT Edge Solution. 

1. [ ] Ensure that the credentials for the Azure Container Registry have been set.

    When set correctly, the `CONTAINER_REGISTRY_USERNAME_<acr-name>` key will have it's value set to the Azure Container Registry service name, and the `CONTAINER_REGISTRY_PASSWORD_<acr-name>` key will have it's value set to the **Password** for the Azure Container Registry service. Keep in mind, the `<acr-name>` placeholders in the keys will be set to the ACR service name (is all lowercase) automatically when the IoT Edge Solution was created.

    The resulting `.env` file contents will look similar to the following:

    ```text
    CONTAINER_REGISTRY_USERNAME_az220acrcah191204=AZ220ACRCAH191204
    CONTAINER_REGISTRY_PASSWORD_az220acrcah191204=Q8YErJFCtqSe9C7BWdHOKEXk+C6uKSuK
    ```

1. [ ] In the **Explorer** view, right-click **deployment.template.json**, and then click **Build and Push IoT Edge Solution**.

    The status of the Build and Push IoT Edge Solution operation is displayed within the Visual Studio Code **TERMINAL** window. Once the process completes, the custom `ObjectCountingModule` IoT Edge Module will have been built, and then the Docker image for the IoT Edge Module will be published to the Azure Container Registry service.

1. [ ] Switch to your Azure portal window.

    If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] On your Resource group tile, to open your Azure Container Registry (ACR) service, click **AZ220ACR{YOUR-ID}**.

1. [ ] On the left side navigation menu, under **Services**, click **Repositories**.

1. [ ] On the **Repositories** pane, notice that the `objectcountingmodule` repository now exists within the ACR service.

    This was created when the custom `ObjectCountingModule` IoT Edge Module was published from within Visual Studio Code.
@@@warning
**Note**: If the repository is not present, review the output from the Push action and ensure that you did not leave references to the `:5000` port number with your ACR references; you can do an **Edit**, **Find in Files** to confirm this.  You should also validate your credentials in the `.env` file and validate that you performed the `docker login` step earlier in the lab.
@@@

1. [ ] Under **Repositories**, click **objectcountingmodule**.

1. [ ] On the **objectcountingmodule** blade, under **Tags**, notice that there is a tag named **0.0.1-amd64**.

1. [ ] To open a details pane for this Tag, click **0.0.1-amd64**.

    Notice the properties listed, including _Repository_, _Tag_, _Tag creation date_,  _Tag last updated date_, and other properties displaying information about the Tag.

1. [ ] Save a copy of the values for the **Repository** and **Tag** properties.

    To copy the values, you can click on the **Copy to clipboard** button located to the right of the displayed values.

    You will need the Repository and Tag names to pull down this specific version of the Docker image for the custom IoT Edge Module to run in an IoT Edge Device.

    The format of the Docker image Repository and Tag names combined will be in the following format:

    ```text
    <repository-name>:<tag>
    ```

    Here's an example of a full Docker image name for the `objectcountingmodule` IoT Edge Module:

    ```text
    objectcountingmodule:0.0.1-amd64
    ```

1. [ ] Navigate to your Azure IoT Hub resource.

    With the custom `objectcountingmodule` IoT Edge Module published to Azure Container Registry (ACR), the next step is to create a new IoT Edge Device within IoT Hub and configure it to run the new custom IoT Edge Module.

1. [ ] On the **AZ-220-HUB-{YOUR-ID}** blade, on the left side navigation menu under **Automatic Device Management**, click **IoT Edge**.

1. [ ] On the **IoT Edge** pane, at the top of the pane, click **Add an IoT Edge device**.

1. [ ] On the **Create a device** blade, under **Device ID**, enter **objectcountingdevice**

1. [ ] Under **Authentication type**, ensure that **Symmetric key** is selected, and ensure that the **Auto-generate keys** checkbox is selected.

    For this unit, we'll keep the IoT Edge Module registration simple by choosing _Symmetric key_ authentication. With the _Auto-generate keys_ option selected, the IoT Hub with automatically generate authentication keys for this device.

1. [ ] At the bottom of the blade, click **Save**.

1. [ ] On the **IoT Edge** pane, under **Device ID**, click **objectcountingdevice**.

1. [ ] At the top of the **objectcountingdevice** blade, click **Set Modules**.

1. [ ] On the **Set modules on device: objectcountingdevice** blade, under **Container Registry Settings**, enter the following values:

    * **Name**: Enter the **Registry name** of the Azure Container Registry (e.g. `az220acrcah191204`)
    * **Address**: Enter the **Login server** (or DNS name) of the Azure Container Registry service (ex: `az220acrcah191204.azurecr.io`)
    * **User Name**: Enter the **Username** for the Azure Container Registry service
    * **Password**: Enter the **password** for the Azure Container Registry service
</br>
@@@warning
**Note**: The Azure Container Registry (ACR) service _Registry name_, _Login server_, _Username_, and _Password_ can be found on the **Access keys** pane for the service.
@@@

1. [ ] On the **Set modules on device: objectcountingdevice** blade, under **IoT Edge Modules**, click **Add**, and then click **IoT Edge Module**.

1. [ ] On the **Add IoT Edge Module** pane, under **IoT Edge Module Name**, enter **objectcountingmodule**

1. [ ] Under **Module Settings**, to enter the full URI and tagged name of the Docker image for the custom IoT Edge Module, use the following format:

    ```text
    <container-registry-login-server>/<repository-name>:<tag>
    ```

    Be sure to replace the placeholders within the above **Image URI** format with the appropriate values:

    * `<container-registry-login-server>` * The **Login server**, or DNS name, for the Azure Container Registry service.
    * `<repository-name>` * The **Repository name** for the Custom IoT Edge Module's Docker image, that was copied previously.
    * `<tag>` * The **Tag** for the Custom IoT Edge Module's Docker image, that was copied previously.
</br>

    The resulting **Image URI** to be entered into the field will be similar to the following:

    ```text
    az220acrcah191204.azurecr.io/objectcountingmodule:0.0.1-amd64
    ```

1. [ ] Leave the rest of the settings at their defaults, and then click **Add**.

1. [ ] On the **Set modules on device: objectcountingdevice** blade, at the bottom of the blade, click **Next: Routes >**.

1. [ ] On the **Routes** tab, review the default route settings.

    The editor will display the configured default route for the IoT Edge Device. At this time, it should be configured with a route that sends all messages from all modules to Azure IoT Hub:

    * Name: **route**
    * Value: `FROM /messages/* INTO $upstream`
</br>
1. [ ] To the right of the default route, click **Remove route**.

1. [ ] Add the following two routes:

    | NAME | VALUE |
    | --- | --- |
    | `AllMessagesToObjectCountingModule` | `FROM /* INTO BrokeredEndpoint("/modules/objectcountingmodule/inputs/input1")` |
    | `ObjectCountingModuleToIoTHub` | `FROM /messages/modules/objectcountingmodule/outputs/* INTO $upstream` |

1. [ ] Review the value assigned to the **AllMessagesToObjectCountingModule** route.

    This route specifies the **Source** value of `/*`. This applies the route to all device-to-cloud messages or twin change notifications from any module or leaf device.

    This route specifies the **Destination** value of `BrokeredEndpoint("/modules/objectcountingmodule/inputs/input1")`. This sends all messages from the Source of this route to the `objectcountingmodule` IoT Edge Module's input.

1. [ ] Review the value assigned to the **ObjectCountingModuleToIoTHub** route.

    This route specifies the **Source** value of `/messages/modules/objectcountingmodule/outputs/*`. This applies the route to all messages output from the `objectcountingmodule` IoT Edge Module.

    This route specifies the **Destination** value of `$upstream`. This sends all messages from the Source of this route to the Azure IoT Hub service within Microsoft Azure.
@@@warning
**Note**:  For more information on the configuration of Message Routing with Azure IoT Hub and IoT Edge Module, reference the following links:
     * [Learn how to deploy modules and establish routes in IoT Edge](https://docs.microsoft.com/en-us/azure/iot-edge/module-composition)
     * [IoT Hub message routing query syntax](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-routing-query-syntax)
@@@

1. [ ] At the bottom of the blade, click **Next: Review + create >**.

1. [ ] Review the Deployment Manifest for the device, and then click **Create**.

 This completes the development of the `objectcountingmodule` custom IoT Edge Module. Now that an IoT Edge Device is registered, the modules specified and the routes configured, the `objectcountingmodule` is ready to be deployed once the associated IoT Edge Device is connected to the Azure IoT Hub as shown in previous labs.

@@@success
**Congratulations**! you have now completed this lab!
@@@
