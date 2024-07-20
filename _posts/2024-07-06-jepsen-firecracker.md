---
layout: post
title:  "Using Firecracker VMs to run Jepsen test"
date:   2024-07-06 09:00:00 +0800
categories: firecracker jepsen distributed systems database
---

[Jepsen test](https://github.com/jepsen-io/jepsen) requires us to have a running cluster for the DB under test. The DB nodes can be on VMs, bare-metal, on the cloud, as long as we have network, ssh and sudo/root access.  **Why not run it with Firecracker?**

This post journals my journey working through the [Jepsen tutorial](https://github.com/jepsen-io/jepsen/blob/main/doc/tutorial/index.md) with running etcd (the DB under test) on Firecracker VMs.

> Sidenote: I'm not sure how containers are supported because Jepsen uses `iptables` to simulate network issues. It seems like [LXC](https://github.com/jepsen-io/jepsen?tab=readme-ov-file#lxc) is supported while Docker is not. Could be worth digging into this next time.

### Firecracker control plane

I created a [simple Firecracker API server](https://github.com/k-jingyang/firecracker-cp) in 2022. This API server allows you to create multiple Firecracker VMs via a HTTP API where:

1. Networking is configured (i.e. the host machine can reach the VM, while the VM can go to the internet).
2. Each VM is passed an SSH public key on creation, so we can ssh into the VM.
3. The rootfs image is made into a squashfs (read only) and overlayed with tmpfs (in-mem fs), so that the same rootfs image can be reused to boot other VMs.

This makes it very easy for me to spin up disposable VMs via a `curl` call

```bash
INSTANCES=1
for i in $(seq $INSTANCES); do curl -X POST localhost:3000/vm -d @- <<EOF
{ "sshPubKey" : "$(cat ~/.ssh/id_rsa.pub)" }
EOF
done
```

### The lack of sudo

The rootfs image provided by the Firecracker team is Ubuntu, while the Jepsen tutorial uses Debian. Luckily, Jepsen test provides Ubuntu support.

```clojure
(ns jepsen.etcdemo
  (:require [jepsen.os.ubuntu :as ubuntu]))

(defn etcd-test
  "Test etcd"
  [opts]
  (let [quorum (boolean (:quorum opts))]
    (merge tests/noop-test opts {:name "etcd"
                                 :os ubuntu/os})))
````

However, when running the Jepsen test, we face an error.

```bash
$ lein run test --time-limit 30  --node FIRECRACKER_VM_IP --ssh-private-key ~/.ssh/id_rsa --password "" --username root --test-count 1 --concurrency 5

STDIN:
null

STDOUT:


STDERR:
bash: line 1: sudo: command not found
```

So, it seems like our rootfs image does not contain `sudo`. Even though we're logged in as root, Jepsen uses sudo to setup the VMs. It's much easier to provide support for sudo than to change Jepsen's underlying implementation.

### Building my own rootfs

I found out that we can build our rootfs easily using Docker images. A docker image is a rootfs after all.

- <https://github.com/firecracker-microvm/firecracker/blob/main/docs/rootfs-and-kernel-setup.md#creating-a-rootfs-image>
- <https://jvns.ca/blog/2021/01/23/firecracker--start-a-vm-in-less-than-a-second/>

```dockerfile
FROM ubuntu:22.04

# install necessary packages
RUN apt update
RUN apt install -y sudo \
    openssh-server \
    init 
```

Hence, I installed the necessary packages on-top of the official Ubuntu image.

- `sudo` because Jepsen requires this
- `openssh-server` because the official image does not have SSH installed
- `init` because Docker images doesn't come with an init, while Firecracker VMs requires it

Docker containers generally do not need `init`. However, there are some [use cases](https://github.com/krallin/tini/issues/8) for it. The nuances of this deserves a separate post.

After building the docker image, I then pushed it to DockerHub.

```bash
docker build -t kjingyang/ubuntu:22.04 .
docker push kjingyang/ubuntu:22.04
```

I then updated firecracker-cp to accept a docker image as an input. It uses the [rootfs_builder](https://github.com/ForAllSecure/rootfs_builder) library to pull from DockerHub to create the rootfs image.

```bash
INSTANCES=1
for i in $(seq $INSTANCES); do curl -X POST localhost:3000/vm -d @- <<EOF  
{"sshPubKey" : "$(cat ~/.ssh/id_rsa.pub)", "dockerImage" : "docker.io/kjingyang/ubuntu:22.04" }
EOF
done
```

Using the new instances, run `lein run test` again and we will not face the `bash: line 1: sudo: command not found` error.

### etcd timeouts and bridge networking

After fixing the `sudo` issue, we encounter a different issue, our etcd commands always times out.

```
INFO [2024-07-06 13:24:40,086] jepsen worker 0 - jepsen.util 0  :invoke :read   [0 nil]
INFO [2024-07-06 13:24:40,098] jepsen worker 2 - jepsen.util 2  :invoke :read   [0 nil]
INFO [2024-07-06 13:24:40,103] jepsen worker 3 - jepsen.util 3  :invoke :write  [0 4]
INFO [2024-07-06 13:24:40,113] jepsen worker 4 - jepsen.util 4  :invoke :read   [0 nil]
INFO [2024-07-06 13:24:40,118] jepsen worker 1 - jepsen.util 1  :invoke :write  [0 1]
INFO [2024-07-06 13:24:45,115] jepsen worker 3 - jepsen.util 3  :info   :write  [0 4]   :timeout
INFO [2024-07-06 13:24:45,115] jepsen worker 2 - jepsen.util 2  :fail   :read   [0 nil] :timeout
INFO [2024-07-06 13:24:45,115] jepsen worker 2 - jepsen.util 2  :invoke :write  [0 1]
INFO [2024-07-06 13:24:45,115] jepsen worker 0 - jepsen.util 0  :fail   :read   [0 nil] :timeout
INFO [2024-07-06 13:24:45,119] jepsen worker 4 - jepsen.util 4  :fail   :read   [0 nil] :timeout
```

Checking the etcd logs shows that the etcd nodes could not communicate with each other.

```
2024-07-06 05:19:59.382962 I | raft: 7466dc0c8072fbc4 [logterm: 1, index: 3] sent MsgVote request to ee298f474ec4f928 at term 4
2024-07-06 05:19:59.382985 I | raft: 7466dc0c8072fbc4 [logterm: 1, index: 3] sent MsgVote request to 3e2e81306ce9d286 at term 4
2024-07-06 05:20:01.091372 W | rafthttp: health check for peer 3e2e81306ce9d286 could not connect: <nil>
2024-07-06 05:20:01.096462 W | rafthttp: health check for peer ee298f474ec4f928 could not connect: dial tcp 172.16.0.144:2380: getsockopt: no route to host
2024-07-06 05:20:01.182534 I | raft: 7466dc0c8072fbc4 is starting a new election at term 4
2024-07-06 05:20:01.182676 I | raft: 7466dc0c8072fbc4 became candidate at term 5
2024-07-06 05:20:01.182767 I | raft: 7466dc0c8072fbc4 received MsgVoteResp from 7466dc0c8072fbc4 at term 5
2024-07-06 05:20:01.182793 I | raft: 7466dc0c8072fbc4 [logterm: 1, index: 3] sent MsgVote request to 3e2e81306ce9d286 at term 5
2024-07-06 05:20:01.182814 I | raft: 7466dc0c8072fbc4 [logterm: 1, index: 3] sent MsgVote request to ee298f474ec4f928 at term 5
```

This is because my CNI config was set to `ptp`, which only allow the VM <-> Host communication and not VM <-> VM.

```json
{
  "name": "fcnet",
  "cniVersion": "0.4.0",
  "plugins": [
    {
      "type": "ptp",
      "ipMasq": true,
      "ipam": {
        "type": "host-local",
        "subnet": "172.16.0.0/24",
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

Configuring my CNI as described in my [previous post](2024-06-15-firecracker-bridge.md) resolves this issue.

```json
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
        "subnet": "172.16.0.0/24",
        "resolvConf": "/home/jingyang/Workspace/firecracker-cp/resolv.conf"
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

### iptables issue and building my own linux kernel

After introducing the nemesis into the test, I faced an iptables error.

```bash
STDIN:
null

STDOUT:


STDERR:
sudo: unable to resolve host localhost.localdomain: Name or service not known
iptables/1.8.7 Failed to initialize nft: Protocol not supported
```

There are not much resources on this issue. Based on this [tweet](https://x.com/iximiuz/status/1610401406002814976), it seems like nftables is not supported on the kernel distributed by the Firecracker team. I found an [opennebula forum post](https://forum.opennebula.io/t/enable-nftables-in-firecracker-default-5-10-kernel-config/11985) that solves this issue by building their own Linux kernel and enabling supporting for nftables.

- Not sure if I messed up the config given in the forum post but it didn't work for me

I then followed the instructions in [Firecracker docs](https://github.com/firecracker-microvm/firecracker/blob/main/docs/rootfs-and-kernel-setup.md#manual-compilation) to compile the kernel with my custom configuration using `make menuconfig`.

- I selected almost all options that relate to Netfilter, so it may not be the smallest workable kernel.
- <a href="/assets/files/jepsen-firecracker-kernel.txt" target="_blank">This</a> was the config file I ended up with

#### Linux kernel y or m

After reading this [stackoverflow post](https://stackoverflow.com/questions/5392756/what-does-m-mean-in-kernel-configuration-file), I selected y instead of m to include Netfilter functionalities into the kernel directly.

I believe this is simpler so that we don't have to run any additional commands after booting up the VM and before we run the Jepsen test.

### Finally

Now, if I were to specify the Firecracker API server to use my custom kernel image,

```golang
import fc "github.com/firecracker-microvm/firecracker-go-sdk"

func main() {
    config := fc.Config{
      SocketPath:      path.Join(socketDir, sockName),
      LogPath:         stdout.Name(),
      LogLevel:        "Info",
      KernelImagePath: "vmlinux-6.1", // my custom built kernel
      KernelArgs:      "console=ttyS0 reboot=k panic=1 pci=off overlay_root=ram ssh_disk=/dev/vdb init=/sbin/overlay-init",
      }
    ctx := context.Background()
    uVM, _ := fc.NewMachine(ctx, config)
    _ := uVM.Start(ctx)
}
```

and then run `lein test` after spinning up a few VMs, we can now run the Jepsen test!

```
INFO [2024-07-14 10:28:16,721] jepsen worker 4 - jepsen.util 14 :ok     :read   [11 3]
INFO [2024-07-14 10:28:16,733] jepsen worker nemesis - jepsen.util :nemesis     :info   :start  [:isolated {"172.16.0.157" #{"172.16.0.156" "172.16.0.158"}, "172.16.0.156" #{"172.16.0.157"}, "172.16.0.158" #{"172.16.0.157"}}]
INFO [2024-07-14 10:28:16,754] jepsen worker 0 - jepsen.util 5  :invoke :write  [11 4]
INFO [2024-07-14 10:28:16,756] jepsen worker 0 - jepsen.util 5  :ok     :write  [11 4]
INFO [2024-07-14 10:28:16,767] jepsen worker 3 - jepsen.util 3  :info   :cas    [9 [1 3]]       :timeout
INFO [2024-07-14 10:28:16,791] jepsen worker 1 - jepsen.util 11 :invoke :read   [11 nil]
INFO [2024-07-14 10:28:16,792] jepsen worker 1 - jepsen.util 11 :ok     :read   [11 3]
INFO [2024-07-14 10:28:16,828] jepsen worker 2 - jepsen.util 7  :invoke :write  [11 4]
INFO [2024-07-14 10:28:16,830] jepsen worker 2 - jepsen.util 7  :ok     :write  [11 4]
INFO [2024-07-14 10:28:16,850] jepsen worker 3 - jepsen.util 8  :invoke :write  [11 0]
```

### Conclusion

It has been fun getting the Jepsen tutorial to work with Firecracker VMs! I learnt quite a fair bit through this process:

- What Jepsen test is, and how it works
- We can build custom rootfs from docker images
- Docker images don't usually come with `init`
- We need to use the bridge CNI for inter-VM communication
- Building the Linux kernel
