Freebird Client/Server Message Formats (through websocket)
===============

## Table of Contents

1. [Overiew](#Overiew)  
2. [Interfaces](#Interfaces)  
    - [Request](#Request)  
    - [Response](#Response)  
    - [Indication](#Indication)  
3. [Data Model](#DataModel)  
    - [Request](#RequestData)  
    - [Response](#ResponseData)  
    - [Indication](#IndicationData)  
4. [RSP Status Codes](#RspCodes)  
5. [Appendix](#Appendix)  
    - [Device Information (devInfo) Object](#devInfoObj)  
    - [Gadget Information (gadInfo) Object](#gadInfoObj)  
    - [Netcore Information (ncInfo) Object](#ncInfoObj)  
    - [Gadget Classes](#gadClasses)  
    - [ Attribute Report Configuration Object](#reportCfg)  

<br />
  
<a name="Overiew"></a>  
## 1. Overview  
  
This document describes the APIs of how a freebird web Client can communicate with the freebird Server through [websocket](http://www.websocket.org/). The APIs are based on a _**Request**_, _**Response**_, and _**Indication**_ messages model. The message object (JSON) has a `__intf` field to denote which type a message is. The message type can be **'REQ'**, **'RSP'**, or **'IND'**.  
  
The freebird framework has _**net**_, _**dev**, _ and _**gad**_ subsystems responsible for network, device, and gadget management, respectively. In brief, a network is formed with many devices, and each device may have some gadgets on it. A gadget is the real application in a manchine network.  
  
Let's take a wifi weather station in a machine network for example. The weather station is made up of temperature, humidity, and pm2.5 sensors, where each sensor is a gadget. A device, such as Arduino(with wifi connection), ESP8266, MT7688, RaspberryPi or Beaglebone, is the carrier of applications(gadgets). Now we know, this weather station has 3 gadgets on it, but only has a single device in it. Here is another example, we have a bluetooth low-energy (BLE) light switch in the network. This is a simple one-device-with-one-gadget machine, we can say "there is only one gadget, a light switch, implemented on a TI CC2540 BLE SoC device."  
  
The concept of _**net**_, _**dev**_, and _**gad**_ subsystems in freebird framework well separates the machine management and application management from each other. This brings developers a more clear, convenient and flexible way in building up a IoT machine network.  
  
********************************************
  
<br />
  
<a name="Interfaces"></a>  
## 2. Interfaces  
  
In freebird, the web-Client and Server communicates with each other through websocket along with a JSON message. 
The `__intf` field in a message can be 'REQ', 'RSP' or 'IND' to denote the interface.  
  
<a name="Request"></a>
### Request  
  
- **Direction**:  
    Client sends to Server to request something or to ask the server to perform an operation.  
- **Interface**:  
    __intf = 'REQ'  
- **Message keys**:  
    { __intf, subsys, seq, id, cmd, args }  
  
    | Property | Type            | Description                                                                                                               |
    |----------|-----------------|---------------------------------------------------------------------------------------------------------------------------|
    | __intf   | String          | 'REQ'                                                                                                                     |
    | subsys   | String          | Only 3 types accepted. They are 'net', 'dev', and 'gad' to denote which subsystem is this message going to                |
    | seq      | Number          | Sequence number of this REQ/RSP transaction                                                                               |
    | id       | Number          | Id of the sender. id is meaningless if `subsys === 'net'`. id is **device id** if `subsys === 'dev'`. id is **gadget id** if `subsys === 'gad'`. It is noticed that id = 0 is reserved for the freebird web-Client and Server |
    | cmd      | String          | Command Identifier corresponding to the API name                                                                          |
    | args     | Object          | A value-object that contains command arguments. Please see section [Request Data Model](#RequestData) to learn more about the `args` data object |
  
- **Message Example**:  
  
    ```js
    { 
        __intf: 'REQ',
        subsys: 'net',
        seq: 3,
        id: 0,
        cmd: 'getDevs',
        args: {
            ids: [ 2, 4, 18, 61 ]
        }
    }
    ```
********************************************
  
<br />
  
<a name="Response"></a>
### Response  
  
- **Direction**:  
    Server responds to Client with the results of the client asking for.  
- **Interface**:  
    __intf = 'RSP'  
- **Message keys**:  
    { __intf, subsys, seq, id, cmd, status, data }  
  
    | Property | Type            | Description                                                                                                                        |
    |----------|-----------------|------------------------------------------------------------------------------------------------------------------------------------|
    | __intf   | String          | 'RSP'                                                                                                                              |
    | subsys   | String          | Only 3 types accepted. They are 'net', 'dev', and 'gad' to denote which subsystem is this message coming from                      |
    | seq      | Number          | Sequence number of this REQ/RSP transaction                                                                                        |
    | id       | Number          | Id of the sender. id is meaningless if `subsys === 'net'`. id is **device id** if `subsys === 'dev'`. id is **gadget id** if `subsys === 'gad'`. It is noticed that id = 0 is reserved for the freebird web-Client and Server  |
    | cmd      | String          | Command Identifier corresponding to the API name                                                                                   |
    | status   | Number          | [Status code of the response](#RspCodes)                                                                                           |
    | data     | Depends         | Data along with the response. To learn more about the data format corresponding to each command, please see section [Response Data Model](#ResponseData). |
  
- **Message Example**:  
  
    ```js
    { 
        __intf: 'RSP',
        subsys: 'net',
        seq: 17,
        id: 0,
        cmd: 'getAllDevIds',
        status: 0,
        data: [ 2, 4, 18, 61 ]
    }
    ```
********************************************
  
<br />
  
<a name="Indication"></a>
### Indication  
  
- **Direction**:  
    Server indicates Client  
- **Interface**:  
    __intf = 'IND'  
- **Message keys**:  
    { __intf, subsys, type, id, data }  

    | Property | Type            | Description                                                                                                                                 |
    |----------|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------|
    | __intf   | String          | 'IND'                                                                                                                                       |
    | subsys   | String          | Only 3 types accepted. They are 'net', 'dev', and 'gad' to denote which subsystem is this indication coming from                            |
    | type     | String          | There are few types of indication accepted, such as 'attrChanged'. Please see section [Indication types](#IndTypes) for details             |
    | id       | Number          | Id of the sender. id is meaningless if `subsys === 'net'`. id is **device id** if `subsys === 'dev'`. id is **gadget id** if `subsys === 'gad'` |
    | data     | Depends         | Data along with the indication. Please see section [Indication Data Model](#IndicationData) to learn more about the indication data format  |
  
<a name="IndTypes"></a>
- **Indication types**:  
  
    | Indication Type | Description                                                                         |
    |-----------------|-------------------------------------------------------------------------------------|
    | 'permitJoining' | Server is now allowing devices to join the network.                                 |
    | 'netChanged'    | Network parameter(s) of a device has changed                                        |
    | 'statusChanged' | Status of a device has changed. The status can be 'online', 'sleep', and 'offline'  |
    | 'devIncoming'   | A device is incoming                                                                |
    | 'devLeaving'    | A device is leaving                                                                 |
    | 'gadIncoming'   | A gadget is incoming                                                                |
    | 'gadLeaving'    | A gadget is leaving                                                                 |
    | 'attrReport'    | A report message of certain attribute(s) on a gadget                                |
    | 'attrChanged'   | Attribue(s) on a gadget or a device has changed                                     |

- **Message Example**:  
  
    ```js
    { 
        __intf: 'IND',
        subsys: 'gad',
        type: 'attrChanged',
        id: 147,            // sender of this indication is a gadget with id = 147
        data: {
            sensorValue: 24
        }
    }
    ```
  
********************************************
  
<br />
  
<a name="DataModel"></a>  
## 3. Data Model  
  
The data model presents the `args` and `data` formats in the REQ/RSP/IND messages.  
  
<a name="RequestData"></a>
### Request  
  
The request message is an object with keys { __intf, subsys, seq, id, cmd, args }. The following table gives the details of each API.  
  
| Subsystem | Command Name   | Arguments (args)         | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------|----------------|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| net       | 'getAllDevIds' | { [ncName] }             | Get identifiers of all devices on freebird Server. **ncName** is a string and is optional. If **ncName** is given, only identifiers of devices managed by that netcore will be returned from Server.                                                                                                                                                                                                                                                                                                                                                                                   |
| net       | 'getAllGadIds' | { [ncName] }             | Get identifiers of all gadgets on freebird Server. **ncName** is a string and is optional. If **ncName** is given, only identifiers of gadgets managed by that netcore will be returned from Server.                                                                                                                                                                                                                                                                                                                                                                                   |
| net       | 'getDevs'      | { ids }                  | Get information of devices by their ids. **ids** is an array of numbers and each number is a device id, i.e., given `{ ids: [ 5, 6, 77 ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| net       | 'getGads'      | { ids }                  | Get gadget information by gadget id. **ids** is an array of numbers and each number is a gadget id, i.e., given `{ ids: [ 23, 14, 132 ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| net       | 'getNetcores'  | { ncNames }              | Get netcore information by netcore names. **ncNames** is an array of strings and each string is a netcore name, i.e., given `{ ncNames: [ 'ble-core', 'zigbee-core' ] }`.                                                                                                                                                                                                                                                                                                                                                                                                                     |
| net       | 'getBlacklist' | { ncName }               | Get blacklist of the banned devices. **ncName** is the netcore name of which you like to get the blacklist from. **ncName** should be a string. i.e., given `{ ncName: 'ble-core' }`                                                                                                                                                                                                                                                                                                                                                                                                                  |
| net       | 'permitJoin'   | { ncName, duration }     | Allow or disallow devices to join the network. **ncName** is the name of which netcore you like to allow for device joining and **ncName** should be a string. **duration** is the time in seconds which should be a number. Set duration to 0 will immediately close the admission. For example, given `{ ncName: 'zigbee-core', duration: 60 }` will allow zigbee devices to join the zigbee network for 60 seconds.                                                                                                         |
| net       | 'maintain'     | { ncName }               | Maintain the network. **ncName** is the name of which netcore you like to maintain. **ncName** should be a string. When a netcore starts to maintain its own network, all devices managed by it will be refreshed. For example, given `{ ncName: 'ble-core' }` to let the BLE netcore do its maintenance.                                                                                                                                                                                                                                                                                   |
| net       | 'reset'        | { ncName }               | Reset the network. **ncName** is the name of which netcore you like to reset. **ncName** should be a string. Reset a network will remove all devices managed by that netcore. Once reset, the banned devices in the netcore blacklist will also be removed.                                                                                                                                                                                                                                                                                                                                          |
| net       | 'enable'       | { ncName }               | Enable the network. **ncName** is the name of which netcore you like to enable. (The netcore is enabled by default.)                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| net       | 'disable'      | { ncName }               | Disable the network. **ncName** is the name of which netcore you like to disable. If netcore is disabled, no messages can be send out and received from remote devices. That is, messages will be ignored and you will not get any message from the netcore on freebird Server.                                                                                                                                                                                                                                                                                                                |
| dev       | 'read'         | { id, attrName }         | Read an attribute on a device. **id** is the id of which device you like to read from. **attrName** is the attribute you like to read. For example, given `{ id: 20, attrName: 'location' }` to read the location attribute from the device with id = 20.                                                                                                                                                                                                                                                                                                                                    |
| dev       | 'write'        | { id, attrName, value }  | Write a value to an attribute on a device. **id** is the id of which device you like to write a value to. **attrName** is the attribute to be written. For example, given `{ id: 20, attrName: 'location', value: 'kitchen' }` to set the device location attribute to 'kitchen'.                                                                                                                                                                                                                                                                                                               |
| dev       | 'remove'       | { id }                   | Remove a device from the network. **id** is the id of which device to remove.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| dev       | 'identify'     | { id }                   | Identify a device in the network. **id** is the id of which device to be identified.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| dev       | 'ping'        | { id }                   | Ping a device in the network. **id** is the id of which device you like to ping.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| dev       | 'ban'          | { ncName, permAddr }     | Ban a device from the network. Once a device has been banned, freebird will always reject its joining request. If a device is already in the network, freebird will first remove it from the network. **ncName** is the netcore that manages the device you'd like to ban. **permAddr** is the permanent address of  the banned device. For example, given `{ ncName: 'zigbee-core', permAddr: '0x00124b0001ce4b89' }` to ban a zigbee device with an IEEE address of 0x00124b0001ce4b89. The permanent address depends on protocol, such as IEEE address for zigbee devices, BD address for BLE devices, and MAC address for IP-based devices. |
| dev       | unban        | { ncName, permAddr }     | Unban a device. **ncName** is the netcore that manages the banned device. **permAddr** is the permanent address of the banned device. For example, given `{ ncName: 'zigbee-core', permAddr: '0x00124b0001ce4b89' }` to unban a zigbee device with an IEEE address of 0x00124b0001ce4b89.                                                                                                                                                                                                                                                                                              |
| gad       | 'read'         | { id, attrName }         | Read an attribute on a gadget. **id** is the id of which gadget you like to read from. **attrName** is the attribute you like to read. For example, given `{ id: 2316, attrName: 'sensorValue' }` to read the sensed value attribute from a temperature sensor (the sensor is a gadget with id = 2316).                                                                                                                                                                                                                                                                                       |
| gad       | 'write'        | { id, attrName, value }  | Write a value to an attribute on a gadget. **id** is the id of which gadget you like to write a value to. **attrName** is the attribute to be written. For example, given `{ id: 1314, attrName: 'onOff', value: 1 }` to turn on a light bulb (the light bulb is a gadget with id = 1314).                                                                                                                                                                                                                                                                                                     |
| gad       | 'exec'         | { id, attrName[, params] } | Invoke a remote procedure on a gadget. **id** is the id of which gadget you like to perform its particular procedure. **attrName** is the attribute name of an executable procedure. **params** is an array of parameters given in order to meet the procedure signature. The signature depends on how a developer declare his(/her) own procedure. For example, given `{ id: 9, attrName: 'blink', value: [ 10, 500 ] }` to blink a LED on a gadget 10 times with 500ms interval.                                                                                                             |
| gad       | 'setReportCfg' | { id, attrName, [rptCfg](#reportCfg) }    | Set the condition for an attribute reporting from a gadget.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| gad       | 'getReportCfg' | { id, attrName }         | Get the report settings of an attribute on a gadget.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
  
********************************************
  
<a name="ResponseData"></a>
### Response  
  
The response message is an object with keys { __intf, subsys, seq, id, cmd, status, data }. `status` shows if the request is successful. The `data` field contains the respond data according to the request. `data` will be null(or undefined) if the request is unsuccessful.  
  
| Subsystem | Command Name   | Response Data Type | Data Description                                     | Example                             |
|-----------|----------------|--------------------|------------------------------------------------------|-------------------------------------|
| net       | 'getAllDevIds' | Number[]           | Array of device identifiers                          | [ 1, 2, 3, 8, 12 ]                  |
| net       | 'getAllGadId's | Number[]           | Array of gadget identifiers                          | [ 2, 3, 5, 11, 12, 13, 14, 15 ]     |
| net       | 'getDevs'      | Object[]           | Array of device information objects                  | [ [devInfo](#devInfoObj), ...  ]    |
| net       | 'getGads'      | Object[]           | Array of gadget information objects                  | [ [gadInfo](#gadInfoObj) , ... ]    |
| net       | 'getNetcores'  | Object[]           | Array of netcore information objects                 | [ [ncInfo](#ncInfoObj), ... ]       |
| net       | 'getBlacklist' | Object[]           | Array of banned device permanent address             | [ '0x00124b0001ce4b89', ... ]       |
| net       | 'permitJoin'   | -                  | Response contains no data                            | null                                |
| net       | 'maintain'     | -                  | Response contains no data                            | null                                |
| net       | 'reset'        | -                  | Response contains no data                            | null                                |
| net       | 'enable'       | -                  | Response contains no data                            | null                                |
| net       | 'disable'      | -                  | Response contains no data                            | null                                |
| dev       | 'read'         | Depends            | The read value. Can be anything                      | 3                                   |
| dev       | 'write'        | Depends            | The written value. Can be anything                   | 'kitchen'                           |
| dev       | 'remove'       | String             | Device permanent address                             | '0x00124b0001ce4b89'                |
| dev       | 'identify'     | -                  | Response contains no data                            | null                                |
| dev       | 'ping'         | Number             | Round-trip time in ms                                | 12                                  |
| dev       | 'ban'          | -                  | Response contains no data                            | null                                |
| dev       | 'unban'        | -                  | Response contains no data                            | null                                |
| gad       | 'read'         | Depends            | The read value. Can be anything                      | 371.42                              |
| gad       | 'write'        | Depends            | The written value. Can be anything                   | false                               |
| gad       | 'exec'         | Depends            | The data returned by the procedure. Can be anything  | 'completed'                         |
| gad       | 'setReportCfg' | Null               | Response contains no data                            | null                                |
| gad       | 'getReportCfg' | Object             | Report settings object                               | [rptCfg](#reportCfg)                |
  

********************************************

<br />
  
<a name="IndicationData"></a>
### Indication  
  
The indication message is an object with keys { __intf, subsys, type, id, data }. `type` shows the type of indication. The `data` field contains the indication data.  

| Subsystem | Indication Type | Data Type | Description                                                                         |
|-----------|-----------------|-----------|-------------------------------------------------------------------------------------|
| net       | 'permitJoining' | Object    | Server is now allowing devices to join the network                                  |
| dev       | 'netChanged'    | Object    | Network parameter(s) of a device has changed                                        |
| dev       | 'statusChanged' | String    | Status of a device has changed. The status can be 'online', 'sleep', and 'offline'  |
| dev       | 'devIncoming'   | Object    | A device is incoming. The indication data is a devInfo object                       |
| dev       | 'devLeaving'    | Number    | A device is leaving. The indication data is the device id                           |
| gad       | 'gadIncoming'   | Object    | A gadget is incoming. The indication data is a gadInfo object                       |
| gad       | 'gadLeaving'    | Number    | A gadget is leaving. The indication data is the gadget id                           |
| gad       | 'attrReport'    | Depends   | Report message of a certain attribute on a gadget                                   |
| dev/gad   | 'attrChanged'   | Object    | Attribute(s) on a gadget or a device has changed                                    |
  
* Indication Example: 'permitJoining'  
  
    ```js
    // indMsg.subsys == 'net'
    // indMsg.data is an object
    {
        netcore: 'zigbee-core',
        leftTime: 60            // netcore does not allow for devices to join the network if leftTime is 0
    }
    ```
  
* Indication Example: 'netChanged'  
  
    ```js
    // indMsg.subsys == 'dev'
    // indMsg.data is an object
    {
        address: {
            dynamic: '192.168.1.32'
        },
        status: 'online'    // status changed, an 'statusChanged' indication will also be fired  
    }
    ```
  
* Indication Example: 'statusChanged'  
  
    ```js
    // indMsg.subsys == 'dev'
    // indMsg.data is a string
    'online'
    ```
  
* Indication Example: 'devIncoming'  
  
    ```js
    // indMsg.subsys == 'dev'
    // indMsg.data is a devInfo object
    {
        id: 18,
        netcore: 'mqtt-core',
        role: 'client',         // depends on protocol. This rol string is defined by the netcore developer
        enable: true,
        status: 'online'
        address: {
            permanent: '00:0c:29:ff:ed:7c',
            dynamic: '192.168.1.24'
        },
        joinTime: 1458008311,
        traffic: {
            in: 12,
            out: 6
        },
        parent: 0,
        gads: [ 5, 6, 7, 8 ]

        manufacturer: 'freebird',
        model: 'lwmqn-7688-duo',
        serial: 'lwmqn-2016-03-15-20',
        version: {
            hardware: 'v1.2.0',
            software: 'v0.8.4',
            firmware: 'v2.0.0'
        },
        power: {
            type: 'line',
            voltage: '5V'
        },

        // name, description, and location are writable and can be modified by users
        name: 'sample_device',
        description: 'This is a device example',
        location: 'bedroom'
    }
    ```
  
* Indication Example: 'devLeaving'  
  
    ```js
    // indMsg.subsys == 'dev'
    // indMsg.data is a number
    27
    ```
  
* Indication Example: 'attrChanged'  
  
    ```js
    // indMsg.subsys == 'dev'
    // indMsg.data is an object to inform the device attribute changes
    {
        description: 'Move from office to kitchen',
        location: 'kitchen'
    }

    // indMsg.subsys == 'gad'
    // indMsg.data is an object to inform the gadget attribute changes
    {
        onOff: 0
    }
    ```
  
* Indication Example: 'gadIncoming'  
  
    ```js
    // indMsg.subsys == 'gad'
    // indMsg.data is a gadInfo object
    {
        id: 32,
        owner: 18,
        enable: true,
        profile: '',
        class: 'lightCtrl',
        attributes: {
            onOff: 1,
            dimmer: 80
        },
        // name and description writable and can be modified by users
        name: 'sampleLight',
        description: 'This is a simple light controller'
    }
    ```
  
* Indication Example: 'gadLeaving'  
  
    ```js
    // indMsg.subsys == 'gad'
    // indMsg.data is a number
    32
    ```
  
* Indication Example: 'attrReport'  
  
    ```js
    // indMsg.subsys == 'gad'
    // indMsg.data is an object to report the gadget attribute
    {
        sensorValue: 18
    }
    ```
  
********************************************

<br />
  
<a name="RspCodes"></a>
## 4. RSP Status Code  
  
[**CAUTION**] The response code is TBD and is subject to change.  
  
| Code Id | Code Name      | Description                                                                        |
|---------|----------------|------------------------------------------------------------------------------------|
| 0       | 'success'      | Response is ok                                                                     |
| 1       | 'fail'         | Operation fails                                                                    |
| 2       | 'busy'         | Server or remote device is busy. Try later                                         |
| 3       | 'unavail'      | Remote device is unreachable, it may be offline or sleeping                        |
| 4       | 'badRequest'   | Request parameter in arguments cannot be recognized or given with wrong type       |
| 5       | 'notFound'     | The allocated device, gadget is not found                                          |
| 6       | 'notAllowed'   | Method is not allowed. For example, write a value to a read-only attribute         |
| 7       | 'unauthorized' | The operation is unauthorized                                                      |
| 8       | 'timeout'      | Request timeout                                                                    |

********************************************

<br />
  
<a name="Appendix"></a>
## 5. Appendix  
  
<a name="devInfoObj"></a>
### Device Information (devInfo) Object
  
* Properties  

    | Property     | Type            | Description                                                                                           |
    |--------------|-----------------|-------------------------------------------------------------------------------------------------------|
    | id           | Number          | Device id                                                                                             |
    | netcore      | String          | Name of the netcore that holds this device                                                            |
    | role         | String          | Device role. Depends on protocol, i.e., 'peripheral' for BLE devices, 'router' for zigbee devices     |
    | enable       | Boolean         | Is this device enabled?                                                                               |
    | status       | String          | Device status, can be 'online', 'sleep', or 'offline'                                                 |
    | address      | Object          | Device permanent and dynamic addresses. { permanent: '00:0c:29:ff:ed:7c', dynamic: '192.168.1.101'    |
    | joinTime     | Number          | Device join time. UNIX time in secs                                                                   |
    | traffic      | Object          | Accumulated inbound and outbound data since device joined. { in: 70, out: 226 } (unit: kBytes)        |
    | parent       | Number          | Parent device id. This id is 0 if device parent is the netcore                                        |
    | gads         | Number[]        | A list of gadget ids that this device owns                                                            |
    | manufacturer | String          | Manufacturer name                                                                                     |
    | model        | String          | Model name                                                                                            |
    | serial       | String          | Serial number in string                                                                               |
    | version      | Object          | Version tags. { hardware: '', software: 'v1.2.2', firmware: 'v0.0.8' }                                |
    | power        | Object          | Power source. { type: 'battery', voltage: '5V' }. The type can be 'line', 'battery' or 'harvester'    |
    | name         | String          | Device name. This attribute can be modified by user (via UI tools)                                    |
    | description  | String          | Device description. This attribute can be modified by user (via UI tools)                             |
    | location     | String          | Device location. This attribute can be modified by user (via UI tools)                                |
  
* Example  
  
    ```js
    {
        id: 6,
        netcore: 'mqtt-core',
        role: 'client',         // depends on protocol
        enable: true,
        status: 'online'
        address: {
            permanent: '00:0c:29:ff:ed:7c',
            dynamic: '192.168.1.73'
        },
        joinTime: 1458008208,
        traffic: {              // accumulated data (kB) since device joined
            in: 86,
            out: 2114
        },
        parent: 0,              // parent is the netcore
        gads: [ 5, 6 ]          // gadgets it owns

        manufacturer: 'freebird',
        model: 'lwmqn-7688-duo',
        serial: 'lwmqn-2016-03-15-01',
        version: {
            hardware: 'v1.2.0',
            software: 'v0.8.4',
            firmware: 'v2.0.0'
        },
        power: {
            type: 'line',
            voltage: '5V'
        },

        // name, description, and location are writable and can be modified by users
        name: 'sample_device',
        description: 'This is a device example',
        location: 'balcony'
    }
    ```

<a name="gadInfoObj"></a>
### Gadget Information (gadInfo) Object
  
* Properties  

    | Property     | Type            | Description                                                                                               |
    |--------------|-----------------|-----------------------------------------------------------------------------------------------------------|
    | id           | Number          | Gadget id                                                                                                 |
    | owner        | Number          | Id of which device owns this gadget                                                                       |
    | enable       | Boolean         | Is this gadget enabled?                                                                                   |
    | profile      | String          | The profile of this gadget                                                                                |
    | class        | String          | The [gadget class](#gadClasses) to denote its application, i.e. 'illuminance', 'temperature', 'lightCtrl' |
    | attributes   | Object          | Attributes of this gadget                                                                                 |
    | name         | String          | Gadget name. This attribute can be modified by user (via UI tools)                                        |
    | description  | String          | Gadget description. This attribute can be modified by user (via UI tools)                                 |
  
* Example  
  
    ```js
    {
        id: 308,
        owner: 26,
        enable: true,
        profile: 'home_automation', // it will be an empty string '' if no profile given
        class: 'lightCtrl',
        attributes: {
            onOff: 1,
            dimmer: 80
        },
        // name and description writable and can be modified by users
        name: 'sampleLight',
        description: 'This is a simple light controller'
    }
    ```
  
<a name="ncInfoObj"></a>
### Netcore Information (ncInfo) Object
  
* Properties  
  
    | Property     | Type            | Description                                                                                      |
    |--------------|-----------------|--------------------------------------------------------------------------------------------------|
    | name         | String          | Netcore name                                                                                     |
    | enable       | Boolean         | Is this netcore enabled?                                                                         |
    | protocol     | Object          | [TBD] Network protocol of this netcore.                                                          |
    | numDevs      | Number          | Number of devices managed by this netcore                                                        |
    | numGads      | Number          | Number of gadgets managed by this netcore                                                        |
    | startTime    | String          | Start time of this netcore. (UNIX time in secs)                                                  |
    | traffic      | Object          | Accumulated inbound and outbound data since netcore started. { in: 70, out: 226 } (unit: kBytes) |
  
* Example  
  
    ```js
    {
        name: 'zigbee-core',
        enable: true,
        protocol: {
            application: 'zcl',
            transport: '',
            network: 'zigbee2007pro',
            link: '',
            phy: 'ieee802.15.4'
        },
        numDevs: 32,
        numGads: 46,
        startTime: 1458008208,
        traffic: {
            in: 44765,
            out: 84114
        }
    }
    ```

  
<a name="gadClasses"></a>
### Gadget Classes

Freebird framework uses the class property on a gadget to define its application. The classes are Smart Object Identfiers defined by [IPSO SmartObject Guideline(Smart Objects Starter Pack1.0)](http://www.ipso-alliance.org/smart-object-guidelines/). Here is the table of Object ids from [lwm2m-id](https://github.com/simenkid/lwm2m-id#5-table-of-identifiers) library.

| Class Name                 | Description            |
|----------------------------|------------------------|
| 'dIn'                      | Digital Input          |
| 'dOut'                     | Digital Output         |
| 'aIn'                      | Analogue Input         |
| 'aOut'                     | Analogue Output        |
| 'generic'                  | Generic Sensor         |
| 'illuminance'              | Illuminance Sensor     |
| 'presence'                 | Presence Sensor        |
| 'temperature'              | Temperature Sensor     |
| 'humidity'                 | Humidity Sensor        |
| 'pwrMea'                   | Power Measurement      |
| 'actuation'                | Actuation              |
| 'setPoint'                 | Set Point              |
| 'loadCtrl'                 | Load Control           |
| 'lightCtrl'                | Light Control          |
| 'pwrCtrl'                  | Power Control          |
| 'accelerometer'            | Accelerometer          |
| 'magnetometer'             | Magnetometer           |
| 'barometer'                | Barometer              |

<a name="reportCfg"></a>
### Attribute Report Configuration (rptCfg) Object
  
* Properties  
  
    | Property     | Type         | Mandatory | Description                                                                                      |
    |--------------|--------------|-----------|--------------------------------------------------------------------------------------------------|
    | pmin         | Number       |  optional | Minimum Period. Minimum time in seconds the gadget should wait from the time when sending the last notification to the time when sending a new notification.                                                                                     |
    | pmax         | Number       |  optional | Maximum Period. Maximum time in seconds the gadget should wait from the time when sending the last notification to the time sending the next notification (regardless if the value has changed).                                                                         |
    | gt           | Number       |  optional | Greater Than. The gadget should notify its attribute when the value is greater than this setting. Only valid for the attribute typed as a number.                                                          |
    | lt           | Number       |  optional | Less Than. The gadget should notify its attribute when the value is smaller than this setting. Only valid for the attribute typed as a number.                                                        |
    | step         | Number       |  optional | Step. The gadget should notify its value when the change of the attribute value, since the last report happened, is greater than this setting.                                                        |
    | enable       | Boolean      | required  | It is set to true for the gadget to start reporting an attribute. Set to false to stop reporting. |
  
* Example  
  
    ```js
    // start reporting with the following settings
    {
        pmin: 10,
        pmax: 60,
        lt: 120
        gt: 260
        enable: true
    }

    // the time chart of reporting: (O: report triggered, xxxxx: pmin, -----: pmax)
    //       O             O     O             O     O             O     O             O
    // |xxxxx|-------------|xxxxx|-------------|xxxxx|-------------|xxxxx|-------------|
    // 0s   10s           70s   80s           140s 150s           210s  220s          280s
    //  <10s> <    60s    >

    // note:
    //     1. only in |---------| duration, any change meets lt or gt condition will be reported
    //     2. In this example, the attribute will be reported when its value is greater than 260 or is less than 120
    ```

  
    ```js
    // start periodical reporting
    {
        pmin: 0,
        pmax: 30,
        enable: true
    }

    // the time chart of reporting: (O: report triggered, xxxxx: pmin, -----: pmax)
    // O             O             O             O             O             0
    // |-------------|-------------|-------------|-------------|-------------|
    // 0s           30s           60s           90s          120s          150s

    // note:
    //     1. In this example, the attribute will be reported every 30 seconds
    ```
  
    ```js
    // set but not start reporting
    {
        gt: 50
        enable: false
    }
    ```
  
    ```js
    // start reporting with the current settings
    {
        enable: true
    }
    ```
  
    ```js
    // stop reporting
    {
        enable: false
    }
    ```
  