---
uid: PIAdapterForOPCUADataSelectionConfiguration
---

# Data selection

In addition to the data source configuration, you need to provide a data selection configuration to specify the data you want the adapter to collect from the data sources.

When you add a data source without providing a data selection configuration, the adapter browses the OPC UA server address space and populates the available OPC UA variables into the data selection configuration.  A comma-separated collection of nodeIds (RootNodeIds) in the data source configuration serves as filters to browse only a subset of the OPC UA server. Data selection items configured this way must still be selected for data retrieval. OPC UA data from OPC UA variables is read through subscriptions (unsolicited reads).

You can decide to have the data selection configuration file generated automatically or you can create it manually yourself.

## Generate default OPC UA data selection configuration file

A default OPC UA data selection file will be created if there is no OPC UA data selection configuration although a valid data source exists.

**Note:** To avoid possible time and resource expensive browse operations due to reasons described previously, OSIsoft recommends that you manually create a data selection file instead of automatically creating the default data selection file. For more information, see [Configure OPC UA data selection](#configure-opc-ua-data-selection).

Complete the following steps to generate a default data selection file:

1. Add an OPC UA adapter with a unique ComponentId. For more information, see [System components configuration](xref:SystemComponentsConfiguration).
  
2. Configure a valid OPC UA data source. For more information, see [PI Adapter for OPC UA data source configuration](xref:PIAdapterForOPCUADataSourceConfiguration).

   Once you complete these steps, a default OPC UA data selection configuration file will be generated in the configuration directory for the corresponding platform.
  
   The following are example locations of the file created. In this example, it is assumed that the ComponentId of the OPC UA component is OpcUa1:

   Windows: `%programdata%\OSIsoft\Adapters\OpcUa\Configuration\OpcUa1_DataSelection.json`
  
   Linux: `/usr/share/OSIsoft/Adapters/OpcUa/Configuration/OpcUa1_DataSelection.json`

3. Copy the file to a different directory.
    - For content structure, see [OPC UA data selection example](#opc-ua-data-selection-examples).

4. Use any text editor to change the value of any **Selected** key from `false` to `true` in the file.

   Once the configuration is updated, the adapter will subscribe to data for all items that are set to *Selected=true*.

5. In the same directory where you edited the file, run the following `curl` command:

    **Note:** `5590` is the default port number. If you selected a different port number, replace it with that value.

      ```bash
      curl -d "@OpcUa1_DataSelection.json" -H "Content-Type: application/json" -X PUT      "http://localhost:5590/api/v1/configuration/OpcUa1/Dataselection"
      ```

## Configure OPC UA data selection

Complete the following steps to configure an OPC UA data selection. Use the `PUT` method in conjunction with the `api/v1/configuration/<ComponentId>/DataSelection` REST endpoint to initialize the configuration.

1. Using a text editor, create an empty text file.

2. Copy and paste an example configuration for an OPC UA data selection into the file.

    For sample JSON, see [OPC UA data selection examples](#opc-ua-data-selection-examples).

3. Update the example JSON parameters for your environment.

    For a table of all available parameters, see [OPC UA data selection parameters](#opc-ua-data-selection-parameters).

4. Save the file. For example, as `ConfigureDataSelection.json`.

5. Open a command line session. Change directory to the location of `ConfigureDataSelection.json`.

6. Enter the following cURL command (which uses the `PUT` method) to initialize the data selection configuration.

    ```bash
    curl -d "@ConfigureDataSelection.json" -H "Content-Type: application/json" -X PUT "http://localhost:5590/api/v1/configuration/OpcUa1/DataSelection"
    ```

    **Notes:**
  
    * If you installed the adapter to listen on a non-default port, update `5590` to the port number in use.
    * If you use a component ID other than `OpcUa1`, update the endpoint with your chosen component ID.
    * For a list of other REST operations you can perform, like updating or deleting a data selection configuration, see [REST URLs](#rest-urls).
    <br/>
    <br/>

## OPC UA data selection schema

The full schema definition for the OPC UA data selection configuration is in the `OpcUa_DataSelection_schema.json` file located in one of the following folders:

Windows: `%ProgramFiles%\OSIsoft\Adapters\OpcUa\Schemas`

Linux: `/opt/OSIsoft/Adapters/OpcUa/Schemas`

## OPC UA data selection parameters

The following parameters are available for configuring an OPC UA data selection:

| Parameter     | Required | Type | Description |
|---------------|----------|------|-------------|
| **Selected** | Optional | `boolean` | Use this field to select or clear a measurement. To select an item, set to true. To remove an item, leave the field empty or set to false. <br><br>Allowed value: `true` or `false` <br>Default value: `true`|
| **Name**      | Optional | `string` | Name of the data item collected from the data source. <br><br>Default value: `null` <br>results in **StreamId** value being used also as a **Name** |
| **NodeId**    | Required | `string` | The NodeId of the variable.<br><br>Examples<br>`"ns=5;AString"`<br>`"ns=2;i=203"`<br>`"ns=<NamespaceIndex>;<IdentifierType>=<Identifer>"` |
| **StreamID** | Optional | `string` | The custom stream ID used to create the streams. If not specified, the adapter will generate a default stream ID based on the measurement configuration. The StreamId serves as the unique identifier of a data selection item. A properly configured custom stream ID follows these rules:<br><br>Is not case-sensitive.<br>Can contain spaces.<br>Cannot start with two underscores ("__").<br>Can contain a maximum of 2000 characters.<br>Cannot use the following characters: / : ? # [ ] @ ! $ & ' ( ) \ * + , ; = % < > &#124;<br>Cannot start or end with a period.<br>Cannot contain consecutive periods.<br>Cannot consist of only periods. |
| **DataFilterID** | Optional | `string` | Enables data filtering for this data selection item if the ID of a data filter is specified.<br>**Note:** If the specified **DataFilterId** does not exist, unfiltered data is sent until that **DataFilterId** is created. |

## OPC UA data selection examples

The following are examples of valid OPC UA data selection configurations:
### Minimal data selection configuration

```json
[
 {
    "NodeId": "ns=5;s=Random1"
  },
  {
    "NodeId": "ns=5;s=Sawtooth1"
  },
  {
    "NodeId": "ns=5;s=Sinusoid1"
  }
]
```

 ### Complete data selection configuration

```json
[
 {
    "Selected": true,
    "Name": "CustomStreamName",
    "NodeId": "ns=5;s=Random1",
    "StreamId": "CustomStreamName"
  },
  {
    "Selected": false,
    "Name": null,
    "NodeId": "ns=5;s=Sawtooth1",
    "StreamId": null
  },
  {
    "Selected": true,
    "Name": "5.Sinusoid1",
    "NodeId": "ns=5;s=Sinusoid1",
    "StreamId": null
  }
]
```

## REST URLs

| Relative URL | HTTP verb | Action |
| ------------ | --------- | ------ |
| api/v1/configuration/\<ComponentId\>/DataSelection  | `GET` | Retrieves the data selection configuration, including all data selection items. |
| api/v1/configuration/\<ComponentId\>/DataSelection  | `PUT` | Configures or updates the data selection configuration. The adapter starts collecting data for each data selection item when the following conditions are met:<br/><br/>&bull; The data selection configuration `PUT` request is received.<br/>&bull; A data source configuration is active. |
| api/v1/configuration/\<ComponentId\>/DataSelection | `DELETE` | Deletes the active data selection configuration. The adapter stops collecting data. |
| api/v1/configuration/\<ComponentId\>/DataSelection | `PATCH` | Allows partial updates of configured data selection items.<br/><br/>**Note:** The request must be an array containing one or more data selection items. Each item in the array must include its **StreamId**. |
| api/v1/configuration/\<ComponentId\>/DataSelection/\<StreamId\> | `PUT` | Updates or creates a new data selection item by **StreamId**. For new items, the adapter starts collecting data after the request is received. |
| api/v1/configuration/\<ComponentId\>/DataSelection/\<StreamId\> | `DELETE` | Deletes a data selection item from the configuration by **StreamId**. The adapter stops collecting data for the deleted item. |

**Note:** Replace \<ComponentId\> with the Id of your OPC UA component, for example _OpcUa1_.
