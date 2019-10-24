# LoRa Server Docker setup

This repository contains a skeleton to setup the Lora Server
project using [docker-compose](https://docs.docker.com/compose/).

**Note:** Please use this `docker-compose.yml` file as a starting point for testing
but keep in mind that for production usage it might need modifications. 

## Directory layout

* `docker-compose.yml`: the docker-compose file containing the services
* `docker-compose-env.yml`: alternate docker-compose file using environment variables, can be run with the docker-compose `-f` flag
* `configuration/lora*`: directory containing the LoRa Server configuration files, see:
* `configuration/postgresql/initdb/`: directory containing PostgreSQL initialization scripts

## Configuration

The LoRa Server components are pre-configured to work with the provided
`docker-compose.yml` file and defaults to the EU868 LoRaWAN band. Please refer
to the `configuration/loraserver/examples` directory for more configuration
examples.

# Data persistence

PostgreSQL and Redis data is persisted in Docker volumes, see the `docker-compose.yml`
`volumes` definition.

## Requirements

Before using this `docker-compose.yml` file, make sure you have [Docker](https://www.docker.com/community-edition)
installed.

## Usage

To start all the LoRa Server components, simply run:

```bash
$ docker-compose up
```

**Note:** during the startup of services, it is normal to see the following errors:

* ping database error, will retry in 2s: dial tcp 172.20.0.4:5432: connect: connection refused
* ping database error, will retry in 2s: pq: the database system is starting up


After all the components have been initialized and started, you should be able
to open http://localhost:8080/ in your browser.

### Add network-server

When adding the network-server in the LoRa App Server web-interface,
you must enter `loraserver:8000` as the network-server `hostname:IP`.


### Add gateway-profile
- Enter Name, Enabled Channels (0,1,2,3,4,5,6,7,8) 
- Select Network Server from drop down

### Add service-profile
- Enter Service-profile Name, Device-status request frequency, Minimum allowed data-rate, Maximum allowed data-rate. Consult the documentation of Gateway Router
- Select Network Server from drop down

### Add gateway
- Enter Gateway Name, Gateway Description, Gateway Id (Consult with Gateway manufacturer)
- Select Network Server from drop down
- Select Gateway Profile from drop down

### Add device-profile
A device-profile defines the device capabilities and boot parameters
that are needed by the network-server for setting the LoRaWAN radio
access service. These information elements shall be provided by the
end-device manufacturer.

When creating a device-profile, LoRa App Server will create the actual
profile on the selected network-server, and will keep a reference record
so it knows to which organization it belongs.

**Note:** changes made to some of the fields will require the reactivation
of the device.

## Payload codecs

**Note:** the raw `base64` encoded payload will always be available, even when
a codec has been configured.

### Cayenne LPP

When selecting the Cayenne LPP codec, LoRa App Server will decode and encode
following the [Cayenne Low Power Payload](https://mydevices.com/cayenne/docs/lora/)
specification.

### Custom JavaScript codec functions

When selecting the Custom JavaScript codec functions option, you can write your
own (JavaScript) functions to decode an array of bytes to a JavaScript object
and encode a JavaScript object to an array of bytes. Package [otto](https://github.com/robertkrimen/otto), 
which targets ES5, is used as a JavaScript interpreter, so ES6 features (e.g. Typed Arrays) are not supported.

#### Decoder function skeleton

{{<highlight js>}}
// Decode decodes an array of bytes into an object.
//  - fPort contains the LoRaWAN fPort number
//  - bytes is an array of bytes, e.g. [225, 230, 255, 0]
// The function must return an object, e.g. {"temperature": 22.5}
function Decode(fPort, bytes) {
  return {};
}
{{< /highlight >}}

#### Encoder function skeleton

{{<highlight js>}}
// Encode encodes the given object into an array of bytes.
//  - fPort contains the LoRaWAN fPort number
//  - obj is an object, e.g. {"temperature": 22.5}
// The function must return an array of bytes, e.g. [225, 230, 255, 0]
function Encode(fPort, obj) {
  return [];
}
{{< /highlight >}}

## Fields / options

The following fields are described by the
[LoRaWAN Backend Interfaces specification](https://www.lora-alliance.org/lorawan-for-developers).
Fields marked with an **X** are implemented by LoRa (App) Server.

- [X] **SupportsClassB** End-Device supports Class B
- [X] **ClassBTimeout** Maximum delay for the End-Device to answer a MAC request or a confirmed DL frame (mandatory if class B mode supported)
- [X] **PingSlotPeriod** Mandatory if class B mode supported
- [X] **PingSlotDR** Mandatory if class B mode supported
- [X] **PingSlotFreq** Mandatory if class B mode supported
- [X] **SupportsClassC** End-Device supports Class C
- [X] **ClassCTimeout** Maximum delay for the End-Device to answer a MAC request or a confirmed DL frame (mandatory if class C mode supported)
- [X] **MACVersion** Version of the LoRaWAN supported by the End-Device
- [X] **RegParamsRevision** Revision of the Regional Parameters document supported by the End-Device
- [X] **SupportsJoin** End-Device supports Join (OTAA) or not (ABP)
- [X] **RXDelay1** Class A RX1 delay (mandatory for ABP)
- [X] **RXDROffset1** RX1 data rate offset (mandatory for ABP)
- [X] **RXDataRate2** RX2 data rate (mandatory for ABP)
- [X] **RXFreq2** RX2 channel frequency (mandatory for ABP)
- [X] **FactoryPresetFreqs** List of factory-preset frequencies (mandatory for ABP)
- [X] **MaxEIRP** Maximum EIRP supported by the End-Device
- [ ] **MaxDutyCycle** Maximum duty cycle supported by the End-Device
- [X] **RFRegion** RF region name (automatically set by LoRa Server)
- [ ] **Supports32bitFCnt** End-Device uses 32bit FCnt (mandatory for LoRaWAN 1.0 End-Device) (always set to `true`)



### References:
- [AN-101D surface-mounted geomagnetic parking sensor](https://drive.google.com/drive/folders/1ekKc2jb-nUbZp3KpUDWwjUzPk5H_6ADi)
- [Gateway Manual](https://drive.google.com/drive/folders/1F3StzJmoC8_WsGZHhBpujz9ZJexIz7er)
