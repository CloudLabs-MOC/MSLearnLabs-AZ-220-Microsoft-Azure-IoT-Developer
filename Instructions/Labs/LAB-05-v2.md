# Lab 05: Integrate IoT Hub with Event Grid

## Lab Overview 

In this lab, you will create a Logic App resource in Azure and configure it to trigger an action when a new IoT device is created. You will start by creating a Logic App and setting up an HTTP trigger that responds to device creation events. After configuring the app to send an automated email using Outlook.com, you will set up an event subscription in Azure IoT Hub that listens for device creation events. Finally, you'll test the setup by adding new IoT devices, verifying that the Logic App successfully sends the email alerts.

## Lab Scenario

Contoso management is impressed with the prototype solutions that you've created using Azure IoT services, and they feel comfortable assigning additional budget the capabilities that you have already demonstrated. They are now asking that you explore the integration of certain operational support capabilities. Specifically, they would like to see how the Azure tools support sending alert notifications to the managers who are responsible for specific work areas. Alert criteria will be defined by the business area managers. The telemetry data arriving at IoT hub will be evaluated to generate the notifications.

You've identified a business manager, Nancy, that you've had success working with in the past. You'll work with her during the initial phase of your solution.

Nancy informs you that her team of facility technicians is responsible for installing the new connected thermostats that will be used to monitor temperature across different cheese caves. The thermostat devices function as IoT devices that can be connected to IoT hub. To get your project started, you agree to create an alert that will generate a notification when a new device has been implemented.

To generate an alert, you will push a device created event type to Event Grid when a new thermostat device is created in IoT Hub. You will create a Logic Apps instance that reacts to this event (on Event Grid) and which will send an email to alert facilities when a new device has been created, specifying the device ID and connection state.

## Lab Objectives

In this lab, you will complete the following:

- Task 1: Create a Logic App resource in the Azure portal
- Task 2: Configure Your Logic App
- Task 3: Configure Azure IoT Hub Event Subscription
- Task 4: Test Your Logic App with New Devices

## Estimated Duration: 120 minutes

## Architecture Diagram

![Lab 9 Architecture](media/LAB_AK_09-architecture-1.png)

### Task 1: Create a Logic App resource in the Azure portal

In this task, you create a new Logic App in Azure. You start by navigating to the Azure portal, searching for "Logic App," and selecting it from the marketplace. After choosing the "Consumption" option for the app, you configure its basic settings, including the subscription, resource group, app name, and region. Once the configuration is set, you review and create the Logic App. After the deployment is complete, you access the newly created app through the resource page.

1. On the Azure portal menu, click **+ Create a resource**.

      ![](media/1lab11.png)
   
1. On the **Search services and marketplace** box, enter **logic app**. From the search results, select **Logic App**.

      ![](media/9lab1.png)

1. Select the **Logic App** from below results.

      ![](media/9lab2.png)

1. On the **Logic App** blade, click **Create**.

      ![](media/9lab3.png)

1. Under the **Create Logic App** select **Consumption** and click on **Select**.

      ![](media/9lab4.png)

1. On the **Basics** tab, under **Create Logic App (Multi-tenant)** with the following settings, click on **Review + create**. Click on **Create**.

    | Setting | Value |
    | --- | --- |
    | Subscription | **Default Subscription (1)**  |
    | Resource group | **Existing Resource group (2)** |
    | Logic App name | **logic-az220-training-<inject key="DeploymentID" enableCopy="false"/> (3)** |
    | Region | **Default region (4)** |
    | Enable Log Analytics | **No (5)** |

    ![](media/az-5-1.png)

    > **Note**: It will take a minute or two for the Logic App deployment to complete.

1. After the deployment is completed, click on **Go to resource**.

1. Here, you can see the newly created Logic app.

      ![](media/9lab6.png)

### Task 2: Configure Your Logic App

In this task, you configure a Logic App in Azure to send an email notification when an HTTP request is received. After setting up the "When a HTTP request is received" trigger, you input a sample JSON payload to generate a schema. Then, you add an action to send an email using Outlook.com, incorporating dynamic content from the event like event type and time. Finally, you save the Logic App and copy the HTTP POST URL, which serves as the Webhook trigger.

1. On the **Logic App** blade, navigate to the **Logic apps designer** under Development Tools and click on **Add a trigger**.

      ![](media/9lab7.png)

1. On the **Add a trigger** section, Search for **When a HTTP request is received (1)** and select it from the results **(2)**.

      ![](media/9lab8.png)

1. Notice that the visual designer opens with the **When a HTTP request is received** trigger selected. Click the trigger to open the details.

1. On the **When a HTTP request is received** trigger, under the **Request Body JSON Schema** textbox, click the **Use sample payload to generate schema** link.

      ![](media/9lab9.png)   

      > **Note**: In the next step you will be adding the **DeviceCreated** sample event schema to the Request Body JSON Schema textbox. This sample, along with a couple of other event schema samples and some associated documentation, can be found at the following link for those who want to learn more: [Azure Event Grid event schema for IoT Hub](https://docs.microsoft.com/en-us/azure/event-grid/event-schema-iot-hub)
      
      > **Note**: In the JSON replace the **id, subscription ID, resource group name, hub name** with the actual values. 

1. Copy the following sample JSON to the **Enter or paste a sample JSON payload.** textbox, and then click **Done**.

    ```json
    [{
      "id": "56afc886-767b-d359-d59e-0da7877166b2",
      "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
      "subject": "devices/LogicAppTestDevice",
      "eventType": "Microsoft.Devices.DeviceCreated",
      "eventTime": "2018-01-02T19:17:44.4383997Z",
      "data": {
        "twin": {
          "deviceId": "LogicAppTestDevice",
          "etag": "AAAAAAAAAAE=",
          "deviceEtag": "null",
          "status": "enabled",
          "statusUpdateTime": "0001-01-01T00:00:00",
          "connectionState": "Disconnected",
          "lastActivityTime": "0001-01-01T00:00:00",
          "cloudToDeviceMessageCount": 0,
          "authenticationType": "sas",
          "x509Thumbprint": {
            "primaryThumbprint": null,
            "secondaryThumbprint": null
          },
          "version": 2,
          "properties": {
            "desired": {
              "$metadata": {
                "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
              },
              "$version": 1
            },
            "reported": {
              "$metadata": {
                "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
              },
              "$version": 1
            }
          }
        },
        "hubName": "egtesthub1",
        "deviceId": "LogicAppTestDevice"
      },
      "dataVersion": "1",
      "metadataVersion": "1"
    }]
    ```

      > **Note**: The **Enter or paste a sample JSON payload** field is a rich editor that automatically inserts opening and closing braces, etc. In the LODS environment, if the "type text" option is used to copy the JSON above directly into the **Enter or paste a sample JSON payload** field, extra braces will be added and the content will be invalid. Instead, open **Notepad** within the LODS VM first, and then send the text to **Notepad**. From there, you can copy the text into the field without error.

1. Notice that the **Request Body JSON Schema** textbox is now populated with a JSON schema that was automatically generated based on the sample JSON that you provided.

      ![](media/9lab11.png)

1. Below the **When a HTTP request is received** trigger, click **+(1)** and select **Add an action(2)**.

      ![](media/9lab12.png)

1. Click **Add an operation** and in the search textbox, enter **Outlook.com**

1. In the list of Actions, search for **Send an email (V2)** and scroll down to the **Office 365 Outlook**, and then click **Send an email (V2)**.

      ![](media/az5-4.png)

      > **Note**:  These instructions walk through configuring the Logic App to send an email using an **Outlook.com** email address. Alternatively, the Logic App can also be configured to send email using the Office 365 Outlook or Gmail connectors as well.

1. On the **Outlook.com** connector, click **Sign in**, and then follow the prompts to authenticate with an existing Outlook.com account.

      ![](media/9lab14.png)

     > **Note**: If **The browser has blocked the popup window** this pop up appears, at top of the page select **Always allow pop-ups and redirects from hhtps://portal.azure.com** and select **Done**.

      ![](media/az70011.png)

1. When prompted to sign in with your credentials.

   - **Email/Username:** <inject key="AzureAdUserEmail"></inject>

        ![](media/9lab15.png)
 
   - **Password:** <inject key="AzureAdUserPassword"></inject>

       ![](media/9lab16.png)

1. If prompted to **stay signed in**, you can click **No**.

1. If prompted to Let this app access your info? (1 of 1 apps) select **Accept**.

1. On the **Send an email (V2)** action, in the **To** field, enter <inject key="AzureAdUserEmail"></inject>.

1. In the **Subject** field, enter **IoT Hub alert(2)**.

1. In the **Body** field, enter the following message content **(3)**:

    ```text
    This is an automated email to inform you that:

    {eventType} occurred at {eventTime}

    IoT Hub: {hubName}
    Device ID: {deviceID}
    Connection state: {connectionState}
    ```
    ![](media/az-5-4.png)

1. To replace the values, click on the the **Dynamic Content** icon shown in the below image.

      ![](media/az5-5.png)

1. Type **eventType (1)** search box and **select (2)** it .

      ![](media/az5-3.png)

1. Repeat the same step by replacing the **curly-braces entries** with the actual **Dynamic content values**.

      > **Note**: When you add the first dynamic content value, because the input data schema is for an array, the Logic Apps Designer will automatically change the e-mail action to be nested inside of a **For each** action. When this happens, the **Send an email (V2)** action will collapse. To reopen your email message, click **Send an email (V2)**, and then continue editing the message body.

      ![](media/az5-6.png)

1. At the top of the designer, to save all changes to the Logic App Workflow, click on **Save** .

     ![](media/az5-7.png)

1. To expand the _When a HTTP request is received_ trigger, click on **When a HTTP request is received(1)**. Copy the value for the **HTTP POST URL(2)** in a notepad.

      ![](media/9lab18.png)

    > **Note**: This URL is the Web Hook endpoint that is used to call the Logic App trigger via HTTPS. Notice the **sig** query string parameter and it's value. The **sig** parameter contains the shared access key that is used to authenticate requests to the Web Hook endpoint.

1. Save the URL for future reference in the notepad or in any other text editor.

      > **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
      > - Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task.
      > - If not, carefully read the error message and retry the step, following the instructions in the lab guide. 
      > - If you need any assistance, please contact us at labs-support@spektrasystems.com. We are available 24/7 to help you out.
      
      <validation step="a35059f4-513a-43b9-8c82-28b61dd53339" />

## Task 3: Configure Azure IoT Hub Event Subscription

Azure IoT Hub integrates with Azure Event Grid so that you can send event notifications to other services and trigger downstream processes. You can configure business applications to listen for IoT Hub events so that you can react to critical events in a reliable, scalable, and secure manner. For example, build an application that updates a database, creates a work ticket, and delivers an email notification every time a new IoT device is registered to your IoT hub. In this exercise, you will create an Event Subscription within Azure IoT Hub to setup Event Grid integration that will trigger a Logic App to send an alert email.

1. Search for **Iot Hub** and select it. Select **iot-az220-training-<inject key="DeploymentID" enableCopy="false"/>**.

      ![](media/az5-2.png)

1. On the **IoT Hub** blade, on the left side navigation menu, select **Events (1)**. On the **Events** pane, at the top, click **+ Event Subscription (2)**.

      ![](media/az5-1.png)

1. On the Create Event Subscription blade, provide the following settings (leave others with their defaults):

    | Setting | Value |
    | --- | --- |
    | Name | **MyDeviceCreateEvent (1)** |
    | EventSchema | **Event Grid Schema (2)** |
    | TOPIC DETAILS >> System Topic Name | **device-creation (3)** |
    | EVENT TYPES >> Filter to Event Types dropdown | De-select all of the choices except **Device Created(4)** |
    | ENDPOINT DETAILS >> Endpoint Type dropdown | **Web Hook (5)** |
    | ENDPOINT DETAILS | Click on **Configure an endpoint (6)** |
    | Select Web Hook | Paste the **URL** **(7)** that you copied from your logic app, then click **Confirm Selection(8)**. |

      ![](media/az5-8.png)

1. At the top of the pane, click on **Filters**. Under **ADVANCED FILTERS**, click on **Add new filter(1)**, and then fill in the fields with these values:

    | Setting | Value |
    | --- | --- |
    | Key | **Subject (2)** |
    | Operator | Select **String begins with (3)** |
    | Value | Enter **devices/sensor-th (4)** |

1. To save the event subscription, click on **Create (5)**.

    ![](media/9lab22.png)

## Task 4: Test Your Logic App with New Devices

In this task, you will test your Logic App by creating a new device to trigger an event notification email.

1. On your Azure portal, navigate to your Iot Hub **iot-az220-training-<inject key="DeploymentID" enableCopy="false"/>** blade if it is not displaying.

1. On the left side navigation menu, under **Device Management**, click on **Devices(1)**. At the top of the IoT devices blade, click on **+ Add Device(2)**.

    ![](media/9lab23.png)

      > **Note**: If you see error loading devices tab, refresh the azure portal page.

1. In the **Device ID** field, enter **sensor-th-0050 (1)**. Leave all other fields at the defaults, and then click **Save (2)**.

    ![](media/9lab24.png)

1. To test the event subscription filters, create additional devices using the following device IDs:

    * `sensor-v-3002`
    * `sensor-th-0030`
    * `sensor-v-3003`

    If you added the four examples total, your list of IoT devices should look like the following image:

    ![](media/9lab25.png)

1. Copy the following link in a private window and sign with your **Username** and **Password** available in the VM's Environment tab.
   
    ```text
    https://outlook.office365.com/mail/
    ```
    ![](media/az5-9.png)

     > **Note**: You can see that only the devices starting with **sensor-th** have been triggered, as this is the configuration we set up in our event subscription.
   
      > **Congratulations** on completing the task! Now, it's time to validate it. Here are the steps:
      > - Hit the Validate button for the corresponding task. If you receive a success message, you can proceed to the next task.
      > - If not, carefully read the error message and retry the step, following the instructions in the lab guide. 
      > - If you need any assistance, please contact us at labs-support@spektrasystems.com. We are available 24/7 to help you out.

      <validation step="95ed4346-5887-405f-be41-8a58ea461960" />

## Summary

In this lab, you have created an Azure Logic App to automatically send emails based on IoT events. You have configured an Azure IoT Hub event subscription to trigger the Logic App and set up new IoT devices to generate events that activate the workflow.

## You have successfully completed the lab!
