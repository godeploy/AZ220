### Lab 2 - Getting Started with Azure IoT Services

@@@secondary
**Lab Scenario**

You are an Azure IoT Developer working for Contoso, a company that crafts and distributes gourmet cheeses. You have been tasked with exploring Azure and the Azure IoT services that will be used in the IoT solution that the company is planning to implement. You have become familiar with the Azure portal and created a resource group for your project. Now you need to begin your investigation of Azure IoT services.

**In This Lab**

In this lab, you will begin exploring the Azure IoT services that will be used to provision and connect IoT devices for your solution (IoT Hub and IoT Hub Device Provisioning Service). The lab includes the following exercises:

* Naming Resources with Unique Names
* Create an IoT Hub using the Azure portal
* Examine features of the IoT Hub service
* Create a Device Provisioning Service and link it to your IoT Hub
* Examine features of the Device Provisioning Service
@@@

#### Exercise 1: Naming Resources with Unique Names

@@@secondary
Throughout this course you will be creating resources. To ensure consistency across the labs and to help in tidying up resources whenever you have finished with them, we will be providing you with the names you should use. However, many of these resources expose services that can be consumed across the web, which means they must have globally unique names. To achieve this, you will be using a unique identifier that will be added to the end of the resource name. Let's create your unique ID.
@@@

**Unique ID**

Your unique ID will be constructed using your initials and the current date using the following pattern:

```text
YourInitialsYYMMDD
```

So, your initials followed by the last two digits of the current year, the current numeric month, and the current numeric day. Here are some examples:

```text
GWB190123
BHO190504
CAH191216
DM190911
```

In some cases, you may be asked to use the lowercase version of your unique ID:

```text
gwb190123
bho190504
cah191216
dm190911
```

Whenever you are expected to use your unique ID, you will see `{YOUR-ID}`. You will replace the entire string (including the `{}`) with your unique value.

Make a note of your unique ID now and **use the same value through the entire course** - don't update the date each day.

Let's review some examples of resources and the names associated with them.

#### Resource Groups

A resource group must have a unique name within a subscription; however, it does not need to be globally unique. Therefore, throughout this course you will be using the resource group name: **AZ-220-RG**.

  > **Information:** Resource Group Name - **AZ-220-RG**

#### Publicly Visible Resources

Many of the resources that you create will have publicly-addressable (although secured) endpoints and therefore must have globally unique. Examples of such resources include IoT Hubs, Device Provisioning Services, and Azure Storage Accounts. For each of these you will be provided with a name template and expected to replace `{YOUR-ID}` with your unique ID. Here are some examples:

If your Unique ID is: **CAH191216**

| Resource Type | Name Template | Example |
| :--- | :--- | :--- |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ | AZ-220-HUB-CAH191216 |
| Device Provisioning Service | AZ-220-DPS-_{YOUR-ID}_ | AZ-220-DPS-CAH191216 |
| Azure Storage Account (name must be lower-case and no dashes) | az220storage_{YOUR-ID}_ | az220storagecah191216 |

You may also be required to update values within bash scripts and C# source files as well as entering the names into the Azure Portal UI. Here are some examples:

```bash
#!/bin/bash

YourID="{YOUR-ID}"
RGName="AZ-220-RG"
IoTHubName="AZ-220-HUB-$YourID"

```

Notice that `YourID="{YOUR-ID}"` should be updated to `YourID="CAH191216"` - you do not change `$YourID`. Similarly, in C# you might see:

```csharp
private string _yourId = "{YOUR-ID}";
private string _rgName = "AZ-220-RG";
private string _iotHubName = $"AZ-220-HUB-{_yourId}";
```

Again, `private string _yourId = "{YOUR-ID}";` should be updated to `private string _yourId = "CAH191216";` - you do not change `_yourId`.

#### Exercise 2: Create an IoT Hub using the Azure portal

@@@secondary
The Azure IoT Hub is a fully managed service that enables reliable and secure bidirectional communications between IoT devices and Azure. The Azure IoT Hub service provides the following:

* Establish bidirectional communication with billions of IoT devices
* Multiple device-to-cloud and cloud-to-device communication options, including one-way messaging, file transfer, and request-reply methods.
* Built-in declarative message routing to other Azure services.
* A queryable store for device metadata and synchronized state information.
* Secure communications and access control using per-device security keys or X.509 certificates.
* Extensive monitoring for device connectivity and device identity management events.
* SDK device libraries for the most popular languages and platforms.

There are several methods that you can use to create an IoT Hub. For example, you can create an IoT Hub resource using the Azure portal, which is what you will do in ths task. But you can also create an IoT Hub (and other resources) programmatically. During this course we will be investigating additional methods that can be used to to create and manage Azure resources, including Azure CLI and PowerShell.
@@@

#### Task 1: Use the Azure portal to create a resource (IoT Hub)

1. [ ] Login to the Azure Portal using your Azure account credentials. 

 ```cli
https://portal.azure.com
 ```

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.  You may find it easiest to use an InPrivate / Incognito browser session to avoid accidentally using the wrong account.

1. [ ] Notice that the AZ-220 dashboard that you created in the previous task has been loaded.

    You will be adding resources to your dashboard as the course continues.

1. [ ] On the Azure portal menu, click **+ Create a resource**.

    The Azure Marketplace is a collection of all the resources you can create in Azure. The marketplace contains resources from both Microsoft and the community.

1. [ ] In the Search textbox, type **IoT** and then press **Enter**.
@@@warning
**Note**: Marketplace services provided by private contributors may include a cost that is not covered by a Microsoft Azure Pass or other Microsoft credit offering.
@@@

1. [ ] On the search results blade, click **IoT Hub**.

    Notice the list of _Useful Links_ displayed on this blade.

1. [ ] In the list of links, notice that there is a link to Documentation.

    No need to explore this documentation now, but it is worth noting. The _IoT Hub Documentation_ page is the root page for IoT Hub resources and documentation. You can use this page to explore current documentation and find tutorials and other resources that will help you to explore activities that are outside the scope of this course. We will refer you to the docs.microsoft.com site throughout this course for additional reading on specific topics.

    If you did open the documentation link, use your browser to close the documentation tab and navigate back to the Azure portal tab.

##### Task 2: Create an IoT Hub with required property settings

1. [ ] To begin the process of creating your new IoT Hub, click **Create**.

@@@info
**Tip:** In the future, there are two other ways to get to the **Create** experience of any Azure resource type:

1. If you have the service in your Favorites, you can click the service to navigate to the list of instances, then click the **+ Add** button at the top.
        
2. You can search for the service name in the **Search** box at the top of the portal to get to the list of instances, then click the **+ Add** button at the top.
@@@

Next, you need to specify information about the Hub and your subscription. The following steps walk you through the settings, explaining each of the fields as you fill them in.

1. [ ] On the _IoT hub_ blade, on the _Basics_ tab, ensure that the Azure **Subscription** that you intend to use for this course is selected.

1. [ ] To the right of **Resource Group**, open the **Select existing** dropdown, and then click **AZ-220-RG**

    This is the resource group that you created in the previous lab. We will be grouping the resources that we create for this course together in the same resource group. This should help you to clean up your resources when you have completed the course.

1. [ ] To the right of **Region**, open the drop-down list and select the same region as your resource group.  Make sure it supports Event Grid.

    As we saw previously, Azure is supported by a series of datacenters that are placed in regions all around the world. When you create something in Azure, you deploy it to one of these datacenter locations.
@@@warning
**Note**:  For the current list of regions that support Event Grid, see the following link: [Products available by region](https://azure.microsoft.com/en-us/global-infrastructure/services/?products=event-grid&regions=all)
@@@
@@@warning
**Note**:  When picking a region to host your app, keep in mind that picking a region close to your end users will decrease load/response times. If you are on the other side of the world from your end users, you should not be picking the region nearest you.
@@@

1. [ ] To the right of **IoT Hub Name**, enter a globally unique name for your IoT Hub.

    To provide a globally unique name, enter **AZ-220-HUB-_{YOUR-ID}_** (remember to replace **_{YOUR-ID}_** with the unique ID you created in Lab 1.).

    For example: **AZ-220-HUB-CAH191021**

    The name of your IoT Hub must be globally unique because it is a publicly accessible resource that you must be able to access from any IP connected device.

    Consider the following when you specify a unique name for your new IoT Hub:

    * The value that you apply to _IoT Hub Name_ must be unique across all of Azure. This is true because the value assigned to the name will be used in the IoT Hub's connection string. Since Azure enables you to connect devices from anywhere in the world to your hub, it makes sense that all Azure hubs must be accessible from the Internet using the connection string and that connection strings must therefore be unique. We'll explore connection strings later in this lab.

    * The value that you assign to _IoT Hub Name_ cannot be changed once the app service has been created. If you do need to change the name, you'll need to create a new IoT Hub, re-register your devices to it, and delete your old IoT Hub.

    * The _IoT Hub Name_ field is a required field.
@@@warning
**Note**:  Azure will ensure that the name you enter is unique. If the name that you enter is not unique, Azure will display an asterisk at the end of the name field as a warning. You can append the name suggested above with '**-01**' or '**-02**' as necessary to achieve a globally unique name.
@@@

1. [ ] At the top of the blade, click **Size and scale**.

    Take a minute to review the information presented on this blade.

1. [ ] To the right of Pricing and scale tier, open the dropdown and then select **S1: Standard tier** if it is not already selected.

    You can choose from several tier options depending on how many features you want and how many messages you send through your solution per day. The _S1_ tier that we are using in this course allows a total of 400,000 messages per unit per day and provides the all of the services that are required in this training. We won't actually need 400,000 messages per unit per day, but we will be using features provided at this tier level, such as _Cloud-to-device commands_, _Device management_, and _IoT Edge_. IoT Hub also offers a free tier that is meant for testing and evaluation. It has all the capabilities of the standard tier, but limited messaging allowances. However, you cannot upgrade from the free tier to either basic or standard. The free tier is intended for testing and evaluation. It allows 500 devices to be connected to the IoT hub and up to 8,000 messages per day. Each Azure subscription can create one IoT Hub in the free tier.
@@@warning
**Note**:  The _S1 - Standard_ tier has a cost of $25.00 USD per month per unit. We will be specifying 1 unit. For details about the other tier options, see [Choosing the right IoT Hub tier for your solution](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-scaling).
@@@

1. [ ] To the right of **Number of S1 IoT Hub units**, ensure that **1** is selected.

    As mentioned above, the pricing tier that you choose establishes the number of messages that your hub can process per unit per day. To increase the number of messages that you hub can process without moving to a higher pricing tier, you can increase the number of units. For example, if you want the IoT hub to support ingress of 800,000 messages, you choose *two* S1 tier units. For the IoT courses created by Microsoft we will be using just 1 unit.

1. [ ] Under _Advanced Settings_, ensure that **Device-to-cloud partitions** is set to **4**.

    The number of partitions relates the device-to-cloud messages to the number of simultaneous readers of these messages. Most IoT hubs will only need four partitions, which is the default value. For this course we will create our IoT Hub using the default number of partitions.

1. [ ] At the top of the blade, click **Review + create**.

    Take a minute to review the settings that your provided.

1. [ ] At the bottom of the blade, to finalize the creation of your IoT Hub, click **Create**.

    Deployment can take a minute or more to complete. You can open the Azure portal Notification pane to monitor progress.

1. [ ] Notice that after a couple of minutes you receive a notification stating that your IoT Hub was successfully deployed to your **AZ-220-RG** resource group.

1. [ ] On the Azure portal menu, click **Dashboard**, and then click **Refresh**.

    You should see that your resource group tile lists your new IoT Hub.

#### Exercise 3: Examine the IoT Hub Service

@@@secondary
As we have already noted, the IoT Hub is a managed service, hosted in the cloud, that acts as a central message hub for bi-directional communication between your IoT application and the devices it manages.

IoT Hub's capabilities help you build scalable, full-featured IoT solutions such as managing industrial equipment used in manufacturing, tracking valuable assets in healthcare, monitoring office building usage, and many more scenarios. IoT Hub monitoring helps you maintain the health of your solution by tracking events such as device creation, device failures, and device connections.
@@@

#### Task 1: Explore the IoT Hub Overview blade

1. [ ] If necessary, log in to the Azure Portal using your Azure account credentials.

 ```cli
https://portal.azure.com
 ```

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Verify that your AZ-220 dashboard is being displayed.

1. [ ] On the AZ-220-RG resource group tile, click **AZ-220-HUB-_{YOUR-ID}_**

    When you first open your IoT Hub, it will display the _Overview_ blade. As you can see, the area at the top of this blade provides some essential information about your IoT Hub service, such as datacenter location and subscription. But this blade also includes tiles that provide information about how you are using your hub and recent activities. Let's take a look at these tiles before exploring further.

1. [ ] At the bottom-left of the _Overview_ blade, notice the **IoT Hub Usage** tile.
@@@warning
**Note**:  The tiles flow based upon the width of the browser, so the layout may be a little different than described.
@@@

    This tile provides a quick overview of what is connected to your hub and message count. As we add devices and start sending messages, this tile will provide nice "at-a-glance" information.

1. [ ] To the right of the _IoT Hub Usage_ tile, notice the **Device twin operations** tile and the **Device to cloud messages** tile.

    The _Device to cloud messages_ tile provides a quick view of the incoming messages from your devices over time. You will be registering a device and sending messages to your hub in the next module.

    You will be learning about device twins and device twin operations during the modules of this course that cover device configuration, device provisioning, and device management. For now all you need to know is that each device that you register with your IoT Hub will have a device twin that you can use when you need to manage the device.

##### Task 2: View features of IoT Hub using the navigation menu

1. [ ] On the IoT Hub blade, take a minute to scan the left-side navigation menu options.

    As you would expect, these options open blades that provide access to properties and features of your IoT Hub. They also give you access to devices that are connected to your hub.

1. [ ] On the left-side menu, under **Explorers**, click **IoT devices**

    This blade can be used to add, modify, and delete devices registered to your hub. You will get pretty familiar with this blade by the end of this course.

1. [ ] On the left-side menu, near the top, click **Activity log**

    As the name implies, this blade gives you access to a log that can be used to review activities and diagnose issues. You can also define queries that help with routine tasks. Very handy.

1. [ ] On the left-side menu, under **Settings**, click **Built-in endpoints**

    IoT Hub exposes "endpoints" that enable external connections. Essentially, an endpoint is anything connected to or communicating with your IoT Hub. You should see that your hub already has two endpoints defined:

    * **Events**
    * **Cloud to device messaging**
</br>
1. [ ] On the left-side menu, under **Messaging**, click **Message routing**

    The IoT Hub message routing feature enables you to route incoming device-to-cloud messages to service endpoints such as Azure Storage containers, Event Hubs, and Service Bus queues. You can also create routing rules to perform query-based routes.

1. [ ] At the top of the blade, click **Custom endpoints**.

    Custom endpoints (such as Service Bus queue, Blob storage and the others listed here) are often used within an IoT implementation.

1. [ ] Take a minute to review the menu options under **Settings**
@@@warning
**Note**:  This lab exercise is only intended to be an introduction to the IoT Hub service and get you more comfortable with the UI, so don't worry if you feel a bit overwhelmed at this point. We will be walking you through the process of configuring and managing your IoT Hub, devices, and communications as this course continues.
@@@

#### Exercise 4: Create a Device Provisioning Service using the Azure portal

@@@secondary
The Azure IoT Hub Device Provisioning Service is a helper service for IoT Hub that enables zero-touch, just-in-time provisioning to the right IoT hub without requiring human intervention. The Device Provisioning Service provides the following:

* Zero-touch provisioning to a single IoT solution without hardcoding IoT Hub connection information at the factory (initial setup)
* Load balancing devices across multiple hubs
* Connecting devices to their owner's IoT solution based on sales transaction data (multitenancy)
* Connecting devices to a particular IoT solution depending on use-case (solution isolation)
* Connecting a device to the IoT hub with the lowest latency (geo-sharding)
* Reprovisioning based on a change in the device
* Rolling the keys used by the device to connect to IoT Hub (when not using X.509 certificates to connect)

There are several methods that you can use to create an instance of the IoT Hub Device Provisioning Service. For example, you can use the Azure portal, which is what you will do in ths task. But you can also create a DPS instance using Azure CLI or an Azure Resource Manager Template.
@@@

#### Task 1: Use the Azure portal to create a resource (Device Provisioning Service)

1. [ ] If necessary, log in to Azure Portal using your Azure account credentials.

 ```cli
https://portal.azure.com
 ```

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] On the Azure portal menu, click **+ Create a resource**.

    The Azure Marketplace is a collection of all the resources you can create in Azure. The marketplace contains resources from both Microsoft and the community.

1. [ ] In the Search textbox, type **Device Provisioning Service** and then press Enter.

1. [ ] On the search results blade, click **IoT Hub Device Provisioning Service**.

    Notice the list of _Useful Links_ displayed on this blade.

1. [ ] In the list of links, notice that there is a link to Documentation.

    Again, there is no need to explore this documentation now, but is is good to know that it is available. The IoT Hub Device Provisioning Service Documentation page is the root page for DPS. You can use this page to explore current documentation and find tutorials and other resources that will help you to explore activities that are outside the scope of this course. We will refer you to the docs.microsoft.com site throughout this course for additional reading on specific topics.

    If you did open the documentation link, use your browser to close the documentation tab and navigate back to the Azure portal tab.

##### Task 2: Create a Device Provisioning Service with required property settings

1. [ ] To begin the process of creating your new DPS instance, click **Create**.

    Next, you need to specify information about the Hub and your subscription. The following steps walk you through the settings, explaining each of the fields as you fill them in.

1. [ ] Under **Name**, enter a unique name for your Device Provisioning Service.

    To provide a unique name, enter **AZ-220-DPS-_{YOUR-ID}_**.

    For example: **AZ-220-DPS-CAH191216**

1. [ ] Under **Subscription**, ensure that the subscription you are using for this course is selected.

1. [ ] Under **Resource Group**, open the **Select existing** dropdown, and then click **AZ-220-RG**

    This is the resource group that you created in the previous lab. We will be grouping the resources that we create for this course together in the same resource group. This should help you to clean up your resources when you have completed the course.

1. [ ] Under **Location**, open the drop-down list and select the same region as your resource group.

    As we saw previously, Azure is supported by a series of datacenters that are placed in regions all around the world. When you create something in Azure, you deploy it to one of these datacenter locations.
@@@warning
**Note**:  When picking a datacenter to host your app, keep in mind that picking a datacenter close to your end users will decrease load/response times. If you are on the other side of the world from your end users, you should not be picking the datacenter nearest you.
@@@

1. [ ] At the bottom of the blade, click **Create**.

    Deployment can take a minute or more to complete. You can open the Azure portal Notification pane to monitor progress.

1. [ ] Notice that after a couple of minutes you receive a notification stating that your IoT Hub Device Provisioning Service instance was successfully deployed to your **AZ-220-RG** resource group.

1. [ ] On the Azure portal menu, click **Dashboard**, and then click **Refresh**.

    You should see that your resource group tile lists your new IoT Hub Device Provisioning Service.

##### Task 3: Link your IoT Hub and Device Provisioning Service.

1. [ ] Notice that the AZ-220 dashboard lists both your IoT Hub and DPS resources.

    You should see both your IoT Hub and DPS resources listed - (you may need to hit **Refresh** if the resources were only recently created)

1. [ ] On your Resource group tile, click **AZ-220-DPS-_{YOUR-ID}_**.

1. [ ] On the _Device Provisioning Service_ blade, under **Settings**, click **Linked IoT hubs**.

1. [ ] At the top of the blade, click **+ Add**.

    You will use the _Add link to IoT hub_ blade to provide the information required to link your Device Provisioning service instance to an IoT hub.

1. [ ] On the _Add link to IoT hub_ blade, ensure that the **Subscription** dropdown is displaying the subscription that you are using for this course.

    The subscription is used to provide a list of the available IoT hubs.

1. [ ] Open the IoT hub dropdown, and then click **AZ-220-HUB-_{YOUR-ID}_**.

    This is the IoT Hub that you created in the previous exercise.

1. [ ] In the Access Policy dropdown, click **iothubowner**.

    The _iothubowner_ credentials provide the permissions needed to establish the link with the specified IoT hub.

1. [ ] To complete the configuration, click **Save**.

    You should now see the selected hub listed on the Linked IoT hubs blade. You might need to click **Refresh** to show Linked IoT hubs.

1. [ ] On the Azure portal menu, click **Dashboard**.

#### Exercise 5: Examine the Device Provisioning Service

@@@secondary
The IoT Hub Device Provisioning Service is a helper service for IoT Hub that enables zero-touch, just-in-time provisioning to the right IoT hub without requiring human intervention, enabling customers to provision millions of devices in a secure and scalable manner.
@@@

#### Task 1: Explore the Device Provisioning Service Overview blade

1. [ ] If necessary, log in to Azure Portal using your Azure account credentials.

 ```cli
https://portal.azure.com
 ```

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Verify that your AZ-220 dashboard is being displayed.

1. [ ] On the _AZ-220-RG_ resource group tile, click **AZ-220-DPS-_{YOUR-ID}_**

    When you first open your Device Provisioning Service instance, it will display the _Overview_ blade. As you can see, the area at the top of this blade provides some essential information about your DPS instance, such as status, datacenter location and subscription. This blade also provides the _Quick Links_ section, which provide access to:

    * [Azure IoT Hub Device Provisioning Service Documentation](https://docs.microsoft.com/en-us/azure/iot-dps/)
    * [Learn more about IoT Hub Device Provisioning Service](https://docs.microsoft.com/en-us/azure/iot-dps/about-iot-dps)
    * [Device Provisioning concepts](https://docs.microsoft.com/en-us/azure/iot-dps/concepts-service)
    * [Pricing and scale details](https://azure.microsoft.com/en-us/pricing/details/iot-hub/)
</br>

    You can explore these links to learn more.

##### Task 2: View features of Device Provisioning Service using the navigation menu

1. [ ] Take a minute to scan the left navigation area options.

    As you might expect, these options open blades that provide access to activity logs, properties and feature of the DPS instance.

1. [ ] On the left-side menu, near the top, click **Activity log**

    As the name implies, this blade gives you access to a log that can be used to review activities and diagnose issues. You can also define queries that help with routine tasks. Very handy.

1. [ ] On the left navigation area, under **Settings**, click **Quick Start**.

    This blade lists the steps to start using the Iot Hub Device Provisioning Service, links to documentation and shortcuts to other blades for configuring DPS.

1. [ ] On the left navigation area, under **Settings**, click **Shared access policies**.

    This blade provides management of access policies, lists the existing policies and the associated permissions.

1. [ ] On the left navigation area, under **Settings**, click **Linked IoT hubs**.

    Here you can see the linked IoT Hub from earlier. The Device Provisioning Service can only provision devices to IoT hubs that have been linked to it. Linking an IoT hub to an instance of the Device Provisioning service gives the service read/write permissions to the IoT hub's device registry; with the link, a Device Provisioning service can register a device ID and set the initial configuration in the device twin. Linked IoT hubs may be in any Azure region. You may link hubs in other subscriptions to your provisioning service.

1. [ ] On the left navigation area, under **Settings**, click **Certificates**.

    Here you can manage the X.509 certificates that can be used to secure your Azure IoT hub using the X.509 Certificate Authentication. You will investigate X.509 certificates in a later lab.

1. [ ] On the left navigation area, under **Settings**, click **Manage enrollments**.

    Here you can manage the enrollment groups and individual enrollments.

    Enrollment groups can be used for a large number of devices that share a desired initial configuration, or for devices all going to the same tenant. An enrollment group is a group of devices that share a specific attestation mechanism. Enrollment groups support both X.509 as well as symmetric. All devices in the X.509 enrollment group present X.509 certificates that have been signed by the same root or intermediate Certificate Authority (CA). Each device in the symmetric key enrollment group present SAS tokens derived from the group symmetric key. The enrollment group name and certificate name must be alphanumeric, lowercase, and may contain hyphens.

    An individual enrollment is an entry for a single device that may register. Individual enrollments may use either X.509 leaf certificates or SAS tokens (from a physical or virtual TPM) as attestation mechanisms. The registration ID in an individual enrollment is alphanumeric, lowercase, and may contain hyphens. Individual enrollments may have the desired IoT hub device ID specified.

1. [ ] Take a minute to review some of the other menu options under **Settings**

@@@warning
**Note**:  This lab exercise is only intended to be an introduction to the IoT Hub Device Provisioning Service and get you more comfortable with the UI, so don't worry if you feel a bit overwhelmed at this point. We will be covering DPS in much more detail as the course continues.
@@@

@@@success
**Congratulations**! you have now completed this lab!
@@@
