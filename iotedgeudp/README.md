# Azure IoT Edge UDP Messages Processor #

Based on original work [danigian/iot-edge-udp](https://github.com/danigian/iot-edge-udp).

The Azure IoT Edge Module is the folder structure

- Azure IoT
	- BrunoEdgeSolution
		- modules
			- UDPFwder

This module will

  - Open a UDP server port to listen UDP messages
  - When a message is received, the message will be stored in an Azure Blob Storage for IoT Edge as a file
  - The UDP server will also trigger a Azure IoT device event with the UDP Message as a property of the message

Current module is published in Azure Container Registry available AzureIoTEdgeStorageCnnString

- ebiot.azurecr.io/udpfwder:0.0.80-amd64
- ebiot.azurecr.io/udpfwder:0.0.80-arm32v7


## Software Prerequisites ##

- [**Visual Studio Code**](https://code.visualstudio.com/)
- [**Visual Studio Code IoT Toolkit**](https://github.com/microsoft/vscode-azure-iot-toolkit)

## UDP Messages Forwarder Environment Variables ##

- **LogEnabled** *(boolean)*. Define if log must be written to the docker container. Default value, False.
- **SaveMessageToAzureStorage** *(boolean)*. If True, when a message is received it will be saved to the define Storage in Storage Connection String. Default value, False.
- **TriggerMessageAsDeviceProperty** *(boolean)*. If True, when a message is received it will be added as a property in an Azure IoT device event message. Default value, False.
- **EdgeUdpPort** *(int)*. UDP to be opened. Default value is 1208
- **UDPMessagePropertyName** *(string)*. Name of the property to be used to store the UDP Message, raised as a Azure IoT device event. Default value is 'UDPMessage'.
- **OutputTarget** *(string)*. Defines the output target for the Azure IoT device message. Default value is 'Output1'.
- **AzureIoTEdgeStorageCnnString** *(string)*. Defines the Azure Storage Connection String for the local IoT Edge Storage. Default value is ''. 
- **AzureIoTEdgeStorageContainerName** *(string)*. Defines the Container name for the local Azure IoT Edge. Default value is ''
- **AzureIoTEdgeTempFolder** *(string)*. Defines the temp folder location. Default value is '/tmp/'. 
- **BlobUseV11** *(boolean)*. Defines if the module will use V12 or V11 to interact with the Azure Storage Blob (see below)

**Notes**

- If the property [AzureIoTEdgeStorageCnnString] is empty, the module will not send any UDP message to an Azure Storage Blob.
- The default template for a connection string is: [DefaultEndpointsProtocol=http;BlobEndpoint=http://<IoTEdge device IP>:11002/<account name>;AccountName=<account name>;AccountKey=<account key>;]


### UDP Messages Forwarder - Container Create Options ###

In the Container Create Options, the UDP ports must be defined.

	{
		"ExposedPorts": {
			"35001/udp": {}
		},
		"HostConfig": {
			"PortBindings": {
				"35001/udp": [
					{
						"HostPort": "35001"
					}
				]
			}
		}
	}

### UDP Messages Forwarder - Reading Logs ###

In the edge device, the command 'sudo iotedge list' should list the installed modules. The UDP module now is hosted from ' ebiot.azurecr.io/udpfwder'

	pi@rpiIoT01:~ $ sudo iotedge list
	NAME                       STATUS           DESCRIPTION      CONFIG
	azureblobstorageoniotedge  running          Up 2 hours       mcr.microsoft.com/azure-blob-storage:latest
	edgeAgent                  running          Up 2 days        mcr.microsoft.com/azureiotedge-agent:1.0
	edgeHub                    running          Up 2 hours       mcr.microsoft.com/azureiotedge-hub:1.0
	udpmessages                running          Up 2 hours       ebiot.azurecr.io/udpfwder:0.0.78-arm32v7

When the module starts, it will log the Environmental Variables. If LogEnabled is True, the log should be similar to this opened

	02/08/2021 21:13:38 - SaveMessageToAzureStorage: True
	02/08/2021 21:13:38 - TriggerMessageAsDeviceProperty: True
	02/08/2021 21:13:38 - EdgeUdpPort: 35001
	02/08/2021 21:13:38 - UDPMessagePropertyName: UDPMessage
	02/08/2021 21:13:38 - OutputTarget: output1
	02/08/2021 21:13:38 - BlobUseV11: False
	02/08/2021 21:13:38 - AzureIoTEdgeStorageCnnString: DefaultEndpointsProtocol=http;BlobEndpoint=http://<IP>:11002/<AccName>;AccountName=<AccName>;AccountKey=<AccKey>;
	02/08/2021 21:13:38 - AzureIoTEdgeStorageContainerName: cont1
	02/08/2021 21:13:38 - AzureIoTEdgeTempFolder: /tmp/

Everytime a new message is received via UDP, it will be logged. In example

	02/08/2021 19:24:55 - 6 - Message Received
	02/08/2021 19:24:55 -    {"EnqueuedTimeUtc":"2021-02-03T13:04:39.9180000Z","Properties":{"UDPMessage":"{\n  \"color\": \"#33FFCC\",\n  \"group\": \"Plumbing\",\n  \"id\": \"a4da22e3eeff\",\n  \"lastAreaId\": \"30f5ffc0-969a-400b-8feb-e276eff029f5\",\n  \"lastAreaName\": \"Tracking001_2D\",\n  \"lastAreaTS\": 1612357458492,\n  \"name\": \"12345678910\",\n  \"smoothedPosition\": [\n    5.94,\n    4.66,\n    1.2\n  ],\n  \"smoothedPositionAccuracy\": 0.1,\n  \"smoothedPositionWGS84\": [\n    43.2227247,\n    -79.6697913\n  ],\n  \"zones\": [{\n    \"name\": \"L02S02\",\n    \"id\": \"66cc16a5-1a82-43fc-93b7-4802885ee203\"\n  }]\n}"},"SystemProperties":{"connectionDeviceId":"QUUPPA-NUC10i3FNK","connectionModuleId":"udpfwder","connectionAuthMethod":"{\"scope\":\"module\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}","connectionDeviceGenerationId":"637477934251502925","enqueuedTime":"2021-02-03T13:04:39.9180000Z"},"Body":""}
	02/08/2021 19:24:55 -    Start message save to Azure Storage
	02/08/2021 19:24:55 -    Uploading to Blob storage as blob: http://192.168.1.245:11002/rpiiot01storagemessages/cont1/udp-2021-02-08_07-24-55-637484090952010853.json
	02/08/2021 19:24:55 -    Start message event as device property - Output Target: 'output1' - Message Property: 'UDPMessage'
	02/08/2021 19:24:55 -            >> 6 Processed

## Azure Blob Storage and CSharp implementation ##

Ugly hack to support Azure Storage Blob on IoT Edge on Ubuntu

V11 works on Ubuntu Server 18.04 >> https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-dotnet-legacy?view=iotedge-2018-06

V12 (latest) version works on RPi >> https://docs.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-dotnet?view=iotedge-2018-06
