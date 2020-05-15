### Lab 18: Detect if your IoT Device was Tampered with Azure Security Center for IoT'

@@@secondary
**Scenario**

Contoso has built all their solutions with security in mind. However, they want to see how they can better get a unified view of security across all of their on-premises and cloud workloads, including their Azure IoT solutions. Plus, when onboarding new devices, we want to apply security policies across workloads (Leaf devices, Microsoft Edge devices, IoT Hub) to ensure compliance with security standards and improved security posture.

Contoso is adding a brand new assembly line outfitted with new IoT devices to help with the increasing shipping and packing demands for new orders. You want to ensure that any new devices are secured and also want to be able to see security recommendations to continue improving your solution's security in your full end-to-end IoT solution. You will start investigating using Azure IoT Center for IoT for your solution.

Contoso is also installing new connected Thermostats to be able to monitor temperature across different cheese caves. As part of Contoso's security requirements, you will create a custom alert to monitor whether the Thermostats exceed expected telemetry transmission frequency.

We will be enabling the Azure Security Center for IoT to be able to see securely in our end to end IoT solutions. Tasks include:

* Verify Lab Prerequisites
* Create a new IoT Hub
* Enable Azure Security Center for IoT
* Create and register a new Device
* Create a Security Module Twin
* Install C#-based Security Agent on a Linux Device
* Configure Solution
* Configure custom alerts
* Create a console app to trigger the alert
* Review the alert in the Azure Security Center
@@@


#### Exercise 1: Verify Lab Prerequisites


#### Exercise 2: Create an IoT Hub using the Azure portal

In this task, you will use the Azure portal to create an IoT Hub resource.

1. [ ] Login to `https://portal.azure.com` using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Notice that the AZ-220 dashboard that you created in the previous task has been loaded.

    You will be adding resources to your dashboard as the course continues.

1. [ ] On the portal menu, click **+ Create a resource**.

    The Azure Marketplace is a collection of all the resources you can create in Azure. The marketplace contains resources from both Microsoft and the community.

1. [ ] In the Search textbox, type **IoT Hub** and then press Enter.

1. [ ] On the search results blade, click **IoT Hub**.

    Notice the "USEFUL LINKS" displayed on this blade.

1. [ ] In the list of links, click **Documentation**.

    The IoT Hub Documentation page is the root page for IoT Hub resources and documentation. You can use this page to explore current documentation and find tutorials and other resources that will help you to explore activities that are outside the scope of this course. We will refer you to the docs.microsoft.com site throughout this course for additional reading on specific topics.

1. [ ] Use your browser to navigate back to the Azure portal tab.

1. [ ] To begin the process of creating your new IoT Hub, click **Create**.

    Next, you need to specify information about the Hub and your subscription. The following steps walk you through the settings, explaining each of the fields as you fill them in.

1. [ ] On the IoT hub blade, on the **Basics** tab, ensure that the Azure subscription that you intend to use for this course is selected.

1. [ ] To the right of **Resource Group**, open the **Select existing** dropdown, and then click **AZ-220-RG**

    This is the resource group that you created in the previous lab. We will be grouping the resources that we create for this course together in the same resource group. This should help you to clean up your resources when you have completed the course.

1. [ ] To the right of **Region**, open the drop-down list and select the geographic location that is closest to you and also supports Event Grid.

    As we saw previously, Azure is supported by a series of datacenters that are placed in regions all around the world. When you create something in Azure, you deploy it to one of these datacenter locations.
@@@warning
**Note**:  For the current list of Regions that support Event Grid, see the following link: **Products available by region** `https://azure.microsoft.com/en-us/global-infrastructure/services/?products=event-grid&regions=all`
@@@
@@@warning
**Note**:  When picking a datacenter to host your app, keep in mind that picking a datacenter close to your end users will decrease load/response times. If you are on the other side of the world from your end users, you should not be picking the datacenter nearest you.
@@@

1. [ ] To the right of IoT Hub **Name**, enter a globally unique name for your IoT Hub.

    To provide a globally unique name, enter **AZ-220-HUB-_{YOUR-ID}_** (remember to replace **{YOUR-ID}** with the unique ID you created in Lab 1.).

    For example: **AZ-220-HUB-CAH102119**

    The name of your IoT hub must be globally unique because it is a publicly accessible resource that you must be able to access from any IP connected device.

    Consider the following when you specify a unique name for your new IoT Hub:

    * The value that you apply to _IoT Hub Name_ must be unique across all of Azure. This is true because the value assigned to the name will be used in the IoT Hub's connection string. Since Azure enables you to connect devices from anywhere in the world to your hub, it makes sense that all Azure hubs must be accessible from the Internet using the connection string and that connection strings must therefore be unique. We'll explore connection strings later in this lab.

    * The value that you assign to _IoT Hub Name_ cannot be changed once the app service has been created. If you do need to change the name, you'll need to create a new IoT Hub, re-register your devices to it, and delete your old IoT Hub.

    * The _IoT Hub Name_ field is a required field.
@@@warning
**Note**:  Azure will ensure that the name you enter is unique. If the name that you enter is not unique, Azure will display an asterisk at the end of the name field as a warning. You can append the name suggested above with '**-01**' or '**-02**' as necessary to achieve a globally unique name.
@@@

1. [ ] At the top of the blade, click **Size and scale**.

    Take a minute to review the information presented on this blade.

1. [ ] To the right of **Pricing and scale** tier, open the dropdown and then select **S1: Standard tier**.

    You can choose from several tier options depending on how many features you want and how many messages you send through your solution per day. The free tier is intended for testing and evaluation. It allows 500 devices to be connected to the IoT hub and up to 8,000 messages per day. Each Azure subscription can create one IoT Hub in the free tier.

    The **S1** tier that we are using in this course allows a total of 400,000 messages per unit per day and provides the all of the services that are required in this training. We won't actually need 400,000 messages per unit per day, but we will be using features provided at this tier level, such as Cloud-to-device commands, Device management, and IoT Edge. IoT Hub also offers a free tier that is meant for testing and evaluation. It has all the capabilities of the standard tier, but limited messaging allowances. However, you cannot upgrade from the free tier to either basic or standard.
@@@warning
**Note**:  The "S1 - Standard" tier has a cost of $25.00 USD per month per unit. We will be specifying 1 unit.
@@@

    For details about the other tier options, see **Choosing the right IoT Hub tier for your solution** `https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-scaling`

1. [ ] To the right of **Number of S1 IoT Hub units**, ensure that **1** is selected.

    As mentioned above, the pricing tier that you choose establishes the number of messages that your hub can process per unit per day. To increase the number of messages that you hub can process without moving to a higher pricing tier, you can increase the number of units. For example, if you want the IoT hub to support ingress of 700,000 messages, you choose *two* S1 tier units. For the IoT courses created by Microsoft we will be using just 1 unit.

1. [ ] Under **Advanced Settings**, ensure that **Device-to-cloud partitions** is set to **4**.

    The number of partitions relates the device-to-cloud messages to the number of simultaneous readers of these messages. Most IoT hubs will only need four partitions, which is the default value. For this course we will create our IoT Hub using the default number of partitions.

1. [ ] At the top of the blade, click **Review + create**.

1. [ ] At the bottom of the blade, to finalize the creation of your IoT Hub, click **Create**.

    Deployment can take a minute or more to complete. You can open the Azure portal Notification pane to monitor progress.

1. [ ] Notice that after a couple of minutes you receive a notification stating that your IoT Hub was successfully deployed to your **AZ-220-RG** resource group.

1. [ ] On the portal menu, click **Dashboard**, and then click **Refresh**.

    You should see that your Resource group tile lists your new IoT Hub.

#### Exercise 3: Enable Azure Security Center for IoT Hub

@@@secondary
Azure Security Center for IoT enables you to unify security management and enable end-to-end threat detection and analysis across hybrid cloud workloads and your Azure IoT solution.

Azure Security Center for IoT is composed of the following components:

* IoT Hub integration
* Device agents (optional)
* Send security message SDK
* Analytics pipeline
@@@

##### Task 1: Enable Azure Security Center for IoT Hub

You will enable the **Azure Security Center for IoT Hub**. 

1. [ ] On the Azure portal menu, click Dashboard and open your IoT Hub - **CheeseCaveHub-{YOUR-ID}**.

   You can also use the portal search bar by entering your IoT Hub name and then select your IoT Hub resource once it is listed.

1. [ ] Under **Security** menu on the left side, to onboard Azure Security Center for IoT Hub, click on any of the Security blades, such as **Overview**, and click **Secure your IoT solution**.

    After a few moments you may see the message **Onboarding succeeded for this IoT hub, please refresh for changes to take effect** - refresh the page.

##### Task 2: Log Analytics creation

When Azure Security Center for IoT is turned on, a default Azure Log Analytics workspace is created to store raw security events, alerts, and recommendations for your IoT devices, IoT Edge, and IoT Hub.

To change the workspace configuration of Log Analytics:

1. [ ] Open your IoT Hub and then select **Overview** from the Security menu.

1. [ ] At the top of the blade, to show the security settings, click **Settings**. 

    The **Settings** blade is shown, with the **Data Collection** pane displayed. Notice the workspace configuration of Log Analytics settings. They are already setup for you when you enabled Azure Security Center for IoT.

By default, turning on the Azure Security Center for IoT solution automatically secures all IoT Hubs under your Azure subscription.

#### Exercise 4: Create and Register a New Device

@@@secondary
You are now going to create a new IoT device that will later be used to measure vibrations on a new conveyor belt. For this lab, you will be using a VM to act like an IoT device.
@@@

##### Task 1: Create a new IoT Device

First, let's create the VM that will represent our device.

1. [ ] Login to **`https://portal.azure.com`** using your Azure account credentials.

1. [ ] On the portal menu, click **+ Create a resource**, then Search the Marketplace for **Ubuntu Server 18.04 LTS**

1. [ ] In the search results, select the **Ubuntu Server 18.04 LTS** item.

1. [ ] On the **Ubuntu Server 18.04 LTS** item, click **Create**.

1. [ ] On the **Create a virtual machine** blade, select your Azure Subscription and use the **Create new** Resource group option to create a new Resource Group for the VM named **vm-device01**.

1. [ ] In the **Virtual machine name** box, enter **vm-device01** for the name of the Virtual Machine.

1. [ ] In the **Region** dropdown, select the Azure Region closest to you, or the region where your Azure IoT Hub is provisioned.

1. [ ] In the **Availability options** dropdown, select **No infrastructure redundancy required**.
@@@info
**Tip**: Azure offers a range of options for managing availability and resiliency for your applications. Architect your solution to use replicated VMs in Availability Zones or Availability Sets to protect your apps and data from datacenter outages and maintenance events. In this lab, we don't required any high-availability features.
@@@

1. [ ] For **Azure Spot instance**, select **No**.
@@@info
**Tip**: Using Spot VMs allows you to take advantage of Azure unused capacity at a significant cost savings. At any point in time when Azure needs the capacity back, the Azure infrastructure will evict Spot VMs. Therefore, Spot VMs are great for workloads that can handle interruptions like batch processing jobs, dev/test environments, large compute workloads, and more. For this lab, we'll use a traditional VM.
@@@

1. [ ] Notice the **Image** dropdown has the **Ubuntu Server 18.04 LTS** image selected.

1. [ ] Under **Administrator account**, select the **Password** option for **Authentication type**.

1. [ ] Enter an Administrator **Username** and **Password** for the VM.

1. [ ] Notice the **Inbound port rules** is configured to enable inbound **SSH** access to the VM. This will be used to remote into the VM to configure/manage it.

1. [ ] Click **Review + create**.

1. [ ] Once validation passes, click **Create** to begin deploying the virtual machine.
@@@warning
**Note**: Deployment will take approximately 5 minutes to complete. You can continue on to the next unit while it is deploying.
@@@

##### Task 2: Register New Devices

As a device must be registered with your IoT hub before it can connect, let's create the registration.

1. [ ] On the Azure portal menu, click **Dashboard** and open your IoT Hub. You can also in the portal search bar type in your IoT Hub name and select your IoT Hub resource once it pops up.

1. [ ] In the left navigation menu, under **Explorers**, click **IoT devices**.

1. [ ] On the top of IoT devices blade, Click **+ New**

1. [ ] Under **Device ID**, enter **vm-device01**

1. [ ] As we are going to use **symmetric Keys** for authentication, leave the other defaults.

1. [ ] Click **Save**.

#### Exercise 5: Create a Security Module Twin

@@@secondary
Azure Security Center for IoT offers full integration with your existing IoT device management platform, enabling you to manage your device security status as well as make use of existing device control capabilities. Azure Security Center for IoT integration is achieved by making use of the IoT Hub twin mechanism.

Azure Security Center for IoT makes use of the module twin mechanism and maintains a security module twin named azureiotsecurity for each of your devices.

The security module twin holds all the information relevant to device security for each of your devices.

To make full use of Azure Security Center for IoT features, you'll need to create, configure and use these security module twins for your new IoT Edge device.

The security module twin **azureiotsecurity** can be created in two ways:

* [Module batch script](https://github.com/Azure/Azure-IoT-Security/tree/master/security_module_twin) - automatically creates module twin for new devices or devices without a module twin using the default configuration.
* Manually editing each module twin individually with specific configurations for each device.

In this task, you will be creating a security module twin manually.
@@@

1. [ ] Navigate to your new IoT device if you are not there already:

   1. On the Azure portal menu, click Dashboard and open your IoT Hub. You can also in the portal search bar type in your IoT Hub name and select your IoT Hub resource once it pops up.

   1. In your IoT Hub, locate **IoT devices** under **Explorers**.

1. [ ] Click on **vm-device01**.

1. [ ] Click on **+ Add Module Identity**.

1. [ ] In the Module Identity Name field, enter **azureiotsecurity**.

1. [ ] As we will be using Symmetric Keys for authentication, leave all over fields the same and click **Save**.

    You should now see **azureiotsecurity** under **Module Identities** for your device. The connection state is **Disconnected**.
@@@danger
**IMPORTANT**: The Module Identity must be called **azureiotsecurity** and not another unique name.
@@@

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/0ebf66bc-68f4-4864-8f31-14abc6c01604.png)

1. [ ] While you are viewing the **vm-device01** information, copy the device's **Primary Key** to use later.
@@@warning
**Note**: Make sure to copy the device's **Primary Key** and not the connection string.
@@@

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/0269e5ed-8ae9-4bfe-9bbc-b544fdb22ed6.png)

1. [ ] Navigate back to your IoT Hub, click on **Overview**.  Copy your IoT Hub hostname.
@@@warning
**Note**: Example of what an IoT Hub hostname looks like: AZ-220-HUB-CAH102119.azure-devices.net
@@@

#### Exercise 6: Deploy Azure Security Center for IoT C# Security Agent

@@@secondary
Azure Security Center for IoT provides reference architecture for security agents that log, process, aggregate, and send security data through IoT Hub. You will be adding a security agent for C# to deploy on your simulated device (Linux VM). There are C and C# based agents. C agents are recommended for devices with more restricted or minimal device resources.

Security agents support the following features:

* Collect raw security events from the underlying Operating System (Linux, Windows). To learn more about available security data collectors, see Azure Security Center for IoT agent configuration.
* Aggregate raw security events into messages sent through IoT Hub.
* Authenticate with existing device identity, or a dedicated module identity. See Security agent authentication methods to learn more.
* Configure remotely through use of the **azureiotsecurity** module twin. To learn more, see Configure an Azure Security Center for IoT agent.
@@@

##### Task 1: Logging into IoT Device - Linux VM

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Navigate to your newly created virtual machine (**vm-01**)  within the Azure Portal.

1. [ ] On the **Overview** pane of the **Virtual machine** blade, click the **Connect** button at the top.

1. [ ] On the **Overview** pane of the **Virtual machine** blade, look for the **Public IP address**. Use this value, along with the name of the Administrator account you created earlier to create the login command in a text editor:

    ```cmd\sh
    ssh <admin user>@<ip address>
    ```

    The command is formatted similar to `ssh demouser@52.170.205.79`.

1. [ ] At the top of the Azure Portal click on the **Cloud Shell** icon to open up the **Azure Cloud Shell** within the Azure Portal. When the pane opens, choose the option for the **Bash** terminal within the Cloud Shell.

1. [ ] Within the Cloud Shell, paste (or type) the `ssh` command you built earlier, and press **Enter**.

1. [ ] When prompted with **Are you sure you want to continue connecting?**, type `yes` and press Enter. This prompt is a security confirmation since the certificate used to secure the connection to the VM is self-signed. The answer to this prompt will be remembered for subsequent connections, and is only prompted on the first connection.

1. [ ] When prompted to enter the password, enter the Administrator password that was entered when the VM was provisioned.

1. [ ] Once connected, the terminal will change to show the name of the Linux VM, similar to the following. This tells you which VM you are connected to and the current user.

    ```cmd/sh
    demouser@vm-device01:~$
    ```

##### Task 2: Install Linux prerequisites

@@@secondary
Every Azure Security Center for IoT security agent flavor offers the same set of features, and supports similar configuration options. For the C#-based security agent we need **auditd** installed.
@@@

1. [ ] Install auditd (The Linux Audit daemon) on your Linux VM.

    ```bash
    sudo apt-get install auditd audispd-plugins
    ```

##### Task 3: Add Symmetric Keys to your device

With the C# agent you will be connecting to your IoT Hub. This means you will need your device's symmetric key or certificate information. For this lab, you will be using the symmetric key as authentication and will store in a temporary text document on the device. To do this you will need to do the following:

1. [ ] Open the Azure Portal in a new browser tab.

1. [ ] On the Azure portal menu, click **Dashboard** and open your IoT Hub.

   You can also use the portal search bar to type in your IoT Hub name and select your IoT Hub resource once it pops up.

1. [ ] In your IoT Hub, under **Explorers**, click **IoT devices**.

1. [ ] Click on **vm-device01**.

1. [ ] From the list of details, copy your **Primary Key**.
@@@warning
**Note**: Copy the primary key, not the primary key connection string.
@@@

1. [ ] Return the the Azure Cloud Shell browser tab - you should still be connected to your **vm-device01** virtual machine.

1. [ ] Create device Authentication type file with your **vm-device01** device's **Primary Key**.

    ```cmd/sh
    echo "<primary_key>" > s.key
    ```
@@@warning
**Note**: To check if you added the correct Primary key into the file, Open your file with `nano s.key` command. Check to see your device's **Primary Key** is in the file. To exit the nano editor, holding `Ctrl` and `X`. Save file by holding `shift` and `Y`. Then hit enter.
@@@

##### Task 4: Installing Security Agent

1. [ ] Download the recent version of Security Agent for C# to your device.

    ```bash
    wget https://github.com/Azure/Azure-IoT-Security-Agent-CS/releases/download/0.0.6/ubuntu-18.04-x64.tar.gz
    ```

1. [ ] Extract the contents of the package and navigate to the `/Install` folder by entering the following command:

    ```bash
    tar -xzvf ubuntu-18.04-x64.tar.gz && cd Install
    ```

1. [ ] Add execute permissions to the `InstallSecurityAgent` script by running the following command:

    ```bash
    chmod +x InstallSecurityAgent.sh
    ```

1. [ ] Next, run the following command with root privileges. You will need to replace the values with your authentication parameters.

    ```bash
    sudo ./InstallSecurityAgent.sh -i -aui Device -aum SymmetricKey -f <Insert file location of your s.key file> -hn <Insert your full IoT Hub host name> -di vm-device01
    ```

    An example of what the command would look like. Please make sure you swap out for your IoT Hub hostname instead: `sudo ./InstallSecurityAgent.sh -i -aui Device -aum SymmetricKey -f ../s.key -hn CheeseCaveHub-AB20200213.azure-devices.net -di vm-device01`
@@@danger
**IMPORTANT**: Ensure you use the full IoT Hub host name - i.e. **CheeseCaveHub-AB20200213.azure-devices.net** for the `-hn` switch value.
@@@

    This script performs the following function:

    * Installs prerequisites.
    * Adds a service user (with interactive sign in disabled).
    * Installs the agent as a Daemon - assumes the device uses **systemd** for service management.
    * Configures **sudoers** to allow the agent to do certain tasks as root.
    * Configures the agent with the authentication parameters provided.
</br>
1. [ ] A reboot is required to complete agent installation. Type in **"y"** for yes when asked `Do you wish to do it now?`

1. [ ] Within the Cloud Shell, reconnect to your virtual machine with the SSH command you used earlier.

1. [ ] Check the deployment of the Azure Security Center for IoT Agent status by running the following command. Your Azure Security Center for IoT Agent should now be active and running.

    ```cmd/sh
    systemctl status ASCIoTAgent.service
    ```

    You should see output similar to:

    ```log
    ● ASCIoTAgent.service - Azure Security Center for IoT Agent
       Loaded: loaded (/etc/systemd/system/ASCIoTAgent.service; enabled; vendor preset: enabled)
       Active: active (running) since Wed 2020-01-15 19:08:15 UTC; 3min 3s ago
     Main PID: 1092 (ASCIoTAgent)
        Tasks: 7 (limit: 9513)
       CGroup: /system.slice/ASCIoTAgent.service
            └─1092 /var/ASCIoTAgent/ASCIoTAgent
    ```

    Specifically, you should check that the service is **Loaded: loaded** and **Active: active (running)**.
@@@warning
**Note**: If your Azure Security Center for IoT Agent isn't running or active, please check out [Deploy Azure Security Center for IoT C# based security agent for Linux Guide Troubleshooting Section](https://docs.microsoft.com/en-us/azure/asc-for-iot/how-to-deploy-linux-cs). Common issues are that might leave the service **Active: activating** are an incorrect key value or not specifying the full IoT Hub hostname.
@@@

1. [ ] Navigate back to the Azure portal to your  **vm-device01**. To do that, go in your IoT Hub, locate IoT devices under Explorers. Click on **vm-device01**.

1. [ ] Notice that your **azureiotsecurity** Module is now in a **Connected** state.

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/7e779014-cded-45f0-9ea4-116fd4f84eee.png)

Now that your Azure Security Center for IoT device agents on your device is installed, the agents will be able to collect, aggregate and analyze raw security events from your device.

#### Exercise 7: Configure Solution Management

@@@secondary
Azure Security Center for IoT provides comprehensive end-to-end security for Azure-based IoT solutions.

With Azure Security Center for IoT, you can monitor your entire IoT solution in one dashboard, surfacing all of your IoT devices, IoT platforms and back-end resources in Azure.
Once enabled on your IoT Hub, Azure Security Center for IoT automatically identifies other Azure services, also connected to your IoT Hub and related to your IoT solution.

In addition to automatic relationship detection, you can also pick and choose which other Azure resource groups to tag as part of your IoT solution. Your selections allow you to add entire subscriptions, resource groups, or single resources.
@@@

##### Task 1: Open IoT Hub

1. [ ] In your browser, open the Azure Portal and navigate to your IoT Hub.

1. [ ] In the left navigation areas, under **Security**, click **Resources**.

    Note that the list of resources already includes the IoT Hub, the workspace that was created when Azure Security Center for IoT was activated earlier as well as the current subscription.

1. [ ] At the top of the pane, click **Edit**.

    The **Solution Management** pane opens, where you can connect additional Azure resources to your security solution by selecting their owning resource groups.

1. [ ] Under **Subscriptions**, select your current subscription.
@@@warning
**Note:** You can add resources from multiple subscriptions to your security solution.
@@@

1. [ ] Under **Resource groups**, select the resource group for your VM - **vm-device01**.

1. [ ] Click **Apply** - you will need to manually close the **Solution Management** pane.

    Note that the list has updated to reflect resources in the resource group you just added.

 After defining all of the resource relationships, Azure Security Center for IoT leverages Azure Security Center to provide you security recommendations and alerts for these resources.

##### Task 2: View Azure Security Center for IoT in Action

@@@secondary
Now that you have your the security agent installed on your device and your solution configured, take some time to check out the different views for Azure Security Center for IoT.
@@@

1. [ ] Navigate **Overview** under Security. 

   You will see start to see an overview of the health of your devices, hubs, and other resources. You can see the Built-in real-time monitoring, recommendations and alerts that were enabled right when you turn on your Azure IoT Security Center.

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/042f2b48-7518-4e09-8822-a6173222ae40.png)

1. [ ] Navigate **Resources** under Security, to see the health of your resources across your IoT solution.
@@@danger
**IMPORTANT**: The process that evaluates the security configuration of your IoT resources may take up to 24 hours to run, therefore the initial status displayed on the dashboard does not reflect the actual state of your resources. The image below shows the dashboard status once the security evaluation has been performed.
@@@

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/36c3dc07-5f27-445b-88c4-1d8d07dc0ca5.png)

#### Exercise 8: Introduce custom alerts

@@@secondary
Using custom security groups and alerts, takes full advantage of the end-to-end security information and categorical device knowledge to ensure better security across your IoT solution. 
@@@

#### Why use custom alerts?

@@@secondary
You know your IoT devices best.

For customers who fully understand their expected device behavior, Azure Security Center for IoT allows you to translate this understanding into a device behavior policy and alert on any deviation from expected, normal behavior. 
@@@

##### Task 1 - Customize an alert

@@@secondary
As mentioned earlier, customers that understand the specific desired behavior of their solution can configure custom alerts that trigger should the desired behavior exceed expectations. In this unit, you will create a custom alert that monitors **Device to Cloud** messages sent via the **MQTT** protocol.

In our cheese cave scenario, we don't expect our temperature monitor devices to be flooding our solution with telemetry every second. In this scenario, we will expect each device to send at least one and no more than five cloud to device messages in five minutes.
@@@

1. [ ] Login to **`https://portal.azure.com`** using your Azure account credentials and navigate to the IoT Hub you have been using for this module.

1. [ ] Under **Security** in the left navigation area, click **Custom Alerts**.

    Security groups enable you to define logical groups of devices, and manage their security state in a centralized way. These groups can represent devices with specific hardware, devices deployed in a certain location, or any other group suitable to your specific needs.
@@@warning
**Note**: The **default** group is created for you automatically.
@@@

1. [ ] To add a custom alert to the default group, click **default**.

    The **Device Security Group** blade will list all active custom alerts - if this is the first time you have visited this blade, it will be empty.

1. [ ] At the top of the blade, click **Add a custom alert rule**.

    The **Create custom alert rule** pane will open. You will note that the **Device Security Group** field is populated with the **default** group.

1. [ ] In the **Custom Alert** dropdown, select a **Number of device to cloud messages (MQTT protocol) is not in allowed range**.
@@@info
**Tip**: Review the the many custom alerts available. Consider how they can be used to secure your solution.
@@@
@@@warning
**Note**: The **Description** and **Required Properties** will change depending upon the **Custom Alert** that is chosen.
@@@

1. [ ] Under **Required Properties**, in the **Minimal Threshold** field, enter **1**.

    This satisfies the scenario's "at least one message" requirement.

1. [ ] Under **Maximal Threshold**, enter **5**.

    This satisfies the scenario's "no more than five" requirement.

1. [ ] In the **Time Window Size** dropdown, select **00:05:00**.

    This satisfies the scenario's "in five minutes" requirement.
@@@warning
**Note**: There are 4 available time windows:</br>
    * 5 minutes
    * 10 minutes
    * 15 minutes
    * 30 minutes
@@@

1. [ ] Make sure to click **SAVE**. Without saving the new alert, the alert is deleted the next time you close IoT Hub.

    You will be returned to the list of custom alerts. Below is an image that shows a number of custom alerts:

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/6d849abf-0a85-4584-beb9-cff19a5517fd.png)

#### Exercise 9: Configure the Device App

@@@secondary
In the previous unit, a Custom alert was created, In this unit, you will create an IoT Hub device and a C# .Net Core console application that will leverage the **Microsoft.Azure.Devices.Client** nuget package to connect to an IoT Hub. The console application will send telemetry every 10 seconds and is designed to exceed the Device to Cloud message threshold configured in the Custom Alert.
@@@

##### Task 1: Register New IoT Device

A device must be registered with your IoT hub before it can connect.

1. [ ] On the Azure portal menu, click **Dashboard** and open your IoT Hub. You can also in the portal search bar type in your IoT Hub name and select your IoT Hub resource once it pops up.

1. [ ] In the left navigation menu, under **Explorers**, select and open **IoT devices**.

1. [ ] On the top of IoT devices blade, Click  **+ New**

1. [ ] Type in **CheeseCave1-Sensor1** under **Device ID**. As we are going to use **symmetric Keys** for authentication, leave the other defaults.

1. [ ] Click **Save**.

1. [ ] Once the list of devices has updated, to see the device details, click **CheeseCave1-Sensor1**.

1. [ ] Copy the device **Primary connection string** - you will need it for the device app.

##### Task 2: Create a Console App

1. [ ] In Visual Studio Code, select **File > Open Folder**.

1. [ ] Create a new folder named **ThermostatDevice** in the location of your choice, and then click **Select Folder**.

1. [ ] Ensure that file auto-save is enabled by clicking on the **File** menu and checking **Auto Save** if it is blank. You will be copying in several blocks of code, and this will ensure you are always operating against the latest edits of your files.

1. [ ] Open the integrated terminal from Visual Studio Code by selecting **View > Terminal** from the main menu.

1. [ ] In the terminal window, copy and paste the following command.

    ```bash
    dotnet new console
    ```

    This command creates a **Program.cs** file in your folder with a basic "Hello World" program already written, along with a C# project file named **ThermostatDevice.csproj**.

1. [ ] In the terminal window, copy and paste the following command to run the "Hello World" program.

    ```bash
    dotnet run
    ```

    The terminal window displays "Hello world!" as output.

##### Task 3: Connect the App to IoT Hub

1. [ ] At the terminal prompt, copy and paste the following command to install the required NuGet package.

    ```bash
    dotnet add package Microsoft.Azure.Devices.Client
    ```

1. [ ] At the top of the Explorer pane, click **Program.cs** to open the file.

1. [ ] Delete all of the existing content.

1. [ ] Add the following using statements:

    ```csharp
    using System;
    using Microsoft.Azure.Devices.Client;
    using Newtonsoft.Json;
    using System.Text;
    using System.Threading.Tasks;
    ```

1. [ ] Create a namespace for the application by adding the following beneath the using statements:

    ```csharp
    namespace ThermostatDevice
    {
        // add class below here
    }
    ```

1. [ ] To create the **ThermostatDevice** class, add the following code below the comment `// add class below here`:

    ```csharp
    class ThermostatDevice
    {
        // add variables below here
    }
    ```

1. [ ] Add the following variables to contain a reference to the `DeviceClient` and store the device connection string you copied earlier below the `// add variables below here` comment:

    ```csharp
    private static DeviceClient s_deviceClient;
    private readonly static string s_connectionString = "<DEVICE-CONNECTION-STRING>";

    // add SendDeviceToCloudMessagesAsync method below here
    ```
@@@info
**Tip**: Replace `<DEVICE-CONNECTION-STRING>` with the device primary connection string you copied from the Azure Portal earlier.
@@@

1. [ ] To send device to cloud messages to the IoT Hub, add the following method below the `// add SendDeviceToCloudMessagesAsync method below here` comment:

    ```csharp
    private static async void SendDeviceToCloudMessagesAsync()
    {
        // Initial telemetry values
        double minTemperature = 20;
        double minHumidity = 60;
        Random rand = new Random();

        // loop until the user hits CTRL+C
        while (true)
        {
            double currentTemperature = minTemperature + rand.NextDouble() * 15;
            double currentHumidity = minHumidity + rand.NextDouble() * 20;

            // Create JSON message using an anonymous type
            var telemetryDataPoint = new
            {
                temperature = currentTemperature,
                humidity = currentHumidity
            };
            var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
            var message = new Message(Encoding.ASCII.GetBytes(messageString));

            // Add a custom application property to the message.
            // An IoT hub can filter on these properties without access to the message body.
            message.Properties.Add("temperatureAlert", (currentTemperature > 30) ? "true" : "false");

            // Send the telemetry message
            await s_deviceClient.SendEventAsync(message);
            Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, messageString);

            // wait 10 seconds before sending the next message
            await Task.Delay(10000);
        }
    }

    // add main method below here
    ```

    Consider the `await Task.Delay(10000);`  code - a message is being sent every 10 seconds and this will definitely exceed the "no more than 5 messages in 5 minutes" threshold for the custom alert.
@@@info
**Tip**: Read the code comments to understand how the method works.
@@@

1. [ ] To implement the main method that is executed when the app is launched, add the following code below the `// add main method below here` comment:

    ```csharp
    private static void Main(string[] args)
    {
        Console.WriteLine("IoT Hub C# Simulated Thermostat Device. CTRL+C to exit.\n");

        // Connect to the IoT hub using the MQTT protocol
        s_deviceClient = DeviceClient.CreateFromConnectionString(s_connectionString, TransportType.Mqtt);
        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();
    }
    ```
@@@warning
**Note**: When the `DeviceClient` instance is created, the **MQTT** protocol is specified with `TransportType.Mqtt` - this ensures that the Device to Cloud messages are sent using the protocol specified in the custom alert created.
@@@

1. [ ] To build and run the application, enter the following command:

    ```bash
    dotnet run
    ```

    The output will be similar to:

    ```text
    IoT Hub C# Simulated Thermostat Device. CTRL+C to exit.

    2/28/2020 1:32:21 PM > Sending message: {"temperature":25.7199231282435,"humidity":79.50078555359542}
    2/28/2020 1:32:31 PM > Sending message: {"temperature":21.877205091005752,"humidity":61.30029373862794}
    2/28/2020 1:32:41 PM > Sending message: {"temperature":21.245898961204055,"humidity":71.36471955634873}
    2/28/2020 1:32:51 PM > Sending message: {"temperature":32.61750500072609,"humidity":66.07430422961447}
    2/28/2020 1:33:01 PM > Sending message: {"temperature":31.100763578946125,"humidity":79.93955616836416}
    2/28/2020 1:33:11 PM > Sending message: {"temperature":25.02041019034591,"humidity":70.50569472392355}
    ```

    You can leave the app running for the remainder of this module to generate multiple alerts.

#### Exercise 10: Review Security Center Alerts

@@@secondary
In the previous unit, a console application was created an executed. That console app will have sent too much telemetry and will trigger the Custom Alert created earlier. 
@@@
@@@info
**Tip**: The alert was set up to trigger if less than 1 and more than 5 messages where sent from a device to the cloud within a 5 minute time window.
@@@

##### Task 1: Review the Security Center Dashboard

1. [ ] On the Azure portal menu, click **Dashboard** and open your IoT Hub. You can also in the portal search bar type in your IoT Hub name and select your IoT Hub resource once it pops up.

1. [ ] Under **Security** in the left navigation area, click **Overview**. Take a look at the **Threat detection** section. You should see one or more **Low severity** alerts displayed in the **Device security alerts** chart:

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/34769e19-dd53-47a7-b30e-9512925efd0e.png)

    You should also see an entry for the **CheeseCave1-Sensor1** device in the **Devices with the most alerts** tile:

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/f343a450-1ebf-4c4c-835b-93fa03e60631.png)
@@@warning
**Note**: It can take between 10 and 15 minutes for alerts to be displayed on the dashboard.
@@@

1. [ ] To display the **Security Alerts** details, you can click on either the **Device security alert chart** or the **Devices with the most alerts tile**, or, under **Security** in the left navigation area, click **Security Alerts**.

    You will see a list of Security Alerts:

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/eb9db6b8-2c1b-4221-81ab-b4031b785138.png)

    The latest alerts will be marked with a **NEW** label.

1. [ ] Click the latest alert and a detail pane will open. The **General information** provides high-level information concerning the alert. Beneath this, the **Last 10 Affected Devices** should list the **CheeseCave1-Sensor1** device.

    ![Screenshot](https://godeployblob.blob.core.windows.net//labguideimages/AZ-220T00/All-Labs/67631857-4f05-4e94-9918-7d2fc449ba4b.png)
