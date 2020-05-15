### Lab - Automatically provision IoT devices securely and at scale with DPS


@@@secondary
**Scenario**

After validating the device provisioning and de-provisioning process using an Individual Enrollment, the management team has asked you to begin rolling out the process on a larger scale.

To keep the project moving forward you need to demonstrate that the Device Provisioning Service can be used to enroll larger numbers of devices automatically and securely using X.509 certificate authentication.

In this lab, you will complete the following activities:

* Verify that the lab prerequisites are met (that you have the required Azure resources)
* Generate and Configure X.509 CA Certificates using OpenSSL
* Create Group Enrollment in DPS
* Configure simulated device with X.509 Certificate
* Handle device twin desired property Changes
* Automatic Enrollment of simulated device
* Retire Group Enrollment of simulated device
@@@

#### Exercise 1: Verify Lab Prerequisites

@@@secondary
This lab assumes that the following Azure resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |
| Device Provisioning Service | AZ-220-DPS-_{YOUR-ID}_ |

If these resources are not available, you will need to run the **lab06-setup.azcli** script as instructed below before moving on to Exercise 2. The script file is included in the GitHub repository that you cloned locally as part of the dev environment configuration (lab 3).

The **lab06-setup.azcli** script is written to run in a **bash** shell environment - the easiest way to execute this is in the Azure Cloud Shell.
@@@

1. [ ] Using a browser, open the Azure Cloud Shell and login with the Azure subscription you are using for this course.

    If you are prompted about setting up storage for Cloud Shell, accept the defaults.

1. [ ] Verify that the Azure Cloud Shell is using **Bash**.

    The dropdown in the top-left corner of the Azure Cloud Shell page is used to select the environment. Verify that the selected dropdown value is **Bash**.

1. [ ] On the Azure Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. [ ] In the dropdown, click **Upload**.

1. [ ] In the file selection dialog, navigate to the folder location of the GitHub lab files that you downloaded when you configured your development environment.

    In **Lab 3: Setup the Development Environment**, you cloned the GitHub repository containing lab resources by downloading a ZIP file and extracting the contents locally. The extracted folder structure includes the following folder path:

    * Allfiles
    
      * Labs
      
          * 06-Automatic Enrollment of Devices in DPS
            * Setup
</br>

    The lab06-setup.azcli script file is located in the Setup folder for lab 6.

1. [ ] Select the **lab06-setup.azcli** file, and then click **Open**.

    A notification will appear when the file upload has completed.

1. [ ] To verify that the correct file has uploaded in Azure Cloud Shell, enter the following command:

    ```bash
    ls
    ```

    The `ls` command lists the content of the current directory. You should see the lab06-setup.azcli file listed.

1. [ ] To create a directory for this lab that contains the setup script and then move into that directory, enter the following Bash commands:

    ```cli
    mkdir lab6
    ```
    ```cli
    mv lab06-setup.azcli lab6
    ```
    ```cli
    cd lab6
    ```

1. [ ] To ensure the **lab06-setup.azcli** script has the execute permission, enter the following command:

    ```bash
    chmod +x lab06-setup.azcli
    ```

1. [ ] On the Cloud Shell toolbar, to edit the lab06-setup.azcli file, click **Open Editor** (second button from the right - **{ }**).

1. [ ] In the **Files** list, to expand the lab6 folder and open the script file, click **lab6**, and then click **lab06-setup.azcli**.

    The editor will now show the contents of the **lab06-setup.azcli** file.

1. [ ] In the editor, update the values of the `{YOUR-ID}` and `{YOUR-LOCATION}` variables.

    In the sample below, you need to set `{YOUR-ID}` to the Unique ID you created at the start of this course - i.e. **CAH191211**, and set `{YOUR-LOCATION}` to the location that makes sense for your resources.

    ```bash
    #!/bin/bash

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

    Location="{YOUR-LOCATION}"
    ```
@@@warning
**Note**:  The `{YOUR-LOCATION}` variable should be set to the short name for the region. You can see a list of the available regions and their short-names (the **Name** column) by entering this command:
@@@
    
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
    ./lab06-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

    Once the script has completed, you will be ready to continue with the lab.

#### Exercise 2: Generate and Configure X.509 CA Certificates using OpenSSL

@@@secondary
In this exercise, you will generate an X.509 CA Certificate using OpenSSL within the Azure Cloud Shell. This certificate will be used to configure the Group Enrollment within the Device Provisioning Service (DPS).
@@@

##### Task 1: Generate the certificates

1. [ ] If necessary, log in to the Azure portal using the Azure account credentials that you are using for this course.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] In the top right of the portal window, to open the Azure Cloud Shell, click **Cloud Shell**.

    The Cloud Shell button has an icon that appears to represent a command prompt - **`>_`**.

    A Cloud Shell window will open near the bottom of the display screen.

1. [ ] In the upper left corner of the Cloud Shell window, ensure that **Bash** is selected as the environment option.
@@@warning
**Note**:  Both *Bash* and *PowerShell* interfaces for the Azure Cloud Shell support the use of **OpenSSL**. In this unit you will use some helper scripts written for the *Bash* shell.
@@@

1. [ ] At the Cloud Shell command prompt, to download Azure IoT helper scripts, enter the following commands:

    ```sh
    # create certificates directory
    mkdir certificates
    # navigate to certificates directory
    cd certificates

    # download helper script files
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/certGen.sh --output certGen.sh
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_device_intermediate_ca.cnf --output openssl_device_intermediate_ca.cnf
    curl https://raw.githubusercontent.com/Azure/azure-iot-sdk-c/master/tools/CACertificates/openssl_root_ca.cnf --output openssl_root_ca.cnf

    # update script permissions so user can read, write, and execute it
    chmod 700 certGen.sh
    ```

    These helper scripts are being downloaded from the `Azure/azure-iot-sdk-c` open source project hosted on Github. This is an open source project that's a part of the Azure IoT SDK. The `certGen.sh` helper script will help demonstrate the purpose of CA Certificates without diving into the specifics of OpenSSL configuration that's outside the scope of this lab.

    For additional instructions on using this helper script, and for instructions on how to use PowerShell instead of Bash, please see this link: `https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md`
@@@danger
**WARNING**! Certificates created by this helper script **MUST NOT** be used for Production. They contain hard-coded passwords ("*1234*"), expire after 30 days, and most importantly are provided for demonstration purposes to help you quickly understand CA Certificates. When building products against CA Certificates, you'll need to use your own security best practices for certificate creation and lifetime management.
@@@

1. [ ] Review the contents of the script you downloaded using whatever tool you'd prefer (`more`, `code`, `vi`, etc.) to validate the code you downloaded.

1. [ ] The first X.509 certificates needed are CA and intermediate certificates. These can be generated using the `certGen.sh` helper script by passing the `create_root_and_intermediate` option.

    Run the following command within the `~/certificates` directory of the **Azure Cloud Shell** to generate the CA and intermediate certificates:

    ```sh
    ./certGen.sh create_root_and_intermediate
    ```

1. [ ] The previous command generated a Root CA Certificate named `azure-iot-test-only.root.ca.cert.pem` is located within the `./certs` directory.

    Run the following command within the **Azure Cloud Shell** to download this certificate to your local machine so it can be uploaded to DPS.

    ```sh
    download ~/certificates/certs/azure-iot-test-only.root.ca.cert.pem
    ```

##### Task 2: Configurate your DPS to trust the root certificate

1. [ ] Navigate to your **Device Provisioning Service** (DPS) named `AZ-220-DPS-_{YOUR-ID}_` within the Azure portal.

1. [ ] On the left side of the **Device Provisioning Service** blade, in the **Settings** section, click the **Certificates** link.

1. [ ] On the **Certificates** pane, click the **Add** button at the top to start process of uploading the X.509 CA Certificate to the DPS service.

1. [ ] On the **Add Certificate** pane, for **Certificate Name**, enter a logical name for the **Root CA Certificate** into the field. For example, `root-ca-cert`.
@@@warning
**Note**: This name could be the same as the name of the certificate file, or something different. This is a logical name that has no correlation to the **Common Name** within the X.509 CA Certificate.
@@@

1. [ ] For the **Certificate .pem or .cer file.** upload field, select the `azure-iot-test-only.root.ca.cert.pem` CA Certificate that you just downloaded.

1. [ ] Click **Save**.

    Once the X.509 CA Certificate has been uploaded, the _Certificates_ pane will display the certificate with the _Status_ of **Unverified**. Before this CA Certificate can be used to authenticate devices to DPS, you will need to verify **Proof of Possession** of the certificate.

1. [ ] To start the process of verifying **Proof of Possession** of the certificate, click on the **CA Certificate** that was just uploaded to open the **Certificate Details** pane for it.

1. [ ] On the **Certificate Details** pane, click on the **Generate Verification Code** button.  (You may need to scroll to see it.)

1. [ ] Copy the newly generated **Verification Code** that is displayed above the _Generate Verification Code_ button.  The button to the right of the _Verification Code_ textbox will do this for you.

    _Proof of Possession_ of the CA certificate is provided to DPS by uploading a certificate generated from the CA certificate with the verifcation code that was just generated within DPS. This is how you provide proof that you actually own the CA Certificate.
@@@warning
**Note**: You will need to leave the **Certificate Details** pane open while you generate the verification certificate. If you close the pane, you will invalidate the verification code, and will need to generate a new one.
@@@

1. [ ] Open the **Azure Cloud Shell**, if it's not still open from earlier, and navigate to the `~/certificates` directory.

1. [ ] Run the following command, passing in your copied verification code, to create the verification certificate:

    ```sh
    ./certGen.sh create_verification_certificate <verification-code>
    ```

    Be sure to replace the `<verification-code>` placeholder with the **Verification Code** generated by the Azure portal.

    For example, the command run will look similar to the following:

    ```sh
    ./certGen.sh create_verification_certificate 49C900C30C78D916C46AE9D9C124E9CFFD5FCE124696FAEA
    ```

    This generates a _verification certificate_ that is chained to the CA certificate.  The subject of the certificate is the verification code. The generated Verification Certificate named `verification-code.cert.pem` is located within the `./certs` directory of the Azure Cloud Shell.

1. [ ] Run the following command within the **Azure Cloud Shell** to download the verification certificate to your local machine so it can be uploaded to DPS.

    ```sh
    download ~/certificates/certs/verification-code.cert.pem
    ```
@@@warning
**Note**: Depending on the web browser, you may be prompted to allow multiple downloads at this point.  If there appears to be no response to your download command, make sure there's not a prompt elsewhere on the screen asking for permission to allow the download.
@@@

1. [ ] Go back to the **Certificate Details** pane for the CA certificate within DPS.

1. [ ] For **Verification Certificate .pem or .cer file.**, select the newly created, and downloaded, verification certificate file, named `verification-code.cert.pem`.

1. [ ] Click **Verify**.

1. [ ] Verify that in the **Certificates** pane, the **Status** for the certificate is now displayed as _Verified_.  You may need to use the **Refresh** button at the top of the pane (to the right of the **Add** button) to see this change.

#### Exercise 3: Create Group Enrollment (X.509 Certificate) in DPS

@@@secondary
In this exercise, you will create a new individual enrollment for a device within the Device Provisioning Service (DPS) using **certificate attestation**.
@@@

##### Task 1: Create the enrollment

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] On your resource group tile, click **AZ-220-DPS-_{YOUR-ID}_**.

1. [ ] On the left side of the Device Provisioning Service blade, under **Settings**, click **Manage enrollments**.

1. [ ] At the top of the blade, click **Add enrollment group**.

1. [ ] On the **Add Enrollment Group** blade, for **Group name**, enter "**simulated-devices**".

1. [ ] Ensure that the **Attestation Type** is set to **Certificate**.

1. [ ] Ensure the **Certificate Type** field is set to **CA Certificate**.

1. [ ] In the **Primary Certificate** dropdown, select the CA certificate that was uploaded to DPS previously, likely **root-ca-cert**.

1. [ ] Leave the **Secondary Certificate** dropdown set to **No certificate selected**.

1. [ ] Leave **Select how you want to assign devices to hubs** as **Evenly weighted distribution**.

   As you only have one IoT Hub associated with the enrollment, this setting is somewhat unimportant.  In larger environments where you have multiple distributed hubs, this setting will control how to choose what IoT Hub should receive this device enrollment.

1. [ ] Notice that the **AZ-220-HUB-_{YOUR-ID}_** IoT Hub is selected within the **Select the IoT hubs this device can be assigned to:** dropdown.

   This field specifies the IoT Hub(s) this device can be assigned to.

1. [ ] Leave **Select how you want device data to be handled on re-provisioning** as the default value of **Re-provision and migrate data**.

    This field gives you high-level control over the re-provisioning behavior, where the same device (as indicated through the same Registration ID) submits a later provisioning request after already being provisioned successfully at least once.

1. [ ] Notice the **Select the IoT hubs this group can be assigned to** dropdown has the **AZ-220-HUB-*{YOUR-ID}*** IoT Hub selected. This will ensure when the device is provisioned, it gets added to this IoT Hub.

1. [ ] In the **Initial Device Twin State** field, modify the `properties.desired` JSON object to include a property named `telemetryDelay` with the value of `"1"`. This will be used by the Device to set the time delay for reading sensor telemetry and sending events to IoT Hub.

    The final JSON will be like the following:

    ```json
    {
        "tags": {},
        "properties": {
            "desired": {
                "telemetryDelay": "1"
            }
        }
    }
    ```

    This field contains JSON data that represents the initial configuration of desired properties for the device.

1. [ ] Leave **Enable entry** set to **Enable**.

    Generally, you'll want to enable new enrollment entries and keep them enabled.

1. [ ] At the top of the **Add Enrollment** blade, click **Save**.

##### Task 2: Validate the enrollment

1. [ ] In the **Manage enrollments** pane, click on the **Enrollment Groups** tab to view the list of enrollment groups in DPS.

1. [ ] In the list, click on the **simulated-devices** enrollment group that was just created to view the enrollment group details.

1. [ ] In the **Enrollment Group Details** blade, locate the **Certificate Type**, and notice it's set to **CA Certificate**. Also, notice the **Primary Certificate** information is displayed, including the ability to update the certificates if needed.

1. [ ] Locate the **Initial Device Twin State**, and notice the JSON for the Device Twin Desired State contains the `telemetryDelay` property set to the value of `"1"`.

1. [ ] Close the **Enrollment Group Details** pane.

#### Exercise 4: Configure simulated device with X.509 certificate

@@@secondary
In this exercise, you will configure a simulated device written in C# to connect to your Azure IoT Hub via your Device Provisioning Service (DPS) using an X.509 certificate. This exercise will also introduce you to the workflow within the **simulated device** source code within the `/LabFiles` directory, and how it works to authenticate with DPS and send messages to IoT Hub.
@@@

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] Open the Azure Cloud Shell by clicking the **Terminal** icon within the top header bar of the Azure portal, and select the **Bash** shell option.

1. [ ] Within the **Azure Cloud Shell**, navigate to the `~/certificates` directory by running the following command:

    ```sh
    cd ~/certificates
    ```

    The `~/certificates` directory is where the `certGen.sh` helper scripts was downloaded to and used to generate the CA Certificate for DPS previously. This helper script will also be used to generate a device certificate within the CA Certificate chain.

1. [ ] Run the following command to generate an _X.509 device certificate_ within the CA certificate chain with the device name of `simulated-device1`. This certificate will generate a leaf device X.509 certificate for your simulated device. This certificate will be signed by the CA certificate generated previously, and will be used to authenticate the device with the Device Provisioning Service (DPS).

    ```sh
    ./certGen.sh create_device_certificate simulated-device1
    ```

    Notice the device id of `simulated-device1` is passed to the `create_device_certificate` command of the `certGen.sh` script. This command will create a new X.509 certificate that's signed by the CA certificate and has the specified device id set within the _common name_, or `CN=`, value of the device certificate.

    Once the `create_device_certificate` command has completed, the generated X.509 device certificate will be named `new-device.cert.pfx`, and will be located within the `/certs` sub-directory.
@@@warning
**Note**:  This command overwrites the existing certificate. If you want to create a certificate for multiple devices, ensure you copy the `new-device.cert.pfx` each time.
@@@

1. [ ] Run the following command to download the generated X.509 device certificate from the Cloud Shell to your local machine:

    ```sh
    download ~/certificates/certs/new-device.cert.pfx
    ```

    The simulated device will be configured to use this X.509 device certificate to authenticate with the Device Provisioning Service.

1. [ ] Within the Azure portal, navigate to the **Device Provisioning Service** blade, and the **Overview** pane.

1. [ ] Within the **Overview** pane, copy the **ID Scope** for the Device Provisioning Service, and save it for reference later.  (There is a copy button to the right of the value that will appear when you hover over the value.)

    The **ID Scope** will be similar to this value: `0ne0004E52G`

1. [ ] Copy the downloaded `new-device.cert.pfx` X.509 device certificate file to the `/LabFiles` directory; within the root directory along-side the `Program.cs` file. The **simulated device** project will need to access this certificate file when authenticating to the Device Provisioning Service.

    After copied, the certificate file will be located in the following location:

    ```text
    /LabFiles/new-device.cert.pfx
    ```

1. [ ] Using **Visual Studio Code**, open the `/LabFiles` folder.

1. [ ] Open the `SimulatedDevice.csproj` file.

1. [ ] Within the `SimulatedDevice.csproj` file, ensure the following block of XML exists within the `<ItemGroup>` tag of the file. This configuration ensures that the `new-device.cert.pfx` certificate file is copied to the build folder when the C# code is compiled, and made available for the program to access when it executes.

    ```xml
        <None Update="new-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
    ```

    If it's not there, then paste it in before the closing `</Project>` tag. The end of the file should look similar to the following:

    ```xml
            <ItemGroup>
                <None Update="new-device.cert.pfx" CopyToOutputDirectory="PreserveNewest" />
                <PackageReference Include="Microsoft.Azure.Devices.Client" Version="1.21.1" />
                <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Mqtt" Version="1.1.8" />
                <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Amqp" Version="1.1.9" />
                <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Transport.Http" Version="1.1.6" />
            </ItemGroup>
        </Project>
    ```

    The exact `PackageReference` entries may be a bit different, depending on the exact version of the lab code you are using.

1. [ ] Open the `Program.cs` file.

1. [ ] Locate the `GlobalDeviceEndpoint` variable, and notice it's value is set to `global.azure-devices-provisioning.net`. This is the **Global Device Endpoint** for the Azure Device Provisioning Service (DPS) within the Public Azure Cloud. All devices connecting to Azure DPS will be configured with this Global Device Endpoint DNS name.

    ```csharp
    private const string GlobalDeviceEndpoint = "global.azure-devices-provisioning.net";
    ```

1. [ ] Locate the `dpsIdScope` variable, and replace the value with the **ID Scope** of the Device Provisioning Service.

   ```csharp
   private static string dpsIdScope = "<DPS-ID-Scope>";
   ```

1. [ ] Locate the `s_certificateFileName` variable. Notice the value to this variable is set to `new-device.cert.pfx`. This is the name of the X.509 device certificate file that was copied to the `/LabFiles` directory after it was previously generated using the `certGen.sh` helper script within the Cloud Shell. This variable tells the simulated device program code what file contains the X.509 device certificate to use when authenticating with the Device Provisioning Service.

1. [ ] Locate the `s_certificatePassword` variable. This variable contains the password for the X.509 device certificate. Notice that it's already set to `1234`, as this is the default password used by the `certGen.sh` helper script when generating the X.509 certificates.
@@@warning
**Note**:  For the purpose of this lab, the password is hard coded. In a _production_ device, the password will need to be stored in a more secure manner, such as in an Azure Key Vault. Additionally, the certificate file (PFX) should be stored securely on a production device using a Hardware Security Module (HSM).
@@@
    
     An HSM (Hardware Security Module), is used for secure, hardware-based storage of device secrets, and is the most secure form of secret storage. Both X.509 certificates and SAS tokens can be stored in the HSM. HSMs can be used with all attestation mechanisms the provisioning service supports.

1. [ ] Locate the `public static int Main` method. This is the execution entry for the simulated device.

    This method contains code that initiates the use of the X.509 device certificate by calling the `LoadProvisioningCertificate` method to load the certificate. The `LoadProvisioningCertificate` method returns an `X509Certificate2` object that contains the X.509 device certificate from the `new-device.cert.pfx` file.

1. [ ] Locate the `LoadProvisioningCertificate` method, and review the necessary code to load an X.509 certificate from the `new-device.cert.pfx` file.

1. [ ] Notice the `public static int Main` method also contains code that initiates a `security` `SecurityProviderX509Certificate` object for the X.509 Device Certificate. It also creates a `transport` `ProvisioningTransportHandlerAmqp` object that defines the simulated device will be using AMQP as the communications protocol when connecting to Azure IoT Hub.

1. [ ] Notice the `ProvisioningDeviceClient.Create` method is passed the `security` and `transport` objects, as well as the DPS ID scope and DPS global device endpoint, that will be used to register the device with the Device Provisioning Service.

1. [ ] Notice the `ProvisioningDeviceLogic` object is instantiated by passing it the `ProvisioningDeviceClient` (`provClient`) and `security` objects.

    The `ProvisioningDeviceLogic` is a class define within the simulated device that contains code for the device logic. It contains code for running the device by reading from the simulated device sensors, and sending device-to-cloud messages to Azure IoT Hub. It will also be modified later to contain code that updates the device according to changes to device twin desired properties that are sent to the device from the cloud.

1. [ ] Locate the `ProvisioningDeviceLogic.RunAsync` method.

1. [ ] Notice the code that calls the `ProvisioningDeviceClient.RegisterAsync` method. This method Registers the device using the Device Provisioning Service and assigns it to an Azure IoT Hub.

    ```csharp
    DeviceRegistrationResult result = await _provClient.RegisterAsync().ConfigureAwait(false);
    ```

1. [ ] Notice the code that instantiates a new `DeviceAuthenticationWithX509Certificate` object. This is a Device Authentication object that will be used to authenticate the simulated device to Azure IoT Hub using the X.509 Device Certificate. The constructor is being passed the device ID for the device that was registered in DPS, as well as the X.509 device certificate to authenticate the device.

    ```csharp
    var auth = new DeviceAuthenticationWithX509Certificate(result.DeviceId, (_security as SecurityProviderX509).GetAuthenticationCertificate());
    ```

1. [ ] Notice the code that calls the `DeviceClient.Create` method to create a new IoT `DeviceClient` object that is used to communicate with the Azure IoT Hub service.

    ```csharp
    using (iotClient = DeviceClient.Create(result.AssignedHub, auth, TransportType.Amqp))
    {
    ```

    Notice this code also passes the value of `TransportType.Amqp`, telling the `DeviceClient` to communicate with the Azure IoT Hub using the AMQP protocol. Alternatively, Azure IoT Hub can be connected to and communicated with using the MQTT or HTTP protocols, depending on network architecture, device requirements, etc.

1. [ ] Notice the call to the `SendDeviceToCloudMessagesAsync` method. This is a method defined within the simulated device code, and is where the device logic is located to read from simulated sensors and to send device-to-cloud messages to Azure IoT Hub. This method also contains a loop that continues to execute while the simulated device is running.

    ```csharp
    await SendDeviceToCloudMessagesAsync(iotClient);
    ```

1. [ ] Notice the call to the `DeviceClient.CloseAsync` method. This method closes the `DeviceClient` object, thus closing the connection with Azure IoT Hub. This is the last line of code executed when the simulated device shuts down.

    ```csharp
    await iotClient.CloseAsync().ConfigureAwait(false);
    ```

1. [ ] Locate the `SendDeviceToCloudMessagesAsync` method.

1. [ ] Notice the code within the `SendDeviceToCloudMessagesAsync` method contains a `while` loop that has code to generate simulated sensor readings, and then send that data to Azure IoT Hub in a device-to-cloud message.

1. [ ] Next, notice the simulated sensor readings are combined into a JSON object using the following code:

    ```csharp
        var telemetryDataPoint = new
        {
            temperature = currentTemperature,
            humidity = currentHumidity,
            pressure = currentPressure,
            latitude = currentLatitude,
            longitude = currentLongitude
        };
        var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
        var message = new Message(Encoding.ASCII.GetBytes(messageString));

    ```

1. [ ] Moving on, notice that there is a line of code that adds a `temperatureAlert` property to the device-to-cloud `Message`. The value of the property is being set to a `boolean` value representing whether the _temperature_ sensor reading is greater than 30.

    ```csharp
    message.Properties.Add("temperatureAlert", (currentTemperature > 30) ? "true" : "false");
    ```

    This code is a simple example of how to add properties to the `Message` object before sending it to the Azure IoT Hub. This can be used to add additional metadata to the messages that are being send, in addition to the message body content.

1. [ ] After that is the call to the `DeviceClient.SendEventAsync` method. This method accepts the `Message` to send as a parameter, then does the work of sending the device-to-cloud message to Azure IoT Hub.

    ```csharp
    await deviceClient.SendEventAsync(message);
    ```

1. [ ] Notice the last line of code within the `SendDeviceToCloudMessagesAsync` method, that performs a simply delay using the `_telemetryDelay` variable to define how many seconds to wait until sending the next simulated sensor reading.

#### Exercise 5: Handle device twin desired property Changes

@@@secondary
In this exercise, you will modify the simulated device source code to include an event handler to update device configurations based on device twin desired property changes sent to the device from Azure IoT Hub.

Device twins are JSON documents that store device state information including metadata, configurations, and conditions. Azure IoT Hub maintains a device twin for each device that you connect to IoT Hub.

The Device Provisioning Service (DPS) contains the initial device twin desired properties for devices that are registered using Group Enrollment. Once the devices are registered they are created within IoT Hub using this initial device twin configuration from DPS. After registration, the Azure IoT Hub maintains a device twin (and its properties) for each device within the IoT Hub Device Registry.

When the device twin desired properties are updated for a device within Azure IoT Hub, the desired changes are sent to the IoT Device using the `DesiredPropertyUpdateCallback` event using the C# SDK. Handling this event within device code enables the devices configuration and properties to be updated as desired by easily managing the Device Twin state for the device within Azure IoT Hub.

This set of steps will be very similar to steps in earlier labs for working with a simulated device, because the concepts and processes are the same.  The method used for authentication of the provisioning process doesn't change the handing of device twin property changes once the device is provisioned.
@@@

1. [ ] Using **Visual Studio Code**, open the `/LabFiles` folder if it's not still open.

1. [ ] Open the `Program.cs` file if it's not still open.

1. [ ] Locate the `RunAsync` method.
   This is the method that connects the simulated device to Azure IoT Hub using a `DeviceClient` object. You will be adding code immediately after the device connects to Azure IoT Hub that integrates an `DesiredPropertyUpdateCallback` event handler for the device to receive to device twin desired property changes.

1. [ ] Locate the `// TODO 1` comment within the **RunAsync** method, and paste in the following code that calls the `SetDesiredPropertyUpdateCallbackAsync` method to setup the `DesiredPropertyUpdateCallback` event handler to receive device twin desired property changes.

    ```csharp
    // TODO 1: Setup OnDesiredPropertyChanged Event Handling to receive Desired Properties changes
    Console.WriteLine("Connecting SetDesiredPropertyUpdateCallbackAsync event handler...");
    await iotClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).ConfigureAwait(false);
    ```

    Notice this code is configuring the `DeviceClient` to call a method named `OnDesiredPropertyChanged` when device twin property change events are received.

1. [ ] Now that the `SetDesiredPropertyUpdateCallbackAsync` method was used to setup the event handler, the method named `OnDesiredPropertyChanged` that it's configured to call needs to be defined.  To define the `OnDesiredPropertyChanged` method, paste in the following code to the `ProvisioningDeviceLogic` class, below the `RunAsync` method:

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

    Notice the first block of code within the `OnDesiredPropertyChanged` event handler method that reads the `telemetryDelay` property value from the device twin desired properties. This code also take the value specified and modifies the `this._telemetryDelay` variable with the desired value to reconfigure the delay time between sending simulated sensor readings to Azure IoT Hub. Remember, this `this._telemetryDelay` variable is used within the `Task.Delay` call inside the `while` loop of the `SendDeviceToCloudMessagesAsync` method.

    Also notice the second block of the code within the `OnDesiredPropertyChanged` event handler method that reports the current state of the device to Azure IoT Hub. This code calls the `DeviceClient.UpdateReportedPropertiesAsync` method and passes it a **TwinCollection** method that contains the current state of the device properties. This code is how the device reports back to IoT Hub that it received the device twin desired properties changed event, and has now updates it's configurations accordingly. It also reports what the properties are now set to, in case they would be different than the desired state received, so that IoT Hub can maintain an accurate Device Twin that reflects the state of the device.

1. [ ] Now that the device can receive updates to the device twin desired properties from Azure IoT Hub, it also needs to be coded to configure it's initial setup when the simulated device begins execution. To do this the device will need to load the current device twin desired properties from Azure IoT Hub, and configure itself accordingly. Locate the `// TODO 2` comment within the `RunAsync` method, and paste in the following code that loads the current device twin desired properties from Azure IoT Hub for the device, and then uses the same `OnDesiredPropertyChanged` method defined previously to configure the device accordingly.

    ```csharp
    // TODO 2: Load Device Twin Properties since device is just starting up
    Console.WriteLine("Loading Device Twin Properties...");
    var twin = await iotClient.GetTwinAsync().ConfigureAwait(false);
    // Use OnDesiredPropertyChanged event handler to set the loaded Device Twin Properties (re-use!)
    await OnDesiredPropertyChanged(twin.Properties.Desired, null);
    ```

    Notice the call to the `DeviceClient.GetTwinAsync` method. This method can be used by the device to retrieve the current Device Twin state at any time. It's used in this case so the device can configure itself to match the device twin desired properties when the device first starts execution.

    In this case, the `OnDesiredPropertyChanged` event handler method is being reused to keep the configuration of the `telemetryDelay` property based on the device twin desired properties to a single place. This will help make the code easier to maintain over time.

#### Exercise 6: Test the Simulated Device

@@@secondary
In this exercise, you will run the simulated device. When the device is started for the first time, it will connect to the Device Provisioning Service (DPS) and automatically be enrolled using the configured group enrollment. Once enrolled into the DPS group enrollment, the device will be automatically registered within the Azure IoT Hub device registry. Once enrolled and registered, the device will begin communicating with Azure IoT Hub securely using the configured X.509 certificate authentication.
@@@

#### Task 1: Build and run the device

1. [ ] Using **Visual Studio Code**, open the `/LabFiles` folder if it's not still open.

1. [ ] Within **Visual Studio Code**, click the **View** menu, then select **Terminal** to open the Visual Studio Code Terminal window.

1. [ ] Navigate to `/LabFiles` directory within the **Terminal**.

1. [ ] Build and execute the **SimulatedDevice** project by running the `dotnet run` command.

    ```cmd/sh
    dotnet run
    ```
@@@warning
**Note**:  When running `dotnet run` for the simulated device, if a `ProvisioningTransportException` exception is displayed, the most common cause is an **Invalid certificate** error. If this happens, ensure the CA Certificate in DPS, and the Device Certificate for the simulated device application are configured correctly.
@@@
 
     ```text
     localmachine:LabFiles User$ dotnet run
     Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1; PrivateKey: True
     Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1
     RegistrationID = simulated-device1
     ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"2e298c80-0974-493c-9fd9-6253fb055ade","message":"Invalid certificate.","timestampUtc":"2019-12-13T14:55:40.2764134Z"}
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
       at X509CertificateSimulatedDevice.ProvisioningDeviceLogic.RunAsync() in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 121
       at X509CertificateSimulatedDevice.Program.Main(String[] args) in /Users/User/Documents/AZ-220/LabFiles/Program.cs:line 55
     ...
     ```

1. [ ] When the simulated device application runs, the **Terminal** will display the Console output from the app. Notice the X.509 certificate was loaded, the device was registered with the Device Provisioning Service, it was assigned to connect to the **AZ-220-HUB-_{YOUR-ID}_** IoT Hub, and the device twin desired properties are loaded.

    ```text
    localmachine:LabFiles User$ dotnet run
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1
    RegistrationID = simulated-device1
    ProvisioningClient RegisterAsync . . . Device Registration Status: Assigned
    ProvisioningClient AssignedHub: AZ-220-HUB-CP1119.azure-devices.net; DeviceID: simulated-device1
    Creating X509 DeviceClient authentication.
    simulated device. Ctrl-C to exit.
    DeviceClient OpenAsync.
    Connecting SetDesiredPropertyUpdateCallbackAsync event handler...
    Loading Device Twin Properties...
    Desired Twin Property Changed:
    {"$version":1}
    Reported Twin Properties:
    {"telemetryDelay":1}
    Start reading and sending device telemetry...
    ```

    To review the source code for the simulated device, open the `/LabFiles/Program.cs` source code file. Look for several `Console.WriteLine` statements that are used to output the messages seen to the console.

1. [ ] Once the simulated device starts executing, it will begin sending simulated sensor telemetry messages to Azure IoT Hub. Notice the output to the console that displays the JSON of the sensor readings that are being sent to Azure IoT Hub.

    Also, notice the delay, as defined by the `telemetryDelay` Device Twin Property, between each message sent to IoT Hub is currently delaying **1 second** between sending sensor telemetry messages.

    ```text
    Start reading and sending device telemetry...
    12/9/2019 5:47:00 PM > Sending message: {"temperature":24.047539159212047,"humidity":67.00504162675004,"pressure":1018.8478924248358,"latitude":40.129349260196875,"longitude":-98.42877188146265}
    12/9/2019 5:47:01 PM > Sending message: {"temperature":26.628804161040485,"humidity":68.09610794675355,"pressure":1014.6454375411363,"latitude":40.093269544242695,"longitude":-98.22227128174003}
    ```

    Keep the simulated device running for the next task.

#### Task 2: Change the device configuration through its twin

@@@secondary
With the simulated device running, the `telemetryDelay` configuration can be updated by editing the device twin Desired State within Azure IoT Hub. This can be done by configuring the Device in the Azure IoT Hub within the Azure portal.
@@@

1. [ ] Open the **Azure Portal** if it is not already open, and navigate to your **Azure IoT Hub** service.

1. [ ] On the IoT Hub blade, on the left side of the blade, under the **Explorers** section, click on **IoT devices**.

1. [ ] Within the list of IoT devices, click on the **Device ID** (likely **simulated-device1**) for the Simulated Device.
@@@danger
**IMPORTANT**! Make sure you select the device from this lab.
@@@

1. [ ] On the device blade, click the **Device Twin** button at the top of the blade.

    Within the **Device twin** blade, there is an editor with the full JSON for the device twin. This enables you to view and/or edit the device twin state directly within the Azure portal.

1. [ ] Locate the `properties.desired` node within the Device Twin JSON. Add, or update if it already exists, the `telemetryDelay` property to have the value of `"2"`. This will update the `telemetryDelay` of the simulated device to send sensor telemetry every **2 seconds**.

    The resulting JSON for this section of the device twin desired properties will look similar to the following:

    ```json
    "properties": {
        "desired": {
          "telemetryDelay": "2",
          "$metadata": {
            "$lastUpdated": "2019-12-09T22:48:05.9703541Z",
            "$lastUpdatedVersion": 2,
            "telemetryDelay": {
              "$lastUpdated": "2019-12-09T22:48:05.9703541Z",
              "$lastUpdatedVersion": 2
            }
          },
          "$version": 2
        },
    ```

    Leave the `$metadata` and `$version` value of the `properties.desired` node within the JSON. You should only update the `telemetryDelay` value to set the new device twin desired property value.

1. [ ] Click **Save** at the top of the blade to set the new device twin desired properties for the device. Once saved, the updated device twin desired properties will automatically be sent to the simulated device.

1. [ ] Go back to the **Visual Studio Code Terminal** window, where the simulated device is running, and notice the application has been notified of the updated device twin `telemetryDelay` desired property setting.

    The application outputs to the Console some messages that display that the new device twin desired properties have been loaded, and the changes have been set and reported back to the Azure IoT Hub.

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":2,"$version":2}
    Reported Twin Properties:
    {"telemetryDelay":2}
    ```

1. [ ] Notice the simulated device sensor telemetry messages are now being sent to Azure IoT Hub every _2_ seconds, now that the updated device twin desired properties have been sent to the device and the device has updated it's internal state accordingly.

    ```text
    12/9/2019 5:48:06 PM > Sending message: {"temperature":33.89822140284731,"humidity":78.34939097908763,"pressure":1024.9467544610131,"latitude":40.020042418755764,"longitude":-98.41923808825841}
    12/9/2019 5:48:09 PM > Sending message: {"temperature":27.475786026323114,"humidity":64.4175510594703,"pressure":1020.6866468579678,"latitude":40.2089999240047,"longitude":-98.26223221770334}
    12/9/2019 5:48:11 PM > Sending message: {"temperature":34.63600901637041,"humidity":60.95207713588703,"pressure":1013.6262313688063,"latitude":40.25499096898331,"longitude":-98.51199886959347}
    ```

1. [ ] To exit the simulated device app within the **Terminal** window, press **Ctrl-C**.

1. [ ] In the Azure Portal, close the **Device twin** blade.

1. [ ] Still in the Azure Portal, on the Simulated Device blade, again click the **Device Twin** button.

1. [ ] This time, locate the JSON for the `properties.reported` object.

    This contains the state reported by the device. Notice the `telemetryDelay` property exists here as well, and is also set to `2`.  There is also a `$metadata` value that shows you when the value was reported data was last updated and when the specific reported value was last updated.

1. [ ] Again close the **Device twin** blade.

1. [ ] Close the simulated device blade to return back to the IoT Hub blade.

#### Exercise 7: Retire Group Enrollment

@@@secondary
In this exercise, you will retire the enrollment group and its devices from both the Device Provisioning Service and Azure IoT Hub.
@@@

##### Task 1: Retire the enrollment group from the DPS

1. [ ] If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. [ ] On your resource group tile, click **AZ-220-DPS-_{YOUR-ID}_** to navigate to the Device Provisioning Service.

1. [ ] On the Device Provisioning Service settings pane on the left side, click **Manage enrollments**.

1. [ ] Click on **simulated-devices** in the list of **Enrollment Groups**.

1. [ ] On the **Enrollment Group Details** blade, locate the **Enable entry** option and set it to **Disable**, then click **Save** at the top of the blade.

    Disabling the Group Enrollment within DPS allows you to temporarily disable devices within this Enrollment Group. This provides a temporary blacklist of the X.509 certificate used by these devices.

1. [ ] To permanently delete the Enrollment Group, you must delete the enrollment group from DPS. To do this, check the box next to the **simulated-devices** **Group Name** on the **Manage enrollments** pane if it is not already checked, then click the **Delete** button at the top.

1. [ ] When prompted to confirm the action to **Remove enrollment**, click **Yes**.

   Once deleted, the Group Enrollment is completely removed from DPS, and would need to be recreated to add it back.
@@@warning
**Note**:  If you delete an enrollment group for a certificate, devices that have the certificate in their certificate chain might still be able to enroll if a different, enabled enrollment group still exists for the root certificate or another intermediate certificate higher up in their certificate chain.
@@@

##### Task 2: Retire the device from the IoT Hub

@@@secondary
Once the enrollment group has been removed from the Device Provisioning Service (DPS), the device registration will still exist within Azure IoT Hub. To fully retire the devices, you will need to remove that registration as well.
@@@

1. [ ] Within the Azure portal, on your resource group tile, click **AZ-220-HUB-_{YOUR-ID}_** to navigate to the Azure IoT Hub.

1. [ ] On the **IoT Hub** blade, on the left side, under the **Explorers** section, click on the **IoT devices** link.

1. [ ] Notice that the **simulated-device1** device ID still exists within the Azure IoT Hub device registry.

1. [ ] To remove the device, check box next to it in the list, then click the **Delete** button at the top of the pane.

1. [ ] When prompted with "_Are you certain you wish to delete selected device(s)_", click **Yes** to confirm and perform the deletion.

##### Task 3: Verify the retirement

@@@secondary
With the group enrollment deleted from the Device Provisioning Service, and the device deleted from the Azure IoT Hub device registry, the device(s) have fully been removed from the solution.
@@@

1. [ ] Run the simulated device again by executing the `dotnet run` command within the Visual Studio Code **Terminal** window.

    Now that the group enrollment and registered device have been deleted, the simulated device will no longer be able to provision nor connect. When the application attempts to use the configured X.509 certificate to connect to DPS, it will return a `ProvisioningTransportException` error message.

    ```txt
    Found certificate: AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1; PrivateKey: True
    Using certificate AFF851ED016CA5AEB71E5749BCBE3415F8CF4F37 CN=simulated-device1
    RegistrationID = simulated-device1
    ProvisioningClient RegisterAsync . . . Unhandled exception. Microsoft.Azure.Devices.Provisioning.Client.ProvisioningTransportException: {"errorCode":401002,"trackingId":"df969401-c766-49a4-bab7-e769cd3cb585","message":"Unauthorized","timestampUtc":"2019-12-20T21:30:46.6730046Z"}
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.ValidateOutcome(Outcome outcome)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterDeviceAsync(AmqpClientConnection client, String correlationId, DeviceRegistration deviceRegistration)
       at Microsoft.Azure.Devices.Provisioning.Client.Transport.ProvisioningTransportHandlerAmqp.RegisterAsync(ProvisioningTransportRegisterMessage message, CancellationToken cancellationToken)
    ```

    You have completed the registration, configuration, and retirement as part of the IoT devices life cycle with Device Provisioning Service.
