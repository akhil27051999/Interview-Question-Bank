## Configuring Docker Bridge Network

### Step 1: Create a custom Docker bridge network named my_bridge_network.
```sh
bash-5.1$ docker network create --driver bridge my_bridge_network
dc75b836fc70f758979ce580c02f5574d442594bbf863f85dcfd9fc0a0b9d422
```
### Step 2: Run a new container called web_container using the nginx image and connect it to the my_bridge_network.
```sh
bash-5.1$ docker run --name web_container --network my_bridge_network -d nginx
e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f
```
### Step 3: Verify that the container is connected to the bridge network by inspecting the network and container.
```sh
bash-5.1$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
e87c9e07f8e6   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp    web_container

bash-5.1$ docker network inspect my_bridge_network
[
    {
        "Name": "my_bridge_network",
        "Id": "dc75b836fc70f758979ce580c02f5574d442594bbf863f85dcfd9fc0a0b9d422",
        "Created": "2026-01-05T16:36:12.898053584Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f": {
                "Name": "web_container",
                "EndpointID": "dc9f27e91d73a2f84c53fde7e30a8852ce587d0dd00c9abbf34b710b592b8243",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```
### Step 4: Inspect the Container
```sh
bash-5.1$ docker inspect web_container
[
    {
        "Id": "e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f",
        "Created": "2026-01-05T16:36:52.873873187Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 982,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2026-01-05T16:36:53.33867287Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:058f4935d1cbc026f046e4c7f6ef3b1d778170ac61f293709a2fc89b1cff7009",
        "ResolvConfPath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/hostname",
        "HostsPath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/hosts",
        "LogPath": "/var/lib/docker/containers/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f/e87c9e07f8e6eae4feaf2ae9812939edd248b99173aa11704bdcdc6c94579a4f-json.log",
        "Name": "/web_container",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "my_bridge_network",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "private",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": null,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4-init/diff:/var/lib/docker/overlay2/02159621de3c6c8a3fcd77ff42a48e56a87209cae476c57e1e9ecfc3300146ae/diff:/var/lib/docker/overlay2/ecb6f730fa8f8106242397eb6487a6dea1f7a5948ed01f9ccc5f61b8698c2982/diff:/var/lib/docker/overlay2/c6608b3e3b25aa909e3305ca9b0877276aa5295a173d0e1fbab0a351097f283a/diff:/var/lib/docker/overlay2/a2743dd3ebf090fe411a9578385a8a070aefa809be8b0aa50a3a6e83ea28fd5f/diff:/var/lib/docker/overlay2/f3bfd8cb288922c3c02045e77ff7034136fa1eb20c8c71e054b35932b4413677/diff:/var/lib/docker/overlay2/c5b3404a57412fa3108389e91591af61f8e13aeea5c9ffe4a2a5e744db7832b8/diff:/var/lib/docker/overlay2/ec6ea0cffbdd68140e6b702d09012a92b00e816b7f1742e971187e32a66556fd/diff",
                "MergedDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4/merged",
                "UpperDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4/diff",
                "WorkDir": "/var/lib/docker/overlay2/b29ad46ba89795f0e99496c30c5c3f03b96449a7f2178c0c6146c9209e6f39b4/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "e87c9e07f8e6",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.29.4",
                "NJS_VERSION=0.9.4",
                "NJS_RELEASE=1~trixie",
                "PKG_RELEASE=1~trixie",
                "DYNPKG_RELEASE=1~trixie"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "nginx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "13177d4fbeb0904255b0e25e30ea5d35f5a262838cef561202259f51f29710d3",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/13177d4fbeb0",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "my_bridge_network": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "e87c9e07f8e6"
                    ],
                    "NetworkID": "dc75b836fc70f758979ce580c02f5574d442594bbf863f85dcfd9fc0a0b9d422",
                    "EndpointID": "dc9f27e91d73a2f84c53fde7e30a8852ce587d0dd00c9abbf34b710b592b8243",
                    "Gateway": "172.18.0.1",
                    "IPAddress": "172.18.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:12:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```
