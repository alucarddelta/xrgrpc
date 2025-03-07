# gRPC library for Cisco IOS XR

[![GoDoc](https://godoc.org/github.com/nleiva/xrgrpc?status.svg)](https://godoc.org/github.com/nleiva/xrgrpc) 
[![Test](https://github.com/nleiva/xrgrpc/actions/workflows/test.yml/badge.svg)](https://github.com/nleiva/xrgrpc/actions/workflows/test.yml)
[![codecov](https://codecov.io/gh/nleiva/xrgrpc/branch/master/graph/badge.svg)](https://codecov.io/gh/nleiva/xrgrpc) 
[![Go Report Card](https://goreportcard.com/badge/github.com/nleiva/xrgrpc)](https://goreportcard.com/report/github.com/nleiva/xrgrpc)
[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/nleiva/xrgrpc)

Minimalistic library to interact with IOS XR devices using the gRPC framework. Look at the [IOS XR proto file](proto/ems_grpc.proto) for the description of the service interface and the structure of the payload messages. gRPC uses protocol buffers as the Interface Definition Language (IDL).

**Tutorials**: 

- [Programming IOS-XR with gRPC and Go](https://xrdocs.github.io/programmability/tutorials/2017-08-04-programming-ios-xr-with-grpc-and-go/).
- [Validate the intent of network config changes](https://xrdocs.github.io/programmability/tutorials/2017-08-14-validate-the-intent-of-network-config-changes/).

**Other Examples**:

- [A collection of OpenConfig and Cisco IOS XR examples](https://github.com/nleiva/xroc).
- [Parsing Telemetry data from IOS XR YANG models](https://github.com/nleiva/nettable).

The end goal is to enable use-cases where multiple interactions with devices are required. gRPC arises as a strong option to single interface network elements to retrieve info from the devices, apply configurations to it, generate telemetry streams from them, programming the RIB/FIB and so on. The following is a very simple config-validate example:

![oc-config-validate](https://github.com/nleiva/xrgrpc/blob/gh-pages/static/images/oc-config-validateH.gif)

## Table of Contents

- [gRPC library for Cisco IOS XR](#grpc-library-for-cisco-ios-xr)
  - [Table of Contents](#table-of-contents)
  - [Prerequisite Tools](#prerequisite-tools)
  - [Usage](#usage)
    - [Get Config (example/getconfig)](#get-config-examplegetconfig)
    - [Get Operation](#get-operation)
    - [Show Commands](#show-commands)
      - [Clear text](#clear-text)
      - [JSON](#json)
    - [Configuring the router](#configuring-the-router)
      - [CLI config (Merge)](#cli-config-merge)
      - [JSON (Merge)](#json-merge)
      - [JSON (Replace)](#json-replace)
      - [Using a YANG config Template (Merge)](#using-a-yang-config-template-merge)
    - [Removing router config](#removing-router-config)
      - [JSON](#json-1)
    - [CLI config multiple routers simultaneously (Merge)](#cli-config-multiple-routers-simultaneously-merge)
    - [Telemetry](#telemetry)
      - [JSON (GPBKV)](#json-gpbkv)
      - [JSON (GPBKV): Exploring the fields](#json-gpbkv-exploring-the-fields)
      - [JSON (GPBKV): OpenConfig](#json-gpbkv-openconfig)
      - [GPB (Protobuf)](#gpb-protobuf)
    - [Config and Validate](#config-and-validate)
    - [Actions](#actions)
      - [Ping](#ping)
      - [Traceroute](#traceroute)
      - [Log Generation](#log-generation)
      - [Crypto Key Generation](#crypto-key-generation)
    - [Bypass the config file](#bypass-the-config-file)
  - [XR gRPC Config](#xr-grpc-config)
    - [Port range](#port-range)
  - [Certificate file](#certificate-file)
    - [Self-signed certificate for package testing](#self-signed-certificate-for-package-testing)
  - [Generating Go binding from protobuf files](#generating-go-binding-from-protobuf-files)
  - [Running the examples](#running-the-examples)
    - [Links](#links)

## Prerequisite Tools

* [Go (at least Go 1.17)](https://golang.org/dl/)

## Usage

CLI examples to use the library are provided in the [example](example/) folder. The CLI specified in the examples is not definitive and might change as we go.

### Get Config (example/getconfig)

Retrieves the config from one target device described in [config.json](example/input/config.json), for the YANG paths specified in [yangpaths.json](example/input/yangpaths.json). If you want to see it using [OpenConfig models](https://github.com/openconfig/public/tree/master/release/models), you can issue `./getconfig -ypath "../input/yangocpaths.json"` instead.

- `example/getconfig`

```console
example/getconfig$  go run main.go 

config from sandbox-iosxr-1.cisco.com:57777
 {
 "data": {
  "openconfig-interfaces:interfaces": {
   "interface": [
    {
     "name": "BVI511",
     "config": {
      "name": "BVI511",
      "type": "iana-if-type:propVirtual"
     },
     "subinterfaces": {
      "subinterface": [
       {
        "index": 0,
        "openconfig-if-ip:ipv4": {
         "addresses": {
          "address": [
           {
            "ip": "10.200.188.33",
            "config": {
             "ip": "10.200.188.33",
             "prefix-length": 24
            }
           }
          ]
         }
        }
       }
      ]
     }
    },
    {

...

2022/05/11 16:55:36 This process took 715.136564ms
```

### Get Operation

Provides the output of IOS XR operations data (eg. optic light levels) from YANG/JSON formatted request, from the list in [config.json](example/input/config.json). It reads the target from [yangoper.json](example/input/yangoper.json).

-  example/getoper
``` console
$ ./getoper

<snip>
      "optics-db-info": {
      "transport-admin-state": "tas-ui-is",
      "controller-state": "optics-state-up"
     }
    }
   ]
  }
 }
}

2023/05/12 10:08:27 This process took 4.465176769s
```
### Show Commands

Provides the output of IOS XR cli commands for one router defined in [config.json](example/input/config.json). Two output format options are available; Unstructured text and JSON encoded:

#### Clear text

- example/showcmd

```console
$ ./showcmd -cli "show isis database" -enc text

Output from [2001:420:2cff:1204::5502:1]:57344
 
----------------------------- show isis database ------------------------------

IS-IS BB2 (Level-2) Link State Database
LSPID                 LSP Seq Num  LSP Checksum  LSP Holdtime  ATT/P/OL
mrstn-5502-1.cisco.com.00-00* 0x0000000c   0x1558        3066            0/0/0
mrstn-5502-2.cisco.com.00-00  0x00000012   0x6e0c        3066            0/0/0
mrstn-5501-1.cisco.com.00-00  0x0000000c   0x65d5        1150            0/0/0

 Total Level-2 LSP count: 3     Local Level-2 LSP count: 1


2017/07/21 15:37:00 This process took 2.480039252s
```

#### JSON

- example/showcmd

```console
$ ./showcmd -cli "show isis database" -enc json

Config from [2001:420:2cff:1204::5502:1]:57344
 [{
 "Cisco-IOS-XR-clns-isis-oper:isis": {
<snip>
       {
        "system-id": "0151.0250.0002",
        "local-is-flag": false,
        "host-levels": "isis-levels-2",
        "host-name": "mrstn-5502-2.cisco.com"
       },
       {
        "system-id": "0151.0250.0003",
        "local-is-flag": false,
        "host-levels": "isis-levels-2",
        "host-name": "mrstn-5501-1.cisco.com"
       },
       {
        "system-id": "0151.0250.0001",
        "local-is-flag": true,
        "host-levels": "isis-levels-2",
        "host-name": "mrstn-5502-1.cisco.com"
...
2017/07/21 15:37:27 This process took 1.54038192s
```

### Configuring the router

#### CLI config (Merge)

Applies CLI config commands on the device/router from the list in [config.json](example/input/config.json).

- example/setconfig

```console
$ ./setconfig -cli "interface Lo11 ipv6 address 2001:db8::/128"

Config applied to [2001:420:2cff:1204::5502:1]:57344

2017/07/21 15:24:17 This process took 1.779449886s
```

You can verify the config on the router:

```
RP/0/RP0/CPU0:mrstn-5502-1.cisco.com#show run interface lo11
Fri Jul 21 15:24:24.199 EDT
interface Loopback11
 ipv6 address 2001:db8::/128
!
```

#### JSON (Merge)

Applies a YANG/JSON formatted config to one device/router (merges with existing config) from the list in [config.json](example/input/config.json). It reads the target from [yangconfig.json](example/input/yangconfig.json). 

- example/mergeconfig

```console
$ ./mergeconfig 

Config merged on [2001:420:2cff:1204::5502:1]:57344 -> Request ID: 8162, Response ID: 8162

2017/07/21 15:18:07 This process took 1.531427437s
```

You can verify the config on the router:

```
RP/0/RP0/CPU0:mrstn-5502-1.cisco.com#show run interface lo201
Fri Jul 21 15:18:24.046 EDT
interface Loopback201
 description New Loopback 201
 ipv6 address 2001:db8:20::1/128
!
```

#### JSON (Replace)

Applies a YANG/JSON formatted config to one device/router (replaces the config for this section) from the list in [config.json](example/input/config.json). It learns the config to replace from [yangconfigrep.json](example/input/yangconfigrep.json). If we had merged instead, we would have ended up with two IPv6 addresses in this example.

- example/replaceconfig

```console
$ ./replaceconfig 

Config replaced on [2001:420:2cff:1204::5502:1]:57344 -> Request ID: 4616, Response ID: 4616

2017/07/21 15:21:27 This process took 1.623047025s
```

You can verify the config on the router:

```
RP/0/RP0/CPU0:mrstn-5502-1.cisco.com#show run interface lo201
Fri Jul 21 15:21:48.053 EDT
interface Loopback201
 description New Loopback 221
 ipv6 address 2001:db8:22::2/128
!
```

#### Using a YANG config Template (Merge)

Applies a YANG/JSON formatted config to one device/router (merges with existing config) from the list in [config.json](example/input/config.json). It takes a template ([bgp.json](example/input/template/bgp.json)), based on the BGP YANG model [Cisco-IOS-XR-ipv4-bgp-cfg](https://github.com/YangModels/yang/blob/master/vendor/cisco/xr/622/Cisco-IOS-XR-ipv4-bgp-cfg.yang), in this case and the specific parameters from [bgp-parameters.json](example/input/template/bgp-parameters.json).

See below an extract from this [bgp.json](example/input/template/bgp.json) and notice NeighborAddress, PeerASN, Description and LocalAddress are variables to be defined.

```shell
"neighbor": [
 {
  "neighbor-address": "{{.NeighborAddress}}",
  "remote-as": {
   "as-xx": {{.PeerASN.X}},
   "as-yy": {{.PeerASN.Y}}
  },
  "description": "{{.Description}}",
  "update-source-interface": "{{.LocalAddress}}",
  "neighbor-afs": {
   "neighbor-af": [
	{
	 "af-name": "ipv6-unicast",
	 "activate": [
	  null
	 ]
	}
   ]
  }
 }
] 
```

Now we execute and inmediately request the updated BGP config from the device with a subsequent RPC call.

- example/mergetemplate

```console
$ ./mergetemplate 

Config merged on [2001:420:2cff:1204::5502:1]:57344 -> Request ID: 1866, Response ID: 1866


Config from [2001:420:2cff:1204::5502:1]:57344
 {
 "Cisco-IOS-XR-ipv4-bgp-cfg:bgp": {
  "instance": [
<snip>
         "bgp-entity": {
          "neighbors": {
           "neighbor": [
            {
             "neighbor-address": "2001:db8:1::1",
             "remote-as": {
              "as-xx": 0,
              "as-yy": 65535
             },
             "description": "Test",
             "update-source-interface": "Loopback60",
             "neighbor-afs": {
              "neighbor-af": [
<snip>

2017/08/07 18:52:57 This process took 907.395197ms
```

Go includes the [template](https://golang.org/pkg/html/template/) package in its standard library to generate data-driven textual outputs. 
- Give templates and YANG a try in [The Go Playground](https://play.golang.org/p/xRsTkVfCTG).

While templates are cool, I'd recommend exploring one of these alternatives to handle YANG models programmatically.
- [YDK](https://developer.cisco.com/site/ydk/) that takes YANG models as input and produces APIs that mirror the structure of the models.
- [goyang](https://github.com/openconfig/goyang) which is a YANG parser and compiler to produce Go language objects.

### Removing router config

#### JSON

Removes YANG/JSON formatted config on one device/router from [config.json](example/input/config.json). It reads the config to delete from [yangdelconfig.json](example/input/yangdelconfig.json). The following example deletes both interfaces configured in the Merge example. See [yangdelintadd.json](example/input/yangdelintadd.json) to delete just the IP address and [yangdelintdesc.json](example/input/yangdelintdesc.json) for only the description of the interface.

- example/deleteconfig

```console
$ ./deleteconfig 

Config Deleted on [2001:420:2cff:1204::5502:1]:57344 -> Request ID: 2856, Response ID: 2856

2017/07/21 15:06:46 This process took 730.329288ms
```

On the router:

```
RP/0/RP0/CPU0:mrstn-5502-1.cisco.com#show configuration commit changes 1000000039
Mon Jul 17 15:54:59.221 EDT
Building configuration...
!! IOS XR Configuration version = 6.2.2.22I
no interface Loopback201
no interface Loopback301
end
```

### CLI config multiple routers simultaneously (Merge)

Applies CLI config commands to the list of routers specified on [config.json](example/input/config.json). Notice that even though we added two devices, the execution time did NOT increase. This is possible because of the use of [Golang Concurrency](https://blog.golang.org/pipelines) primitives.

- example/setconfiglist

```console
$ ./setconfiglist -cli "interface Lo33 ipv6 address 2001:db8:33::1/128"

Config applied to [2001:420:2cff:1204::5502:2]:57344



Config applied to [2001:420:2cff:1204::5501:1]:57344



Config applied to [2001:420:2cff:1204::5502:1]:57344


2017/07/21 15:32:11 This process took 1.773893901s
```

You can verify the config on the routers:

```
RP/0/RP0/CPU0:mrstn-5501-1.cisco.com#sh run int Lo33
Fri Jul 21 15:32:35.468 EDT
interface Loopback33
 ipv6 address 2001:db8:33::1/128
!
```

```
RP/0/RP0/CPU0:mrstn-5502-1.cisco.com#sh run int Lo33
Fri Jul 21 15:33:07.281 EDT
interface Loopback33
 ipv6 address 2001:db8:33::1/128
!
```

```
RP/0/RP0/CPU0:mrstn-5502-2.cisco.com#sh run int Lo33
Fri Jul 21 15:33:14.504 EDT
interface Loopback33
 ipv6 address 2001:db8:33::1/128
!
```


### Telemetry

#### JSON (GPBKV)

Subscribe to a Telemetry stream. The Telemetry message is defined in [telemetry.proto](proto/telemetry/telemetry.proto). The payload is JSON encoded (self-describing GPB).

- example/telemetry

```console
$ ./telemetry -subs "LLDP"
Time 1500666991103, Path: Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors/details/detail
{
  "NodeId": {
    "NodeIdStr": "mrstn-5502-1.cisco.com"
  },
  "Subscription": {
    "SubscriptionIdStr": "LLDP"
  },
  "encoding_path": "Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors/details/detail",
  "collection_id": 1,
  "collection_start_time": 1500666991103,
  "msg_timestamp": 1500666991103,
  "data_gpbkv": [
    {
      "timestamp": 1500666991108,
      "ValueByType": null,
      "fields": [
...
```

The Subscription ID has to exist on the device <sup>[1](#myfootnote1)</sup>.

```
telemetry model-driven
 sensor-group LLDPNeighbor
  sensor-path Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors/details/detail
 !
 subscription LLDP
  sensor-group-id LLDPNeighbor sample-interval 15000
 !
!
```

#### JSON (GPBKV): Exploring the fields

Same as the previous example using a Cisco native YANG model. However this time we explore the fields in order to produce a custom output.

```go
func exploreFields(f []*telemetry.TelemetryField, indent string) {
	for _, field := range f {
		switch field.GetFields() {
		case nil:
			decodeKV(field, indent)
		default:
			exploreFields(field.GetFields(), indent+" ")
		}
	}
}
```

The result looks like this:

- example/telemetrykv

```console
$ ./telemetrykv
******************************************************************************************
Time 01:24:48PM, Path: Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors/details/detail
******************************************************************************************
  node-name: 0/RP0/CPU0
  interface-name: HundredGigE0/0/0/1
  device-id: mrstn-5502-1.cisco.com
   receiving-interface-name: HundredGigE0/0/0/1
   receiving-parent-interface-name: <No interface>
   device-id: mrstn-5502-1.cisco.com
   chassis-id: 008a.9646.6cd8
   port-id-detail: Hu0/0/0/1
   header-version: 0
   hold-time: 15
   enabled-capabilities: R
   platform:
    port-description: TO calient_fiber_switch, port 001 in/out
    system-name: mrstn-5502-1.cisco.com
    system-description:  6.2.2.22I, NCS-5500
<snip>    
```

#### JSON (GPBKV): OpenConfig

Same example as before, just calling a subscription that uses an OpenConfig model instead. The result looks like this:

- example/telemetrykv

```console
$ ./telemetrykv -subs "BGP-OC"
******************************************************************************************
Time 01:08:03PM, Path: openconfig-bgp:bgp/neighbors/neighbor/state
******************************************************************************************
  instance-name: default
  neighbor-address: 2001:db8:cafe::2
  speaker-id: 0
  description: iBGP session
  local-as: 64512
  remote-as: 64512
  has-internal-link: true
  is-external-neighbor-not-directly-connected: false
  messages-received: 16
  messages-sent: 16
  update-messages-in: 1
  update-messages-out: 1
  messages-queued-in: 0
  messages-queued-out: 0
  connection-established-time: 822
  connection-state: bgp-st-estab
  previous-connection-state: 2
  connection-admin-status: 0
  open-check-error-code: none
   afi: ipv6
    value: 2001:db8:cafe::1
  is-local-address-configured: false
<snip>    
```

The Subscription ID has to exist on the device <sup>[1](#myfootnote1)</sup>.

```
telemetry model-driven
 sensor-group BGPNeighbor-OC
  sensor-path openconfig-bgp:bgp/neighbors/neighbor/state
 !
 subscription BGP-OC
  sensor-group-id BGPNeighbor-OC sample-interval 10000
 !
!
```

#### GPB (Protobuf)

Again, we subscribe to a Telemetry stream but we request the content is encoded with [protobuf](https://developers.google.com/protocol-buffers/). To decode the message we need to look at the "LLDP neighbor details" definition in [lldp_neighbor.proto](proto/telemetry/lldp/lldp_neighbor.proto). We parse the message and modify the output to illustrate how to access to each field on it.

- example/telemetrygpb

```console
$ ./telemetrygpb -subs "LLDP"
Time 1500667512299, Path: Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors/details/detail
{
  "node_name": "0/RP0/CPU0",
  "interface_name": "HundredGigE0/0/0/22",
  "device_id": "mrstn-5502-2.cisco.com"
}
Type:  6.2.2.22I, NCS-5500, Address value:"2001:558:2::2"  

{
  "node_name": "0/RP0/CPU0",
  "interface_name": "HundredGigE0/0/0/21",
  "device_id": "mrstn-5502-2.cisco.com"
}
Type:  6.2.2.22I, NCS-5500, Address value:"2001:558:2::2"  

{
  "node_name": "0/RP0/CPU0",
  "interface_name": "HundredGigE0/0/0/1",
  "device_id": "mrstn-5502-2.cisco.com"
}
Type:  6.2.2.22I, NCS-5500, Address value:"2001:f00:bb::2"
...
```

To print out the entire message, look at the example in [telemetrygpbfull](example/telemetrygpbfull/main.go). To decode the message we need to look at the "LLDP neighbor details" definition in [lldp_neighbor.proto](proto/telemetry/lldp65x/lldp_neighbor.proto).

- example/telemetrygpbfull

```console
$ go run main.go
Time 1557511431921, Path: Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors/details/detail
Decoded Keys:
{
  "node_name": "0/0/CPU0",
  "interface_name": "HundredGigE0/0/0/0",
  "device_id": "mrstn-5502-1.cisco.com"
}
Decoded JSON Neighbors:
{
  "lldp_neighbor": [
    {
      "receiving_interface_name": "HundredGigE0/0/0/0",
      "device_id": "mrstn-5502-1.cisco.com",
      "chassis_id": "008a.9646.6cdf",
      "port_id_detail": "HundredGigE0/0/0/0",
      "hold_time": 15,
      "enabled_capabilities": "R",
      "detail": {
        "peer_mac_address": "00:8a:96:46:6c:00",
        "port_description": "TRANSPORT: mrstn-5502-2.cisco.com",
        "system_name": "mrstn-5502-1.cisco.com",
        "system_description": " 6.5.3.02I, NCS-5500",
        "time_remaining": 11,
        "system_capabilities": "R",
        "enabled_capabilities": "R",
        "network_addresses": {
        ...
```

The Subscription ID has to exist on the device <sup>[1](#myfootnote1)</sup>.

```
telemetry model-driven
 sensor-group LLDPNeighbor
  sensor-path Cisco-IOS-XR-ethernet-lldp-oper:lldp/nodes/node/neighbors/details/detail
 !
 subscription LLDP
  sensor-group-id LLDPNeighbor sample-interval 15000
 !
!
```
<a name="myfootnote1">[1]</a>: [gNMI](https://github.com/openconfig/reference/blob/master/rpc/gnmi/gnmi.proto) defines a variant where you do not need this config.

### Config and Validate

In order to validate the intended state of the network after a config change, we need to need to look at the associated telemetry data. In this example we will configure a BGP neighbor using a BGP config [template](https://golang.org/pkg/html/template/) based on the [OpenConfig BGP YANG model](https://github.com/openconfig/public/tree/master/release/models/bgp). See below an extract of [oc-bgp.json](example/input/template/oc-bgp.json).

```shell
{ "openconfig-bgp:bgp": {
   "global": {
    "config": {
     "as": {{.LocalAs}}
    }
   },
   "neighbors": {
    "neighbor": [
     {
      "neighbor-address": "{{.NeighborAddress}}",
      "config": {
       "neighbor-address": "{{.NeighborAddress}}",
       "peer-as": {{.PeerAs}},
       "description": "{{.Description}}"
      }
<snip>
```

The example will run a config checklist, composed of three items as a result of independent RPC calls.

1. We obtain a gRPC confirmation that config was received by the target.
2. We make a gRPC request to get the running configuration on the target to validate the change submitted was actually applied.
3. We subscribe to a BGP Neighbor State Telemetry stream to track the status changes.

The output of the example is very basic, but illustrates all these points. Notice we receive BGP status every 5 seconds and the neighbor goes from bgp-st-idle to bgp-st-estab.

- example/configvalidate

```console
$ ./configvalidate 
******************************************************************************************

Config merged on [2001:420:2cff:1204::5502:1]:57344 -> Request ID: 3018, Response ID: 3018

******************************************************************************************

BGP Config from [2001:420:2cff:1204::5502:1]:57344


{
 "openconfig-bgp:bgp": {
  "global": {
   "config": {
    "as": 64512,
    "router-id": "162.151.250.1"
   },
   "afi-safis": {
    "afi-safi": [
     {
      "afi-safi-name": "openconfig-bgp-types:ipv6-unicast",
      "config": {
       "afi-safi-name": "openconfig-bgp-types:ipv6-unicast",
       "enabled": true
      }
     }
    ]
   }
  },
  "neighbors": {
   "neighbor": [
    {
     "neighbor-address": "2001:db8:cafe::2",
     "config": {
      "neighbor-address": "2001:db8:cafe::2",
      "peer-as": 64512,
      "description": "iBGP session"
     },
     "afi-safis": {
      "afi-safi": [
       {
        "afi-safi-name": "openconfig-bgp-types:ipv6-unicast",
        "config": {
         "afi-safi-name": "openconfig-bgp-types:ipv6-unicast",
         "enabled": true
        }
       }
      ]
     }
    }
   ]
  }
 }
}

******************************************************************************************

Telemetry from [2001:420:2cff:1204::5502:1]:57344

------------------------------------- Time 02:39:06AM -------------------------------------
BGP Neighbor; IP: 2001:db8:cafe::2, ASN: 64512, State bgp-st-idle 

------------------------------------- Time 02:39:11AM -------------------------------------
BGP Neighbor; IP: 2001:db8:cafe::2, ASN: 64512, State bgp-st-idle 

------------------------------------- Time 02:39:16AM -------------------------------------
BGP Neighbor; IP: 2001:db8:cafe::2, ASN: 64512, State bgp-st-idle 

------------------------------------- Time 02:39:21AM -------------------------------------
BGP Neighbor; IP: 2001:db8:cafe::2, ASN: 64512, State bgp-st-estab
```

The telemetry subscription config is:

```
telemetry model-driven
 sensor-group BGPNeighbor
  sensor-path Cisco-IOS-XR-ipv4-bgp-oper:bgp/instances/instance/instance-active/default-vrf/afs/af/neighbor-af-table/neighbor
 !
 subscription BGP
  sensor-group-id BGPNeighbor sample-interval 2000
 !
!
```

### Actions

There are multiple actions than can be triggered via gRPC on IOS XR devices running 6.3.1 or later. Below the YANG models supported to the date and some examples using this library.

```console
$ cd ~/yang/vendor/cisco/xr/641
$ ls | grep act.yang
Cisco-IOS-XR-cfgmgr-rollback-act.yang
Cisco-IOS-XR-crypto-act.yang
Cisco-IOS-XR-hwmod-mpa-reload-act.yang
Cisco-IOS-XR-infra-statsd-act.yang
Cisco-IOS-XR-ipv4-bgp-act.yang
Cisco-IOS-XR-ipv4-ospf-act.yang
Cisco-IOS-XR-ipv4-ping-act.yang
Cisco-IOS-XR-ipv4-traceroute-act.yang
...
```

#### Ping

- IPv6 Ping (`example/action` with [ping6.json](example/input/action/ping6.json))

```console
$ ./action -act "../input/action/ping6.json"

output from [2001:420:2cff:1204::7816:1]:57344
 {
 "Cisco-IOS-XR-ping-act:output": {...}
}

2018/05/29 15:08:34 This process took 800.050845ms
```

The extended output looks like this:

```json
 {
 "Cisco-IOS-XR-ping-act:output": {
  "ping-response": {
   "ipv6": {
    "destination": "2001:420:2cff:1204::1",
    "repeat-count": "2",
    "data-size": "1350",
    "timeout": "1",
    "pattern": "abcd",
    "rotate-pattern": false,
    "replies": {
     "reply": [
      {
       "reply-index": "1",
       "result": "!"
      },
      {
       "reply-index": "2",
       "result": "!"
      }
     ]
    },
    "hits": "2",
    "total": "2",
    "success-rate": "100",
    "rtt-min": "2",
    "rtt-avg": "2",
    "rtt-max": "3"
   }
  }
 }
}
```

![ipv6-ping](https://github.com/nleiva/xrgrpc/blob/gh-pages/static/images/ipv6_ping.svg)

#### Traceroute

- Dual-Stack traceroute (`example/action` with [traceroute.json](example/input/action/traceroute.json)). Target is an FQDN ([www.cisco.com](www.cisco.com)).

```console
 ./action -act "../input/action/traceroute.json"

output from [2001:420:2cff:1204::7816:1]:57344
 {
 "Cisco-IOS-XR-traceroute-act:output": {
  "traceroute-response": {
   "ipv6": {
    "destination": "2001:420:1101:1::a",
    "hops": {
     "hop": [
      {
       "hop-index": 1,
       "hop-address": "2001:420:2cff:1204::1",
       "hop-hostname": "mrstn-vlan44.cisco.com",
       "probes": {
        "probe": [
         {
          "probe-index": 0,
          "delta-time": 1
         },
         {
          "probe-index": 1,
          "delta-time": 1
         },
         {
          "probe-index": 2,
          "delta-time": 1
         }
        ]
       }
      },
...

2018/05/29 15:12:50 This process took 15.241451154s
```

#### Log Generation

- Log event generation (`example/action` with [log.json](example/input/action/log.json)).

```console
$ ./action -act "../input/action/log.json"

output from [2001:420:2cff:1204::7816:1]:57344

2018/05/29 15:23:59 This process took 42.585985ms
```

On the router

```console
#
RP/0/RP0/CPU0:May 29 19:23:59.297 UTC: emsd[1096]: %OS-SYSLOG-1-LOG_ALERT : gRPC Generated: Device will be under maintenance for 2 hrs for planned activities
```

![log](https://github.com/nleiva/xrgrpc/blob/gh-pages/static/images/log.svg)

#### Crypto Key Generation

- SSH Crypto key generation (`example/action` with [crypto.json](example/input/action/crypto.json)).

```console
$ ./action -act "../input/action/crypto.json"

output from [2001:420:2cff:1204::7816:1]:57344

2018/05/29 15:26:06 This process took 2.721703931s
```

```console
#
RP/0/RP0/CPU0:May 29 19:26:04.249 UTC: cepki[420]: %SECURITY-CEPKI-6-KEY_INFO : crypto key RSA zeroized, label:test
RP/0/RP0/CPU0:May 29 19:26:06.907 UTC: cepki[420]: %SECURITY-CEPKI-6-KEY_INFO : crypto key RSA generated, label:test, modBits:2048
RP/0/RP0/CPU0:May 29 19:26:06.917 UTC: cepki[420]: %SECURITY-CEPKI-6-INFO : key database updated
```

![crypto](https://github.com/nleiva/xrgrpc/blob/gh-pages/static/images/crypto.svg)

### Bypass the config file

You can manually define the target without the config file [config.json](example/input/config.json), by calling the functional options "WithValue". See the snippet below from [definetarget](example/definetarget/main.go).

```go
// Manually specify target parameters.
router, err := xr.BuildRouter(
  xr.WithUsername("admin"),
  xr.WithPassword("C1sco12345"),
  xr.WithHost("sandbox-iosxr-1.cisco.com:57777"),
  xr.WithTimeout(45),
)
```

## XR gRPC Config

The following is the configuration required on the IOS XR device in order to enable gRPC dial-in with TLS support.

```
tpa
 vrf default
  address-family ipv4
   default-route mgmt
  !
 !
!
grpc
 port 57777
 address-family ipv4
!
```

### Port range

While you can select any not-used port on the device, it's recommended to choose one from the 57344-57999 range.

```
mrstn-5502-1 emsd: [1058]: %MGBL-EMS-4-EMSD_PORT_RANGE : The configured port 56500 is outside of the range of [57344, 57999]. It will consume an additional LPTS entry.
```

## Certificate file

Update 6/24/2019: You no longer need to download the certicate file manually. If you don't specify a file, it will be downloaded automatically as in the [getconfig](example/getconfig/main.go) example.

### Self-signed certificate for package testing

This needs to be renewed once a year.

```console
$ openssl req -new -x509 -nodes -subj '/C=US/CN=localhost' \
              -addext "subjectAltName = DNS:localhost" \
              -newkey rsa:4096 -keyout test/key.pem -out \
              test/cert.pem -days 365
```

## Generating Go binding from protobuf files

The Go generated code in [ems_grpc.pb.go](proto/ems/ems_grpc.pb.go) is the result of the following:

- `proto/ems`

```console
$ protoc --go_out=. --go_opt=paths=source_relative \
    --go-grpc_out=. --go-grpc_opt=paths=source_relative \
    --go_opt=Mproto/ems/ems_grpc.proto=proto/ems \
    --go-grpc_opt=Mproto/ems/ems_grpc.proto=proto/ems \
    proto/ems/ems_grpc.proto
```

- The Go generated code in [bgp_nbr_bag.pb.go](proto/telemetry/bgp/bgp_nbr_bag.pb.go) is the result of the following:

```console
$ protoc --go_out=. \
    --go_opt=Mproto/telemetry/bgp/bgp_nbr_bag.proto=proto/telemetry/bgp \
    proto/telemetry/bgp/bgp_nbr_bag.proto
```

- The Go generated code in [lldp_neighbor.pb.go](proto/telemetry/lldp/lldp_neighbor.pb.go) is the result of the following:

```console
$ protoc --go_out=. \
    --go_opt=Mproto/telemetry/lldp/lldp_neighbor.proto=proto/telemetry/lldp \
    proto/telemetry/lldp/lldp_neighbor.proto
```

## Running the examples

After cloning the repo, go the a folder example and execute `go run main.go`. For example:

```console
$ cd example/configvalidate
$ go run main.go 
```

### Links

- [XR YANG models](https://github.com/YangModels/yang/tree/main/vendor/cisco/xr)
- [XR Proto files](https://github.com/ios-xr/model-driven-telemetry/tree/master/protos)
