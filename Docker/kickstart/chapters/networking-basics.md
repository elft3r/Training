# Docker Networking Basics

In this lab you'll explore Docker's networking model and the CLI commands used to manage container networks.

You will complete the following steps as part of this lab.

- [Task 1 - The `docker network` command](#docker_network)
- [Task 2 - List networks](#list_networks)
- [Task 3 - Inspect a network](#inspect)
- [Task 4 - Understand network drivers](#drivers)

## Prerequisites

You will need all of the following to complete this lab:

- A Docker host running Docker Engine 20.10 or higher

## <a name="docker_network"></a>Task 1: The `docker network` command

The `docker network` command is the main command for configuring and managing container networks.

Run `docker network --help` to see the available sub-commands.

```
$ docker network --help

Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks
```

The key operations are: **create** and **rm** to manage networks, **connect** and **disconnect** to attach containers, and **inspect** and **ls** to view details.

## <a name="list_networks"></a>Task 2: List networks

Run `docker network ls` to view existing container networks on your Docker host.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1befe23acd58        bridge              bridge              local
726ead8f4e6b        host                host                local
ef4896538cc7        none                null                local
```

Every Docker installation comes with these three default networks:

| Network | Driver | Purpose |
|---------|--------|---------|
| **bridge** | bridge | Default network for containers. Provides isolated networking on a single host via a Linux bridge (virtual switch). |
| **host** | host | Removes network isolation — the container shares the host's network stack directly. |
| **none** | null | No networking. The container has a loopback interface only. |

Each network has a unique `ID` and `NAME`, and is associated with a single driver. Notice that the "bridge" and "host" networks share the same name as their respective drivers.

## <a name="inspect"></a>Task 3: Inspect a network

Use `docker network inspect` to view configuration details of a network. These details include the name, ID, driver, subnet info, connected containers, and more.

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "1befe23acd58...",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Driver": "default",
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.name": "docker0"
        },
        ...
    }
]
```

Key things to notice:
- The **Subnet** (`172.17.0.0/16`) defines the IP range available to containers
- The **Gateway** (`172.17.0.1`) is the IP of the Linux bridge (`docker0`) on the host
- **enable_ip_masquerade** means containers can reach the internet through NAT
- **enable_icc** (inter-container communication) allows containers on this network to talk to each other

> **NOTE:** The command syntax is `docker network inspect <network>`, where `<network>` can be either the network name or ID.

## <a name="drivers"></a>Task 4: Understand network drivers

Docker uses a pluggable networking architecture. The built-in drivers handle most use cases:

| Driver | Scope | Use Case |
|--------|-------|----------|
| **bridge** | Local | Containers on a single host that need to communicate. The most common driver. |
| **host** | Local | When you need maximum network performance and don't need isolation (container shares host networking). |
| **overlay** | Swarm | Multi-host networking for Docker Swarm services. |
| **macvlan** | Local | When containers need to appear as physical devices on the network (each gets its own MAC address). |
| **none** | Local | Disable networking entirely. |

You can see which drivers are available with `docker info`.

```
$ docker info --format '{{.Plugins.Network}}'
[bridge host ipvlan macvlan null overlay]
```

For the kickstart workshop, we'll focus on the **bridge** driver since it's the most commonly used and the foundation for understanding Docker networking.

## Next Steps
For the next step in the tutorial, head over to [Bridge Networking](./bridge-network.md)
