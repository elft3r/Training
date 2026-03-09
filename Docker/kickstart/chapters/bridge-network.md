# Bridge Networking

In this lab you'll learn how to build, manage, and use **bridge** networks — the most common networking type for containers running on a single Docker host.

You will complete the following steps as part of this lab.

- [Task 1 - Explore the default **bridge** network](#task_1)
- [Task 2 - Run containers on the default bridge](#task_2)
- [Task 3 - Create a user-defined bridge network](#task_3)
- [Task 4 - Test DNS-based service discovery](#task_4)
- [Task 5 - Network isolation between bridges](#task_5)
- [Task 6 - Configure port mapping for external access](#task_6)
- [Task 7 - Cleanup](#task_7)

## Prerequisites

You will need all of the following to complete this lab:

- A Docker host running Docker Engine 20.10 or higher
- Basic familiarity with the `docker` CLI

## <a name="task_1"></a>Task 1: Explore the default bridge network

Every Docker installation comes with three pre-built networks. List them with `docker network ls`.

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1befe23acd58        bridge              bridge              local
726ead8f4e6b        host                host                local
ef4896538cc7        none                null                local
```

The **bridge** network uses the *bridge* driver and is scoped locally (single-host only). It's important to note that the network name and the driver name are both "bridge", but they are not the same thing — one is a specific network, the other is the driver (template) used to create it.

Under the hood, the *bridge* driver creates a Linux bridge (virtual switch) called **docker0** on the host. You can see it with the `ip` command.

```
$ ip addr show docker0
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN
    link/ether 02:42:f1:7f:89:a6 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
```

The **docker0** bridge acts as the gateway (`172.17.0.1`) for all containers connected to the default **bridge** network.

Inspect the **bridge** network to see its configuration.

```
$ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "1befe23acd58...",
        "Scope": "local",
        "Driver": "bridge",
        "IPAM": {
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Containers": {},
        ...
    }
]
```

## <a name="task_2"></a>Task 2: Run containers on the default bridge

The **bridge** network is the default for new containers. If you don't specify `--network`, your container lands here.

Start two Alpine containers.

```
$ docker run -dit --name c1 alpine sh
$ docker run -dit --name c2 alpine sh
```

> **NOTE:** We use Alpine because it includes `ping` out of the box — no need to install extra packages.

Verify both containers are connected to the **bridge** network.

```
$ docker network inspect bridge --format '{{range .Containers}}{{.Name}}: {{.IPv4Address}}{{"\n"}}{{end}}'
c1: 172.17.0.2/16
c2: 172.17.0.3/16
```

Test connectivity **by IP address** between the two containers.

```
$ docker exec c1 ping -c 3 172.17.0.3
PING 172.17.0.3 (172.17.0.3): 56 data bytes
64 bytes from 172.17.0.3: seq=0 ttl=64 time=0.112 ms
64 bytes from 172.17.0.3: seq=1 ttl=64 time=0.098 ms
64 bytes from 172.17.0.3: seq=2 ttl=64 time=0.102 ms
```

Containers on the default bridge can reach each other by IP. Now try reaching the other container **by name**.

```
$ docker exec c1 ping -c 3 c2
ping: bad address 'c2'
```

**This fails.** The default bridge network does **not** provide DNS-based service discovery. Containers can only communicate using IP addresses, which is fragile and impractical. This is one of the key reasons Docker recommends using **user-defined bridge networks** instead.

Also verify that containers can reach the internet.

```
$ docker exec c1 ping -c 3 docker.com
PING docker.com (141.193.213.20): 56 data bytes
64 bytes from 141.193.213.20: seq=0 ttl=37 time=15.2 ms
...
```

Clean up the default bridge containers.

```
$ docker rm -f c1 c2
```

## <a name="task_3"></a>Task 3: Create a user-defined bridge network

User-defined bridge networks are the **recommended** approach for container networking. They provide:
- **Automatic DNS resolution** between containers (by name)
- **Better isolation** from other containers
- **Configurable subnets** and IP ranges

Create a user-defined bridge network.

```
$ docker network create my_bridge
a1b2c3d4e5f6...
```

Inspect the new network.

```
$ docker network inspect my_bridge --format '{{(index .IPAM.Config 0).Subnet}}'
172.18.0.0/16
```

Docker automatically assigned a subnet. You can also specify one explicitly.

```
$ docker network create \
  --subnet=10.0.0.0/24 \
  --gateway=10.0.0.1 \
  custom_bridge
```

Verify both networks exist.

```
$ docker network ls --filter driver=bridge
NETWORK ID          NAME                DRIVER              SCOPE
1befe23acd58        bridge              bridge              local
a1b2c3d4e5f6        my_bridge           bridge              local
d7e8f9a0b1c2        custom_bridge       bridge              local
```

Remove the `custom_bridge` network since we won't use it further.

```
$ docker network rm custom_bridge
```

## <a name="task_4"></a>Task 4: Test DNS-based service discovery

Now start two containers on the **user-defined** `my_bridge` network.

```
$ docker run -dit --name web --network my_bridge nginx:alpine
$ docker run -dit --name client --network my_bridge alpine sh
```

Test connectivity **by container name**.

```
$ docker exec client ping -c 3 web
PING web (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.089 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.105 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.100 ms
```

**It works!** Docker's built-in DNS server automatically resolves container names to their IP addresses on user-defined networks. This is how real-world microservices discover each other — by name, not by IP.

You can even use `wget` to reach the NGINX web server by name.

```
$ docker exec client wget -qO- http://web
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

## <a name="task_5"></a>Task 5: Network isolation between bridges

Containers on **different** bridge networks are isolated from each other by default. This is a fundamental Docker security feature.

Start a container on the default **bridge** network.

```
$ docker run -dit --name isolated alpine sh
```

Try to ping the `web` container (which is on `my_bridge`) from `isolated` (which is on the default `bridge`).

```
$ docker exec isolated ping -c 3 172.18.0.2
PING 172.18.0.2 (172.18.0.2): 56 data bytes

--- 172.18.0.2 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
```

**No connectivity.** The two bridge networks are completely isolated from each other.

### Connecting a container to multiple networks

You can attach a container to additional networks using `docker network connect`.

```
$ docker network connect my_bridge isolated
```

Now `isolated` is connected to **both** the default `bridge` and `my_bridge`. Verify.

```
$ docker exec isolated ping -c 3 web
PING web (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.110 ms
...
```

It can now reach containers on `my_bridge` by name. To remove the connection:

```
$ docker network disconnect my_bridge isolated
```

Clean up the isolated container.

```
$ docker rm -f isolated
```

## <a name="task_6"></a>Task 6: Configure port mapping for external access

Containers on a bridge network are not accessible from outside the Docker host by default. To expose a container's service externally, use **port mapping** with the `-p` flag.

The `web` container from Task 4 is already running NGINX on port 80, but it's only reachable from within the `my_bridge` network. Let's fix that.

Stop and re-create the `web` container with port mapping.

```
$ docker rm -f web
$ docker run -d --name web --network my_bridge -p 8080:80 nginx:alpine
```

This maps port **8080** on the Docker host to port **80** inside the container.

Verify the port mapping.

```
$ docker ps --filter name=web
CONTAINER ID   IMAGE          COMMAND                  PORTS                  NAMES
b747d43fa277   nginx:alpine   "/docker-entrypoint.…"   0.0.0.0:8080->80/tcp   web
```

The `0.0.0.0:8080->80/tcp` mapping means port 8080 on **all host interfaces** forwards to port 80 in the container.

Test external access using `curl` from the Docker host.

```
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

You can also access it from a web browser by navigating to `http://<your-docker-host-ip>:8080`.

> **NOTE:** Port mapping uses port address translation (PAT) under the hood. Outbound traffic from containers is masqueraded/SNATed so it appears to come from the host's IP address.

## <a name="task_7"></a>Task 7: Cleanup

Remove all containers and the custom network.

```
$ docker rm -f web client
$ docker network rm my_bridge
```

Verify cleanup.

```
$ docker ps -a --filter name=web --filter name=client
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES

$ docker network ls --filter driver=bridge
NETWORK ID          NAME                DRIVER              SCOPE
1befe23acd58        bridge              bridge              local
```

## Key Takeaways

| | Default Bridge | User-Defined Bridge |
|---|---|---|
| **DNS resolution** | No — containers must use IP addresses | Yes — containers can reach each other by name |
| **Isolation** | All containers share the same default network | Containers are isolated by network |
| **Recommended** | No — use only for quick tests | Yes — use for all real workloads |
| **Configuration** | Cannot be customized | Custom subnets, IP ranges, and gateways |
| **Connect/disconnect** | Cannot disconnect running containers from primary (default) network | Live connect/disconnect with `docker network connect/disconnect` |

**Best practices:**
- Always create user-defined bridge networks for your applications
- Use container names for service discovery instead of hard-coded IPs
- Use port mapping (`-p`) only for services that need external access
- Use separate networks to isolate groups of containers (e.g., frontend vs backend)

## Next Steps
For the next step in the tutorial, head over to [Docker Secrets](./secrets.md)
