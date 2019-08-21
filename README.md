advlib
======

Library for wireless advertising packet decoding.  Currently supports the following protocols:
* [Bluetooth Low Energy (BLE)](#bluetooth-smart-ble-advertising-packet-library)
* [reelyActive RFID](#reelyactive-rfid-library)

For a live, interactive version of advlib visit [reelyactive.github.io/advlib](https://reelyactive.github.io/advlib/).

This project was published in a scientific paper entitled [Low-Power Wireless Advertising Software Library for Distributed M2M and Contextual IoT](https://reelyactive.com/science/reelyActive-IoT2015.pdf) presented at the [2nd IEEE World Forum on Internet of Things (WF-IoT)](http://wfiot2015.ieee-wf-iot.org/) in Milan, Italy on December 16th, 2015.  See also our presentation [BLE as Active RFID](https://www.slideshare.net/reelyActive/ble-as-active-rfid) from IEEE RFID 2017 for a friendly overview of BLE advertising.

![advlib black box](https://reelyactive.github.io/advlib/web/images/advlib-blackbox.png)


Installation
------------

    npm install advlib


Hello advlib
------------

```javascript
var advlib = require('advlib');

var rawHexPacket = '421655daba50e1fe0201050c097265656c79416374697665';
var processedPacket = advlib.ble.process(rawHexPacket);

console.log(JSON.stringify(processedPacket, null, " "));
```

The console output should appear as follows:

    {
      type: "ADVA-48",
      value: "fee150bada55",
      advHeader: { 
        type: "ADV_NONCONNECT_IND",
        length: 22,
        txAdd: "random",
        rxAdd: "public" 
      },
      advData: {
        flags: [ "LE Limited Discoverable Mode", "BR/EDR Not Supported" ],
        completeLocalName: "reelyActive" 
      } 
    }


Bluetooth Low Energy (BLE) Advertising Packet Library
-----------------------------------------------------

Process a raw packet (as a hexadecimal string) with the following command:

    advlib.ble.process(rawHexPacket);

The library is organised hierarchically so that the separate elements of a packet can be processed individually.  Refer to the index below for details on each element:

* [Header](#header)
* [Address](#address)
* [Data (GAP)](#data-generic-access-profile)
  * [Flags](#flags)
  * [UUID](#uuid)
  * [Local Name](#local-name)
  * [Tx Power](#tx-power)
  * [Slave Connection Interval Range](#slave-connection-interval-range)
  * [Solicitation](#solicitation)
  * [Service Data](#service-data)
  * [Manufacturer Specific Data](#manufacturer-specific-data)
  * [Generic Data](#generic-data)
* [Data (GATT)](#data-generic-attribute-profile)
  * [Member Services](#member-services)
  * [Standard Services](#standard-services)

Complementary to the packet processing hierarchy above is a _common_ folder which contains supporting functions and lookups that are subject to frequent evolution:

* [Common](#common)
  * [Assigned Numbers](#assigned-numbers)
  * [Manufacturers](#manufacturers)
  * [Utilities](#util)


### Header

Process a 16-bit header (as a hexadecimal string) with the following command:

    advlib.ble.header.process(rawHexHeader);

For reference, the 16-bit header is as follows (reading the hexadecimal string from left to right):

| Bit(s) | Function                      |
|-------:|-------------------------------|
| 15     | RxAdd: 0 = public, 1 = random |
| 14     | TxAdd: 0 = public, 1 = random |
| 13-12  | Reserved for Future Use (RFU) |
| 11-8   | Type (see table below)        |
| 7-6    | Reserved for Future Use (RFU) |
| 5-0    | Payload length in bytes       |

And the advertising packet types are as follows:

| Type | Packet           | Purpose                                |
|-----:|------------------|----------------------------------------|
| 0    | ADV_IND          | Connectable undirected advertising     |
| 1    | ADV_DIRECT_IND   | Connectable directed advertising       |
| 2    | ADV_NONCONN_IND  | Non-connectable undirected advertising |
| 3    | SCAN_REQ         | Scan for more info from advertiser     |
| 4    | SCAN_RSP         | Response to scan request from scanner  |
| 5    | CONNECT_REQ      | Request to connect                     |
| 6    | ADV_DISCOVER_IND | Scannable undirected advertising       |

For example:

    advlib.ble.header.process('4216');

would yield:

    {
      rxAdd: "public",
      txAdd: "random",
      type: "ADV_NONCONNECT_IND",
      length: 22
    }


### Address

Process a 48-bit address (as a hexadecimal string) with the following command:

    advlib.ble.address.process(rawHexAddress);

For reference, the 48-bit header is as follows (reading the hexadecimal string from left to right):

| Bit(s) | Address component |
|-------:|-------------------|
| 47-40  | xx:xx:xx:xx:xx:## |
| 39-32  | xx:xx:xx:xx:##:xx |
| 31-24  | xx:xx:xx:##:xx:xx |
| 23-16  | xx:xx:##:xx:xx:xx |
| 15-8   | xx:##:xx:xx:xx:xx |
| 7-0    | ##:xx:xx:xx:xx:xx |

This is best illustrated with an example:

    advlib.ble.address.process('0123456789ab');

Would yield:

    {
      type: "ADVA-48",
      value: "ab8967452301"
    }

which can alternatively be represented as ab:89:67:45:23:01.


### Data (Generic Access Profile)

Process GAP data (as a hexadecimal string) with the following command:

    advlib.ble.data.process(rawHexData);

For reference, the structure of the data is as follows:

| Byte(s)     | Data component                                        |
|------------:|-------------------------------------------------------|
| 0           | Length of the data in bytes (including type and data) |
| 1           | GAP Data Type (see table below)                       |
| 2 to length | Type-specifc data                                     |

The Generic Access Profile Data Types are listed on the [Bluetooth GAP Assigned Numbers website](https://www.bluetooth.org/en-us/specification/assigned-numbers/generic-access-profile).  The following table lists the Data Types, their names and the section in this document in which they are described.

| Data Type | Data Type Name                       | See advlib section |
|----------:|--------------------------------------|--------------------|
| 0x01      | Flags                                | Flags              |
| 0x02      | Incomplete List of 16-bit UUIDs      | UUID               |
| 0x03      | Complete List of 16-bit UUIDs        | UUID               |
| 0x04      | Incomplete List of 32-bit UUIDs      | UUID               |
| 0x05      | Complete List of 32-bit UUIDs        | UUID               |
| 0x06      | Incomplete List of 128-bit UUIDs     | UUID               |
| 0x07      | Complete List of 128-bit UUIDs       | UUID               |
| 0x08      | Shortened Local Name                 | Local Name         |
| 0x09      | Complete Local Name                  | Local Name         |
| 0x0a      | Tx Power Level                       | Tx Power           |
| 0x0d      | Class of Device                      | Generic Data       |
| 0x0e      | Simple Pairing Hash C-192            | Generic Data       |
| 0x0f      | Simple Pairing Randomizer R-192      | Generic Data       |
| 0x10      | Security Manager TK Value            | Generic Data       |
| 0x11      | Security Manager OOB Flags           | Generic Data       |
| 0x12      | Slave Connection Interval Range      | SCIR               |
| 0x14      | 16-bit Solicitation UUIDs            | Solicitation       |
| 0x15      | 128-bit Solicitation UUIDs           | Solicitation       |
| 0x16      | Service Data 16-bit UUID             | Service Data       |
| 0x17      | Public Target Address                | Generic Data       |
| 0x18      | Random Target Address                | Generic Data       |
| 0x19      | Public Target Address                | Generic Data       |
| 0x1a      | Advertising Interval                 | Generic Data       |
| 0x1b      | LE Bluetooth Device Address          | Generic Data       |
| 0x1c      | LE Bluetooth Role                    | Generic Data       |
| 0x1d      | Simple Pairing Hash C-256            | Generic Data       |
| 0x1e      | Simple Pairing Hash Randomizer C-256 | Generic Data       |
| 0x1f      | 32-bit Solicitation UUIDs            | Solicitation       |
| 0x20      | Service Data 32-bit UUID             | Service Data       |
| 0x21      | Service Data 128-bit UUID            | Service Data       |
| 0x22      | LE Secure Con. Confirmation Value    | Generic Data       |
| 0x23      | LE Secure Connections Random Value   | Generic Data       |
| 0x24      | URI                                  | Generic Data       |
| 0x25      | Indoor Positioning                   | Generic Data       |
| 0x26      | Transport Discovery Data             | Generic Data       |
| 0x27      | LE Supported Features                | Generic Data       |
| 0x28      | Channel Map Update Indication        | Generic Data       |
| 0x29      | PB-ADV                               | Generic Data       |
| 0x2a      | Mesh Message                         | Generic Data       |
| 0x2b      | Mesh Beacon                          | Generic Data       |
| 0x3d      | 3-D Information Data                 | Generic Data       |
| 0xff      | Manufacturer Specific Data           | Mfr. Specific Data |


#### UUID 

The following example illustrates the processing of a UUID:

    var payload = '17074449555520657669746341796c656572';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte(s) | Hex String                       | Description                    |
|--------:|:---------------------------------|:-------------------------------|
| 0       | 17                               | Length, in bytes, of type and data |
| 1       | 07                               | GAP Data Type for Complete 128-bit UUIDs | 
| 2 to 17 | 4449555520657669746341796c656572 | reelyActive's 128-bit UUID     |

Which would return:

    { complete128BitUUIDs: "7265656c794163746976652055554944" }

It is also possible to process just the UUID:

    var uuid = '4449555520657669746341796c656572';
    advlib.ble.data.gap.uuid.process(uuid);

Which would simply return "7265656c794163746976652055554944".


#### Local Name 

The following example illustrates the processing of a local name:

    var payload = '12097265656c79416374697665';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte(s) | Hex String             | Description                             |
|--------:|:-----------------------|:----------------------------------------|
| 0       | 12                     | Length, in bytes, of type and data      |
| 1       | 09                     | GAP Data Type for Complete Local Name   | 
| 2 to 12 | 7265656c79416374697665 | ASCII representation of 'reelyActive'   |

Which would return:

    { completeLocalName: "reelyActive" }

It is also possible to process just the local name:

    var name = '7265656c79416374697665';
    advlib.ble.data.gap.localname.process(name);

Which would simply return "reelyActive".


#### Flags

The following example illustrates the processing of flags:

    var payload = '020104';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte | Hex String  | Description                         |
|-----:|:------------|:------------------------------------|
| 0    | 02          | Length, in bytes, of type and data  |
| 1    | 01          | GAP Data Type for flags             | 
| 2    | 04          | Flags byte (see table below)        |

Which would return:

    { flags: [ 'BR/EDR Not Supported' ] }
 
For reference, the bits of the flags byte are as follows:

| Bit | Description                                                    |
|----:|----------------------------------------------------------------|
| 0   | LE Limited Discoverable Mode                                   |
| 1   | LE General Discoverable Mode                                   |
| 2   | BR/EDR Not Supported                                           |
| 3   | Simultaneous LE and BR/EDR to Same Device Capable (Controller) |
| 4   | Simultaneous LE and BR/EDR to Same Device Capable (Host)       |
| 5   | Reserved                                                       |

It is also possible to process just the flags byte:

    var flags = '04';
    advlib.ble.data.gap.flags.process(flags);

Which would simply return [ 'BR/EDR Not Supported' ].


#### Manufacturer Specific Data

The following example illustrates the processing of manufacturer specific data:

    var payload = '03ff2805';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte(s) | Hex String | Description                                     |
|--------:|:-----------|:------------------------------------------------|
| 0       | 03         | Length, in bytes, of type and data              |
| 1       | ff         | GAP Data Type for manufacturer specific data    | 
| 2-3     | 8c00       | Gimbal company identifier code (bytes reversed) |

Which would return:

    {
      manufacturerSpecificData: {
        companyName: 'Lunera Inc.',
        companyIdentifierCode: '2805',
        data: ''
      }
    }


The proprietary data of some manufacturers can be further processed.  The data for those supported will automatically be processed.  See the [Manufacturers](#manufacturers) section for the list of all supported manufacturers.

It is also possible to process just the manufacturer specific data:

    var data = '2805';
    advlib.ble.data.gap.manufacturerspecificdata.process(data);

Which would return the following:

    {
      companyName: 'Lunera Inc.',
      companyIdentifierCode: '0528',
      data: ''
    }



#### TX Power Level 

The following example illustrates the processing of TX power level:

    var payload = '020a7f';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte(s) | Hex String | Description                         |
|--------:|:-----------|:------------------------------------|
| 0       | 02         | Length, in bytes, of type and data  |
| 1       | 0a         | GAP Data Type for TxPower           | 
| 2       | 7f         | TxPower (see table below)           |

TxPower is a two's complement value which is interpreted as follows:

| Hex String  | Power dBm |
|------------:|-----------|
| 7f          | 127 dBm   |
| 00          | 0 dBm     |
| ff          | -128 dBm  |

Which would return:

    { txPower: "127dBm" }

It is also possible to process just the TX power level:

    var power = '7f';
    advlib.ble.data.gap.txpower.process(power);

Which would return "127dBm".
    

#### Slave Connection Interval Range 

The following example illustrates the processing of slave connection interval range:

    var payload = '051200060c80';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte(s) | Hex String | Description                                       |
|--------:|:-----------|:--------------------------------------------------|
| 0       | 05         | Length, in bytes, of type and data                |
| 1       | 12         | GAP Data Type for Slave Connection Interval Range | 
| 2-5     | 00060c80   | Min and max intervals (see table below)           |

The intervals would be intepreted as follows:

| Byte(s) | Hex String | Description                               |
|--------:|:-----------|:------------------------------------------|
| 0-1     | 0006       | Min interval = 6 x 1.25 ms = 7.5 ms       |
| 2-3     | 0c80       | Max interval = 12128 x 1.25 ms = 15160 ms |

Which would return:

    { slaveConnectionIntervalRange: "00060c80" }
      

#### Service Solicitation 

The following example illustrates the processing of a service solicitation UUID:

    var payload = '0314d8fe';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte(s) | Hex String | Description                                          |
|--------:|:-----------|:-----------------------------------------------------|
| 0       | 03         | Length, in bytes, of type and data                   |
| 1       | 14         | GAP Data Type for 16-bit Service Solicitation UUID   | 
| 2-3     | d8fe       | Google's UriBeacon UUID (bytes reversed)             |

Which would return:

    { solicitation16BitUUIDs: "fed8" }

It is also possible to process just the UUID:

    var uuid = 'd8fe';
    advlib.ble.data.gap.solicitation.process(uuid);

Which would simply return "fed8".


#### Service Data 

The following example illustrates the processing of service data:

    var payload = '09160a181204eb150000';
    advlib.ble.data.process(payload);
    
For reference, the example payload is interpreted as follows:

| Byte(s) | Hex String   | Description                         |
|--------:|:-------------|:------------------------------------|
| 0       | 09           | Length, in bytes, of type and data  |
| 1       | 16           | GAP Data Type for service data      | 
| 2-3     | 0a18         | UUID (bytes reversed)               |
| 4-9     | 1204eb150000 | Service Data                        |

Which would return:

    {
      serviceData: {
        uuid : "180a",
        data : "1204eb150000",
        specificationName: "Device Information"
      }
    }

In this case, the service UUID represents one of the GATT [Standard Services](#standard-services) and is processed as such.

Additional examples for service UUIDs among the GATT [Member Services](#member-services) are given in that section below.

It is also possible to process just the service data:

    var data = '0a181204eb150000';
    advlib.ble.data.gap.servicedata.process(data);

Which would simply return:

    {
      uuid: '180a',
      data: '1204eb150000',
      specificationName: 'Device Information'
    }


### Data (Generic Attribute Profile)

GATT service data is automatically processed when the Service Data GAP Type is present (see [Service Data](#service-data)).

Based on the UUID, the serviceData will be parsed as either a [member service](#member-services) or a [standard service](#standard-services), as applicable.  Note that not all services are yet implemented.


#### Member Services

Based on a pilot program for members which allows the SIG to allocate a 16-bit Universally Unique Identifier (UUID) for use with a custom GATT-based service defined by the member.

| UUID   | Member                | Description                           |
|-------:|-----------------------|---------------------------------------|
| 0xfed8 | Google                | UriBeacon (Physical Web)              |
| 0xfeaa | Google                | Eddystone                             |
| 0xfe9a | Estimote              | Estimote Location & Telemetry         |


##### Google

Supports Eddystone ([UID](#eddystone-uid), [URL](#eddystone-url), [TLM](#eddystone-tlm) & [EID](#eddystone-eid)) and the [UriBeacon](#uribeacon) of the Physical Web.

###### UriBeacon

Process UriBeacon data (UUID = 0xfed8).

    advlib.ble.data.gatt.services.members.process(advData);
 
This is best illustrated with an example using the following input:

    advData: {
      serviceData: {
        uuid: "fed8",
        data: "00f2027265656c7961637469766507"
      }
    }
       
For reference, the example serviceData.data is interpreted as follows, based on the [UriBeacon Advertising Packet Specification](https://github.com/google/uribeacon/blob/uribeacon-final/specification/AdvertisingMode.md):

| Byte(s) | Hex String               | Description                   |
|--------:|:-------------------------|:------------------------------|
| 0       | 00                       | UriBeacon flags               |
| 1       | f2                       | UriBeacon TxPower level       | 
| 3       | 02                       | Uri Scheme Prefix (http://)   |
| 4-15    | 7265656c7961637469766507 | Encoded Uri (reelyactive.com) |

Which would add the following properties to advData:

    serviceData: {
      uuid: "fed8",
      data: "00f2027265656c7961637469766507",
      companyName: "Google​",
      uriBeacon: {
        invisibleHint: false,
        txPower: "-14dBm",
        url: "http://reelyactive.com"
      }
    }

###### Eddystone-UID

Process Eddystone-UID data (UUID = 0xfeaa).

    advlib.ble.data.gatt.services.members.process(advData);
 
This is best illustrated with an example using the following input:

    advData: {
      serviceData: {
        uuid: "feaa",
        data: "00128b0ca750095477cb3e770011223344550000"
      }
    }
       
For reference, the example serviceData.data is interpreted as follows, based on the [Eddystone-UID specification](https://github.com/google/eddystone/tree/master/eddystone-uid):

| Byte(s) | Hex String           | Description                   |
|--------:|:---------------------|:------------------------------|
| 0       | 00                   | Eddystone-UID Frame Type      |
| 1       | 12                   | Calibrated TxPower at 0m      | 
| 2-11    | 8b0ca750095477cb3e77 | 10-byte ID Namespace          |
| 12-17   | 001122334455         | 6-byte ID Instance            |
| 18-19   | 0000                 | Reserved for Future Use       |

Which would add the following properties to advData:

    serviceData: {
      uuid: "feaa",
      data: "00128b0ca750095477cb3e770011223344550000",
      eddystone: {
        type: "UID",
        txPower: "18dBm",
        uid: {
          namespace: "8b0ca750095477cb3e77",
          instance: "001122334455"
        }
      }
    }

###### Eddystone-URL

Process Eddystone-URL data (UUID = 0xfeaa).

    advlib.ble.data.gatt.services.members.process(advData);
 
This is best illustrated with an example using the following input:

    advData: {
      serviceData: { 
        uuid: "feaa",
        data: "1012027265656c7961637469766507" 
      }
    }
       
For reference, the example serviceData.data is interpreted as follows, based on the [Eddystone-URL specification](https://github.com/google/eddystone/tree/master/eddystone-url):

| Byte(s) | Hex String               | Description                   |
|--------:|:-------------------------|:------------------------------|
| 0       | 10                       | Eddystone-URL Frame Type      |
| 1       | 12                       | Calibrated TxPower at 0m      | 
| 2       | 02                       | URL Scheme (http://)          |
| 3-14    | 7265656c7961637469766507 | Encoded URL (reelyactive.com) |

Which would add the following properties to advData:

    serviceData: {
      uuid: "feaa",
      data: "1012027265656c7961637469766507",
      eddystone: {
        type: "URL",
        txPower: "18dBm",
        url: "http://reelyactive.com"
      }
    }

###### Eddystone-TLM

Process Eddystone-TLM data (UUID = 0xfeaa).

    advlib.ble.data.gatt.services.members.process(advData);
 
This is best illustrated with an example using the following input:

    advData: {
      serviceData: { 
        uuid: "feaa",
        data: "20000bb81800000000010000000a"  
      }
    }
       
For reference, the example serviceData.data is interpreted as follows, based on the [Eddystone-TLM specification](https://github.com/google/eddystone/tree/master/eddystone-tlm), in this case [Unencrypted TLM](https://github.com/google/eddystone/blob/master/eddystone-tlm/tlm-plain.md):

| Byte(s) | Hex String               | Description                   |
|--------:|:-------------------------|:------------------------------|
| 0       | 20                       | Eddystone-TLM Frame Type      |
| 1       | __00__                   | TLM Version                   | 
| 2-3     | 0bb8                     | Battery Voltage (mV)          |
| 4-5     | 1800                     | Temperature (8:8 fixed point) |
| 6-9     | 00000001                 | Advertising PDU Count         |
| 10-13   | 0000000a                 | Uptime (0.1s resolution)      |

Which would add the following properties to advData:

    serviceData: {
      uuid: 'feaa',
      data: '20000bb81800000000010000000a',
      eddystone: {
        type: "TLM",
        version: "00",
        batteryVoltage: "3000mV",
        temperature: "24C",
        advertisingCount: 1,
        uptime: "1s"
      }
    }

Consider also the example of [Encrypted TLM](https://github.com/google/eddystone/blob/master/eddystone-tlm/tlm-encrypted.md):

| Byte(s) | Hex String               | Description              |
|--------:|:-------------------------|:-------------------------|
| 0       | 20                       | Eddystone-TLM Frame Type |
| 1       | __01__                   | TLM Version              | 
| 2-13    | 112233445566778899aabbcc | Encrypted TLM            |
| 14-15   | 0123                     | Salt                     |
| 16-17   | abcd                     | Message Integrity Check  |

Which would add the following properties to advData:

    serviceData: {
      uuid: 'feaa',
      data: '2001112233445566778899aabbcc0123abcd',
      eddystone: {
        type: "TLM",
        version: "01",
        etlm: "112233445566778899aabbcc",
        salt: "0123",
        mic: "abcd"
      }
    }

###### Eddystone-EID

Process Eddystone-EID data (UUID = 0xfeaa).

    advlib.ble.data.gatt.services.members.process(advData);
 
This is best illustrated with an example using the following input:

    advData: {
      serviceData: {
        uuid: "feaa",
        data: "30001122334455667788"
      }
    }
       
For reference, the example serviceData.data is interpreted as follows, based on the [Eddystone-EID specification](https://github.com/google/eddystone/tree/master/eddystone-eid):

| Byte(s) | Hex String       | Description                 |
|--------:|:-----------------|:----------------------------|
| 0       | 30               | Eddystone-EID Frame Type    |
| 1       | 00               | Calibrated TxPower at 0m    | 
| 2-9     | 1122334455667788 | 8-byte Ephemeral Identifier |

Which would add the following properties to advData:

    serviceData: {
      uuid: "feaa",
      data: "30001122334455667788",
      eddystone: {
        type: "EID",
        txPower: "0dBm",
        eid: "1122334455667788"
      }
    }

##### Estimote

Supports Estimote beacons that send additional data within a service.  Note that [Estimote Nearables](#nearables) use Manufacturer Specific Data instead of a service.  The first byte of the service data specifies the type, of which there are two observed: location (0x00) and telemetry (0x12).

| Byte(s) | Hex String       | Description               |
|--------:|:-----------------|:--------------------------|
| 0       | 12               | Estimote Telemetry packet |
| 1-8     | ceaac4fd251b1e35 | 64-bit static identifier  | 
| 9+      |                  | Type-dependent            |

Process Estimote service data (UUID = 0xfe9a).

    advlib.ble.data.gatt.services.members.process(advData);

Consider the following case.

###### Estimote Telemetry

This is best illustrated with an example using the following input:

    advData: {
      serviceData: { 
        uuid: "fe9a",
        data: "12ceaac4fd251b1e350020c9010196f002" 
      }
    }

| Byte(s) | Hex String       | Description                               |
|--------:|:-----------------|:------------------------------------------|
| 0       | 12               | Estimote Telemetry packet                 |
| 1-8     | ceaac4fd251b1e35 | 64-bit static identifier                  |
| 9       | 00               | Subtype 0x00                              |
| 10      | 20               | Acceleration in X-axis (two's complement) |
| 11      | c9               | Acceleration in Y-axis (two's complement) |
| 12      | 01               | Acceleration in Z-axis (two's complement) |
| 13-16   | 0196f002         | Motion/state/duration bytes?              |

Which would add the following property to advData:

    serviceData: {
      uuid: "fe9a",
      data: "12ceaac4fd251b1e350020c9010196f002",
      estimote: {
        type: "telemetry",
        id: "ceaac4fd251b1e35",
        subtype: "00",
        accelerationX: 0.5,
        accelerationY: -0.859375,
        accelerationZ: 0.015625,
        statusBytes: [ "01", "96", "f0", "02" ]
      }
    }

##### Minew

Supports the Minew S1, E6, E8 and i7 sensor beacons which send data within a service.  Note that the service UUID __0xffe1__ does not appear to be a valid member service granted by the Bluetooth SIG.

###### S1 Temperature/Humidity Beacon

This is best illustrated with an example using the following input:

    advData: {
      serviceData: { 
        uuid: "ffe1",
        data: "a1016416dc27fd0c04a03f23ac" 
      }
    }

| Byte(s) | Hex String   | Description                         |
|--------:|:-------------|:------------------------------------|
| 0       | a1           | Frame type                          |
| 1       | 01           | Product model                       |
| 2       | 64           | Battery level in percent            |
| 3-4     | 16dc         | Temperature in celcius (signed 8.8) |
| 5-6     | 27fd         | Humidity percentage (signed 8.8)    |
| 7-12    | 0c04a03f23ac | MAC address                         |

Which would add the following property to advData:

    serviceData: {
      uuid: "ffe1",
      data: "a1016416dc27fd0c04a03f23ac",
      minew: {
        frameType: "a1",
        productModel: 1,
        batteryPercent: 100,
        temperature: 22.859375,
        humidity: 39.98828125,
        macAddress: "ac:23:3f:a0:04:0c"
      }
    }

###### E6 Light Sensor Beacon

This is best illustrated with an example using the following input:

    advData: {
      serviceData: { 
        uuid: "ffe1",
        data: "a1026401ff04a03f23ac" 
      }
    }

| Byte(s) | Hex String   | Description                     |
|--------:|:-------------|:--------------------------------|
| 0       | a1           | Frame type                      |
| 1       | 02           | Product model                   |
| 2       | 64           | Battery level in percent        |
| 3       | 01           | Visible light observed? (Bit 0) |
| 7-12    | ff04a03f23ac | MAC address                     |

Which would add the following property to advData:

    serviceData: {
      uuid: "ffe1",
      data: "a1026401ff04a03f23ac",
      minew: {
        frameType: "a1",
        productModel: 2,
        batteryPercent: 100,
        visibleLight: true,
        macAddress: "ac:23:3f:a0:04:ff"
      }
    }

###### E8 and i7 Accelerometer Beacon

This is best illustrated with an example using the following input:

    advData: {
      serviceData: { 
        uuid: "ffe1",
        data: "a1036400d70087fffe5705a03f23ac" 
      }
    }

| Byte(s) | Hex String   | Description                         |
|--------:|:-------------|:------------------------------------|
| 0       | a1           | Frame type                          |
| 1       | 03           | Product model                       |
| 2       | 64           | Battery level in percent            |
| 3-4     | 00d7         | Acceleration in X-axis (signed 8.8) |
| 5-6     | 0087         | Acceleration in Y-axis (signed 8.8) |
| 7-8     | fffe         | Acceleration in Z-axis (signed 8.8) |
| 9-14    | 5705a03f23ac | MAC address                         |

Which would add the following property to advData:

    serviceData: {
      uuid: "ffe1",
      data: "a1036400d70087fffe5705a03f23ac",
      minew: {
        frameType: "a1",
        productModel: 3,
        batteryPercent: 100,
        accelerationX: 0.83984375,
        accelerationY: 0.52734375,
        accelerationZ: -0.0078125,
        macAddress: "ac:23:3f:a0:05:57"
      }
    }

###### Minew Info Frame

This is best illustrated with an example using the following input:

    advData: {
      serviceData: { 
        uuid: "ffe1",
        data: "a10864d739a03f23ac504c5553" 
      }
    }

| Byte(s) | Hex String   | Description              |
|--------:|:-------------|:-------------------------|
| 0       | a1           | Frame type               |
| 1       | 08           | Version number           |
| 2       | 64           | Battery level in percent |
| 3-8     | d739a03f23ac | MAC address              |
| 9-12    | 504c5553     | Name (in ASCII)          |

Which would add the following property to advData:

    serviceData: {
      uuid: "ffe1",
      data: "a1036400d70087fffe5705a03f23ac",
      minew: {
        frameType: "a1",
        productModel: 8,
        batteryPercent: 100,
        macAddress: "ac:23:3f:a0:39:d7",
        name: "PLUS"
      }
    }


#### Standard Services

The following GATT Services, assigned in the [GATT Specification](https://developer.bluetooth.org/gatt/services/Pages/ServicesHome.aspx) are identified but not parsed:

- Alert Notification Service
- Automation IO
- Battery Service
- Blood Pressure
- Body Composition 
- Bond Management
- Continous Glucose Monitoring
- Current Time Service
- Cycling Power
- Cycling Speed and Cadence
- Device Information
- Environmental Sensing
- Generic Access 
- Generic Attribute
- Glucose
- Health Thermometer
- Heart Rate
- Human Interface Device
- Immediate Alert
- Indoor Positioning
- Internet Protocol Support
- Link Loss
- Location and Navigation
- Next DST Change Service
- Phone Alert Status Service
- Pulse Oximeter
- Reference Time Update Service
- Running Speed and Cadence
- Scan Parameters
- TX Power
- User Data
- Weight Scale


### Common

Supporting functions and lookups that are subject to frequent evolution are:
* [Assigned Numbers](#assigned-numbers)
* [Manufacturers](#manufacturers)
* [Utilities](#utilities)

#### Assigned Numbers

The Bluetooth SIG maintains a [list of assigned numbers](https://www.bluetooth.org/en-us/specification/assigned-numbers).  The advlib currently implements [16-bit UUIDs for Members](#member-services) and [Company Identifiers](#company-identifiers).

##### Company Identifiers

The Bluetooth SIG maintains a [list of company identifiers](https://www.bluetooth.org/en-us/specification/assigned-numbers/company-identifiers).  Look up a company name from its 16-bit code.

    advlib.ble.common.companyidentifiercodes.companyNames[companyCode];

For example:

    advlib.ble.common.companyidentifiercodes.companyNames['000d'];

would yield:

    "Texas Instruments Inc."

##### Member Services

The Bluetooth SIG maintains a list of 16-bit UUIDs for Members, for members.  In other words, this list is accessible to members.  Look up a company name from its 16-bit service UUID.

    advlib.ble.common.memberservices.companyNames[uuid];

For example:

    advlib.ble.common.memberservices.companyNames['feed'];

would yield:

    "Tile, Inc."

#### Manufacturers

All functions and lookups which represent manufacturer-proprietary data contained in the [Manufacturer Specific Data](#manufacturer-specific-data) data type are included here.  Each manufacturer is contained in a separate subfolder.

##### Apple

Process Apple-proprietary data from a packet's contained advertiser data.

    advlib.ble.common.manufacturers.apple.process(advData);

The first byte of the proprietary data specifies the type:

| Type (Hex String) | Description                             |
|------------------:|:----------------------------------------|
| 01                | Observed on iOS, not explicitly handled |
| 02                | [iBeacon](#ibeacon)                     |
| 05                | [AirDrop](#airdrop)                     |
| 07                | [AirPods](#airpods)                     |
| 08                | Unknown, observed on Macs               |
| 09                | [AirPlay](#airplay) destination         |
| 0a                | [AirPlay](#airplay) source              |
| 0c                | [Handoff](#handoff)                     |
| 10                | [Nearby](#nearby)                       |

###### iBeacon

A specific case of Apple-proprietary data is the iBeacon.  Look up a licensee name from its 128-bit iBeacon UUID.

    advlib.ble.common.manufacturers.apple.ibeacon.licenseeNames[uuid];

For example:

    advlib.ble.common.manufacturers.apple.ibeacon.licenseeNames['f7826da64fa24e988024bc5b71e0893e'];

would yield:

    "Kontakt.io"

Process an iBeacon packet from the contained advertiser data.

    advlib.ble.common.manufacturers.apple.ibeacon.process(advData);

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "004c",
        data: "0215b9407f30f5f8466eaff925556b57fe6d294c903974"
      }
    }

For reference, the iBeacon payload is interpreted as follows:

| Byte(s) | Hex String                       | Description                    |
|--------:|:---------------------------------|:-------------------------------|
| 0       | 26                               | Length, in bytes, of type and data |
| 1       | ff                               | GAP Data Type for manufacturer specific data | 
| 2-3     | 4c00                             | Apple company identifier code (bytes reversed) |
| 4-5     | 0215                             | Identifier code for iBeacon    |
| 6-21    | b9407f30f5f8466eaff925556b57fe6d | UUID (assigned by Apple)       |
| 21-22   | 294c                             | Major                          |
| 23-24   | 9039                             | Minor                          |
| 25      | 74                               | TxPower (see TxPower section)  |

Which would add the following property to advData:

    manufacturerSpecificData: {
      iBeacon: {
        uuid: "b9407f30f5f8466eaff925556b57fe6d",
        major: "294c",
        minor: "9039",
        txPower: "116dBm",
        licenseeName: "Estimote"
      }
    }

###### AirDrop

A specific case of Apple-proprietary data is AirDrop.  Process an AirDrop packet from the contained advertiser data.

    advlib.ble.common.manufacturers.apple.airdrop.process(advData);

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "004c",
        data: "05120000000000000000011bc238fa0000000000"
      }
    }

For reference, the AirDrop payload is interpreted as follows:

| Byte(s) | Hex String                           | Description               |
|--------:|:-------------------------------------|:--------------------------|
| 0       | 12                                   | Length, in bytes, of data |
| 1-18    | 0000000000000000011bc238fa0000000000 | Data                      |

Which would add the following property to advData:

    manufacturerSpecificData: {
      airdrop: { 
        length: 18,
        data: "0000000000000000011bc238fa0000000000"
      }
    }

###### AirPods

A specific case of Apple-proprietary data is AirPods.  Process an AirPods packet from the contained advertiser data.

    advlib.ble.common.manufacturers.apple.airpods.process(advData);

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "004c",
        data: "071901022021880f00000049bcba4477206b0447472c771e53bbeb"
      }
    }

For reference, the AirDrop payload is interpreted as follows:

| Byte(s) | Hex String                           | Description               |
|--------:|:-------------------------------------|:--------------------------|
| 0       | 19                                   | Length, in bytes, of data |
| 1-25    | 01022021880f00000049bcba4477206b0447472c771e53bbeb | Data        |

Which would add the following property to advData:

    manufacturerSpecificData: {
      airpods: { 
        length: 25,
        data: "01022021880f00000049bcba4477206b0447472c771e53bbeb"
      }
    }

###### AirPlay

A specific case of Apple-proprietary data is AirPlay.  Process an AirPlay packet from the contained advertiser data.

    advlib.ble.common.manufacturers.apple.airplay.process(advData);

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "004c",
        data: "09060200c0a80030"
      }
    }

For reference, the AirPlay payload is interpreted as follows:

| Byte(s) | Hex String   | Description               |
|--------:|:-------------|:--------------------------|
| 0       | 06           | Length, in bytes, of data |
| 1-6     | 0200c0a80030 | Data                      |

Which would add the following property to advData:

    manufacturerSpecificData: {
      airplay: { 
        length: 6,
        role: "destination",
        data: "0200c0a80030"
      }
    }

###### Handoff

A specific case of Apple-proprietary data is Handoff.  Process a Handoff packet from the contained advertiser data.

    advlib.ble.common.manufacturers.apple.handoff.process(advData);

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "004c",
        data: "0c0e0000041b59594de21ab6fbbb5cf6"
      }
    }

For reference, the Handoff payload is interpreted as follows:

| Byte(s) | Hex String                   | Description               |
|--------:|:-----------------------------|:--------------------------|
| 0       | 0e                           | Length, in bytes, of data |
| 1-14    | 0000041b59594de21ab6fbbb5cf6 | Data                      |

Which would add the following property to advData:

    manufacturerSpecificData: {
      handoff: { 
        length: 14,
        data: "0000041b59594de21ab6fbbb5cf6"
      }
    }

###### Nearby

A specific case of Apple-proprietary data is Nearby.  Process a Nearby packet from the contained advertiser data.

    advlib.ble.common.manufacturers.apple.nearby.process(advData);

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "004c",
        data: "10020100"
      }
    }

For reference, the Nearby payload is interpreted as follows:

| Byte(s) | Hex String | Description               |
|--------:|:-----------|:--------------------------|
| 0       | 02         | Length, in bytes, of data |
| 1-2     | 0100       | Data                      |

Which would add the following property to advData:

    manufacturerSpecificData: {
      nearby: { 
        length: 2,
        data: "0100"
      }
    }

##### StickNFind

Process StickNFind-proprietary data from a packet's contained advertiser data.

    advlib.ble.common.manufacturers.sticknfind.process(advData);

The first byte of the proprietary data specifies the type:

| Type (Hex String) | Description                   |
|------------------:|:------------------------------|
| 01                | StickNFind Single             |
| 42                | StickNSense Motion            |

##### Estimote

Process Estimote-proprietary data from a packet's contained advertiser data.

    advlib.ble.common.manufacturers.estimote.process(advData);

###### Nearables

A specific case of Estimote-proprietary data is the [Nearable](http://developer.estimote.com/nearables/).  In the absence of official documentation, advlib assumes that all Estimote-proprietary data represents a Nearable and is interpreted based on experimentally-observed behaviour.  Kindly advise or submit a pull request should official documentation be made available.

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "015d",
        data: "012b9e3834cfbfaa710401acc1b202ff3f000353"
    }

| Byte(s) | Hex String       | Description                                  |
|--------:|:-----------------|:---------------------------------------------|
| 0       | 01               | Unknown (Type?). Status byte 0.              |
| 1-8     | 2b9e3834cfbfaa71 | 64-bit Nearables ID                          |
| 9       | 04               | Unknown (Firmware?). Status byte 1.          |
| 10      | 01               | Unknown (Toggling status?). Status byte 2.   |
| 11      | ac               | Temperature (LSB) (upper bits in next byte)  |
| 12      | c1               | Bit 6 current state. Temp. Status byte 3.    |
| 13      | b2               | Unknown (Toggling status?). Status byte 4.   |
| 14      | 02               | Acceleration in X-axis (two's complement)    |
| 15      | ff               | Acceleration in Y-axis (two's complement)    |
| 16      | 3f               | Acceleration in Z-axis (two's complement)    |
| 17      | 00               | Duration of current state (var. resolution)  |
| 18      | 03               | Duration of previous state (var. resolution) |
| 19      | 53               | Unknown (TX power?). Status byte 5.          |

Which would add the following property to advData:

    manufacturerSpecificData: {
      nearable: {
        id: "2b9e3834cfbfaa71",
        temperature: 26.75,
        currentState: "still",
        accelerationX: 0.03125,
        accelerationY: -0.015625,
        accelerationZ: 0.984375,
        currentStateSeconds: 0,
        previousStateSeconds: 3,
        statusBytes: [ "01", "04", "01", "c1", "b2", "53" ]
      }
    }

##### Radius Networks

###### AltBeacon

Process an AltBeacon packet from the contained advertiser data.

    advlib.ble.common.manufacturers.radiusnetworks.altbeacon.process(advData);

This is best illustrated with an example using the following input:

    advData: {
      manufacturerSpecificData: {
        companyIdentifierCode: "0118",
        data: "beac00010203040506070809101112131415161718190069"
      }
    }

For reference, the AltBeacon payload is interpreted as follows:

| Byte(s) | Hex String                               | Description           |
|--------:|:-----------------------------------------|:----------------------|
| 0-1     | beac                                     | AltBeacon code        |
| 2-21    | 0001020304050607080910111213141516171819 | ID                    | 
| 22      | 00                                       | Reference RSSI        |
| 23      | 69                                       | Manufacturer reserved |

Which would add the following property to advData:

    manufacturerSpecificData: {
      altBeacon: {
        id: "0001020304050607080910111213141516171819",
        refRSSI: "0dBm",
        mfgReserved: "69"
      }
    }


#### Utilities

##### PDU

More info to come.


reelyActive RFID Library
------------------------

Process a raw packet (as a hexadecimal string) with the following command:

    advlib.reelyactive.process(rawHexPacket);


What's next?
------------

This is an active work in progress.  Expect regular changes and updates, as well as improved documentation!  If you'd like to contribute, kindly read our [Node.js style guide](https://github.com/reelyactive/node-style-guide) and [contact us](https://www.reelyactive.com/contact/) or make a pull request.


License
-------

MIT License

Copyright (c) 2015-2019 [reelyActive](https://www.reelyactive.com)

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR 
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE 
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER 
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, 
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN 
THE SOFTWARE.
