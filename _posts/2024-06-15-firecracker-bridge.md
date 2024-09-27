---
layout: post
title:  "Using bridge networking with Firecracker Go SDK"
date:   2024-06-15 09:00:00 +0800
categories: firecracker
excerpt: ""
---

> tl;dr: You can follow this guide in [firecracker-containerd](https://github.com/firecracker-microvm/firecracker-containerd/blob/main/docs/getting-started.md#cni-setup) to configure bridge networking with CNI

I was trying to work through the [Jepsen tutorial](https://github.com/jepsen-io/jepsen/tree/main/doc/tutorial) with Firecracker uVMs, and this required spinning up an etcd cluster on 5 uVMs. 
Inter-uVM connectivity was necessary because the etcd nodes has to communicate with each other.

[Firecracker SDK](https://github.com/firecracker-microvm/firecracker-go-sdk) integrates nicely with CNI to provide networking capabilities to the VMs and provides a simple API.

```go
import fc "github.com/firecracker-microvm/firecracker-go-sdk"

func main() {
    config := fc.Config{
        NetworkInterfaces: fc.NetworkInterfaces{
            fc.NetworkInterface{
                // finds the CNI configuration in in /etc/cni/conf.d by default
                CNIConfiguration: &fc.CNIConfiguration{ 
                    NetworkName: "fcnet", // this name must match the name in the CNI config file  
                },
                AllowMMDS: true,
            },
        },
    }

    ctx := context.Background()
    uVM, _ := fc.NewMachine(ctx, config)
  _ := uVM.Start(ctx)
}
```

### A little on CNI

CNI provides a specification for **plugins** to provide networking functionalities to containers.

From the [CNI website](https://www.cni.dev/docs/spec/#summary), the CNI specification defines:

- A format for administrators to define network configuration.
- A protocol for container runtimes to make requests to network plugins.
- A procedure for executing plugins based on a supplied configuration.
- A procedure for plugins to delegate functionality to other plugins.
- Data types for plugins to return their results to the runtime.

### Using the bridge CNI

This was how I configured my CNI config file `etc/cni/conf.d/fcnet.conflist`. Note that the name **fcnet** matches the `NetworkName` specified with the sdk.

```text
{
  "name": "fcnet",
  "cniVersion": "0.4.0",
  "plugins": [
    {
      "type": "bridge",
      "ipMasq": true,
      "bridge":"mynet0",
      "isDefaultGateway": true,
      "ipam": {
        "type": "host-local",
        "subnet": "172.16.0.0/24"
      }
    },
    {
      "type": "firewall"
    },
    {
      "type": "tc-redirect-tap"
    }
  ]
}
```

`isDefaultGateway` must be configured true, otherwise the bridge interface will only operate at layer 2 and will not be assigned an IP address. This means that you will not be able to reach your uVMs via IP addresses.

- `isGateway` works too

`isDefaultGateway` supposedly also configures the virtual network to use that bridge interface as the default gateway. However, it seems like the interface is used as the default gateway whether I use `isGateway` or `isDefaultGateway`

*isDefaultGateway*

```bash
root@localhost:~# ip r
default via 172.16.0.1 dev eth0 
172.16.0.0/24 dev eth0 proto kernel scope link src 172.16.0.117 
```

*isGateway*

```bash
root@localhost:~# ip r
default via 172.16.0.1 dev eth0 
172.16.0.0/24 dev eth0 proto kernel scope link src 172.16.0.116 
```

### A simple documentation PR

For the benefit of subsequent newbies, it'll be nice to have a working example of bridge networking in the README of the Firecracker Go SDK.

<https://github.com/firecracker-microvm/firecracker-go-sdk/pull/575>
