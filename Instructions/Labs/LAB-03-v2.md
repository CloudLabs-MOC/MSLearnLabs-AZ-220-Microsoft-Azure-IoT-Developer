# Lab 03: Individual Enrollment of a Device in DPS

## Lab Overview

In this lab, you will begin by reviewing the lab prerequisites and you will run a script if needed to ensure that your Azure subscription includes the required resources. You will then create a new individual enrollment in DPS that uses Symmetric Key attestation and specifies an initial Device Twin State (telemetry rate) for the device. With the device enrollment saved, you will go back into the enrollment and get the auto-generated Primary and Secondary keys needed for device attestation. Next, you create a simulated device and verify that device connects successfully with IoT hub and that the initial device twin properties are applied by the device as expected. To finish up, you will complete a deprovisioning process that securely removes the device from your solution by both disenrolling and deregistering the device (from DPS and IoT hub respectively).

## Lab Scenario

Contoso management is pushing for an update to their existing Asset Monitoring and Tracking Solution. The update will use IoT devices to reduce the manual data entry work that is required under the current system and provide more advanced monitoring during the shipping process. The solution relies on the ability to provision IoT devices when shipping containers are loaded and deprovision the devices when the container arrives at the destination. The best option for managing the provisioning requirements appears to be the IoT Hub Device Provisioning Service (DPS).

The proposed system will use IoT devices with integrated sensors for tracking the location, temperature, and pressure of shipping containers during transit. The IoT devices will be placed within the existing shipping containers that Contoso uses to transport their cheese, and will connect to Azure IoT Hub using vehicle-provided WiFi. The new system will provide continuous monitoring of the product environment and enable a variety of notification scenarios when issues are detected. The rate at which telemetry is sent to IoT hub must be configurable.

In Contoso's cheese packaging facility, when an empty container enters the system it will be equipped with the new IoT device and then loaded with packaged cheese products. The IoT device will be auto-provisioned to IoT hub using DPS. When the container arrives at the destination, the IoT device will be retrieved and must be fully deprovisioned (disenrolled and deregistered). The recovered devices will be recycled and re-used for future shipments following the same auto-provisioning process.

## Lab Objectives

In this lab, you will perform:
 - Exercise 1: Create new individual enrollment (Symmetric keys) in DPS
 - Exercise 2: Configure Simulated Device
 - Exercise 3: Test the Simulated Device
 - Exercise 4: Deprovision the Device

## Estimated Duration: 120 minutes

## Architecture Diagram

![Lab 5 Architecture](media/LAB_AK_05-architecture.png)

## Exercise 1: Create new individual enrollment (Symmetric keys) in DPS

In this exercise, you will create a new individual enrollment for a device within the Device Provisioning Service (DPS) using _symmetric key attestation_. You will also configure the initial device state within the enrollment. After saving your enrollment, you will go back in and obtain the auto-generated attestation Keys that get created when the enrollment is saved.

### Task 1: Create the enrollment

In this task, you will create an individual enrollment in the Device Provisioning Service.

1. On the Azure portal, navigate to Resource group and then select the resource group named **az220rg-<inject key="DeploymentID" enableCopy="false" />.**

   ![](./media/v2img1.png)

1. Select the **dps-az220-training-<inject key="DeploymentID" enableCopy="false" />** from the resource list.

   ![](./media/az-3-1.png)

1. On the left-side menu under **Settings**, click on **Manage enrollments(1)** then navigate to **Individual enrollments(2)** and click on **+ Add individual enrollment(3)**.

   ![](./media/az-3-2.png)

1. On the **Add Enrollment** blade under **Registration + provisioning**, fill the details as follows: 

    | Settings | Values |
    |  -- | -- |
    | **Attestation** | Choose **Symmetric Key** **(1)** from dropdown |
    | **Registration ID** | Enter **sensor-thl-1000** **(2)** |
    | **Reprovision policy** | Select **Reprovision device and migrate current state** **(3)** |
    | Click on | **Next: IoT hubs >** **(4)** |

      ![](./media/az-3-3.png)
   

1. On the **Add Enrollment** blade under **IoT hubs**, fill the details as follows:

     | Settings | Values |
     |  -- | -- |
     | **Target IoT hubs** | Select your **IoT hub** **(1)** from dropdown |
     | **Allocation policy** | Select **Evenly weighted distribution** **(2)** |
     | Click on | **Next: Device settings >** **(3)** |

      ![](./media/az-3-41.png)
   
1. Under **Device settings** in the **Desired properties** field delete the existing content and add the below json data. This field contains JSON data that represents the initial configuration of desired properties for the device. The data that you entered will be used by the Device to set the time delay for reading sensor telemetry and sending events to IoT Hub.

    The final JSON will be like the following:

    ```json
    {
      "telemetryDelay": "2"
    }
    ```
    ![](./media2/lab03updatedimg1.png)

1. Click on **Review + create** and select **Create** after validation is successful.

### Task 2: Review Enrollment and Obtain Authentication Keys

In this task, you will be reviewing the enrollment created and obtain the keys for further tasks.

1. On the **Manage enrollments(1)** pane, to view the list of individual device enrollments then click on **Individual enrollments(2)**.

    ![](./media/az-3-17.png)

1. Under **Registration ID**, click on **sensor-thl-1000**. This blade enables you to view the enrollment details for the individual enrollment that you just created.

1. Copy the **Primary Key** and **Secondary Key** values for this device enrollment, and then paste them in any text editor such as notepad for later use.

   ![](./media/lab5img10.png)

## Exercise 2: Configure Simulated Device

In this exercise, you will configure a Simulated Device written in C# to connect to Azure IoT using the individual enrollment created in the previous exercise. You will also add code to the Simulated Device that will read and update device configuration based on the device twin within Azure IoT Hub.

The simulated device that you create in this exercise represents an IoT device that will be located within a shipping container/box, and will be used to monitor Contoso products while they are in transit. The sensor telemetry from the device that will be sent to Azure IoT Hub includes Temperature, Humidity, Pressure, and Latitude/Longitude coordinates of the container. The device is part of the overall asset tracking solution.

> **Note**: You may have the impression that creating this simulated device is a bit redundant with what you created in the previous lab, but the attestation mechanism that you implement in this lab is quite different from what you did previously. In the previous lab, you used a shared access key to authenticate, which does not require device provisioning, but also does not give the provisioning management benefits (such as leveraging device twins), and it requires fairly large distribution and management of a shared key. In this lab, you are provisioning a unique device through the Device Provisioning Service.

### Task 1: Create the Simulated Device

In this task, you will be creating the simulating device using the dotnet project.

1. On the left-side menu of the **dps-az220-training-<inject key="DeploymentID" enableCopy="false" />** blade, click on **Overview(1)**. In the top-right area of the blade, hover the mouse pointer over value assigned to **ID Scope(2)** then click on **Copy to clipboard** and then paste it in a Notepad for later use.

    ![](./media/az-3-61.png)

1. To open Visual Studio Code, locate the **Visual Studio Code** icon on your desktop. Double-click the icon to launch the application.

   ![](./media/new-az-220-lab3-1.png)

1. On the **File(1)** menu, click on **Open Folder(2)**.

   ![](./media/az-3-22.png)
   
1. Navigate to `C:\LabFiles\az-220\MSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer-stage-rowancollege\Allfiles\Labs\05-Individual Enrollment of Device in DPS\Starter\ContainerDevice` press Enter and then click on **Select folder**.

   ![](./media/az-3-8.png)

1. If the pop up appears click on **Yes, I trust the authors**.

   ![](./media/az-3-9.png)

1. Open integrated Terminal in **Visual studio code** click on **Three dots(...) >> Terminal(1)** and then **New Terminal(2).**

   ![](./media/az-3-10.png)
   
1.  At the Terminal command prompt, to restore all the application NuGet packages, enter the following command:

    ```cmd/sh
    dotnet restore
    ```

1. In the Visual Studio Code **explorer** pane, click on **Program.cs**.

1. In the code editor, near the top of the Program class, locate the **dpsIdScope** variable.

1. Update the following values:
   - Update the value assigned to **dpsIdScope (1)** using the ID Scope that you copied from the Device Provisioning Service.
   - Locate the **registrationId** variable, and update the assigned value using **sensor-thl-1000 (2)**.
   - Update the **individualEnrollmentPrimaryKey (3)** and **individualEnrollmentSecondaryKey (4)** variables using the **Primary Key** and **Secondary Key** values that you copied in Exercise 1 Task 2.

     ![](./media/az-3-23.png)    

### Task 2: Add the provisioning code

In this task, you will be adding the code to the dotnet project for provisioning and also implement the code that provisions the device via DPS and creates a DeviceClient instance that can be used to connect to the IoT Hub.

1. Take a minute to scan through the code in the **Program.cs** file.

1. In the code editor, locate the `// INSERT Main method below here` comment.

1. To create the **Main** method for your simulated device application, enter the following code:

    ```csharp
    public static async Task Main(string[] args)
    {

        using (var security = new SecurityProviderSymmetricKey(registrationId,
                                                                individualEnrollmentPrimaryKey,
                                                                individualEnrollmentSecondaryKey))
        using (var transport = new ProvisioningTransportHandlerAmqp(TransportFallbackType.TcpOnly))
        {
            ProvisioningDeviceClient provClient =
                ProvisioningDeviceClient.Create(GlobalDeviceEndpoint, dpsIdScope, security, transport);

            using (deviceClient = await ProvisionDevice(provClient, security))
            {
                await deviceClient.OpenAsync().ConfigureAwait(false);

                // INSERT Setup OnDesiredPropertyChanged Event Handling below here

                // INSERT Load Device Twin Properties below here

                // Start reading and sending device telemetry
                Console.WriteLine("Start reading and sending device telemetry...");
                await SendDeviceToCloudMessagesAsync(deviceClient);

                await deviceClient.CloseAsync().ConfigureAwait(false);
            }
        }
    }
    ```

     ![](./media/az-3-vs1.png)    
    

1. Locate the `// INSERT ProvisionDevice method below here` comment.

1. To create the **ProvisionDevice** method, enter the following code:

    ```csharp
    private static async Task<DeviceClient> ProvisionDevice(ProvisioningDeviceClient provisioningDeviceClient, SecurityProviderSymmetricKey security)
    {
        var result = await provisioningDeviceClient.RegisterAsync().ConfigureAwait(false);
        Console.WriteLine($"ProvisioningClient AssignedHub: {result.AssignedHub}; DeviceID: {result.DeviceId}");
        if (result.Status != ProvisioningRegistrationStatusType.Assigned)
        {
            throw new Exception($"DeviceRegistrationResult.Status is NOT 'Assigned'");
        }

        var auth = new DeviceAuthenticationWithRegistrySymmetricKey(
            result.DeviceId,
            security.GetPrimaryKey());

        return DeviceClient.Create(result.AssignedHub, auth, TransportType.Amqp);
    }
    ```

     ![](./media/az-3-vs2.png)    

1. Locate the **SendDeviceToCloudMessagesAsync** method.

1. At the bottom of the **SendDeviceToCloudMessagesAsync** method, notice the call to `Task.Delay()`.

1. Near the top of the **Program** class, locate the **telemetryDelay** variable declaration. Notice that the default value for the delay is set to **2** second. Your next step is to integrate the code that uses a device twin value to control the delay time.

### Task 3: Integrate Device Twin Properties

In order to use the device twin properties (from Azure IoT Hub) on a device, you need to create the code that accesses and applies the device twin properties. In this case, you want to update your simulated device code to read the telemetryDelay device twin Desired Property, and then assign that value to the corresponding **telemetryDelay** variable in your code. You also want to update the device twin Reported Property (maintained by IoT Hub) to have a record of the delay time that is currently implemented on our device.

1. In the Visual Studio Code editor, locate the **Main** method.

1. Locate the `// INSERT Setup OnDesiredPropertyChanged Event Handling below here` comment.

1. To set up the DeviceClient for an OnDesiredPropertyChanged event, enter the following code:

    ```csharp
    await deviceClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).ConfigureAwait(false);
    ```

     ![](./media/az-3-vs3.png)    

1. Locate the `// INSERT OnDesiredPropertyChanged method below here` comment.

1. To create the **OnDesiredPropertyChanged** method, enter the following code:

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
    {
        Console.WriteLine("Desired Twin Property Changed:");
        Console.WriteLine($"{desiredProperties.ToJson()}");

        // Read the desired Twin Properties
        if (desiredProperties.Contains("telemetryDelay"))
        {
            string desiredTelemetryDelay = desiredProperties["telemetryDelay"];
            if (desiredTelemetryDelay != null)
            {
                telemetryDelay = int.Parse(desiredTelemetryDelay);
            }
            // if desired telemetryDelay is null or unspecified, don't change it
        }

        // Report Twin Properties
        var reportedProperties = new TwinCollection();
        reportedProperties["telemetryDelay"] = telemetryDelay.ToString();
        await deviceClient.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        Console.WriteLine("Reported Twin Properties:");
        Console.WriteLine($"{reportedProperties.ToJson()}");
    }
    ```

     ![](./media/az-3-vs4.png)    

1. In the **Main** method, locate the `// INSERT Load Device Twin Properties below here` comment.

1. To read the device twin desired properties and configure the device to match on device startup, enter following code:

    ```csharp
    var twin = await deviceClient.GetTwinAsync().ConfigureAwait(false);
    await OnDesiredPropertyChanged(twin.Properties.Desired, null);
    ```

     ![](./media/az-3-vs5.png)    

1. On the Visual Studio Code **File** menu, click on **Save**. Your simulated device will now use the device twin properties from Azure IoT Hub to set the delay between telemetry messages.

    > **Tip**: When inserting code, the code layout may not be ideal. You can have Visual Studio Code format the document for you by right-clicking in the code editor pane and then clicking **Format Document**. You can achieve the same result by opening the **Task** pane (press **F1**) and typing **Format Document** and then pressing **Enter**. And on Windows, the shortcut for this task is **SHIFT+ALT+F**. If it asks to install the formatter. You need to install it and then use format document using the formatter.

## Exercise 3: Test the Simulated Device

In this exercise, you will run the Simulated Device and verify that it's sending sensor telemetry to Azure IoT Hub. You will also change the rate at which telemetry is sent to Azure IoT Hub by updating the telemetryDelay device twin property for the simulated device within Azure IoT Hub.

### Task 1: Build and run the device

In this task you will build the dotnet project and run to send the telemetry data.

1. Ensure that you have your code project open in Visual Studio Code. click on **Terminal(1)** and then **New Terminal(2)**.

   ![](./media/az-3-24.png)

1. In the Terminal pane, ensure the command prompt shows the directory path for the `Program.cs` file.

1. At the command prompt, to build and run the Simulated Device application, enter the following command:

    ```cmd/sh
    dotnet run
    ```

    > **Note**: When the Simulated Device application runs, it will first write some details about it's status to the console (terminal pane).

1. Notice that the JSON output following the `Desired Twin Property Changed:` line contains the desired value for the `telemetryDelay` for the device. You can scroll up in the terminal pane to review the output. It should be similar to the following:

    ```text
    ProvisioningClient AssignedHub: iot-az220-training-{your-id}.azure-devices.net; DeviceID: sensor-thl-1000
    Desired Twin Property Changed:
    {"$version":1,"telemetryDelay":"2"}
    Reported Twin Properties:
    {"telemetryDelay":"2"}
    Start reading and sending device telemetry...
    ```

1. Notice that the Simulated Device application begins sending telemetry events to the Azure IoT Hub. The telemetry events include values for `temperature`, `humidity`, `pressure`, `latitude`, and `longitude`, and should be similar to the following:

    ```text
    11/6/2019 6:38:55 PM > Sending message: {"temperature":25.59094770373355,"humidity":71.17629229611545,"pressure":1019.9274696347665,"latitude":39.82133964767944,"longitude":-98.18181981142438}
    11/6/2019 6:38:57 PM > Sending message: {"temperature":24.68789062681044,"humidity":71.52098010830628,"pressure":1022.6521258267584,"latitude":40.05846882452387,"longitude":-98.08765031156229}
    11/6/2019 6:38:59 PM > Sending message: {"temperature":28.087463226675737,"humidity":74.76071353757787,"pressure":1017.614206096327,"latitude":40.269273772972454,"longitude":-98.28354453319591}
    11/6/2019 6:39:01 PM > Sending message: {"temperature":23.575667940813894,"humidity":77.66409506912534,"pressure":1017.0118147748344,"latitude":40.21020096551372,"longitude":-98.48636739129239}
    ```
    
1. Leave the simulated device app running.

### Task 2: Verify Telemetry Stream sent to Azure IoT Hub

In this task, you will use the Azure CLI to verify telemetry sent by the simulated device is being received by Azure IoT Hub.

1. In the Azure portal, Click on the **Cloudshell** icon to open Cloudshell.

   ![](./media/az-5-9.png)

1. When the **Welcome to Azure Cloud Shell** message is displayed, select **Bash**.

   ![](./media/v2img17.png)

1. Select **No Storage Azzount Required** and  Under **Subscription**, ensure the correct subscription is selected. Click on **Apply**

   ![](./media/v2img18.png)

1. Run the following Azure CLI command. Make sure to replace `{IoTHubName}` with the actual name it looks similar to **iot-az220-training-<inject key="DeploymentID" enableCopy="false" />**.

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} 
    ```

    _Be sure to replace the **{IoTHubName}** placeholder with the name of your Azure IoT Hub._

    > **Note**: After entering the command
    > - If prompted **The command requires the extension azure-iot. Do you want to install it now? The command will continue to run after the extension is installed. (Y/n): Y**.
    > - If prompted **Dependency update (uamqp 1.2) required for IoT extension version: 0.24.0. 
Continue? (y/n) -> y.**

     ![](./media/az-3-20.png)

### Task 3: Change the device configuration through its twin

In this task, you will be changing the twin property and will verify that device will automaticcly accomodate the changes made. With the simulated device running, the `telemetryDelay` configuration can be updated by editing the device twin Desired State within Azure IoT Hub. This can be done by configuring the Device in the Azure IoT Hub within the Azure portal.

1. Open the Azure portal (if it is not already open), and then navigate to your Azure IoT Hub service **iot-az220-training-<inject key="DeploymentID" enableCopy="false" />**

     ![](./media/az-3-21.png)

1. On the IoT Hub blade, on the left-side menu under **Device Management**, click on **Devices(1)** and then click on **sensor-thl-1000(2)** under **Device ID**,

     ![](./media/az-3-14.png)

1. On the **sensor-thl-1000** device blade, at the top of the blade, click on **Device Twin**. The **Device twin** blade provides an editor with the full JSON for the device twin. This enables you to view and/or edit the device twin state directly within the Azure portal.

     ![](./media/new-az-220-lab3-2.png)

1. Locate the JSON for the `properties.desired` object. This contains the desired state for the device. Notice the `telemetryDelay` property already exists, and is set to `"2"`, as was configured when the device was provisioned based on the Individual Enrollment in DPS.

     ![](./media/new-az-220-lab3-3.png)

     ![](./media/az-3-16.png)

1. To update the value assigned to the `telemetryDelay` desired property, change the value to `"5"`.

1. At the top of the **Device twin** blade, click on **Save**. The `OnDesiredPropertyChanged` event will be triggered automatically within the code for the Simulated Device, and the device will update its configuration to reflect the changes to the device twin Desired state.

1. Switch to the Visual Studio Code window that you are using to run the simulated device application.

1. In Visual Studio Code, scroll to the bottom of the Terminal pane.

1. Notice that the device recognizes the change to the device twin properties. The output will show a message that the `Desired Twin Property Changed` along with the JSON for the new desired`telemetryDelay` property value. Once the device picks up the new configuration of device twin desired state, it will automatically update to start sending sensor telemetry every 5 seconds as now configured.

    ```text
    Desired Twin Property Changed:
    {"telemetryDelay":"5","$version":2}
    Reported Twin Properties:
    {"telemetryDelay":"5"}
    4/21/2020 1:20:16 PM > Sending message: {"temperature":34.417625961088405,"humidity":74.12403526442313,"pressure":1023.7792049974805,"latitude":40.172799921919186,"longitude":-98.28591913777421}
    4/21/2020 1:20:22 PM > Sending message: {"temperature":20.963297521678403,"humidity":68.36916032636965,"pressure":1023.7596862048422,"latitude":39.83252821949164,"longitude":-98.31669969393461}
    ```

1. Switch to the browser page where you are running the Azure CLI command in the Azure Cloud Shell. Ensure that you are still running the `az iot hub monitor-events` command. If it isn't running, re-start the command.

1. Notice that the telemetry events sent to Azure IoT Hub being received at the new interval of 5 seconds.

1. Use **Ctrl+C** to stop both the `az` command and the Simulated Device application.

1. Switch to your browser window for the Azure portal.

1. Close the **Device twin** blade.

1. Still in the Azure Portal, on the **sensor-thl-1000** device blade, click **Device Twin**.

1. Locate the JSON for the `properties.reported` object. This portion of the JSON contains the state reported by the device. Notice the `telemetryDelay` property exists here as well, and is also set to `5`.  There is also a `$metadata` value that shows you when the value was reported data was last updated and when the specific reported value was last updated.

1. Close the **Device twin** blade.

1. Close the simulated device blade, and then close the IoT Hub blade.

## Exercise 4: Deprovision the Device

In your Contoso scenario, when the shipping container arrives at it's final destination, the IoT device will be removed from the container and returned to a Contoso location. Contoso will need to deprovision the device before it can be tested and placed in inventory. In the future the device could be provisioned to the same IoT hub or an IoT hub in a different region. Complete device deprovisioning is an important step in the life cycle of IoT devices within an IoT solution.
In this exercise, you will perform the tasks necessary to deprovision the device from both the Device Provisioning Service (DPS) and Azure IoT Hub. To fully deprovision an IoT device from an Azure IoT solution it must be removed from both of these services.

### Task 1: Disenroll the device from the DPS

In this task, you will be deleteing the device from the enrollments

1. On your Resource group tile, to open your Device Provisioning Service, click on **dps-az220-training-<inject key="DeploymentID" enableCopy="false" />**.

     ![](./media/az-3-1.png)

1. On the left-side menu under **Settings**, click on **Manage enrollments(1)**. On the **Manage enrollments** pane, to view the list of individual device enrollments, click on **Individual Enrollments(2)**.

     ![](./media/az-3-17.png)

1. To the left of **sensor-thl-1000**, click the checkbox. At the top of the blade, click on **Delete**

     ![](./media/az-3-18.png)

    > **Note**: Deleting the individual enrollment from DPS will permanently remove the enrollment. To temporarily disable the enrollment, you can set the **Enable entry** setting to **Disable** within the **Enrollment Details** for the individual enrollment.

1. On the **Remove enrollment** prompt, click on **Yes**. The individual enrollment is now removed from the Device Provisioning Service (DPS). To complete the deprovisioning process, the **Device ID** for the Simulated Device also must be removed from the **Azure IoT Hub** service.

      ![](./media/lab5img8.png)

### Task 2: Deregister the device from the IoT Hub

In this task you will delete the device from the IoT hub device management.

1. In the Azure portal, navigate back to your Dashboard.

1. On your Resource group tile, to open your Azure IoT Hub blade, click on **iot-az220-training-<inject key="DeploymentID" enableCopy="false" />**.

     ![](./media/az-3-21.png)

1. On the left-side menu under **Device management**, click on **Devices**.

1. To the left of **sensor-thl-1000**, click the checkbox.

    > **Note**: Make sure you select the device representing the simulated device that you used for this lab.

1. At the top of the blade, click on **Delete**.

   ![](./media/az-3-19.png)

1. On the **Are you sure you want to delete selected devices?** prompt, click on **Yes**.

    > **Note**:  Deleting the device ID from IoT Hub will permanently remove the device registration. To temporarily disable the device from connecting to IoT Hub, you can set the **Enable connection to IoT Hub** to **Disable** within the properties for the device.

## Summary

In this lab, you have configured the enrollment in the Device Provision Service, built a device which sends the telemetry data to the IoT hub also tested the device by changing configurations and finally deprovisioned device at last.

### You have successfully completed the Lab!