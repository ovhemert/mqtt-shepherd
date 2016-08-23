# mqtt-shepherd
Network server and manager for the lightweight MQTT machine network (LWMQN)  
  
[![NPM](https://nodei.co/npm/mqtt-shepherd.png?downloads=true)](https://nodei.co/npm/mqtt-shepherd/)  

[![Travis branch](https://img.shields.io/travis/lwmqn/mqtt-shepherd/master.svg?maxAge=2592000)](https://travis-ci.org/lwmqn/mqtt-shepherd)
[![npm](https://img.shields.io/npm/v/mqtt-shepherd.svg?maxAge=2592000)](https://www.npmjs.com/package/mqtt-shepherd)
[![npm](https://img.shields.io/npm/l/mqtt-shepherd.svg?maxAge=2592000)](https://www.npmjs.com/package/mqtt-shepherd)

<br />

## Table of Contents

1. [Overiew](#Overiew)  
2. [Features](#Features)  
3. [Installation](#Installation)  
4. [Basic Usage](#Basic)  
5. [APIs and Events](#APIs)  
6. [Message Encryption](#Encryption)  
7. [Auth Policies](#Auth)  
8. [Status Code](#StatusCode)  

<a name="Overiew"></a>
## 1. Overview

Lightweight MQTT machine network ([**LWMQN**](http://lwmqn.github.io)) is an architecture that follows part of [**OMA LWM2M v1.0**](http://technical.openmobilealliance.org/Technical/technical-information/release-program/current-releases/oma-lightweightm2m-v1-0) specification to meet the minimum requirements of machine network management. LWMQN is also an open source project that offers a solution of establishing a local area machine network with MQTT, and it can be a replacement of the cloud-based solution if you don't really need the cloud (which becomes an option). Not only has LWM2M-like interfaces, LWMQN also utilizes the [IPSO Smart Object](http://www.ipso-alliance.org/) as its fundamental of resource organization, this leads to a comprehensive and consistent way in describing real-world gadgets.  
* LWMQN project provides you with a server-side **mqtt-shepherd** library and a client-side [**mqtt-node**](https://github.com/lwmqn/mqtt-node) library to run your machine network with JavaScript and node.js. With these two libraries and node.js, you can have your own authentication, authorization and encryption subsystems to secure your network easily. LWMQN project is trying to let you build an IoT machine network with less pain.  

![LWMQN Network](https://github.com/lwmqn/documents/blob/master/media/lwmqn_net.png)

* This module, **mqtt-shepherd**, is an implementation of LWMQN Server that can run on platfroms equipped with node.js.  
* LWMQN Client and Server benefits from [IPSO data model](http://www.ipso-alliance.org/ipso-community/resources/smart-objects-interoperability/), which leads to a very comprehensive way for the Server to use an URI-style *path* to allocate and query Resources on Client Devices.  
* In the following example, both of these two requests is to read the sensed value from a temperature sensor on a Client Device.  
  
    ```js
    qnode.readReq('temperature/0/sensorValue', function (err, rsp) {
        console.log(rsp); // { status: 205, data: 18 }
    });

    qnode.readReq('3304/0/5700', function (err, rsp) {
        console.log(rsp); // { status: 205, data: 18 }
    });
    ```
  
* The goal of **mqtt-shepherd** is to let you build and manage an MQTT machine network with less efforts, it is implemented as a server-side application framework with functionality of network and devices management, e.g. permission of device joining, device authentication, reading resources, writing resources, observing resources, and executing a procedure on the remote Devices. Furthermore, thanks to the power of node.js, making your own RESTful APIs to interact with your machines is also possible.  
  

**Note**:  
* IPSO uses **_Object_**, **_Object Instance_** and **_Resource_** to describe the hierarchical structure of resources on a Client Device, where `oid`, `iid`, and `rid` are identifiers of them respectively used to allocate resources on a Client Device.  
* An IPSO **_Object_** is like a Class, and an **_Object Instance_** is an entity of such Class. For example, when you have many 'temperature' sensors, you have to use an unique `iid` on each Object Instance to distinguish one entity from the other.  

<br />

#### Acronyms and Abbreviations
* **Server**: LWMQN Server
* **Client** or **Client Device**: LWMQN Client 
* **MqttShepherd**: Class exposed by `require('mqtt-shepherd')`  
* **MqttNode**: Class to create a software endpoint of a remote Client Device on the Server
* **qserver**: Instance of MqttShepherd Class 
* **qnode**: Instance of MqttNode Class  

<br />

<a name="Features"></a>
## 2. Features

* Communication based on MQTT protocol  
* Based on [Mosca](https://github.com/mcollina/mosca/wiki), an MQTT broker on node.js.  
* Hierarchical data model in Smart-Object-style (IPSO)  
* Easy to query resources on a Client Device  
* LWM2M-like interfaces for Client/Server interaction  
* Embedded persistence ([NeDB](https://github.com/louischatriot/nedb)) and auto-reload at boot-up for Client Devices  
* Simple machine network managment  
  
<a name="Installation"></a>
## 3. Installation

> $ npm install mqtt-shepherd --save
  
<a name="Basic"></a>
## 4. Basic Usage

```js
var MqttShepherd = require('mqtt-shepherd');
var qserver = new MqttShepherd();   // create a LWMQN server

qserver.on('ready', function () {
    console.log('Server is ready.');

    // when server is ready, allow devices to join the network within 180 secs
    qserver.permitJoin(180);
});

qserver.start(function (err) {      // start the sever
    if (err)
        console.log(err);
});

// That's all to start a LWMQN Server.
// Now the Server is going to auotmatically tackle most of the network managing things.
```
  
<a name="APIs"></a>
## 5. APIs and Events  
  
This moudle provides you with **MqttShepherd** and **MqttNode** classes.  

* **MqttShepherd** class brings you a LWMQN Server with network managing facilities, i.e., start/stop the Server, permit device joining, find a joined node. This document uses `qserver` to denote the instance of this class.  

* **MqttNode** is the class for creating a software endpoint at server-side to represent the remote Client Device. This document uses `qnode` to denote the instance of this class. You can invoke methods on a `qnode` to operate the remote Client.  

* MqttShepherd APIs  
    * [new MqttShepherd()](#API_MqttShepherd)  
    * [qserver.start()](#API_start)  
    * [qserver.stop()](#API_stop)  
    * [qserver.reset()](#API_reset)  
    * [qserver.permitJoin()](#API_permitJoin)  
    * [qserver.info()](#API_info)  
    * [qserver.list()](#API_list)  
    * [qserver.find()](#API_find)  
    * [qserver.findByMac()](#API_findByMac)  
    * [qserver.remove()](#API_remove)  
    * [qserver.announce()](#API_announce)  
    * Events: [ready](#EVT_ready), [error](#EVT_error), [permitJoining](#EVT_permit), [ind](#EVT_ind), and [message](#EVT_message)  

* MqttNode APIs  
    * [qnode.readReq()](#API_readReq)  
    * [qnode.writeReq()](#API_writeReq)  
    * [qnode.executeReq()](#API_executeReq)  
    * [qnode.writeAttrsReq()](#API_writeAttrsReq)  
    * [qnode.discoverReq()](#API_discoverReq)  
    * [qnode.observeReq()](#API_observeReq)  
    * [qnode.pingReq()](#API_pingReq)  
    * [qnode.maintain()](#API_maintain)  
    * [qnode.dump()](#API_dump)  
    

<br />

## MqttShepherd Class
Exposed by `require('mqtt-shepherd')`  
  
***********************************************

<a name="API_MqttShepherd"></a>
### new MqttShepherd([name][, settings])
Create an instance of the `MqttShepherd` class. The created instance is a LWMQN Server.  
  
**Arguments:**  

1. `name` (_String_): Server name. A default name `'mqtt-shepherd'` will be used if not given.  
2. `settings` (_Object_): Settings for the Mosca MQTT broker. If not given, the default settings will be applied, i.e. port 1883 for the broker, LevelUp for presistence. You can set up your own backend, like mongoDB, Redis, Mosquitto, or RabbitMQ, through this option. Please refer to the [Mosca wiki page](https://github.com/mcollina/mosca/wiki/Mosca-advanced-usage) for details.  
    
**Returns:**  
  
* (_Object_): qserver

**Examples:**  

* Create a server and name it

```js
var MqttShepherd = require('mqtt-shepherd');
var qserver = new MqttShepherd('my_iot_server');
```

* Create a server that starts on a specified port

```js
var qserver = new MqttShepherd('my_iot_server', {
    port: 9000
});
```

* Create a server with other backend ([example from Mosca wiki](https://github.com/mcollina/mosca/wiki/Mosca-advanced-usage#--mongodb))

```js
var qserver = new MqttShepherd('my_iot_server', {
    port: 1883,
    backend: {  // This is the pubsubsettings you will see in Mosca wiki  
        type: 'mongo',        
        url: 'mongodb://localhost:27017/mqtt',
        pubsubCollection: 'ascoltatori',
        mongo: {}
    }
});
```

*************************************************

<a name="API_start"></a>
### .start([callback])
Start the qserver.  

**Arguments:**  

1. `callback` (_Function_): `function (err) { }`. Get called after the initializing procedure is done.  

  
**Returns:**  
  
* _none_

**Examples:**  
    
```js
qserver.start(function (err) {
    if (!err)
        console.log('server initialized.');
});
```

*************************************************

<a name="API_stop"></a>
### .stop([callback])
Stop the qserver.  

**Arguments:**  

1. `callback` (_Function_): `function (err) { }`. Get called after the server closed.  

  
**Returns:**  
  
* _none_

**Examples:**  
    
```js
qserver.stop(function (err) {
    if (!err)
        console.log('server stopped.');
});
```

*************************************************

<a name="API_reset"></a>
### .reset([mode,] [callback])
Reset the server. After the server restarted, a `'ready'` event will be fired. A hard reset (`mode == true`) will clear all joined qnodes and the database.  

**Arguments:**  

1. `mode` (_Boolean_): `true` for a hard reset, and `false` for a soft reset. Default is `false`.  
1. `callback` (_Function_): `function (err) { }`. Get called after the server restarted.  
  
**Returns:**  
  
* _none_

**Examples:**  
    
```js
qserver.on('ready', function () {
    console.log('server is ready');
});

// default is soft reset
qserver.reset(function (err) {
    if (!err)
        console.log('server restarted.');
});

// hard reset
qserver.reset(true, function (err) {
    if (!err)
        console.log('server restarted.');
});
```

*************************************************

<a name="API_permitJoin"></a>
### .permitJoin(time)
Allow or disallow devices to join the network.  

**Arguments:**  

1. `time` (_Number_): Time in seconds for qsever allowing devices to join the network. Set `time` to `0` can immediately close the admission.  

**Returns:**  
  
* _none_

**Examples:**  
    
```js
qserver.on('permitJoining', function (joinTimeLeft) {
    console.log(joinTimeLeft);
});

// allow devices to join for 180 seconds, this will also trigger 
// a 'permitJoining' event at each tick of countdown.
qserver.permitJoin(180);
```

*************************************************

<a name="API_info"></a>
### .info()
Returns the qserver information.

**Arguments:**  

1. none  
  
**Returns:**  
  
* (_Object_): An object that contains information about the Server. Properties in this object are given in the following table.  

    | Property       | Type    | Description                                                    |
    |----------------|---------|----------------------------------------------------------------|
    | name           | String  | Server name                                                    |
    | enabled        | Boolean | Server is up(`true`) or down(`false`)                          |
    | net            | Object  | Network information, `{ intf, ip, mac, routerIp }`             |
    | devNum         | Number  | Number of devices managed by this qserver                      |
    | startTime      | Number  | Unix Time (secs)                                               |
    | joinTimeLeft   | Number  | How many seconds left for allowing devices to join the Network |

**Examples:**  
    
```js
qserver.info();

/*
{
    name: 'my_iot_server',
    enabled: true,
    net: {
        intf: 'eth0',
        ip: '192.168.1.99',
        mac: '00:0c:29:6b:fe:e7',
        routerIp: '192.168.1.1'
    },
    devNum: 36,
    startTime: 1454419506,
    joinTimeLeft: 28
}
*/
```

*************************************************
<a name="API_list"></a>
### .list([clientIds])
List records of the registered Client Devices.  

**Arguments:**  

1. `clientIds` (_String_ | _String[]_): A single client id or an array of client ids to query for their records. All device records will be returned if `clientIds` is not given.  
  
**Returns:**  
  
* (_Array_): Information of Client Devices. Each record in the array is an object with the properties shown in the following table. An element in the array will be `undefined` if the corresponding Client Device is not found.  

    | Property     | Type    | Description                                                                                                                  |
    |--------------|---------|------------------------------------------------------------------------------------------------------------------------------|
    | clientId     | String  | Client id of the device                                                                                                      |
    | joinTime     | Number  | Unix Time (secs). When a device joined the network.                                                                          |
    | lifetime     | Number  | Lifetime of the device. If there is no message coming from the device within lifetime, qserve will deregister this device    |
    | ip           | String  | Ip address of the server                                                                                                     |
    | mac          | String  | Mac address                                                                                                                  |
    | version      | String  | LWMQN version                                                                                                                |
    | objList      | Object  | IPSO Objects and Object Instances. Each key in `objList` is the `oid` and each value is an array of `iid` under that `oid`.  |
    | status       | String  | `online` or `offline`                                                                                                        |

**Examples:**  
    
```js
console.log(qserver.list([ 'foo_id', 'bar_id', 'no_such_id' ]));

/*
[
    {
        clientId: 'foo_id',          // record for 'foo_id'
        joinTime: 1454419506,
        lifetime: 12345,
        ip: '192.168.1.112',
        mac: 'd8:fe:e3:e5:9f:3b',
        version: '',
        objList: {
            3: [ 1, 2, 3 ],
            2205: [ 7, 5503 ]
        },
        status: 'online'
    },
    {
        clientId: 'bar_id',          // record for 'bar_id'
        joinTime: 1454419706,
        lifetime: 12345,
        ip: '192.168.1.113',
        mac: '9c:d6:43:01:7e:c7',
        version: '',
        objList: {
            3: [ 1, 2, 3 ],
            2205: [ 7, 5503 ]
        },
        status: 'sleep',
    },
    undefined                        // record not found for 'no_such_id'
]
*/

console.log(qserver.list('foo_id'));

/* An array will be returned even a single string is argumented.
[
    {
        clientId: 'foo_id',          // record for 'foo_id'
        joinTime: 1454419506,
        lifetime: 12345,
        ip: '192.168.1.112',
        mac: 'd8:fe:e3:e5:9f:3b',
        version: '',
        objList: {
            3: [ 1, 2, 3 ],
            2205: [ 7, 5503 ]
        },
        status: 'online'
    }
]
*/
```

*************************************************
<a name="API_find"></a>
### .find(clientId)
Find a registered Client (qnode) on qserver by clientId.  

**Arguments:**  

1. `clientId` (_String_): Client id of the device to find for.  

  
**Returns:**  
  
* (_Object_): qnode. Returns `undefined` if not found.  

**Examples:**  
    
```js
var qnode = qserver.find('foo_id');

if (qnode) {
    // do something upon the qnode, like qnode.readReq()
}
```

*************************************************
<a name="API_findByMac"></a>
### .findByMac(macAddr)
Find registered Clients (qnodes) by the specified mac address. This method will always returns you an array, because there may be many Clients living in the same machine to share the same mac address.  

**Arguments:**  

1. `macAddr` (_String_): Mac address of the device(s) to find for. The address in case-insensitive.  

  
**Returns:**  
  
* (_qnode[]_): Array of found qnodes. Returns an empty array if not found.  

**Examples:**  
    
```js
var qnodes = qserver.findByMac('9e:65:f9:0b:24:b8');

if (qnodes.length) {
    // do something upon the qnodes
}
```

*************************************************

<a name="API_remove"></a>
### .remove(clientId[, callback])
Deregister and remove a qnode from the network.

**Arguments:**  

1. `clientId` (_String_): Client id of the qnode to be removed.  
2. `callback` (_Function_): `function (err, clientId) { ... }` will be called after removal. `clientId` is client id of the removed qnode.  
  
**Returns:**  
  
* _none_

**Examples:**  
    
```js
qserver.remove('foo', function (err, clientId) {
    if (!err)
        console.log(clientId);
});
```

*************************************************

<a name="API_announce"></a>
### .announce(msg[, callback])
The Server can use this method to announce(/broadcast) any message to all Clients.  

**Arguments:**  

1. `msg` (_String_ | _Buffer_): The message to announce. Remember to stringify if the message is a data object.  
2. `callback` (_Function_): `function (err) { ... }`. Get called after message announced.  
  
**Returns:**  
  
* _none_

**Examples:**  
    
```js
qserver.announce('Rock on!');
```

*************************************************

<a name="EVT_ready"></a>
### Event: 'ready'  
Listener: `function () { }`  
Fired when Server is ready.  

*************************************************

<a name="EVT_error"></a>
### Event: 'error'  
Listener: `function (err) { }`  
Fired when there is an error occurs.  

*************************************************

<a name="EVT_permit"></a>
### Event: 'permitJoining'
Listener: `function (joinTimeLeft) {}`  
Fired when the Server is allowing for devices to join the network, where `joinTimeLeft` is number of seconds left to allow devices to join the network. This event will be triggered at each tick of countdown.  

*************************************************

<a name="EVT_ind"></a>
### Event: 'ind'
Listener: `function (msg) { }`  
Fired when there is an incoming indication message. The `msg` is an object with the properties given in the table:  

| Property       | Type             | Description                                                                                                                     |
|----------------|------------------|---------------------------------------------------------------------------------------------------------------------------------|
| type           | String           | Indication type, can be `'devIncoming'`, `'devLeaving'`, `'devUpdate'`, `'devNotify'`, `'devChange'`, and `'devStatus'`.        |
| qnode          | Object \| String | qnode instance, except that when `type === 'devLeaving'`, qnode will be a string of the clientId (since qnode has been removed) |
| data           | Depends          | Data along with the indication, which depends on the type of indication                                                         |


* ##### devIncoming  
    When there is a Client Device incoming to the network, qserver will fire an `'ind'` event along with this type of indication. The Client Device can be either a new registered one or an old one that logs in again.  

    * msg.type: `'devIncoming'`  
    * msg.qnode: qnode  
    * msg.data: `undefined`  

* ##### devLeaving  
    When there is a Client Device leaving the network, qserver will fire an `'ind'` event along with this type of indication.  

    * msg.type: `'devLeaving'`  
    * msg.qnode: `'foo_clientId'`, the clientId of which qnode is leaving  
    * msg.data: `9e:65:f9:0b:24:b8`, the mac address of which qnode is leaving.  

* ##### devUpdate  
    When there is a Client Device that publishes an update of its device attribute(s), qserver will fire an `'ind'` event along this type of indication.  

    * msg.type: `'devUpdate'`  
    * msg.qnode: qnode  
    * msg.data: `{ ip: '192.168.0.36' }`, an object that contains the updated attribute(s). There may be fields of `status`, `lifetime`, `ip`, and `version` in this object.  

* ##### devNotify  
    When there is a Client Device that publishes a notification of its Object Instance or Resource, qserver will fire an `'ind'` event along this type of indication.  

    * msg.type: `'devNotify'`
    * msg.qnode: qnode
    * msg.data: Content of the notification. This object has fileds of `oid`, `iid`, `rid`, and `data`.
        - `data` is an Object Instance if `oid` and `iid` are given but `rid` is null or undefined
        - `data` is a Resource if `oid`, `iid` and `rid` are given (data type depends on the Resource)
  
        ```js
        // example of an Object Instance notification
        {
            oid: 'humidity',
            iid: 0,
            data: {             // Object Instance
                sensorValue: 32
            }
        }

        // example of a Resource notification
        {
            oid: 'humidity',
            iid: 0,
            rid: 'sensorValue',
            data: 32            // Resource value
        }
        ```

* ##### devChange  
    When the Server perceives that there is any change of _Resources_ from notifications or read/write responses, qserver will fire an `'ind'` event along this type of indication.  

    * msg.type: `'devChange'`  
    * msg.qnode: qnode  
    * msg.data: Content of the changes. This object has fileds of `oid`, `iid`, `rid`, and `data`.  
        - `data` is an object that contains only the properties changed in an Object Instance. In this case, `oid` and `iid` are given but `rid` is null or undefined  
        - `data` is the new value of a Resource. If a Resource itself is an object, then `data` will be an object that contains only the properties changed in that Resource. In this case, `oid`, `iid` and `rid` are given (data type depends on the Resource)
        ```js
        // changes of an Object Instance
        {
            oid: 'temperature',
            iid: 0,
            data: {
                sensorValue: 12,
                minMeaValue: 12
            }
        }

        // change of a Resource 
        {
            oid: 'temperature',
            iid: 1,
            rid: 'sensorValue',
            data: 18
        }
        ```
    * Note
        - The diffrence between `'devChange'` and `'devNotify'` is that data along with `'devNotify'` is which a Client Device like to notify even if there is no change of it. A periodical notification is a good example, a Client Device has to report something under observation even there is no change of that thing.
        - If the Server does notice there is really something changed, it will then fire `'devChange'` to report the change(s). It is suggested to use `'devChange'` indication to update your GUI views, and to use `'devNotify'` indication to log data.

* ##### devStatus  
    When there is a Client Device going online, going offline, or going to sleep, qserver will fire an `'ind'` event along this type of indication.  

    * msg.type: `'devStatus'`  
    * msg.qnode: qnode  
    * msg.data: `'online'`, `'sleep'`, or `'offline'`  


*************************************************

<a name="EVT_message"></a>
### Event: 'message'
Listener: `function(topic, message, packet) {}`  
Emitted when the Server receives any published packet from any remote Device.  

1. `topic` (_String_): topic of the received packet  
2. `message` (_Buffer_): payload of the received packet  
3. `packet` (_Object_): the received packet, as defined in [mqtt-packet](https://github.com/mqttjs/mqtt-packet#publish)  


***********************************************

<br />

## MqttNode Class
This class provides you with methods to perform remote operations upon a registered Client Device. An instance of this class is denoted as `qnode` in this document.  

***********************************************
<a name="API_readReq"></a>
### qnode.readReq(path, callback)
Remotely read a target from the Client Device. Response will be passed through the second argument of the callback.  

**Arguments:**  

1. `path` (_String_): Path of the allocated _Object_, _Object Instance_, or _Resource_ on the remote Client Device.  
2. `callback` (_Function_): `function (err, rsp) { }`
    - `err` (_Object_): Error object
    - `rsp` (_Object_): The response is an object that has a status code along with the returned data from the remote Client Device.  

    | Property | Type    | Description                                                                                                                                                                   |
    |----------|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    |  status  | Number  | [Status code](#StatusCode) of the response. Possible status codes are 205, 400, 404, and 405.                                                                                 |
    |  data    | Depends | `data` can be the value of an Object, an Object Instance, or a Resource. Note that when an unreadable Resource is read, the returned value will be a string `'_unreadable_'`. |
  

**Returns:**  
  
* _none_

**Examples:**  
    
```js
qnode.readReq('temperature/1/sensedValue', function (err, rsp) {
    console.log(rsp);       // { status: 205, data: 87 }
});

// Target not found
qnode.readReq('/noSuchObject/0/foo', function (err, rsp) {
    console.log(rsp);       // { status: 404, data: undefined }
});

// Target not found
qnode.readReq('/temperature/0/noSuchResource/', function (err, rsp) {
    console.log(rsp);       // { status: 404, data: undefined }
});

// Target is unreadable
qnode.readReq('/temperature/0/foo', function (err, rsp) {
    console.log(rsp);       // { status: 405, data: '_unreadable_' }
});
```

***********************************************
<a name="API_writeReq"></a>
### qnode.writeReq(path, val[, callback])
Remotely write a value to the allocated _Resource_ on a Client Device. The response will be passed through the second argument of the callback.  

**Arguments:**  

1. `path` (_String_): Path of the allocated _Resource_ on the remote Client Device.
2. `val` (_Depends_): The value to write to the _Resource_.
3. `callback` (_Function_): `function (err, rsp) { }`. The `rsp` object that has a status code along with the written data from the remote Client Device.

    | Property | Type    | Description                                                                                                         |
    |----------|---------|---------------------------------------------------------------------------------------------------------------------|
    |  status  | Number  | [Status code](#StatusCode) of the response. Possible status codes are 204, 400, 404, and 405.                       |
    |  data    | Depends | `data` is the written value. It will be a string `'_unwritable_'` if the Resource is not allowed for writing.       |

**Returns:**  
  
* _none_

**Examples:**  
    
```js
// write successfully
qnode.writeReq('digitalOutput/0/appType', 'lightning', function (err, rsp) {
    console.log(rsp);   // { status: 204, data: 'lightning' }
});

qnode.writeReq('digitalOutput/0/dOutState', 0, function (err, rsp) {
    console.log(rsp);   // { status: 204, data: 0 }
});

// target not found
qnode.writeReq('temperature/0/noSuchResource', 1, function (err, rsp) {
    console.log(rsp);   // { status: 404, data: undefined }
});

// target is unwritable
qnode.writeReq('digitalInput/1/dInState', 1, function (err, rsp) {
    console.log(rsp);   // { status: 405, data: '_unwritable_' }
});
```

***********************************************
<a name="API_executeReq"></a>
### qnode.executeReq(path[, args][, callback])
Invoke an excutable Resource on the Client Device. An excutable Resource is like a remote procedure call.  

**Arguments:**  

1. `path` (_String_): Path of the allocated Resource on the remote Client Device.  
2. `args` (_Array_): The arguments to the procedure.  
3. `callback` (_Function_): `function (err, rsp) { }`. The `rsp` object has a status code to indicate whether the operation succeeds. There will be a `data` field if the procedure does return something back, and the data type depends on the implementation at client-side.  

    | Property | Type    | Description                                                                                                              |
    |----------|---------|--------------------------------------------------------------------------------------------------------------------------|
    |  status  | Number  | [Status code](#StatusCode) of the response. Possible status codes are 204, 400, 404, 405, and 500.                       |
    |  data    | Depends | What will be returned depends on the client-side implementation.                                                         |

  
**Returns:**  
  
* (_none_)

**Examples:**  
    
```js
// assuming there is an executable Resource (procedure) with singnatue
// function(n) { ... } to blink an LED n times.
qnode.executeReq('led/0/blink', [ 10 ] ,function (err, rsp) {
    console.log(rsp);       // { status: 204 }
});

// assuming there is an executable Resource with singnatue
// function(edge, duration) { ... } to count how many times the button 
// was pressed within `duration` seconds.
qnode.executeReq('button/0/blink', [ 'falling', 20 ] ,function (err, rsp) {
    console.log(rsp);       // { status: 204, data: 71 }
});

// Something went wrong at Client-side
qnode.executeReq('button/0/blink', [ 'falling', 20 ] ,function (err, rsp) {
    console.log(rsp);       // { status: 500 }
});

// arguments cannot be recognized, in this example, 'up' is an invalid parameter
qnode.executeReq('button/0/blink', [ 'up', 20 ] ,function (err, rsp) {
    console.log(rsp);       // { status: 400 }
});

// Resource not found
qnode.executeReq('temperature/0/noSuchResource', function (err, rsp) {
    console.log(rsp);       // { status: 404 }
});

// invoke an unexecutable Resource
qnode.executeReq('temperature/0/sensedValue', function (err, rsp) {
    console.log(rsp);       // { status: 405 }
});
```

***********************************************
<a name="API_writeAttrsReq"></a>
### qnode.writeAttrsReq(path, attrs[, callback])
Configure the report settings of a Resource, an Object Instance, or an Object. This method can also be used to cancel an observation by assigning the `attrs.cancel` to `true`.  
    
**Note**
* This API **won't start reporting** of the notifications, call observe() method if you want to turn the report on.  

**Arguments:**  

1. `path` (_String_): Path of the allocated Resource, Object Instance, or Object on the remote Client Device.  
2. `attrs` (_Object_): Parameters of the report settings.  

    | Property | Type    | Mandatory | Description                                                                                                                                                                                             |
    |----------|---------|-----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | pmin     | Number  | optional  | Minimum Period. Minimum time in seconds the Client Device should wait from the time when sending the last notification to the time when sending a new notification.                                     |
    | pmax     | Number  | optional  | Maximum Period. Maximum time in seconds the Client Device should wait from the time when sending the last notification to the time sending the next notification (regardless if the value has changed). |
    | gt       | Number  | optional  | Greater Than. The Client Device should notify its value when the value is greater than this setting. Only valid for the Resource typed as a number.                                                     |
    | lt       | Number  | optional  | Less Than. The Client Device should notify its value when the value is smaller than this setting. Only valid for the Resource typed as a number.                                                        |
    | stp      | Number  | optional  | Step. The Client Device should notify its value when the change of the Resource value, since the last report happened, is greater than this setting.                                                    |
    | cancel   | Boolean | optional  | Set to `true` for a Client Device to cancel observation on the allocated Resource or Object Instance.                                                                                                   |

3. `callback` (_Function_):  `function (err, rsp) { }`. The `rsp` object has a status code to indicate whether the operation is successful.  

    | Property | Type    | Description                                                                                                    |
    |----------|---------|----------------------------------------------------------------------------------------------------------------|
    |  status  | Number  | [Status code](#StatusCode) of the response. Possible status codes are 204, 400, and 404.                       |

  
**Returns:**  
  
* (_none_)

**Examples:**  
    
```js
// set successfully
qnode.writeAttrsReq('temperature/0/sensedValue', {
    pmin: 10,
    pmax: 600,
    gt: 45
}, function (err, rsp) {
    console.log(rsp);       // { status: 200 }
});

// cancel the observation on a Resource
qnode.writeAttrsReq('temperature/0/sensedValue', {
    cancel: true
}, function (err, rsp) {
    console.log(rsp);       // { status: 200 }
});

// taget not found
qnode.writeAttrsReq('temperature/0/noSuchResource', {
    gt: 20
}, function (err, rsp) {
    console.log(rsp);       // { status: 404 }
});

// parameter cannot be recognized
qnode.writeAttrsReq('temperature/0/noSuchResource', {
    foo: 60
}, function (err, rsp) {
    console.log(rsp);       // { status: 400 }
});
```

***********************************************
<a name="API_discoverReq"></a>
### qnode.discoverReq(path, callback)
Discover report settings of a Resource or, an Object Instance ,or an Object on the Client Device.  

**Arguments:**  

1. `path` (_String_):  Path of the allocated Resource, Object Instance, or Object on the remote Client Device.
2. `callback` (_Function_): `function (err, rsp) { }`. The `rsp` object has a status code along with the parameters of report settings.  

    | Property | Type    | Description                                                                                                                                                                                         |
    |----------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    |  status  | Number  | [Status code](#StatusCode) of the response. Possible status codes are 205, 400, 404, and 405.                                                                                                       |
    |  data    | Object  | `data` is an object of the report settings. If the discoved target is an Object, there will be an additional field `data.resrcList` to list all its Resource idetifiers under each Object Instance. |
  
**Returns:**  
  
* (_none_)

**Examples:**  
    
```js
// discover a Resource successfully
qnode.discoverReq('temperature/0/sensedValue', function (err, rsp) {
    console.log(rsp);   // { status: 205, data: { pmin: 10, pmax: 600, gt: 45 }
});

// discover an Object successfully
qnode.discoverReq('temperature/', function (err, rsp) {
    console.log(rsp);
    /*
    {
      status: 205,
      data: {
         pmin: 10,
         pmax: 600,
         gt: 45,
         resrcList: {
             0: [ 1, 3, 88 ],    // Instance 0 has Resources 1, 3, and 88
             1: [ 1, 2, 6 ]      // Instance 1 has Resources 1, 2, and 6
         }
      }
    }
    */
});
```

***********************************************
<a name="API_observeReq"></a>
### qnode.observeReq(path[, callback])
Start observing a Resource on the Client Device. Please listen to event `'ind'` with indication type `'devNotify'` to get the reports.  

**Arguments:**  

1. `path` (_String_): Path of the allocated Resource on the remote Client Device.  
2. `callback` (_Function_): `function (err, rsp) { }`. The `rsp` object has a status code to indicate whether the operation succeeds.  

    | Property | Type    | Description                                                                                                         |
    |----------|---------|---------------------------------------------------------------------------------------------------------------------|
    |  status  | Number  | [Status code](#StatusCode) of the response. Possible status codes are 205, 400, 404, and 405.                       |

  
**Returns:**  
  
* (_none_)

**Examples:**  
    
```js
// observation starts successfully
qnode.observeReq('temperature/0/sensedValue', function (err, rsp) {
    console.log(rsp);       // { status: 205 }
});

// An Object is not allowed for observation
qnode.observeReq('temperature/', function (err, rsp) {
    console.log(rsp);       // { status: 400 }
});

// target is not allowed for observation
qnode.observeReq('temperature/0', function (err, rsp) {
    console.log(rsp);       // { status: 405 }
});

// target not found
qnode.observeReq('temperature/0/noSuchResource', function (err, rsp) {
    console.log(rsp);       // { status: 404 }
});
```

***********************************************
<a name="API_pingReq"></a>
### qnode.pingReq(callback)
Ping the remote Client Device.  

**Arguments:**  

1. `callback` (_Function_): `function (err, rsp) { }`. The `rsp` is a response object with a status code to tell the result of pinging. `rsp.data` is the approximate round-trip time in milliseconds.  

    | Property | Type    | Description                                                                   |
    |----------|---------|-------------------------------------------------------------------------------|
    |  status  | Number  | [Status code](#StatusCode) of the response. Possible status code is 200 (OK). |
    |  data    | Number  | Approximate round trip time in milliseconds.                                  |

**Returns:**  
  
* (_none_)

**Examples:**  
    
```js
qnode.pingReq(function (err, rsp) {
    if (!err)
        console.log(rsp);   // { status: 200, data: 12 }, round-trip time is 12 ms
});
```

*************************************************
<a name="API_maintain"></a>
### .maintain([callback])
Maintain this Client Device. This will refresh its record on qserver by rediscovering the remote qnode.  

**Arguments:**  

1. `callback` (_Function_): `function (err, lastTime) { ... }`. Get called with the timestamp `lastTime` (ms) after this maintenance finished. An error occurs when the request is timeout or the qnode is offline.  
  
**Returns:**  
  
* _none_

**Examples:**  
    
```js
qnode.maintain(function (err, lastTime) {
    if (!err)
        console.log(lastTime);  // 1470192227322 (ms, from 1970/1/1)
});
```
***********************************************
<a name="API_dump"></a>
### qnode.dump()
Dump record of the Client Device.  

**Arguments:**  

1. none  
  
**Returns:**  
  
* (_Object_): A data object of qnode record.

| Property         | Type    | Description                          |
|------------------|---------|--------------------------------------|
|  clientId        | String  | Client id of the device              |
|  ip              | String  | Ip address of the server             |
|  mac             | String  | Mac address                          |
|  lifetime        | Number  | Lifetime of the device               |
|  version         | String  | LWMQN version                        |
|  joinTime        | Number  | Unix Time (secs)                     |
|  objList         | Object  | Resource ids of each Object Instance |
|  so              | Object  | Contains IPSO Object(s)              |

**Examples:**  
    
```js
console.log(qnode.dump());

/* 
{
    clientId: 'foo_id',
    ip: '192.168.1.114',
    mac: '9e:65:f9:0b:24:b8',
    lifetime: 86400,
    version: 'v0.0.1',
    joinTime: 1460448761,
    objList: {
        '1': [ 0 ],
        '3': [ 0 ],
        '4': [ 0 ],
        '3303': [ 0, 1 ],
        '3304': [ 0 ]
    },
    so: {
        lwm2mServer: {  // oid is 'lwm2mServer' (1)
            '0': {
                shortServerId: null,
                lifetime: 86400,
                defaultMinPeriod: 1,
                defaultMaxPeriod: 60,
                regUpdateTrigger: '_unreadable_'
            }
        },
        device: {       // oid is 'device' (3)
            '0': {
                manuf: 'LWMQN_Project',
                model: 'dragonball',
                reboot: '_unreadable_',
                availPwrSrc: 0,
                pwrSrcVoltage: 5,
                devType: 'Env Monitor',
                hwVer: 'v0.0.1',
                swVer: 'v0.2.1'
            }
        },
        connMonitor: {  // oid is 'connMonitor' (4)
            '0': {
                ip: '192.168.1.114',
                routeIp: ''
            }
        },
        temperature: {              // oid is 'temperature' (3303)
            0: {                    //   iid = 0
                sensedValue: 18,    //     rid = 'sensedValue', its value is 18
                appType: 'home'     //     rid = 'appType', its value is 'home'
            },
            1: {
                sensedValue: 37,
                appType: 'fireplace'
            }
        },
        humidity: {                 // oid is 'humidity' (3304)
            0: {
                sensedValue: 26,
                appType: 'home'
            }
        }
    }
}
*/
```

***********************************************

<br />

<a name="Encryption"></a>
## 6. Message Encryption  

By default, the Server won't encrypt the message. You can override the encrypt() and decrypt() methods to implement your own message encryption and decryption. If you did, you should implement the encrypt() and decrypt() methods at your Client Devices as well.  

Note: You may like to distribute pre-configured keys to your Clients and utilize the [authentication](#Auth) approach to build your own security subsystem.  

***********************************************
### qserver.encrypt(msg, clientId, cb)
Method of encryption. Overridable.  

**Arguments:**  

1. `msg` (_String_ | _Buffer_): The outgoing message.  
2. `clientId` (_String_): Indicates the Client of this message going to.  
3. `cb` (_Function_): `function (err, encrypted) {}`, the callback you should call and pass the encrypted message to it after encryption.  
  

***********************************************
### qserver.decrypt(msg, clientId, cb)
Method of decryption. Overridable.  

**Arguments:**  

1. `msg` (_Buffer_): The incoming message which is a raw buffer.  
2. `clientId` (_String_): Indicates the Client of this message coming from.  
3. `cb` (_Function_): `function (err, decrypted) {}`, the callback you should call and pass the decrypted message to it after decryption.  
  
***********************************************

**Encryption/Decryption Example:**  

```js
var qserver = new MqttShepherd('my_iot_server');

// In this example, I simply encrypt the message with a constant password 'mysecrete'.
// You may like to get the password according to different Clients by `clientId` if you have
// a security subsystem.

qserver.encrypt = function (msg, clientId, cb) {
    var msgBuf = new Buffer(msg),
        cipher = crypto.createCipher('aes128', 'mysecrete'),
        encrypted = cipher.update(msgBuf, 'binary', 'base64');

    try {
        encrypted += cipher.final('base64');
        cb(null, encrypted);
    } catch (err) {
        cb(err);
    }
};

qserver.decrypt = function (msg, clientId, cb) {
    msg = msg.toString();
    var decipher = crypto.createDecipher('aes128', 'mysecrete'),
        decrypted = decipher.update(msg, 'base64', 'utf8');

    try {
        decrypted += decipher.final('utf8');
        cb(null, decrypted);
    } catch (err) {
        cb(err);
    }
};
```

***********************************************
<br />

<a name="Auth"></a>
## 7. Authentication and Authorization Policies

Override methods within `qserver.authPolicy` to authorize a Client. These methods include `authenticate()`, `authorizePublish()`, and `authorizeSubscribe()`.  

***********************************************
### qserver.authPolicy.authenticate(client, username, password, cb)  
Method of user authentication. Override at will.  
The default implementation authenticate all Clients.  

**Arguments:**  

1. `client` (_Object_): A mqtt client instance from [Mosca](http://mcollina.github.io/mosca/docs/lib/client.js.html#Client).  
2. `username` (_String_): Username given by a Client during connection.  
3. `password` (_Buffer_): Password given by a Client during connection.  
4. `cb` (_Function_): `function (err, valid) {}`, the callback you should call and pass a boolean flag `valid` to tell if this Client is authenticated.  
  
**Example:**  

```js
qserver.authPolicy.authenticate = function (client, username, password, cb) {
    var authorized = false,
        clientId = client.id;

    // This is just an example. 
    queryUserFromSomewhere(username, function (err, user) {     // maybe query from a local database
        if (err) {
            cb(err);
        } else if (username === user.name && password === user.password) {
            client.user = username;
            authorized = true;
            cb(null, authorized);
        } else {
            cb(null, authorized);
        }
    });
};
```

***********************************************
### qserver.authPolicy.authorizePublish(client, topic, payload, cb)  
Method of authorizing a Client to publish to a topic. Override at will.  
The default implementation authorize every Client, that was successfully registered, to publish to any topic.  

**Arguments:**  

1. `client` (_Object_): A mqtt client instance from [Mosca](http://mcollina.github.io/mosca/docs/lib/client.js.html#Client).  
2. `topic` (_String_): The topic to publish to.  
3. `payload` (_String_ | _Buffer_): The data to publish out.  
4. `cb` (_Function_): `function (err, authorized) {}`, the callback you should call and pass a boolean flag `authorized` to tell if a Client is authorized to publish the topic.  
  
**Example:**  

```js
qserver.authPolicy.authorizePublish = function (client, topic, payload, cb) {
    var authorized = false,
        clientId = client.id,
        username = client.user;

    // This is just an example. 
    passToMyAuthorizePublishSystem(clientId, username, topic, function (err, authorized) {
        cb(err, authorized);
    });
};
```

***********************************************
### qserver.authPolicy.authorizeSubscribe(client, topic, cb)  
Method of authorizing a Client to subscribe to a topic. Override at will.  
The default implementation authorize every Client, that was successfully registered, to subscribe to any topic.  

**Arguments:**  

1. `client` (_Object_): A mqtt client instance from [Mosca](http://mcollina.github.io/mosca/docs/lib/client.js.html#Client).  
2. `topic` (_String_): The topic to subscribe to.  
3. `cb` (_Function_): `function (err, authorized) {}`, the callback you should call and pass a boolean flag `authorized` to tell if a Client is authorized to subscribe to the topic.  
  
**Example:**  

```js
qserver.authPolicy.authorizeSubscribe = function (client, topic, cb) {
    var authorized = false,
        clientId = client.id,
        username = client.user;

    // This is just an example. 
    passToMyAuthorizeSubscribeSystem(clientId, username, topic, function (err, authorized) {
        cb(err, authorized);
    });
};
```

Please refer to Mosca Wiki to learn more about [Authentication & Authorization](https://github.com/mcollina/mosca/wiki/Authentication-&-Authorization)  


***********************************************
<a name="StatusCode"></a>
## 8. Status Code  

| Status Code               | Description                                                                                                                                                                              |
|---------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 200 (Ok)                  | Everything is fine                                                                                                                                                                       |
| 204 (Changed)             | The Client accepted this writing request successfully                                                                                                                                    |
| 400 (BadRequest)          | There is an unrecognized attribute/parameter within the request message                                                                                                                  |
| 404 (NotFound)            | The Client not found                                                                                                                                                                     |
| 405 (MethodNotAllowed)    | If you are trying to change either `clientId` or `mac`, to read something unreadable, to write something unwritable, and execute something unexecutable, then you will get this response |
| 500 (InternalServerError) | The Client has some trouble                                                                                                                                                              |
