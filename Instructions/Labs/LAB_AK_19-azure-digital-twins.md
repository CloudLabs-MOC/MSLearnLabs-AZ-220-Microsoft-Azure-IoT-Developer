---
lab:
    title: 'Lab 19: Develop Azure Digital Twins solutions'
    module: 'Module 11: Develop with Azure Digital Twins'
---

# Develop Azure Digital Twins solutions

## Lab Scenario

Contoso Management has decided to take the next step in their digital evolution and develop a model of their cheese making facility using Azure Digital Twins. With Azure Digital Twins, it is possible to create and interact with live models of real-world environments. First, each individual element is modeled as a digital twin. Then, these models are connected into a knowledge graph that can respond to live events and be queried for information.

In order to demonstrate the advantages of Azure Digital Twins for Contoso stakeholders, you have been asked to build a proof-of-concept prototype that demonstrates how the telemetry of an existing Cheese Cave device can be incorporated into the following (simplified) model hierarchy:

* Cheese Factory
* Cheese Cave
* Cheese Cave Device

In this first prototype, you have been asked to demonstrate solutions for the following scenarios:

* Mapping an IoT hub device to a corresponding digital twin and ingesting telemetry data from IoT hub into Azure Digital Twins
* Updating the property of a parent digital twin when a child digital twin's property/telemetry is updated (update the currentTemp property of a Cheese Cave twin when the temperature property of a Cheese Cave Device twin changes)
* Routing digital twin telemetry values from the Azure Digital Twins instance to a downstream service (Time Series Insights)

The following resources will be created:

![Lab 19 Architecture](media/LAB_AK_19-architecture.png)

## In This Lab

In this lab, you will complete the following activities:

* Configure the lab prerequisites (the required Azure resources)
* Create and configure digital twins
  * Create a digital twin by using the supplied DTDL
  * Build an Azure Digital Twins graph using digital twin instances
* Implement Azure Digital Twins graph interactions (ADT Explorer)
  * Query the Azure Digital Twins Graph
  * Update properties on Azure Digital Twins entities in the graph
* Integrate Azure Digital Twins with upstream and downstream systems
  * Ingest IoT device messages and translate messages to Azure Digital Twins
  * Configure Azure Digital Twins routes and endpoints to publish telemetry to Time Series Insights (TSI)

## Lab Instructions

### Exercise 1 - Configure Lab Prerequisites

#### Task 1 - Create resources

**Important**: Due to resource/policy restrictions, this lab cannot be run using Azure account credentials provided by the MS Learn lab sandbox. You can complete this lab using a free trial or other personal Azure subscription.

This lab assumes that the following Azure resources are available:

| Resource Type  | Resource Name                |
| :------------- | :--------------------------- |
| Resource Group | rg-az220                     |
| IoT Hub        | iot-az220-training-{your-id} |
| TSI            | tsi-az220-training-{your-id} |
| TSI Access Policy | access1                   |

To ensure these resources are available, complete the following steps.

1. In the virtual machine environment, open a Microsoft Edge browser window, and then navigate to the following Web address:

    +++https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fMicrosoftLearning%2fMSLearnLabs-AZ-220-Microsoft-Azure-IoT-Developer%2fmaster%2fAllfiles%2FARM%2Flab19.json+++

    > **NOTE**: Whenever you see the green "T" symbol, for example +++enter this text+++, you can click the associated text and the information will be typed into the current field within the virtual machine environment.

1. When prompted to Sign in, enter the Azure account credentials associated with your free pass.

    > **NOTE**: If you are not using a free or promotional account, you may be responsible for fees associated with the Azure services that you will be using. We recommend using the suggested Azure resource groups for all resources created during the lab and deleting those resource groups before exiting the lab virtual machine environment. Use the Azure portal outside of the lab environment to verify that the resources have been deleted. This will help to minimize your costs.

    Once you have signed in, the **Custom deployment** page will be displayed.

1. Under **Project details**, in the **Subscription** dropdown, ensure that the Azure subscription that you intend to use for this lab is selected.

1. In the **Resource group** dropdown, select **rg-az220**.

    > **NOTE**: If **rg-az220** is not listed:
    >
    > 1. Under the **Resource group** dropdown, click **Create new**.
    > 1. Under **Name**, enter **rg-az220**.
    > 1. Click **OK**.

1. Under **Instance details**, in the **Region** dropdown, select the region closest to you.

    > **NOTE**: If the **rg-az220** group already exists, the **Region** field is set to the region used by the resource group and is read-only.

1. In the **Your ID** field, enter a unique ID value that includes your initials followed by the current date (using a "YourInitialsYYMMDD" pattern).

    The first part of your unique ID will be your initials in lower-case. The second part will be the last two digits of the current year, the current numeric month, and the current numeric day. For example:

    ccj220101

    During this lab, you will see **{your-id}** listed as part of the suggested resource name whenever you need to enter your unique ID. The **{your-id}** portion of the suggested resource name is a placeholder. You will replace the entire placeholder string (including the **{}**) with your unique value.

1. In the **Course ID** field, enter **az220**.

    +++az220+++

1. To determine the current user object ID, open the **Cloud Shell** and execute the following command:

    ```sh
    az ad signed-in-user show --query objectId -o tsv
    ```

    In the Azure portal, the Cloud Shell button has an icon that resembles a command prompt.

    > **Note**: If the cloud shell has not been configured, follow these steps:

    1. When the **Welcome to Azure Cloud Shell** message is displayed, select **Bash**.

    1. Under **Subscription**, ensure the correct subscription is displayed.

    1. To specify storage options, click **Show advanced settings**.

    1. Under **Resource group**, ensure **Use existing** is selected and select a resource group associated with your account.

        If there is not resource group defined, select **create new** and specify **rg-account**.
 
    1. Under **Storage account**, select **Create new** and enter the following: **stoaz220{your-id}**.

    1. Under **File share**, select **Create new** and enter the following **cloudshell**.

    1. To finish to configuration of the cloud shell, click **Create storage**.

1. Copy the displayed object ID value, and then close the cloud shell.

1. In the **Object ID** field, enter the object ID copied from the above step.

1. To validate the template, click **Review + create**.

1. If validation passes, click **Create**.

    The deployment will start. It will take several minutes to deploy the required Azure resources.

1. While the Azure resources are being created, open a text editor tool (Notepad is accessible from the **Start** menu, under **Windows Accessories**). 

    > **NOTE**: You will be using the text editor to store some configuration values associated with your Azure resources.

1. In your text editor, enter the following text labels:

    +++connectionString:+++

    +++deviceConnectionString:+++

    +++devicePrimaryKey:+++

    +++storageAccountName:+++

1. Switch back to the Azure portal window and wait for the deployment to finish.

    You will see a notification when deployment is complete.

1. Once the deployment has completed, in the left navigation area, click **Outputs**.

1. In your text editor, create a record of the following Outputs for use later in the lab:

    * connectionString
    * deviceConnectionString
    * devicePrimaryKey
    * storageAccountName

    The Azure resources required for this lab are now available.

#### Task 2 - Verify tools

1. In your virtual machine environment, open a Windows Command Prompt window.

1. To display the version of Azure CLI that is installed locally, enter the following command:

    ```powershell
    az --version
    ```

1. Verify that azure-cli version 2.11.0 or later is listed.

    Azure CLI version 2.11 provides the ability to upgrade to the latest version. To upgrade, enter the following command:

        ```powershell
        az upgrade
        ```

    > **NOTE**: If Azure CLI is not installed, you must install it before you continue:

    1. Open your browser, and then navigate to the Azure CLI tools download page: [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest "Azure CLI Install")

        You should be installing the latest version of the Azure CLI tools. The current version of Azure CLI (as of February 2022) is version 2.33, but be aware that new versions are released monthly, so the latest version is likely to have changed.

    1. On the **Install Azure CLI** page, select the install option for your OS (such as **Install on Windows**), and then follow the on-screen instructions to install the Azure CLI tool.

### Exercise 2 - Create an instance of the Azure Digital Twins resource

In this exercise, the Azure portal will be used to create an Azure Digital Twins instance. The connection data for Azure Digital Twins will then be stored in a text file for later use. Finally the current user will be assigned a role to allow the Azure Digital Twins resource to be accessed.

#### Task 1 - Use the Azure portal to create a resource (Azure Digital Twins)

1. Switch to the browser window containing your Azure portal.

1. On the Azure portal menu, click **Dashboard**.

    On the All resources tile, you should see the resources listed that were created for this lab.

1. On the Azure portal menu, click **+ Create a resource**.

    The **New** blade that opens is a front-end to the Azure Marketplace, which is a collection of all the resources you can create in Azure. The marketplace contains resources from both Microsoft and the community.

1. In the **Search the marketplace** text box, type **Azure Digital Twins**

1. When the search results appear, select **Azure Digital Twins**, and then click **Create**.

1. On the **Create Resource** pane, under **Subscription**, ensure that the subscription you are using for this lab is selected.

    > **Note**: Your account must have the administrator role for the subscription

1. For the **Resource group**, select **rg-az220**.

1. For the **Resource name**, enter **adt-az220-training-{your-id}**.

    +++adt-az220-training-{your-id}+++

    > **NOTE**: Be sure to replace the your-id placeholder, including the curly brackets, value with the unique ID that you created earlier.

1. In the **Region** dropdown, select the region where your resource group and Azure IoT hub are provisioned (or the closest region available).

1. Under **Grant access to resource**, to ensure that you can use the **Digital Twins Explorer** app, check **Assign Azure Digital Twins Owner Role**.

    > **Note**: To manage the elements within an instance, a user needs access to Azure Digital Twins data plane APIs. Select the suggested role above grants the current user full access to the data plane APIs. You can also use Access Control (IAM) to choose appropriate roles later. You can learn more about Azure Digital Twins Security [here](https://docs.microsoft.com/azure/digital-twins/concepts-security)

1. To review the values entered, click **Review + create**.

1. To start the deployment process, click **Create**.

    After a moment the **Deployment is in progress** will be displayed. Wait for the deployment to complete.

1. Once your deployment is complete, select **Go to resource**.

    You should see the Overview pane for your Azure Digital Twins resource, which includes a body section titled **Get started with Azure Digital Twins**.

#### Task 2 - Save the connection data to a reference file

1. Take a moment to review the Overview page of your Azure Digital Twins blade.

1. On the left-side navigation menu, under **Settings**, click **Properties**.

1. Copy the values of the **Name** and **Resource group** properties to you text editor (Notepad) and label the values as **ADT name:** and **ADT rg:**

1. Navigate back to the **Overview** page of your Azure Digital Twins instance.

1. In the **Essentials** section of the page, locate the **Host name** field.

1. Hover the mouse pointer over the **Host name** field, use the icon than appears to the right of the value to copy the host name to the clipboard, and then paste it into the text file.

1. In the text file, convert the host name to a connection url to your digital twins instance by adding **https://** to the start of the host name.

    The modified url will be similar to:

    ```http
    https://adt-az220-training-dm030821.api.eus.digitaltwins.azure.net
    ```

1. To label the new value, mark it as **ADT url:**

1. Save the text editor file.

    You can name it something like "adt lab info.txt".

The Azure Digital Twin resource is now created and the user account has been updated so that the resource can be accessed via APIs.

### Exercise 3 - Create a graph of the models

As part of a modeling activity, analysts would consider many factors, such as the Cheese Cave Device message content, and create mappings in DTDL **Property** and **Telemetry** field definitions. In order to use these DTDL code fragments, they would be incorporated into an **Interface** (the top-level code item for a model). However, the **Interface** for a Cheese Cave Device model would form just a small part of the Azure Digital Twins environment for a Contoso Cheese Factory. As modeling an environment that represents an entire factory is beyond the scope of this lab, a greatly simplified environment that focuses on a Cheese Cave Device model, an associated Cheese Cave model, and a Factory model will be considered instead. The model hierarchy is as follows:

* Cheese Factory Interface
* Cheese Cave Interface
* Cheese Cave Device Interface

Considering the hierarchy of Interface definitions above, and the relationships between them, it can be said that **a Cheese Factory has Cheese Caves**, and **a Cheese Cave has Cheese Cave Devices**.

When designing the Digital Twin models for an Azure Digital Twins environment, it is best to use a consistent approach for creating the IDs used for the Interfaces, Schemas, and Relationships. Each entity within the environment has an **@id** property (it is required for Interfaces) and should uniquely identify that entity. The format of the ID value is that of a **digital twin model identifier (DTMI)**. A DTMI has three components: scheme, path, and version. The scheme and path are separated by a colon `:`, while path and version are separated by a semicolon `;`. The format looks like this: `<scheme> : <path> ; <version>`. The value for scheme within the DTMI formatted ID is always **dtmi**.

An example for the ID value of a Contoso cheese factory would be: **dtmi:com:contoso:digital_factory:cheese_factory;1**.

In this example, the scheme value is **dtmi** as expected and the version is set to **1**. The `<path>` component within this ID value utilizes the following taxonomy:

* The source of the model - **com:contoso**
* The model category - **digital_factory**
* The type within the category - **cheese_factory**

> **TIP**: To learn more about the DTMI format, review the following resource:
> * [Digital Twin Model Identifier](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md#digital-twin-model-identifier)

Recalling the model hierarchy and relationships identified above, an example of the IDs used could be:

| Interface                      | ID                                                                     |
| :----------------------------- | :--------------------------------------------------------------------- |
| Cheese Factory Interface       | dtmi:com:contoso:digital_factory:cheese_factory;1                      |
| Cheese Cave Interface        | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        |
| Cheese Cave Device Interface | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 |

And the relationships between IDs could be:

| Relationship | ID                                                                | From ID                                                         | To ID                                                                  |
| :----------- | :---------------------------------------------------------------- | :-------------------------------------------------------------- | :--------------------------------------------------------------------- |
| Has Caves  | dtmi:com:contoso:digital_factory:cheese_factory:rel_has_caves;1 | dtmi:com:contoso:digital_factory:cheese_factory;1               | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        |
| Has Devices  | dtmi:com:contoso:digital_factory:cheese_cave:rel_has_devices;1  | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1 | dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 |

> **NOTE**: The Lab 19 **Models** folder is part of the lab resources that you downloaded before starting this lab. The folder path is:
>
>   * Allfiles
>       * Labs
>           * 19-Azure Digital Twins
>               * Final
>                   * Models
>
>  The complete models referenced in this exercise are available in this folder location.
>
> **Note**: If you have trouble finding the Allfiles folder, check your Windows Desktop folder.

For the purposes of this exercise, the interfaces have already been defined for each of the digital twins that will be used in the proof-of-concept, it is time to construct the actual graph of digital twins. The flow for building a graph is straightforward:

* Import the model definitions
* Create twin instances from the appropriate models
* Create relationships between the twin instances using the defined model relationships

There are a number of ways that this flow can be achieved:

* Using Azure CLI commands from the command-line or in scripts
* Programmatically using one of the SDKs or directly via the REST APIs
* Using tools such as the ADT Explorer sample

As the ADT Explorer includes rich visualization of an Azure Digital Twins graph, it is well suited for building out the simple model for the proof-of-concept. However, larger, more complex, models are also supported and a comprehensive bulk import/export capability helps with iterative design. During this exercise the following tasks will be completed:

* Access the ADT Explorer (Preview) via the Azure Portal
* Import the Contoso Cheese models
* Use the models to create digital twins
* Add relationships to the graph
* Learn how to use delete twins, relationships and models from Azure Digital Twins
* Bulk import a graph into Azure Digital Twins

#### Task 1 - Access the ADT Explorer

The **ADT Explorer** is a an application for the Azure Digital Twins service. The app connects to an Azure Digital Twins instance and provides the following features/capabilities:

* Upload and explore models
* Upload and edit graphs of twins
* Visualize the twins graph with a number of layout techniques
* Edit properties of twins
* Run queries against the twins graph

The ADT Explorer is incorporated into the Azure Portal as a preview feature and is also available as a standalone sample application. In this lab, the version incorporated into the Azure Portal will be used.

1. In your Azure portal window, navigate to the Digital Twins instance **Overview** page.

1. Near the top of the **Overview** page, to open the ADT Explorer in a new browser tab, click **Open Azure Digital Twins Explorer (preview)**.

    A new browser tab hosting the ADT Explorer will open. Enable popups on the Azure Digital Twins Explorer page if prompted to do so.

    > **Important**: If you are prompted to login to Azure, ensure you use the same account that you used when creating the Azure Digital Twins instance, otherwise you will not have access to the data plane APIs and will see errors.

1. On the **Azure Digital Twins URL** popup, enter the value of the Azure Digital Twins URL that you created in Notepad, and then click **Save**.

The **ADT Explorer** sample application is now ready for use. Loading models is your next task, so don't be alarmed if you see an error message telling you that there are no models available.

#### Task 2 - Import Models

In order to create Digital Twins in Azure Digital Twins, it is necessary to first upload models. There are a number of ways that models can be uploaded:

* [Data Plane SDKs](https://docs.microsoft.com/azure/digital-twins/how-to-use-apis-sdks)
* [Data Plane REST APIs](https://docs.microsoft.com/rest/api/azure-digitaltwins/)
* [Azure CLI](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest)
* The import feature of the [ADT Explorer](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/)

The first two options are more appropriate for programmatic scenarios, whereas the Azure CLI can be useful in **configuration as code** scenarios or "one-off" requirements. The **ADT Explorer** app provides an intuitive way to interact with Azure Digital Twins.

> **TIP**: What is **configuration as code**? As configuration is written as source code (for example, scripts containing Azure CLI commands), you can use best development practices to optimise it, such as: creating reusable definitions of model uploads, parameterization, using loops to create multiple instances of the models and so on. These scripts can then be stored in source code control to ensure they are retained, version controlled, etc.

In this task, you will use Azure CLI commands and the ADT Explorer sample app to upload the models included in the **Allfiles\Labs\19-Azure Digital Twins\Final\Models** folder.

1. Open a Windows Command Prompt window.

1. To ensure that you are using correct Azure account credentials, login to Azure using the following command:

    ```powershell
    az login
    ```

1. When prompted, enter the Azure account credentials that you are using for this lab.

1. At the command prompt, to upload the **Cheese Factory Interface**, enter the following command:

    > **NOTE**: You may want to construct the following command in your text editor before entering it at the command prompt.
 
    ```powershell
    az dt model create --models "{file-root}\Allfiles\Labs\19-Azure Digital Twins\Final\Models\CheeseFactoryInterface.json" -n adt-az220-training-{your-id}
    ```

    > **IMPORTANT**: Ensure that you replace the **{file-root}** with the folder location where the resource files for this lab are located, and replace **{your-id}** with your unique identifier.

    By default, the root-file location is: +++C:\Users\Student\Desktop+++

    If successful, output similar to the following will be displayed.

    ```json
    [
        {
            "decommissioned": false,
            "description": {},
            "displayName": {
            "en": "Cheese Factory - Interface Model"
            },
            "id": "dtmi:com:contoso:digital_factory:cheese_factory;1",
            "uploadTime": "2021-03-24T19:56:53.8723857+00:00"
        }
    ]
    ```

1. Return to the **ADT Explorer**.

    > **TIP**: Click the **Refresh** button in the **MODELS** explorer to update the list of models.

    The uploaded **Cheese Factory - Interface Model** should be listed:

    ![ADT Explorer MODEL VIEW with Factory Model](media/LAB_AK_19-modelview-factory.png)

1. To import the remaining two models using the **ADT Explorer**, in the **MODELS** explorer, click the **Upload a Model** icon

    ![ADT Explorer MODEL VIEW Upload a Model button](media/LAB_AK_19-modelview-addmodel.png)

1. In the **Open** dialog, navigate to the **Models** folder, select the **CheeseCaveInterface.json** and the **CheeseCaveDeviceInterface.json** files, and then click **Open**.

    The two files will then be uploaded to Azure Digital Twins and the models added. Once complete, the **MODELS** explorer will update and list all three models.

Now that the models are uploaded, Digital Twins can be created.

#### Task 3 - Creating Twins

In an Azure Digital Twins solution, the entities in your environment are represented by digital twins. A digital twin is an instance of one of your custom-defined models. It can be connected to other digital twins via relationships to form a twin graph: this twin graph is the representation of your entire environment.

Similar to models, digital twins and relationships can be created in multiple ways:

* [Data Plane SDKs](https://docs.microsoft.com/azure/digital-twins/how-to-use-apis-sdks)
* [Data Plane REST APIs](https://docs.microsoft.com/rest/api/azure-digitaltwins/)
* [Azure CLI](https://docs.microsoft.com/cli/azure/ext/azure-iot/dt?view=azure-cli-latest)
* The import feature of the [ADT Explorer](https://docs.microsoft.com/samples/azure-samples/digital-twins-explorer/digital-twins-explorer/)

As before, the first two options are more appropriate for programmatic scenarios, whereas the Azure CLI remains useful in **configuration as code** scenarios or "one-off" requirements. The most intuitive way to create digital twins and relationships is via the **ADT Explorer**, however there are some limitations when it comes to initializing properties.

1. Open the command line window that you used to upload the CheeseFactoryInterface model.

1. To create a digital twin from the Cheese Factory model using the Azure CLI, enter the following command:

    ```powershell
    az dt twin create --dt-name adt-az220-training-{your-id} --dtmi "dtmi:com:contoso:digital_factory:cheese_factory;1" --twin-id factory_1 --properties "{file-root}\Allfiles\Labs\19-Azure Digital Twins\Final\Properties\FactoryProperties.json"
    ```

    Ensure you replace the **{file-root}** with the folder the companion files for this lab are located, and replace **{your-id}** with your unique identifier.

    Notice the following:

    * The **--dt-name** value specifies the Azure Digital Twins twin instance.
    * The **--dtmi** value specifies the Cheese Factory model uploaded earlier
    * The **--twin-id** specifies the ID given to the digital twin
    * The **--properties** value provides a file path for a JSON document that will be used to initialize the twin. Alternatively, simple JSON can also be specified in-line.

    If successful, the output of the command is similar to:

    ```json
    {
        "$dtId": "factory_1",
        "$etag": "W/\"09e781e5-c31f-4bf1-aed4-52a4472b0c5b\"",
        "$metadata": {
            "$model": "dtmi:com:contoso:digital_factory:cheese_factory;1",
            "FactoryName": {
                "lastUpdateTime": "2021-03-24T21:51:04.1371421Z"
            },
            "GeoLocation": {
                "lastUpdateTime": "2021-03-24T21:51:04.1371421Z"
            }
        },
        "FactoryName": "Contoso Cheese 1",
        "GeoLocation": {
            "Latitude": 47.64319985218156,
            "Longitude": -122.12449651580214
        }
    }
    ```

    Notice that the **$metadata** property contains an object that tracks the last time properties were updated.

1. The **FactoryProperties.json** file contains the following JSON:

    ```json
    {
        "FactoryName": "Contoso Cheese 1",
        "GeoLocation": {
            "Latitude": 47.64319985218156,
            "Longitude": -122.12449651580214
        }
    }
    ```

    The property names match the DTDL Property values declared in the Cheese Factory Interface.

    > **NOTE**: The complex property **GeoLocation** is assigned via a JSON object with **Latitude** and **Longitude** properties.

1. In a browser, return to the **ADT Explorer**.

1. To display the digital twins created so far, click **Run Query**.

    > **NOTE**: Queries and the Query Language will be discussed shortly.

    After a few moments, the **factory_1** digital twin should be displayed in the **TWIN GRAPH** view.

    ![ADT Explorer GRAPH VIEW Factory 1](media/LAB_AK_19-graphview-factory_1.png)

1. To view the digital twin properties, in the **TWIN GRAPH** view, click **factory_1**.

    The properties for **factory_1** are displayed in the **Property View** as nodes in a tree view.

1. To view the longitude and latitude property values, click **GeoLocation**.

    Notice that the values are consistent with those in the **FactoryProperties.json** file.

1. To create another digital twin from the Cheese Factory model, in the **MODELS** explorer, locate the **Cheese Factory** model, and then click **Create a Twin**

    ![ADT Explorer MODEL VIEW Create a Twin button](media/LAB_AK_19-modelview-createtwin.png)

1. When prompted for the **New Twin Name** enter **factory_2** and then click **Save**.

1. To view the digital twin properties for **factory_2**, in the **TWIN GRAPH** view, click **factory_2**.

    Notice that the **FactoryName** and **GeoLocation** properties are uninitialized.

1. To set the **factoryName**, position the mouse cursor to the right of the property - a textbox control should appear. Enter **Cheese Factory 2**.

    ![ADT Explorer Property View enter factory name](media/LAB_AK_19-propertyexplorer-factoryname.png)

1. In the **PROPERTIES** Explorer pane, to save the update to the property, select the **Save changes** icon.

    > **Note**: The Save changes icon appears identical to the Save Query icon located to the right of the Run Query button. You don't want the Save Query icon.

    Selecting Save changes will result in a JSON Patch being created and sent to update the digital twin. The **Patch Information** will be displayed in a dialog. Notice that as this is the first time the value has been set, the **op** (operation) property is **add**. Subsequent changes to the value would be **replace** operations - to see this, click **Run Query** to refresh the **TWIN GRAPH** view before making another update.

   > **TIP**: To learn more about a JSON Patch document, review the following resources:
   > * [Javascript Object Notation (JSON) Patch](https://tools.ietf.org/html/rfc6902)
   > * [What is JSON Patch?](http://jsonpatch.com/)

1. In the **PROPERTIES** explorer, examine the **factory_2** **GeoLocation** property - notice the values for **Latitude** and **Longitude** are shown as **not set**.

    > **Info**: Earlier versions of the ADT Explorer did not support editing "sub-properties" via the UI - this feature is a welcome addition.

1. Update the **Latitude** and **Longitude** values as follows:

    | Property name | Value |
    | :-- | :-- |
    | Latitude | +++47.64530450740752+++ |
    | Longitude | +++-122.12594819866645+++ |

1. In the **PROPERTIES** Explorer pane, to save the update to the properties, select the **Save changes** icon.

    Notice that the Patch Information is once again displayed.

1. Add the following digital twins by selecting the appropriate model in **MODELS** explorer and clicking **Create a Twin**:

    | Model Name                             | Digital Twin Name |
    | :------------------------------------- | :---------------- |
    | Cheese Cave - Interface Model        | cave_1          |
    | Cheese Cave - Interface Model        | cave_2          |
    | Cheese Cave Device - Interface Model | device_1          |
    | Cheese Cave Device - Interface Model | device_2          |

    ![ADT Explorer graph view displaying created twins](media/LAB_AK_19-graphview-createdtwins.png)

Now that some twins have been created, it is time to add some relationships.

#### Task 4 - Adding relationships

Twins are connected into a twin graph by their relationships. The relationships that a twin can have are defined as part of its model.

For example, the **Cheese Factory** model defines a "contains" relationship that targets twins of type **Cheese Cave**. With this definition, Azure Digital Twins will allow you to create **rel_has_caves** relationships from any **Cheese Factory** twin to any **Cheese Cave** twin (including any twins that may be subtypes of a **Cheese Cave** - for example a specialized **Cheese Cave** for a specific cheese).

The result of this process is a set of nodes (the digital twins) connected via edges (their relationships) in a graph.

Similar to Models and Twins, relationships can be created in multiple ways.

1. To create a relationship via the Azure CLI, return to the command prompt and execute the following command:

    ```powershell
    az dt twin relationship create -n adt-az220-training-{your-id} --relationship-id factory_1_has_cave_1 --relationship rel_has_caves --twin-id factory_1 --target cave_1
    ```

    Ensure you replace **{your-id}** with your unique identifier.

    If successful, the output of the command is similar to:

    ```json
    {
        "$etag": "W/\"cdb10516-36e7-4ec3-a154-c050afed3800\"",
        "$relationshipId": "factory_1_has_cave_1",
        "$relationshipName": "rel_has_caves",
        "$sourceId": "factory_1",
        "$targetId": "cave_1"
    }
    ```

1. To visualize the relationship, return to the **ADT Explorer** window.

1. To display the updated digital twins, click **Run Query**.

    The diagram will refresh and the new relationship will be displayed.

    ![ADT Explorer graph view with a relationship](media/LAB_AK_19-graphview-relationship.png)

    If you don't see the relationship, refresh the browser window and then run the query.

1. To add a relationship using the **ADT Explorer**, first click **cave_1** to select it, and then **right-click** **device_1**. 

1. In the displayed context menu, select **Add relationships**.

    If the popup menu is cut of at the bottom of the display screen you may need to adjust the disply in order to select the Add relationships command.

1. In the **Create Relationship** dialog, under **Source ID**, confirm that **cave_1** is displayed.

1. Under **Target ID**, confirm that **device_1** is displayed.

1. Under **Relationship**, select **rel_has_devices**.

    > **NOTE**: Unlike relationships created with the Azure CLI, there is no equivalent UI to supply a **$relationshipId** value. Instead, a GUID will be assigned.

1. To Create the relationship, click **Save**.

    The relationship will be created and the diagram will update to display the relationship. The diagram now shows that **factory_1** has **cave_1**, which has **device_1**.

1. Add two more relationships:

    | Source    | Target   | Relationship    |
    | :-------- | :------- | :-------------- |
    | factory_1 | cave_2 | rel_has_caves |
    | cave_2  | device_2 | rel_has_devices |

    The graph should now appear similar to:

    ![ADT Explorer graph view with updated graph](media/LAB_AK_19-graphview-updatedgraph.png)

1. To view the layout options for the **TWIN GRAPH** view, click the **Choose Layout** button.

    ![ADT Explorer graph view choose layout](media/LAB_AK_19-twingraph-chooselayout.png)

    The **TWIN GRAPH** view can use different algorithms to layout the graph. The **Klay** layout is selected by default. You can try selecting different layouts to see how the graph is impacted.

#### Task 5 - Deleting models, relationships and Twins

During the design process for modeling with Azure Digital Twins, it is likely that a number of proof-of-concepts will be created, many of which will be deleted. Similar to the other operations on digital twins, there are programmatic approaches (API, SDK, and CLI) to deleting models and twins, and you can also use the **ADT Explorer**.

> **NOTE**: One thing to note is that the delete operations are asynchronous and although, for example, a REST API call or a delete in **ADT Explorer** may appear to complete instantly, it may take a few minutes for the operation to complete within the Azure Digital Twins service. Attempting to upload revised models with the same name as recently deleted models may fail unexpectedly until the back-end operations have completed.

1. To delete the **factory_2** digital twin via the CLI, return to your command prompt window, and then enter the following command:

    ```powershell
    az dt twin delete -n adt-az220-training-{your-id} --twin-id factory_2
    ```

    Unlike other commands, no output is displayed (unless the command produces an error).

1. To delete the relationship between **factory_1** and **cave_1**, enter the following command:

    ```powershell
    az dt twin relationship delete -n adt-az220-training-{your-id} --twin-id factory_1 --relationship-id factory_1_has_cave_1
    ```

    > **NOTE**: This command requires the relationship ID value for the relationship that you created. To view a relationship ID in the Azure Digital Twins Explorer, select the relationship and review the displayed properties. Update the command above with the relationship ID displayed in the Explorer before running the command.

    You can also use Azure CLI to view relationship IDs for a given twin. For example, to view the relationship IDs for **factory_1**, you can enter the following command:

    ```powershell
    az dt twin relationship list -n adt-az220-training-{your-id} --twin-id factory_1
    ```

    If you ran this command prior to deleting the relationship to cave 1, you would see output similar to the following:

    ```json
    [
        {
            "$etag": "W/\"a6a9f506-3cfa-4b62-bcf8-c51b5ecc6f6d\"",
            "$relationshipId": "47b0754a-25d1-4b71-ac47-c2409bb08535",
            "$relationshipName": "rel_has_caves",
            "$sourceId": "factory_1",
            "$targetId": "cave_2"
        },
        {
            "$etag": "W/\"b5207e88-7c86-498f-a272-7f81dde88dee\"",
            "$relationshipId": "factory_1_has_cave_1",
            "$relationshipName": "rel_has_caves",
            "$sourceId": "factory_1",
            "$targetId": "cave_1"
        }
    ]
    ```

1. To delete a model, enter the following command:

    ```powershell
    az dt model delete -n adt-az220-training-{your-id} --dtmi "dtmi:com:contoso:digital_factory:cheese_factory;1"
    ```

    Again no output is displayed.

    > **IMPORTANT**: This command deleted the factory model and succeeded, even though a digital twin **factory_1** still exists. Digital twins that were created using the deleted model can still be found by querying the graph, however, properties of the twin can no longer be updated without the model. Be very careful when completing model management tasks (versioning, deleting, etc.) to avoid creating inconsistent graphs.

1. To display the recent changes to the Azure Digital Twins environment, return to the **ADT Explorer**, and then refresh the models and run the query.

    In addition to the **factory_2** twin being deleted, the **Cheese Factory** model should be missing from the **MODELS** explorer and there should be no relationship between **factory_1** and **cave_1** in the **TWIN GRAPH** view.

1. To select the relationship between **cave_1** and **device_1**, click the line between the two twins.

    The line should thicken indicating it is selected, and the relationship properties will be displayed.

1. To delete the relationship, right-lick the line and select **Delete relationship(s)** from the context menu, and confirm by clicking **Delete**.

    The relationship will be deleted and the graph will update.

1. To delete the **device_1** digital twin, right-click **device_1**, and select **Delete twin(s)** from the context menu.

    > **NOTE**: By using **CTRL** and left-click, multiple twins can be selected. To delete them, right-click the final twin and select **Delete twin(s)** from the context menu.

1. To delete the **Cheese Cave Device** model from the **MODELS** explorer, click the associated **Delete Model** button, and confirm by clicking **Delete**.

1. To delete all models, click **Delete All Models** at the top of the **MODELS** explorer.

    > **IMPORTANT**: Use with care - there is no undo capability!

At this point, the Azure Digital Twins instance should be clear of all models, twins and relationships. Don't worry - in the next task, the **Import Graph** feature will be used to create a new graph.

#### Task 6 - Bulk import with ADT Explorer

The **ADT Explorer** supports the import and export of a digital twin graph. The **Export** capability serializes the most recent query results to a JSON-based format, including models, twins, and relationships. The **Import** capability deserializes from either a custom Excel-based format or the JSON-based format generated on export. Before import is executed, a preview of the graph is presented for validation.

The excel import format is based on the following columns:

* **ModelId**: The complete dtmi for the model that should be instantiated.
* **ID**: The unique ID for the twin to be created
* **Relationship**: A twin id with an outgoing relationship to the new twin
* **Relationship name**: The name for the outgoing relationship from the twin in the previous column
* **Init data**: A JSON string that contains **Property** settings for the twins to be created

> **NOTE**: The Excel import capability **does not** import model definitions, only twins and relationships. The JSON format supports models as well.

The following table shows the twins and relationships that will be created in this task (the **Init Data** values are removed for readability):

| ModelID                                                                | ID             | Relationship (From) | Relationship Name | Init Data |
| :--------------------------------------------------------------------- | :------------- | :------------------ | :---------------- | :-------- |
| dtmi:com:contoso:digital_factory:cheese_factory;1                      | factory_1      |                     |                   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        | cave_1       | factory_1           | rel_has_caves   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        | cave_2       | factory_1           | rel_has_caves   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1        | cave_3       | factory_1           | rel_has_caves   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 | sensor-th-0055 | cave_1            | rel_has_devices   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 | sensor-th-0056 | cave_2            | rel_has_devices   |           |
| dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1 | sensor-th-0057 | cave_3            | rel_has_devices   |           |

The spreadsheet **cheese-factory-scenario.xlsx** can be found in the **{file-root}\Allfiles\Labs\19-Azure Digital Twins\Final\Models** folder.

1. In a browser, return to the **ADT Explorer**.

1. To import your models using the **ADT Explorer**, in the **MODELS** explorer, click the **Upload a Model** icon

1. In the **Open** dialog, navigate to the **Models** folder, select the **CheeseFactoryInterface.json**, **CheeseCaveInterface.json**, and **CheeseCaveDeviceInterface.json** files, and then click **Open**.

    This will reload all of the models.

1. To import the **cheese-factory-scenario.xlsx** spreadsheet, click **Import Graph**.

    ![ADT Explorer graph view import graph](media/LAB_AK_19-graphview-importgraph.png)

1. In the **Open** dialog, navigate to the **Models** folder and select the **cheese-factory-scenario.xlsx** file, then click **Open**.

    A preview of the graph to be imported is displayed in an **IMPORT** view:

    ![ADT Explorer graph view import preview](media/LAB_AK_19-graphview-importpreview.png)

1. To complete the import, click **Start Import**.

    An **Import Successful** dialog will be displayed, detailing that 7 twins and 6 relationships were imported. Click **Close** to proceed.

1. Return to the **TWIN GRAPH** view, and click **Run Query**.

    The imported graph should now be displayed. You can click on each twin to view the properties (each twin has been initialized with values).

1. To export the current graph as JSON, click **Export Graph** (next to the **Import Graph** button used earlier).

    The **Export** view is displayed with a **Download** link in the top-left corner.

1. To download the model in JSON, click the **Download** link.

    The browser will download the model.

1. To view the JSON, open the downloaded file in Visual Studio Code.

    If the JSON is shown as a single line, reformat the JSON using the **Format Document** command via the command palette or by using a right-click in the editor view and then selecting Format Document.

    The JSON has three primary sections:

    * **digitalTwinsFileInfo** - contains the version of the exported file format
    * **digitalTwinsGraph** - contains the instance data for every twin and relationship that was displayed on the exported graph (i.e. only those displayed based upon the query)
    * **digitalTwinsModels** - the model definitions

    > **NOTE**: Unlike the Excel format, the JSON file includes the model definitions, meaning that everything can be imported with just the one file.

1. To import the JSON file, use **ADT Explorer** to delete the models and twins following the instructions in earlier tasks, and then import the JSON file that was just created by using the **Import graph** button. 

    > **NOTE**: Be sure to save the imported graph.

1. Notice that the models, twins and their properties, and the relationships are recreated.

This twin graph will be used as the basis for the exercise on querying.

### Exercise 4 - Query the graph using ADT Explorer

>**NOTE**: This exercise requires the graph imported in the previous exercise.

Now let's review the digital twin graph query language.

You can query the digital twin graph that you just built, to get information about the digital twins and relationships it contains. You write these queries in a custom, SQL-like query language, referred to as the Azure Digital Twins query language. This language is also similar to the query language for Azure IoT Hub.

Queries can be made through the Digital Twins REST API and with the SDKs. In this exercise, you'll be using the Azure Digital Twins explorer sample app to handle the API calls for you. Additional tools will be explored later in this lab.

> **NOTE**: The **ADT Explorer** is  designed to visualize a graph and can only display entire twins, rather than just single values selected from a twin, such as the name.

#### Task 1 - Query using the ADT Explorer

In this task, the ADT Explorer will be used to execute graph queries and render the results as a graph. Twins can be queried by properties, model type and by relationships. Queries can be combined into compound queries using combination operators that can query for more than one type of twin descriptor at a a time.

1. Return to the **ADT Explorer** window.

1. Ensure the **QUERY EXPLORER** query is set to the following:

    ```sql
    SELECT * FROM digitaltwins
    ```

    If you are familiar with SQL, you will realize that this query will return everything from digital twins.

1. To run this query, click **Run Query**.

    As expected, the entire graph is displayed.

1. To save this as a named query, click on the **Save** icon (just to the right of the **Run Query** button).

1. In the **Save Query** dialog, enter the name **All Twins** and click **Save**.

    This query will then be saved locally and be available in the **Saved Queries** drop down to the left of the query text box. To delete a saved query, click on the **X** icon next to the name of the query when the **Saved Queries** drop down is open.

    > **TIP**: Run the **All Twins** query to return to the full view at any time.

1. To create a query that filters the graph so that only **Cheese Cave** twins are displayed, enter the following query:

    ```sql
    SELECT * FROM digitaltwins 
    WHERE IS_OF_MODEL('dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1')
    ```

1. Review the query and verify that it matches the suggested syntax, and then click **Run Query**.

    If you get an error, ensure that there is a space between **digitaltwins** and **WHERE**.
 
    The graph will now display just the 3 **Cheese Cave** twins.

    Save this query as **Just Caves**.

1. To create a query that displays only the **Cheese Cave** twins that are **inUse**, enter the following query:

    ```sql
    SELECT * FROM digitaltwins 
    WHERE IS_OF_MODEL('dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1') 
    AND inUse = true
    ```

1. Review the query and verify that it matches the suggested syntax, and then click **Run Query**.

    The graph should now display just **cave_3** and **cave_1**.

1. To display only the **Cheese Cave** twins that are **inUse** and have a **temperatureAlert**, enter and run the following query:

    ```sql
    SELECT * FROM digitaltwins 
    WHERE IS_OF_MODEL('dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave;1') 
    AND inUse = true 
    AND temperatureAlert = true
    ```

    The graph should now display just **cave_3**.

1. To use relationships to find the parent of the device **sensor-th-0055** via a join, enter the following query:

    ```sql
    SELECT Parent FROM digitaltwins Parent 
    JOIN Child RELATED Parent.rel_has_devices 
    WHERE Child.$dtId = 'sensor-th-0055'
    ```

    The **cave_1** twin should be displayed.

    For those familiar with the SQL JOIN, the syntax used here will look different from what you might be used to. Notice that the name of the relationship **rel_has_devices** is specified, rather than correlating this JOIN with a key value in a WHERE clause or specifying a key value inline with the JOIN definition. This correlation is computed automatically, as the relationship properties themselves identify the target entity. Here is the relationship definition:

    ```json
    {
        "@type": "Relationship",
        "@id": "dtmi:com:contoso:digital_factory:cheese_cave:rel_has_devices;1",
        "name": "rel_has_devices",
        "displayName": "Has devices",
        "target": "dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1"
    }
    ```

#### Task 2 - Query for properties using the ADT Explorer

A key limitation of the **ADT Explorer** is that it is designed to render a graph and the primary display cannot show the results for queries that return just properties. In this task, you will learn how it is possible to see the results of such queries without resorting to coding solutions.

1. To run a valid query that returns just a property, enter the following query and click **Run Query**:

    ```sql
    SELECT Parent.desiredTemperature FROM digitaltwins Parent 
    JOIN Child RELATED Parent.rel_has_devices 
    WHERE Child.$dtId = 'sensor-th-0055'
    ```

    Despite the fact that the query will run without error, no graph is displayed. However, there is a way to view the results in **ADT Explorer**, and you will open the **Output** pane to view the query results during the next few steps.

1. To open the **Output** pane, click the **Settings** icon at the top-right of the page.

1. On the dialog that appears, under **View**, enable **Output**, and then close the dialog.

    The **Output** pane should appear at the bottom of the page.

1. Rerun the query above and review the contents of the **Output** pane.

    The OUTPUT pane should display the **Requested query**, and then show the returned JSON. The JSON should be similar to the following:

    ```json
    {
        "queryCharge": 20.259999999999998,
        "connection": "close",
        "content-encoding": "gzip",
        "content-type": "application/json; charset=utf-8",
        "date": "Thu, 25 Mar 2021 21:34:40 GMT",
        "strict-transport-security": "max-age=2592000",
        "traceresponse": "00-182f5e54efb95c4b8b3e2a6aac15499f-9c5ffe6b8299584e-01",
        "transfer-encoding": "chunked",
        "vary": "Accept-Encoding",
        "x-powered-by": "Express",
        "value": [
            {
            "desiredTemperature": 50
            }
        ],
        "continuationToken": null
    }
    ```

    Along with additional result metadata, notice that the **value** property contains the selected **desiredTemperature** property and value.

### Exercise 5 - Configure and launch device simulator

In the preceding exercises, the digital twin model and graph for the proof-of-concept were created. In order to demonstrate how to route device message traffic from IoT Hub to Azure Digital Twins, it is useful to use a device simulator. In this exercise, you will be configuring a simulated device app to send telemetry to your IoT Hub.

#### Task 1: Open the device simulator project

In this task, the Cheese Cave Device simulator app will be opened in Visual Studio Code in preparation for configuration.

1. Open **Visual Studio Code**.

1. On the **File** menu, click **Open Folder** and then navigate to the **Starter** folder for Lab 19.

    The Lab 19 **Starter** folder is part of the lab resources that you downloaded before starting this lab. The folder path is:

    * Allfiles
        * Labs
            * 19-Azure Digital Twins
                * Starter
                    * CheeseCaveDevice

    > **Note**: If you have trouble finding the Allfiles folder, check your Windows Desktop folder.

1. Click **CheeseCaveDevice**, and then click **Select Folder**.

    You should see the following files listed in the EXPLORER pane of Visual Studio Code:

    * CheeseCaveDevice.csproj
    * Program.cs

1. To open the code file, click **Program.cs**.

    A cursory glance will reveal that this simulated device application uses symmetric Key authentication, sends both telemetry and logging messages to the IoT Hub, and includes simulated sensor readings for the telemetry.

1. On the **Terminal** menu, click **New Terminal**.

    Notice the directory path indicated as part of the command prompt. You do not want to start building this project within the folder structure of a previous lab project.

1. At the terminal command prompt, to verify the application builds, enter the following command:

    ```bash
    dotnet build
    ```

    The output will be similar to:

    ```text
    > dotnet build
    Microsoft (R) Build Engine version 16.5.0+d4cbfca49 for .NET Core
    Copyright (C) Microsoft Corporation. All rights reserved.

    Restore completed in 39.27 ms for D:\Az220\AllFiles\Labs\19-Azure Digital Twins\Starter\CheeseCaveDevice\CheeseCaveDevice.csproj.
    CheeseCaveDevice -> D:\Az220\AllFiles\Labs\19-Azure Digital Twins\Starter\CheeseCaveDevice\bin\Debug\netcoreapp3.1\CheeseCaveDevice.dll

    Build succeeded.
        0 Warning(s)
        0 Error(s)

    Time Elapsed 00:00:01.16
    ```

In the next task, you will configure the connection string and review the application.

#### Task 2: Configure connection and review code

The simulated device app that you will build in this task simulates an IoT device that monitors temperature and humidity. The app will simulate sensor readings and communicate sensor data every two seconds and is the same app that was built in Lab 15.

1. In **Visual Studio Code**, ensure that the Program.cs file is open.

1. In the code editor, locate the following line of code:

    ```csharp
    private readonly static string deviceConnectionString = "<your device connection string>";
    ```

1. Replace the **your device connection string** (including the angle brackets) with the device connection string that you saved to your text editor.

    This is the only change that you need to implement before sending telemetry to the IoT Hub.

    > **NOTE**: You saved both a Device and Service connection string. Be sure to provide the Device connection string.

1. On the **File** menu, click **Save**.

#### Task 3: Test your Code to Send Telemetry

In this task, the configured simulator app is launched and the the successful transmission of telemetry is verified.

1. In Visual Studio Code, ensure that you still have the Terminal open.

1. At the Terminal command prompt, to run the simulated device app, enter the following command:

    ```bash
    dotnet run
    ```

   This command will run the **Program.cs** file in the current folder.

1. Notice the output being sent to the Terminal.

    You should quickly see console output, similar to the following:

    ![Console Output](media/LAB_AK_19-cheesecave-telemetry.png)

    > **Note**:  Green text is used to indicate when things are working as they should be. Red text is used to indicate when there is a problem. If you don't get a screen similar to the image above, start by checking your device connection string.

1. Leave this app running.

    You need to be sending telemetry to IoT Hub later in this lab.

### Exercise 6 -  Set up Azure Function to ingest data

A key part of the proof-of-concept is to demonstrate how data from a device can be delivered to Azure Digital Twins. Data can be ingested into Azure Digital Twins through external compute resources such as Virtual Machines, Azure Functions, and Logic Apps. In this exercise, a function app will be invoked by an IoT Hub's built-in Event Grid. The function app receives the data and uses the Azure Digital Twins APIs to set properties on the appropriate digital twin instance.

#### Task 1 - Create and configure a function app

In order to configure an IoT Hub event grid endpoint to route telemetry to an Azure Function, it is necessary to first create the Azure Function. In this task, an Azure Function App is created that provides the execution context in which individual Azure Functions run.

In order to access Azure Digital Twins and it's APIs, it is necessary to utilize a service principal with the appropriate permissions. During this task, a service principal is created for the function app and then assigned the appropriate permission. Once the function app has the appropriate permission, any Azure Functions that execute within the function app context will use that service principal and will therefore have permission to access Azure Digital Twins.

The function app context also provides an environment for managing app settings for one or more functions. This capability will be used to define a setting that contains the Azure Digital Twins connection string which can then be read by the Azure Functions. Encapsulating connection strings and other configurations values in app settings is considered a much better practice than hard-coding the values in the function code.

1. Open the browser window containing your Azure portal, and then open the Azure Cloud Shell.

1. At the Cloud Shell command prompt, to create an Azure Function App, enter the following command:

    ```bash
    az functionapp create --resource-group rg-az220 --consumption-plan-location {your-location} --name func-az220-hub2adt-training-{your-id} --storage-account staz220training{your-id} --functions-version 3
    ```

    > **Note**: Remember to replace the **{your-location}** and **{your-id}** tokens above.

    The Azure function requires that a bearer token to be passed to it in order to authenticate with Azure Digital Twins. To make sure that this token is passed, you'll need to create a managed identity for the function app.

1. To create (assign) the system-managed identity for the function app and display the associated principal Id, enter the following command:

    ```bash
    az functionapp identity assign -g rg-az220 -n func-az220-hub2adt-training-{your-id} --query principalId -o tsv
    ```

    > **Note**: Remember to replace the **{your-id}** token above.

    The output will be similar to following:

    ```bash
    1179da2d-cc37-48bb-84b3-544cbb02d194
    ```

    This is the principal ID that was assigned to the function app - you will need the principal ID in the next step.

1. To assign the **Azure Digital Twins Data Owner** role to the Function App principal, enter the following command:

    ```bash
    az dt role-assignment create --dt-name adt-az220-training-{your-id} --assignee {principal-id} --role "Azure Digital Twins Data Owner"
    ```

    > **Note**: Remember to replace the **{your-id}** and **{principal-id}** tokens above. The **{principal-id}** value was displayed as the output of the previous step.

    Now that the principal has been assigned to the Azure Function App, that principal must be assigned the **Azure Digital Twins Data Owner** role so that it can access the Azure Digital Twins instance.

1. In order to supply the Azure Digital Twin instance URL to the Azure Function App as an environment variable, enter the following command:

    ```bash
    az functionapp config appsettings set -g rg-az220 -n func-az220-hub2adt-training-{your-id} --settings "ADT_SERVICE_URL={adt-url}"
    ```

    > **Note**: Remember to replace the **{your-id}** and **{adt-url}** tokens above. The **{adt-url}** value was saved to your text editor file in an earlier task and will be similar to `https://adt-az220-training-dm030821.api.eus.digitaltwins.azure.net`.

    Once complete, the command lists all of the available settings. The Azure Function will now be able to obtain the Azure Digital Twins service URL by reading the **ADT_SERVICE_URL** value.

#### Task 2 - Review Contoso.AdtFunctions Project

In this task you will review the Azure Function that will be executed whenever an event occurs on the associated Event Grid. The event will be processed and the message and telemetry will be routed to Azure Digital Twins.

1. Open a new instance of **Visual Studio Code**.

1. In the Visual Studio Code window, on the **File** menu, click **Open Folder**, and then navigate to the **Contoso.AdtFunctions** folder.

    The Lab 19 **Contoso.AdtFunctions** folder is part of the lab resources that you downloaded before starting this lab. The folder path is:

    * Allfiles
        * Labs
            * 19-Azure Digital Twins
                * Final
                    * Contoso.AdtFunctions

1. Click **Contoso.AdtFunctions**, and then click **Select Folder**.

1. Open the **Contoso.AdtFunctions.csproj** file.

    Notice the project references the following NuGet Packages:

    * The **Azure.DigitalTwins.Core** package contains the SDK for the Azure Digital Twins service. This library provides access to the Azure Digital Twins service for managing twins, models, relationships, etc.
    * The **Microsoft.Azure.WebJobs.Extensions.EventGrid** package provides functionality for receiving Event Grid webhook calls in Azure Functions, allowing you to easily write functions that respond to any event published to Event Grid.
    * The **Microsoft.Azure.WebJobs.Extensions.EventHubs** package provides functionality for receiving Event Hub webhook calls in Azure Functions, allowing you to easily write functions that respond to any event published to Event Hub.
    * The **Microsoft.NET.Sdk.Functions** packages includes a build task for building .NET function projects.
    * The **Azure.identity** package contains the implementation of the Azure SDK Client Library for Azure Identity. The Azure Identity library provides Azure Active Directory token authentication support across the Azure SDK. It provides a set of TokenCredential implementations which can be used to construct Azure SDK clients which support AAD token authentication
    * The **System.Net.Http** package provides a programming interface for modern HTTP applications, including HTTP client components that allow applications to consume web services over HTTP and HTTP components that can be used by both clients and servers for parsing HTTP headers.

1. In Visual Studio Code, open the **HubToAdtFunction.cs** file.

1. To review the member variables for the function, locate the **//Your Digital Twins URL is stored in an application setting in Azure Functions** comment and review the code below it:

    ```csharp
    //Your Digital Twins URL is stored in an application setting in Azure Functions.
    private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
    private static readonly HttpClient httpClient = new HttpClient();
    ```

    Notice that the **adtInstanceUrl** variable is assigned the value of the **ADT_SERVICE_URL** environment variable defined earlier in the exercise. The code also follows a best practice of using a single, static, instance of the **HttpClient**.

1. Locate the **Run** method declaration, review the following comments:

    ```csharp
    [FunctionName("HubToAdtFunction")]
    public async static Task Run([EventGridTrigger] EventGridEvent eventGridEvent, ILogger log)
    ```

    Notice the use of the **FunctionName** attribute to mark the **Run** method as the entry point **Run** for **HubToAdtFunction**. The method is also declared `async` as the code to update the Azure Digital Twin runs asynchronously.

    The **eventGridEvent** parameter is assigned the Event Grid event that triggered the function call and the **log** parameter provides access to a logger that can be used for debugging.

    > **TIP**: To learn more about the Azure Event Grid trigger for Azure Functions, review the resource below:
    > * [Azure Event Grid trigger for Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-grid-trigger?tabs=csharp%2Cbash)

1. To review how to log informational data, locate the following code:

    ```csharp
    log.LogInformation(eventGridEvent.Data.ToString());
    ```

    The **ILogger** interface is defined in the **Microsoft.Extensions.Logging** namespace and aggregates most logging patterns to a single method call. In this case, a log entry is created at the **Information** level - other methods exists for various levels including critical, error. etc. As the Azure Function is running in the cloud, logging is essential during development and production.

    > **TIP:** To learn more about the **Microsoft.Extensions.Logging** capability, review the following resources:
    > * [Logging in .NET](https://docs.microsoft.com/dotnet/core/extensions/logging?tabs=command-line)
    > * [Microsoft.Extensions.Logging namespace](https://docs.microsoft.com/dotnet/api/microsoft.extensions.logging?view=dotnet-plat-ext-5.0&viewFallbackFrom=netcore-3.1)
    > * [ILogger Interface](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.ilogger?view=dotnet-plat-ext-5.0&viewFallbackFrom=netcore-3.1)

1. Locate the following code to understand how the **adtInstanceUrl** variable value is checked:

    ```csharp
    if (adtInstanceUrl == null)
    {
        log.LogError("Application setting \"ADT_SERVICE_URL\" not set");
        return;
    }
    ```

    This code checks if the **adtInstanceUrl** variable has been set - if not, the error is logged and the function exits. This demonstrates the value of logging to capture the fact that the function has been incorrectly configured.

1. To ensure any exceptions are logged, a **try..catch** loop is used:

    ```csharp
    try
    {
        // ... main body of code
    }
    catch (Exception e)
    {
        log.LogError(e.Message);
    }
    ```

    Notice that the exception message is logged.

1. To see how the function app principal is used to authenticate to Azure Digital Twins and create a client instance, locate the **// INSERT authentication code below here** comment and review the following code:

    ```csharp
    ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
    DigitalTwinsClient client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
    log.LogInformation($"Azure digital twins service client connection created.");
    ```

    Notice the use of the **ManagedIdentityCredential** class. This class attempts authentication using the managed identity that has been assigned to the deployment environment earlier. Once the credential is returned, it is used to construct an instance of the **DigitalTwinsClient**. The client contains methods to retrieve and update digital twin information, like models, components, properties and relationships.

1. To review the code that starts to process the Event Grid event, locate the **// INSERT event processing code below here** comment and review the following code below it:

    ```csharp
    if (eventGridEvent != null && eventGridEvent.Data != null)
    {
        // Read deviceId and temperature for IoT Hub JSON.
        JObject deviceMessage = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
        string deviceId = (string)deviceMessage["systemProperties"]["iothub-connection-device-id"];
        var fanAlert = (bool)deviceMessage["properties"]["fanAlert"]; // cast directly to a bool
        var temperatureAlert = deviceMessage["properties"].SelectToken("temperatureAlert") ?? false; // JToken object
        var humidityAlert = deviceMessage["properties"].SelectToken("humidityAlert") ?? false; // JToken object
        log.LogInformation($"Device:{deviceId} fanAlert is:{fanAlert}");
        log.LogInformation($"Device:{deviceId} temperatureAlert is:{temperatureAlert}");
        log.LogInformation($"Device:{deviceId} humidityAlert is:{humidityAlert}");

        var bodyJson = Encoding.ASCII.GetString((byte[])deviceMessage["body"]);
        JObject body = (JObject)JsonConvert.DeserializeObject(bodyJson);
        log.LogInformation($"Device:{deviceId} Temperature is:{body["temperature"]}");
        log.LogInformation($"Device:{deviceId} Humidity is:{body["humidity"]}");
    }
    ```

    Notice the use of JSON deserialization to access the event data. The event data JSON will be similar to:

    ```JSON
    {
        "properties": {
            "sensorID": "S1",
            "fanAlert": "false",
            "temperatureAlert": "true",
            "humidityAlert": "true"
        },
        "systemProperties": {
            "iothub-connection-device-id": "sensor-th-0055",
            "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
            "iothub-connection-auth-generation-id": "637508617957275763",
            "iothub-enqueuedtime": "2021-03-11T03:27:21.866Z",
            "iothub-message-source": "Telemetry"
        },
        "body": "eyJ0ZW1wZXJhdHVyZSI6OTMuOTEsImh1bWlkaXR5Ijo5OC4wMn0="
    }
    ```

    The message **properties** and **systemProperties** are easily accessible using an indexer approach, however where properties are optional, such as **temperatureAlert** and **humidityAlert**, the use of **SelectToken** and a null-coalescing operation is required to prevent an exception being thrown.

    > **TIP**: To learn more about the null-coalescing operator **??**, review the following content:
    > * [?? and ??= operators (C# reference)](https://docs.microsoft.com/dotnet/csharp/language-reference/operators/null-coalescing-operator)

    The message **body** contains the telemetry payload and is ASCII encoded JSON. Therefore, it must first be decoded and then deserialized before the telemetry properties can be accessed.

    > **TIP**: To learn more about the event schema, review the following resource:
    > * [Event schema](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-grid-trigger?tabs=csharp%2Cbash#event-schema)

1. To inspect the code that updates the digital twin, locate the **// INSERT ADT update code below here** comment and review the following code below it:

    ```csharp
    //Update twin
    var patch = new Azure.JsonPatchDocument();
    patch.AppendReplace<bool>("/fanAlert", fanAlert); // already a bool
    patch.AppendReplace<bool>("/temperatureAlert", temperatureAlert.Value<bool>()); // convert the JToken value to bool
    patch.AppendReplace<bool>("/humidityAlert", humidityAlert.Value<bool>()); // convert the JToken value to bool

    await client.UpdateDigitalTwinAsync(deviceId, patch);

    // publish telemetry
    await client.PublishTelemetryAsync(deviceId, null, bodyJson);
    ```

    There are two approaches being used to apply data to the digital twin - the first via property updates using a JSON patch, the second via the publishing of telemetry data.

    The Azure Digital Twins client utilizes a JSON Patch document to add or update digital twin properties. The JSON Patch defines a JSON document structure for expressing a sequence of operations to apply to a JSON document. The various values are added to the patch as append or replace operations, and the Azure Digital Twins is then updated asynchronously.

   > **TIP**: To learn more about a JSON Patch document, review the following resources:
   > * [Javascript Object Notation (JSON) Patch](https://tools.ietf.org/html/rfc6902)
   > * [What is JSON Patch?](http://jsonpatch.com/)
   > * [JsonPatchDocument Class](https://docs.microsoft.com/dotnet/api/azure.jsonpatchdocument?view=azure-dotnet)

   > **IMPORTANT**: The digital twin instance must have existing values before the **AppendReplace** operation is used.

   Notice that the telemetry data is handled differently than the properties - rather than being used to set digital twin properties, it is instead being published as telemetry events. This mechanism ensures that the telemetry is available to be consumed by any downstream subscribers to the digital twins event route.

   > **NOTE**: The digital twins event route must be defined before publishing a telemetry message, otherwise the message will not be routed for consumption.

The function is ready to be published.

> **NOTE**: The **TelemetryFunction.cs** function will be reviewed in a later task.

#### Task 3 - Publish Functions

1. In Visual Studio Code, on the left side menu, click **Azure**.

    This will display a new information pane to the left of the code editor view. The AZURE pane includes sections for various Azure resource types.

1. Take a moment to review the contents of the FUNCTIONS section of the Azure pane.

    Notice that your local project "Contoso.AdtFunctions" is listed.

1. Click **Local Project**, and then select the initialize function for Visual Studio use button.

1. In the FUNCTIONS section, click **Deploy to Function App**.

    ![Visual Studio Code deploy to function app](media/LAB_AK_19-deploy-to-function-app.png)

1. When prompted, make these selections:

    * **Sign in to Azure**: If prompted, sign into Azure
    * **Select subscription**: If prompted, select the subscription you are using for this lab.
    * **Select Function App in Azure**: Select **func-az220-hub2adt-training-{your-id}**.

    When asked to confirm the deploy, click **Deploy**.

    The function will then be compiled and, if successful, deployed. This may take a few moments.

1. Wait for the deployment to complete.

    When the deployment is complete, the following prompt to be displayed:

    ![Visual Studio Code deployment complete - select stream logs](media/LAB_AK_19-function-stream-logs.png)

1. In the deployment complete dialog, click **Stream logs**.

1. In the confirmation dialog that appears, to enable application logging, click **Yes**.

    The **OUTPUT** pane will now display the log stream for the deployed function - this will timeout after 2 hours. There will be some status information displayed, however there will not be any diagnostic information from the function itself until it is launched. That will be covered in the next exercise.

    The streaming can be stopped or started at any time by right-clicking the Azure function in Visual Studio Code and select **Start Streaming Logs** or **Stop Streaming Logs**:

    ![Visual Studio Code Azure Function start streaming logs](media/LAB_AK_19-start-function-streaming.png)

### Exercise 7 - Connect IoT Hub to the Azure Function

In this exercise, you will configure your IoT hub to publish events to the Azure Function that you created in the previous exercise. The events will be published as they occur. The telemetry generated by the simulated device created earlier will then be routed to the Azure Digital Twins instance.

1. Open the browser window for your Azure portal.

1. Navigate to your **iot-az220-training-{your-id}** IoT hub.

1. In the left-hand navigation area, select **Events**.

1. To add an event subscription, click **+ Event Subscription**.

1. In the **EVENT SUBSCRIPTION DETAILS** section, in the **Name** field, enter **device-telemetry**.

    +++device-telemetry+++

1. In the **Event Schema** dropdown, ensure **Event Grid Schema** is selected.

1. In the **TOPIC DETAILS** section, verify that the **Topic Type** is set to **IoT Hub** and the **Source Resource** is set to **iot-az220-training-{your-id}**.

1. In the **System Topic Name** field, enter **Twin-Topic**

    +++Twin-Topic+++

1. In the **EVENT TYPES** section, in the **Filter to Event Types** dropdown, select _only_ **Device Telemetry**.

1. In the **ENDPOINT DETAILS** section, in the **Endpoint Type** dropdown, select **Azure Function**.

    Notice that the UI updates to provide for the selection of an endpoint.

1. In the **Endpoint** field, click **Select an endpoint**.

1. In the **Select Azure Function** pane, under **Subscription**, ensure the correct subscription is selected.

1. Under **Resource group**, ensure **rg-az220** is selected.

1. Under **Function app**, select **func-az220-hub2adt-training-{your-id}**.

1. Under **Slot**, confirm **Production** is selected.

1. Under **Function**, confirm **HubToAdtFunction** is selected.

1. To choose this endpoint, click **Confirm Selection**.

1. Verify that **HubToAdtFunction** is now the specified Endpoint.

    On the **Create Event Subscription** pane, in the **ENDPOINT DETAILS** section, the **Endpoint** field should now display **HubToAdtFunction**.

1. To create this event subscription, click **Create**.

    After the subscription has been created, you should see messages in the Azure Functions log stream that you configured in the preceding exercise. The Azure Functions log stream shows the telemetry being received from Event Grid. It also shows any errors that occur when connecting to Azure Digital Twins or updating the twin.

    The log output for a successful function call will be similar to:

    ```log
    2021-03-12T19:14:17.180 [Information] Executing 'HubToAdtFunction' (Reason='EventGrid trigger fired at 2021-03-12T19:14:17.1797847+00:00', Id=88d9f9e8-5cfa-4a20-a4cb-36e07a78acd6)
    2021-03-12T19:14:17.180 [Information] {
    "properties": {
        "sensorID": "S1",
        "fanAlert": "false",
        "temperatureAlert": "true",
        "humidityAlert": "true"
    },
    "systemProperties": {
        "iothub-connection-device-id": "sensor-th-0055",
        "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
        "iothub-connection-auth-generation-id": "637508617957275763",
        "iothub-enqueuedtime": "2021-03-12T19:14:16.824Z",
        "iothub-message-source": "Telemetry"
    },
    "body": "eyJ0ZW1wZXJhdHVyZSI6NjkuNDcsImh1bWlkaXR5Ijo5Ny44OX0="
    }
    2021-03-12T19:14:17.181 [Information] Azure digital twins service client connection created.
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 fanAlert is:False
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 temperatureAlert is:true
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 humidityAlert is:true
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 Temperature is:69.47
    2021-03-12T19:14:17.181 [Information] Device:sensor-th-0055 Humidity is:97.89
    2021-03-12T19:14:17.182 [Information] Executed 'HubToAdtFunction' (Succeeded, Id=88d9f9e8-5cfa-4a20-a4cb-36e07a78acd6, Duration=2ms)
    ```

    Here is an example log if the Digital Twin instance is not found:

    ```log
    2021-03-11T16:35:43.646 [Information] Executing 'HubToAdtFunction' (Reason='EventGrid trigger fired at 2021-03-11T16:35:43.6457834+00:00', Id=9f7a3611-0795-4da7-ac8c-0b380310f4db)
    2021-03-11T16:35:43.646 [Information] {
    "properties": {
        "sensorID": "S1",
        "fanAlert": "false",
        "temperatureAlert": "true",
        "humidityAlert": "true"
    },
    "systemProperties": {
        "iothub-connection-device-id": "sensor-th-0055",
        "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
        "iothub-connection-auth-generation-id": "637508617957275763",
        "iothub-enqueuedtime": "2021-03-11T16:35:43.279Z",
        "iothub-message-source": "Telemetry"
    },
    "body": "eyJ0ZW1wZXJhdHVyZSI6NjkuNzMsImh1bWlkaXR5Ijo5OC4wOH0="
    }
    2021-03-11T16:35:43.646 [Information] Azure digital twins service client connection created.
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 fanAlert is:False
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 temperatureAlert is:true
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 humidityAlert is:true
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 Temperature is:69.73
    2021-03-11T16:35:43.647 [Information] Device:sensor-th-0055 Humidity is:98.08
    2021-03-11T16:35:43.648 [Information] Executed 'HubToAdtFunction' (Succeeded, Id=9f7a3611-0795-4da7-ac8c-0b380310f4db, Duration=2ms)
    2021-03-11T16:35:43.728 [Error] Service request failed.
    Status: 404 (Not Found)

    Content:
    {"error":{"code":"DigitalTwinNotFound","message":"There is no digital twin instance that exists with the ID sensor-th-0055. Please verify that the twin id is valid and ensure that the twin is not deleted. See section on querying the twins http://aka.ms/adtv2query."}}

    Headers:
    Strict-Transport-Security: REDACTED
    traceresponse: REDACTED
    Date: Thu, 11 Mar 2021 16:35:43 GMT
    Content-Length: 267
    Content-Type: application/json; charset=utf-8
    ```

1. Return to the **ADT Explorer** instance in the browser and query the graph.

    You should be able to see that the **fanAlert**, **temperatureAlert** and **humidityAlert** properties have been updated (associated with Sensor/Device or Cave nodes).

At this point, the data ingestion from device (in this case a device simulator) into Azure Digital Twins has been demonstrated. The next exercise will demonstrate how telemetry can be streamed to Time Series Insights (TSI).

### Exercise 8 - Connect Azure Digital Twins to TSI

In this exercise, you will complete the final part of the proof-of-concept - streaming the device telemetry sent from the Cheese Cave Device sensor-th-0055 via the simulator, through Azure Digital Twins to Time Series Insights.

> **NOTE**: The setup script for this lab created the Azure Storage Account, Time Series Insights Environment and Access Policy that will be used during this exercise.

Routing event data from Azure Digital Twins to TSI is achieved by the following basic flow:

* Azure Digital Twins publishes notification events to an Event Hub (adt2func)
* An Azure Function is triggered by events from the adt2func Event Hub, creates a new event for TSI, adds a partition key and published to another Event Hub (func2tsi)
* TSI subscribes to events published to the func2tsi Event Hub.

In order to implement this flow, the following resources must be created (in addition to Azure Digital Twins and TSI):

* An Azure Function
* An Event Hub Namespace
* Two Event Hubs - one for events from Azure Digital Twins to the Azure Function, another for events from the Azure Function for TSI.

The Azure function can serve a number of purposes:

* Map device specific telemetry message formats (property names, data types, etc.) to a single format for TSI. This can provide a consolidated view across different devices as well as insulate the TSI solution from changes to other parts of the solution.
* Enrich the messages from other sources
* Add a field to each event for use as the Time Series ID Property within TSI.

#### Task 1 - Create event hub namespace

An Event Hubs namespace provides DNS integrated network endpoints and a range of access control and network integration management features such as IP filtering, virtual network service endpoint, and Private Link and is the management container for one of multiple Event Hub instances (or topics, in Kafka parlance). The two Event Hubs required for this solution will be created within this namespace.

1. Switch to your browser window containing your Azure portal.

1. On the Azure portal menu, click **+ Create a resource**.

1. In the Search textbox, type **Event Hubs** and then click the **Event Hubs** search result.

1. To create an **Event Hub**, click **Create**.

    The **Create Namespace** page will open.

1. On the **Create Namespace** blade, in the **Subscription** dropdown, ensure that the Azure subscription that you are using for this lab is selected.

1. To the right of **Resource group**, open the dropdown, and then click **rg-az220**

1. In the **Namespace name** field, enter **evhns-az220-training-{your-id}**.

    +++evhns-az220-training-{your-id}+++

    Be sure to replace the **{your-id}** placeholder. This resource is publicly accessible and must have a unique name.

1. To the right of **Location**, open the drop-down list and select the same location that you selected for your resource group.

1. To the right of **Pricing tier**, open the drop-down list and select **Standard**.

    > **TIP**: In this exercise, the **Basic** tier would also work, however in most production scenarios, **Standard** would be the better choice:
    >
    > * **Basic**
    >   * 1 Consumer Group
    >   * 100 Brokered Connections
    >   * Ingress events - $0.028 per million (at time of writing)
    >   * Message retention - 1 day
    > * **Standard**
    >   * 20 Consumer Group
    >   * 1000 Brokered Connections
    >   * Ingress events - $0.028 per million (at time of writing)
    >   * Message retention - 1 day
    >   * Additional storage - up to 7 days
    >   * Publisher policies

1. To the right of **Throughput units**, leave the selection at **1**.

    > **TIP**: The throughput capacity of Event Hubs is controlled by throughput units. Throughput units are pre-purchased units of capacity. A single throughput lets you:
    >
    > * Ingress: Up to 1 MB per second or 1000 events per second (whichever comes first).
    > * Egress: Up to 2 MB per second or 4096 events per second.

1. Leave **Enable Auto-Inflate** unchecked.

    Auto-Inflate automatically scales the number of Throughput Units assigned to your Standard Tier Event Hubs Namespace when your traffic exceeds the capacity of the Throughput Units assigned to it. You can specify a limit to which the Namespace will automatically scale.

1. To start the validation of the data entered, click **Review + create**.

1. Once validation has succeeded, click **Create**.

    After a few moments the resource will be deployed. Click **Go to resource**.

This namespace will contain the event hub used to integrate the digital twin telemetry with an Azure function, and an event hub that will take the output of the Azure function and integrate it with Time Series Insights.

#### Task 2 - Add an event hub for Azure Digital Twins

This task will create an Event Hub that will subscribe to the twin telemetry events and pass them to an Azure Function.

1. On the **Overview** page of the **evhns-az220-training-{your-id}** Event Hubs namespace, click **+ Event Hub**.

1. On the **Create Event Hub** page, under **Name**, enter **evh-az220-adt2func**

    +++evh-az220-adt2func+++

    > **Note**: As the event hub is scoped within a globally unique namespace, the event hub name itself need not be globally unique.

1. Under **Partition Count**, leave the value as **1**.

    > **TIP**: Partitions are a data organization mechanism that relates to the downstream parallelism required in consuming applications. The number of partitions in an event hub directly relates to the number of concurrent readers you expect to have.

1. Under **Message Retention**, leave the value as **1**.

    > **TIP**: This is the retention period for events. You can set the retention period between 1 and 7 days.

1. Under **Capture**, leave the value set to **Off**.

    > **TIP**: Azure Event Hubs Capture enables you to automatically deliver the streaming data in Event Hubs to an Azure Blob storage or Azure Data Lake Store account of your choice, with the added flexibility of specifying a time or size interval. Setting up Capture is fast, there are no administrative costs to run it, and it scales automatically with Event Hubs throughput units. Event Hubs Capture is the easiest way to load streaming data into Azure, and enables you to focus on data processing rather than on data capture.

1. To create the Event Hub, click **Create**.

    After a moment, the Event Hub is created and Event Hubs namespace **Overview** is displayed. If necessary, scroll to the bottom of the page and the **evh-az220-adt2func** Event Hub is listed.

#### Task 3 - Add an authorization rule to the event hub

Each Event Hubs namespace and each Event Hubs entity (an event hub instance or a Kafka topic) has a shared access authorization policy made up of rules. The policy at the namespace level applies to all entities inside the namespace, irrespective of their individual policy configuration. For each authorization policy rule, you decide on three pieces of information: name, scope, and rights. The name is a unique name in that scope. The scope is the URI of the resource in question. For an Event Hubs namespace, the scope is the fully qualified domain name (FQDN), such as **https://evhns-az220-training-{your-id}.servicebus.windows.net/**.

The rights provided by the policy rule can be a combination of:

* Send – Gives the right to send messages to the entity
* Listen – Gives the right to listen or receive to the entity
* Manage – Gives the right to manage the topology of the namespace, including creation and deletion of entities

In this task, an authorization rule with **Listen** and **Send** permissions will be created.

1. On the **Overview** page of the **evhns-az220-training-{your-id}** namespace, click the **evh-az220-adt2func** Event Hub.

1. In the left navigation area, under **Settings**, click **Shared access policies**.

    An empty list of policies that are specific to his event hub is shown.

1. To create a new authorization rule, click **+ Add**.

1. In the **Add SAS Policy** pane, under policy name, enter **ADTHubPolicy**.

    +++ADTHubPolicy+++

1. In the list of permissions, check only **Send** and **Listen**.

1. Click **Create** to save the authorization rule.

    After a moment, the pane closes and list of policies refresh.

1. To retrieve the primary connection string for the authorization rule, click **ADTHubPolicy** in the list.

    The **SAS Policy: ADTHubPolicy** pane will open.

1. Copy the **Connection string-primary key** value and add it to your text file.

1. Close the **SAS Policy: ADTHubPolicy** pane.

#### Task 4 - Add an event hub endpoint to the Azure Digital Twins instance

Now that the Event Hub is created, it must be added as an endpoint that the Azure Digital Twins instance can use as an output to send events.

1. Navigate to the **adt-az220-training-{your-id}** instance.

1. In the left navigation area, under **Connect outputs**, click **Endpoints**.

    A list of endpoints, will be displayed.

1. To add a new endpoint, click **+ Create an endpoint**.

1. In the **Create an endpoint** pane, under **Name**, enter **eventhub-endpoint**.

    +++eventhub-endpoint+++

1. Under **Endpoint type**, select **Event Hub**.

    The UI updates to include fields to specify the Event Hub details.

1. In the **Subscription** dropdown, ensure that the Azure subscription that you intend to use for this lab is selected.

1. In the **Event hub namespace** dropdown, select **evhns-az220-training-{your-id}**.

1. In the **Event Hub** dropdown, select **evh-az220-adt2func**.

1. In the  **Authorization rule** dropdown, select **ADTHubPolicy**.

   This is the authorization rule that was created earlier.

1. To create the endpoint, click **Save**.

    The pane closes and, after a moment, the endpoint list updates to include the new endpoint.

#### Task 5 - Add a route to send telemetry to the event hub

With the addition of the Event Hub endpoint to the Azure Digital Twins instance, it is now necessary to create a route that sends twin telemetry events to this endpoint.

1. In the left navigation area, under **Connect outputs**, click **Event routes**.

    A list of existing routes are displayed.

1. To add a new event route, click **+ Create an event route**.

1. In the **Create an event route** pane, under **Name**, enter **eventhub-telemetryeventroute**.

    +++eventhub-telemetryeventroute+++

1. In the **Endpoint** dropdown, select **eventhub-endpoint**.

1. Under **Add a event route filter**, leave the **Advanced editor** set to **Disabled**.

    The advanced editor supports entering a specific filtering expression - for this task, the UI is sufficient.

1. In the **Event types** dropdown, select only **Telemetry**.

    Notice the **Filter** field displays the generated filter expression.

1. To create the event route, click **Save**.

    The pane closes and, after a moment, the event route list updates to include the new route.

#### Task 6 - Create TSI Event Hub and Policy

This time, the Event Hub and authorization rule will be created using the Azure CLI.

1. Return to the command prompt and verify the session is still logged in by entering:

    ```powershell
    az account list
    ```

    If prompted to run `az login`, do so and login.

1. To create the Event Hub between the Azure Function and TSI, enter the following command:

    ```powershell
    az eventhubs eventhub create --name evh-az220-func2tsi --resource-group rg-az220 --namespace-name evhns-az220-training-{your-id}
    ```

    Remember to replace **{your-id}**.

1. To create an authorization rule with listen and send permissions on the new Event Hub, enter the following command:

    ```powershell
    az eventhubs eventhub authorization-rule create --rights Listen Send --resource-group rg-az220  --eventhub-name evh-az220-func2tsi --name TSIHubPolicy --namespace-name evhns-az220-training-{your-id}
    ```

    Remember to replace **{your-id}**.

1. To retrieve the primary connection string for the authorization rule, enter the following command:

    ```powershell
    az eventhubs eventhub authorization-rule keys list --resource-group rg-az220 --eventhub-name evh-az220-func2tsi --name TSIHubPolicy --namespace-name evhns-az220-training-{your-id} --query primaryConnectionString -o tsv
    ```

    Remember to replace **{your-id}**.

    The output will be similar to:

    ```text
    Endpoint=sb://evhns-az220-training-dm030821.servicebus.windows.net/;SharedAccessKeyName=TSIHubPolicy;SharedAccessKey=x4xItgUG6clhGR9pZe/U6JqrNV+drIfu1rlvYHEdk9I=;EntityPath=evh-az220-func2tsi
    ```

1. Copy the **Connection string-primary key** value and add it to your text file.

    The two connection strings will be required in the next task.

#### Task 7 - Add the Endpoint addresses as app settings for Azure Function

In order for an Azure Function to connect to an Event Hub, it must have access to the connection string for a policy with the appropriate rights. In this scenario, two Event Hubs are involved  - the hub that publishes the event from Azure Digital Twins and the hub that publishes the data transformed by the Azure Function to TSI.

1. To supply the Event Hub authorization rule connections strings as environments variable, in the Azure portal, navigate to the **func-az220-hub2adt-training-{your-id}** instance.

1. In the left navigation area, under **Settings**, click **Configuration**.

1. On the **Configuration** page, on the **Application settings** tab, review the current **Application settings** that are listed.

    The **ADT_SERVICE_URL** that was added earlier via the CLI should be listed.

1. To add an environment variable for the adt2func rule connection string, click **+ New application setting**.

1. On the **Add/Edit application setting** pane, in the **Name** field, enter **ADT_HUB_CONNECTIONSTRING**

    +++ADT_HUB_CONNECTIONSTRING+++

1. In the **Value** field, enter the ADT connection string value that was saved to your text file and ends with **EntityPath=evh-az220-adt2func**.

    The value should be similar to the following:

    ```
    Endpoint=sb://evhns-az220-training-dm030821.servicebus.windows.net/;SharedAccessKeyName=ADTHubPolicy;SharedAccessKey=fHnhXtgjRGpC+rR0LFfntlsMg3Z/vjI2z9yBb9MRDGc=;EntityPath=evh-az220-adt2func
    ```

1. To close the pane, click **OK**.

    > **IMPORTANT**: The setting is not yet saved.

1. To add an environment variable for the func2tsi rule connection string, click **+ New application setting**.

1. On the **Add/Edit application setting** pane, in the **Name** field, enter **TSI_HUB_CONNECTIONSTRING**

    +++TSI_HUB_CONNECTIONSTRING+++

1. In the **Value** field, enter the authorization rule connection string value that was saved to your text file and ends with **EntityPath=evh-az220-func2tsi**.

    The value should be similar to the following:

    ```
    Endpoint=sb://evhns-az220-training-dm030821.servicebus.windows.net/;SharedAccessKeyName=TSIHubPolicy;SharedAccessKey=x4xItgUG6clhGR9pZe/U6JqrNV+drIfu1rlvYHEdk9I=;EntityPath=evh-az220-func2tsi
    ```

1. To close the pane, click **OK**.

    > **IMPORTANT**: The setting is not yet saved.

1. To save the both of the new settings, click **Save**, and then click **Continue**.

    > **NOTE**: Any change to the application settings will restart the functions.

#### Task 8 - Review a telemetry Azure Function

In this task, the second Azure function will be reviewed. This function will be responsible for mapping the device telemetry messages to an alternate format for TSI. This approach has the advantage of being able to handle changes to the device telemetry format without changing the TSI solution.

1. In Visual Studio Code, open the **Contoso.AdtFunctions** project.

1. Open the **TelemetryFunction.cs** file.

1. Locate the **Run** method definition and review the code.

    ```csharp
    public static class TelemetryFunction
    {
        [FunctionName("TelemetryFunction")]
        public static async Task Run(
            [EventHubTrigger("evh-az220-adt2func", Connection = "ADT_HUB_CONNECTIONSTRING")] EventData[] events,
            [EventHub("evh-az220-func2tsi", Connection = "TSI_HUB_CONNECTIONSTRING")] IAsyncCollector<string> outputEvents,
            ILogger log)
        {
            var exceptions = new List<Exception>();

            foreach (EventData eventData in events)
            {
                try
                {
                    // main processing code here
                }
                catch (Exception e)
                {
                    // We need to keep processing the rest of the batch - capture this exception and continue.
                    // Also, consider capturing details of the message that failed processing so it can be processed again later.
                    exceptions.Add(e);
                }
            }

            // Once processing of the batch is complete, if any messages in the batch failed processing throw an exception so that there is a record of the failure.

            if (exceptions.Count > 1)
                throw new AggregateException(exceptions);

            if (exceptions.Count == 1)
                throw exceptions.Single();
        }
    }
    ```

    Take a moment to look at the **Run** method definition. The **events** parameter makes use of the **EventHubTrigger** attribute - the attribute's constructor takes the name of the event hub, the **optional** name of the consumer group (**$Default** is used if omitted), and the name of an app setting that contains the connection string. This configures the function trigger to respond to an event sent to an event hub event stream. As **events** is defined as an array of EventData, it can be populated with a batch of events.

    > **TIP** To learn more about the **EventHubTrigger**, review the following resource:
    > [Azure Event Hubs trigger for Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-hubs-trigger?tabs=csharp)

    The next parameter, **outputEvents** has the **EventHub** attribute - the attribute's constructor takes the name of the event hub and the name of an app setting that contains the connection string. Adding data to the **outputEvents** variable will publish it to the associated Event Hub.

    As this function is processing a batch of events, a way to handle errors is to create a collection to hold exceptions. The function will then iterate through each event in the batch, catching exceptions and adding them to the collection. Skip[ to the end of the method, and you will see that if there are multiple exceptions, an **AggregaeException** is created with the collection, if a single exception is generated, then the single exception is thrown.

1. To review the code that checks to see if the event contains Cheese Cave Device telemetry, locate the **// INSERT check telemetry below here** comment and review the following code below it:

    ```csharp
    if ((string)eventData.Properties["cloudEvents:type"] == "microsoft.iot.telemetry" &&
        (string)eventData.Properties["cloudEvents:dataschema"] == "dtmi:com:contoso:digital_factory:cheese_factory:cheese_cave_device;1")
    {
        // INSERT TSI Event creation below here
    }
    else
    {
        log.LogInformation($"Not Cheese Cave Device telemetry");
        await Task.Yield();
    }
    ```

    This code checks that the current event is telemetry from a Cheese Cave Device digital twin - if not, logs that it isn't and then forces the method to complete asynchronously - this can make better use of resources.

    > **TIP**: To learn more about the use of **await Task.Yield();** review the following resource:
    > * [Task.Yield Method](https://docs.microsoft.com/dotnet/api/system.threading.tasks.task.yield?view=net-5.0)

1. To review the code that processes the event and creates a message for TSI, locate the **// INSERT TSI Event creation below here** comment and review the following code below it:

    ```csharp
    // The event is Cheese Cave Device Telemetry
    string messageBody = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
    JObject message = (JObject)JsonConvert.DeserializeObject(messageBody);

    var tsiUpdate = new Dictionary<string, object>();
    tsiUpdate.Add("$dtId", eventData.Properties["cloudEvents:source"]);
    tsiUpdate.Add("temperature", message["temperature"]);
    tsiUpdate.Add("humidity", message["humidity"]);

    var tsiUpdateMessage = JsonConvert.SerializeObject(tsiUpdate);
    log.LogInformation($"TSI event: {tsiUpdateMessage}");

    await outputEvents.AddAsync(tsiUpdateMessage);
    ```

    As the **eventData.Body** is defined as an **ArraySegment**, rather than just an array, the portion of the underlying array that contains the **messageBody** must be extracted, and then deserialized.

    > **TIP**: To learn more about an **ArraySegment** review the following content:
    > * [ArraySegment&lt;T&gt; Struct](https://docs.microsoft.com/dotnet/api/system.arraysegment-1?view=net-5.0)

    A **Dictionary** is then instantiated to hold the key/value pairs that will make up the properties sent within the TSI event. Notice that the **cloudEvents:source** property (which contains the fully qualified twin ID - similar to **adt-az220-training-dm030821.api.eus.digitaltwins.azure.net/digitaltwins/sensor-th-0055**) is assigned to the **\$dtId** key. This key has special meaning as the Time Series Insights environment created during setup is using **\$dtId** as the **Time Series ID Property**.

    The **temperature** and **humidity** values are extracted from the message and added to the TSI update.

    The update is then serialized to JSON and added to the **outputEvents** which publishes the update to the Event Hub.

    The function is now ready to be published.

#### Task 9 - Publish the function app to Azure

1. In the Azure Functions extension for Visual Studio Code, select **Deploy to Function App**:

    ![Visual Studio Code deploy to function app](media/LAB_AK_19-deploy-to-function-app.png)

1. When prompted, make these selections:

    * **Select subscription**: If prompted, select the subscription you are using for this lab.
    * **Select Function App in Azure**: Select **func-az220-hub2adt-training-{your-id}**.

    When asked to confirm the deploy, click **Deploy**.

    The function will then be compiled and, if successful, deployed. This may take a few moments.

1. Once the deployment has completed, the following prompt will be displayed:

    ![Visual Studio Code deployment complete - select stream logs](media/LAB_AK_19-function-stream-logs.png)

1. Click **Stream logs**, and then, in the confirmation dialog to enable application logging, click **Yes**.

    The **OUTPUT** pane will now display the log stream for the deployed function - this will timeout after 2 hours. There will be some status information displayed, however there will not be any diagnostic information from the function itself until it is launched. That will be covered in the next exercise.

    The streaming can be stopped or started at any time by right-clicking the Azure function in Visual Studio Code and select **Start Streaming Logs** or **Stop Streaming Logs**:

    ![Visual Studio Code Azure Function start streaming logs](media/LAB_AK_19-start-function-streaming.png)

#### Task 10 - Configure TSI

1. Switch to your Azure Portal window and navigate to your dashboard.

1. Select the **tsi-az220-training-{your-id}** resource.

    > **NOTE**: This resource was created by the setup script. If you have not run it, do so now - the script will not impact existing resources.

1. On the **Overview** pane, in the **Essentials** section, locate the **Time Series ID Property** field and the value - **$dtid**. This value was specified during the creation of the TSI environment and should match with a field in the event data being streamed to TSI.

    > **IMPORTANT**: The **Time Series ID Property** value is specified during the creation of the TSI environment and cannot be changed later.

1. In the left navigation area, under **Settings**, click **Event Sources**.

    The list of **Event Sources** will be displayed - at this point, it should be empty.

1. To add a new Event Source, click **+ Add**.

1. On the **New event source** pane, under **Event source name**, enter **adt-telemetry**

    +++adt-telemetry+++

1. Under **Source**, select **Event Hub**.

1. Under **Import option**, select **Use Event Hub from available subscriptions**.

1. Under **Subscription ID**, select the subscription being used for this lab.

1. Under **Event Hub namespace**, select **evhns-az220-training-{your-id}**

1. Under **Event Hub name**, select **evh-az220-func2tsi**.

1. Under **Event Hub policy value**, select **TSIHubPolicy**.

    Notice that the read-only field **Event Hub policy key** is automatically populated.

1. Under **Event Hub consumer group**, select **$Default**.

    > **TIP**: As there is only one event reader for the **evh-az220-func2tsi** Event Hub, using the **$Default** consumer group is fine. If more readers were to be added, then it is recommended that one consumer group per reader is used. Consumer groups are created on the Event Hub.

1. Under **Start time**, select **All my data**.

    Notice there are other options available, such as **Beginning now (default)**, that may be more suitable for a production environment.

1. Under **Event serialization format**, notice that the read-only value is **JSON**.

1. Under **Timestamp property name**, leave the value empty.

    > **TIP**: This specifies the name of the event property that should be used as the event timestamp. When not specified, the event enqueue time within the event source will be used as the event timestamp.

1. To create the Event Source, click **Save**.

#### Task 11 - Visualize the telemetry data in Time Series Insights

Now, data should be flowing into your Time Series Insights instance, ready to be analyzed. Follow the steps below to explore the data coming in.

1. In the browser, return to the **Overview** pane of the **tsi-az220-training-{your-id}** resource.

1. To navigate to the **TSI Explorer**, click **Go to TSI Explorer**.

    The **TSI Explorer** will open in a new tab in the browser.

1. In the explorer, you will see Azure Digital Twins shown on the left.

    ![TSI Explorer data](media/LAB_AK_19-tsi-explorer.png)

1. To add telemetry to the graph, click the twin and select **EventCount**, **humidity** and **temperature**, and then click **Add**

    Select an appropriate time range, and the displayed data will be similar to:

    ![TSI Explorer showing time series data](media/LAB_AK_19-tsi-explorer-data.png)

    > **Note**: It may take some time for sufficient data to be displayed.

    By default, digital twins are stored as a flat hierarchy in Time Series Insights, however they can be enriched with model information and a multi-level hierarchy for organization. You can write custom logic to automatically provide this information using the model and graph data already stored in Azure Digital Twins.

    Congratulations - you are now passing device telemetry data to Time Series Insights.

1. Delete the Azure resources that you created during the lab.

    If you have one or more Azure resource groups dedicated to this lab, it is recommended that you delete them before exiting the lab environment.

    > **Note**: Resources that your created during this lab include an associated fee. If you are using your own Azure account, be sure to clean up your resources to minimize any charges.
