docker1.12有关network的特性增加

特性
Built-in Virtual-IP based internal and ingress load-balancing using IPVS
Routing Mesh using ingress overlay network
Secured multi-host overlay networking using encrypted control-plane and Data-plane
MacVlan driver is out of experimental
Add driver filter to network ls
Adding network filter to docker ps –filter
Add –link-local-ip flag to create, run and network connect to specify a container’s link-local address
Add network label filter support
Removed dependency on external KV-Store for Overlay networking in Swarm-Mode
Add container’s short-id as default network alias
run options –dns and –net=host are no longer mutually exclusive
Fix DNS issue when renaming containers with generated names
Allow both network inspect -f {{.Id}} and network inspect -f {{.ID}} to address inconsistency with inspect output

比如关于：Add driver filter to network ls。现在可以使用driver作为过滤条件来确认network的情况

[root@host31 ~]# docker network ls --filter driver=bridge
NETWORK ID          NAME                DRIVER              SCOPE
e2836311817e        bridge              bridge              local
[root@host31 ~]#


docker network的种类

在刚刚安装完docker之后，下面三个network是被自动地创建出来的。
network种类
none
host
bridge
[root@host31 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e2836311817e        bridge              bridge              local
58211460fd1f        host                host                local
a157ec9146b7        none                null                local
[root@host31 ~]#


初期状态

使用network inspect命令可以看到以上三种network最初的状态。
docker network inspect none

[root@host31 ~]# docker network inspect none
[
    {
        "Name": "none",
        "Id": "a157ec9146b720cb38981fa1a22390b60c78fcd4396a1d50d979427f480799d6",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
[root@host31 ~]#

docker network inspect host

[root@host31 ~]# docker network inspect host
[
    {
        "Name": "host",
        "Id": "58211460fd1f3da1bbc392a43ddd2b79a8bec663620b7783cefcf910940ddcd9",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
[root@host31 ~]#


docker network inspect bridge


[root@host31 ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "e2836311817eabd7b2d28e3bbc2ae5e7a545a8652446d52ca77cd55fa7ba50d1",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
[root@host31 ~]#


创建一个container加入none

用一下命令可以创建一个centos的container将其加入none的network中。
docker run -it --name container_none --network=none centos /bin/bash
1
[root@host31 ~]# docker run -it --name container_none --network=none centos /bin/bash
[root@0dfd0712c5ca /]#
1
2
另外打开一个终端，让我们来看看发生了什么
[root@host31 tmp]# docker network inspect none
[
    {
        "Name": "none",
        "Id": "a157ec9146b720cb38981fa1a22390b60c78fcd4396a1d50d979427f480799d6",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": {
            "0dfd0712c5cab81f3328a39aa5f57723c957915b67d5bc235fb514120bd03f56": {
                "Name": "container_none",
                "EndpointID": "a7b8a817f1cf42fa3566eb0327b337d2352f0f8efa5ceec4d10f96b69e13ffc4",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
[root@host31 tmp]#


加入了none的network的container，我们可以从上面通过他的Name等发现就是刚刚穿件的container_none，下面我们来看一下这个container中有哪些特点。
[root@0dfd0712c5ca /]# ping www.baidu.com
ping: unknown host www.baidu.com
[root@0dfd0712c5ca /]#
[root@0dfd0712c5ca /]# ping 192.168.32.31
connect: Network is unreachable
[root@0dfd0712c5ca /]#
[root@0dfd0712c5ca /]# ping localhost
PING localhost (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.338 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.082 ms
^C
--- localhost ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.082/0.210/0.338/0.128 ms
[root@0dfd0712c5ca /]#


除了自己谁都连接不通，是它的特点。
创建一个container加入host

用一下命令可以创建一个centos的container将其加入host的network中。
docker run -it --name container_host --network=host centos /bin/bash


[root@host31 ~]# docker run -it --name container_host --network=host centos /bin/bash
[root@host31 /]#


怎么回事，不是-i方式启动的麽，另外怎么目录变了呢。另外打开一个终端，让我们来看看发生了什么
[root@host31 tmp]# docker ps |grep container_host
43b4f08151e2        centos              "/bin/bash"         7 minutes ago       Up 6 minutes                            container_host
[root@host31 tmp]#
这个就是host的方式的container，上面提示的[root@host31 /]已经不是在宿主机，而是在container_host中了，我们可以简单的确认一下，比如至少用centos官方最新镜像启动的container中是不可能有我们安装的docker1.12的。
[root@host31 /]# hostname
host31
[root@host31 /]# docker info
bash: docker: command not found
[root@host31 /]#
虽然你的hostname跟宿主机一样，但是我们都知道那是你的马甲了。通过下面的inspect也能看到其已经加入host网络中了。
[root@host31 tmp]# docker network inspect host
[
    {
        "Name": "host",
        "Id": "58211460fd1f3da1bbc392a43ddd2b79a8bec663620b7783cefcf910940ddcd9",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": {
            "43b4f08151e2da050e26aa62b27f68229cd112a963a35e5fcb7b6ed47e0e7f11": {
                "Name": "container_host",
                "EndpointID": "365f5858203d3d5162edf7350fa1094174df29f60d9978f90aa975068f93db74",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
[root@host31 tmp]#


加入了host的network的container，我们可以从上面通过他的Name等发现就是刚刚穿件的container_host，下面我们来看一下这个container中有哪些特点。
[root@host31 /]# ping -w1 www.baidu.com
PING www.a.shifen.com (14.215.177.38) 56(84) bytes of data.
64 bytes from 14.215.177.38: icmp_seq=1 ttl=128 time=61.2 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 61.262/61.262/61.262/0.000 ms
[root@host31 /]#
[root@host31 /]# ping container_host
ping: unknown host container_host
[root@host31 /]#



跟none不同,它不再是只能自己跟自己通信. 它和外部是连通的。
创建一个container加入bridge

用一下命令可以创建一个centos的container将其加入bridge的network中。因为缺省不指定就是这种方式, 我们平时没有意识到network的存在,其实是使用的bridge的方式
docker run -it --name container_bridge centos /bin/bash

[root@host31 ~]# docker run -it –name container_bridge centos /bin/bash 
[root@743d5689399a /]#


另外打开一个终端，让我们来看看发生了什么
[root@host31 tmp]# docker ps |grep container_bridge
743d5689399a        centos              "/bin/bash"         42 seconds ago      Up 41 seconds                           container_bridge
[root@host31 tmp]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "e2836311817eabd7b2d28e3bbc2ae5e7a545a8652446d52ca77cd55fa7ba50d1",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "743d5689399aab527f83a0708763970bc671801ff377ac791f9aee2b58de4b34": {
                "Name": "container_bridge",
                "EndpointID": "c0ad0de740ed65b7c6e8e63fc34e42e807c9d82822341bef8f474dcca8fc4272",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
[root@host31 tmp]#

加入了bridge的network的container，我们可以从上面通过他的Name等发现就是刚刚创建的container_bridge，下面我们来看一下这个container中有哪些特点。
[root@743d5689399a /]# ping -w1 www.baidu.com
PING www.a.shifen.com (103.235.46.39) 56(84) bytes of data.
64 bytes from 103.235.46.39: icmp_seq=1 ttl=127 time=255 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 255.315/255.315/255.315/0.000 ms

[root@743d5689399a /]#
[root@743d5689399a /]# ping container_bridge
ping: unknown host container_bridge
[root@743d5689399a /]#



none模式下的多个container

启动两个container加入 none中
[root@host31 ~]# docker run -it --network=none centos /bin/bash
[root@a2a37d0ddc0b /]#



[root@host31 ~]# docker run -it --network=none centos /bin/bash
[root@b8b7f66f1c12 /]#



启动后container的确认
[root@host31 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b8b7f66f1c12        centos              "/bin/bash"         41 seconds ago      Up 41 seconds                           determined_visvesvaraya
a2a37d0ddc0b        centos              "/bin/bash"         54 seconds ago      Up 53 seconds                           jovial_goldstine
[root@host31 ~]#



docker network inspect none确认详细
[root@host31 ~]# docker network inspect none
[
    {
        "Name": "none",
        "Id": "a157ec9146b720cb38981fa1a22390b60c78fcd4396a1d50d979427f480799d6",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": {
            "a2a37d0ddc0b5915422ce5ec5948ef715acfe84110abed11100373c4fc89c37a": {
                "Name": "jovial_goldstine",
                "EndpointID": "f0acf4f20f924a5fe5d6754fbfb61aa458e87500f02a564736c197fb7207f43d",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            },
            "b8b7f66f1c12354fe784092697c93531d2966a0dee27ce301a558ba1dfd5b975": {
                "Name": "determined_visvesvaraya",
                "EndpointID": "91e7e7e1b1a5bfe7188be69baae53b01c181978094b2f19010b949fc6ebd1527",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
[root@host31 ~]#



从inspect的结果可以清楚地看到IPv4Address根本都没有被设定，基本应该只有lo的127.0.0.1，所以none的模式下多个container即使在同一个network中也无法相互之间通信。
host模式下的多个container

启动两个container加入 host中，host模式下，容器与主机共享网络Namespace，拥有与主机相同的网络设备。
[root@host31 ~]# docker run -it --network=host --name host_container1 centos /bin/bash
[root@host31 /]#



[root@host31 ~]# docker run -it --network=host --name host_container2 centos /bin/bash
[root@host31 /]#



启动后container的确认
[root@host31 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
91223e0ce984        centos              "/bin/bash"         39 seconds ago      Up 38 seconds                           host_container2
f7d361caf164        centos              "/bin/bash"         52 seconds ago      Up 52 seconds                           host_container1
[root@host31 ~]#



docker network inspect host确认详细
[root@host31 ~]# docker network inspect host
[
    {
        "Name": "host",
        "Id": "58211460fd1f3da1bbc392a43ddd2b79a8bec663620b7783cefcf910940ddcd9",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
        },
        "Internal": false,
        "Containers": {
            "91223e0ce984105d5dfa24260148bc0dc9254c7211d0ca6e76faf54cd9d3c42f": {
                "Name": "host_container2",
                "EndpointID": "bc0adbea4290527df3fe23d353a45ba324567de143d8b57f2dc195046cdfeee1",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            },
            "f7d361caf164f67a3a5670dd9f2b826ccd2054c17989bf101a81b78480a69101": {
                "Name": "host_container1",
                "EndpointID": "06e60004d1e0672fdf14dc775c10b5a2459867e7a1b760f338b769bd32811436",
                "MacAddress": "",
                "IPv4Address": "",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
[root@host31 ~]#



确认2个container都能连接外网但是无法使用container名称ping通。
[root@host31 ~]# docker run -it --network=host --name host_container1 centos /bin/bash
[root@host31 /]# ping host_container1
ping: unknown host host_container1
[root@host31 /]# ping host_container2
ping: unknown host host_container2
[root@host31 /]# ping -w1 www.baidu.com
PING www.a.shifen.com (14.215.177.37) 56(84) bytes of data.
64 bytes from 14.215.177.37: icmp_seq=1 ttl=128 time=66.3 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 66.304/66.304/66.304/0.000 ms
[root@host31 /]#



[root@host31 ~]# docker run -it --network=host --name host_container2 centos /bin/bash
[root@host31 /]# ping host_container2
ping: unknown host host_container2
[root@host31 /]# ping host_container1
ping: unknown host host_container1
[root@host31 /]# ping -w1 www.baidu.com
PING www.a.shifen.com (61.135.169.121) 56(84) bytes of data.
64 bytes from 61.135.169.121: icmp_seq=1 ttl=128 time=22.2 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 22.245/22.245/22.245/0.000 ms
[root@host31 /]#



bridge模式下的多个container

启动两个container加入 bridge中
[root@host31 ~]# docker run -it --network=bridge --name bridge_container1 centos /bin/bash
[root@5a55638c0ac2 /]#


[root@host31 ~]# docker run -it --network=bridge --name bridge_container2 centos /bin/bash
[root@8e4621500007 /]#


启动后container的确认
[root@host31 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
8e4621500007        centos              "/bin/bash"         51 seconds ago       Up 50 seconds                           bridge_container2
5a55638c0ac2        centos              "/bin/bash"         About a minute ago   Up About a minute                       bridge_container1
[root@host31 ~]#



docker network inspect bridge确认详细
[root@host31 ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "5da066475a0d5d0e54ba9e6f64f80342eb188d0455b85a69b50bf222f13ce9a3",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "5a55638c0ac2971e204668294ba3bb2d83487ef94f5af506f83bef773d4fab64": {
                "Name": "bridge_container1",
                "EndpointID": "63ee42bae5309639d1a512d8e4b7b81f3b8c83f454ca076bd5d8513ec637c88e",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "8e4621500007b357d69c417b5f7025258bb4efc3f0c3076c3cff8d9ebb351a14": {
                "Name": "bridge_container2",
                "EndpointID": "9f824c05d490e95c84a21684bd627a1005a1464dcb74b8647e2ad31be126cafc",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
[root@host31 ~]#



通过确认，bridge_container1和bridge_container2被赋予了如下IP
container名	IP
bridge_container1	172.17.0.2
bridge_container2	172.17.0.3
确认除了使用container名之外，使用IP是可以相互之前ping通的
[root@host31 ~]# docker run -it --network=bridge --name bridge_container1 centos /bin/bash
[root@5a55638c0ac2 /]# ping bridge_container1
ping: unknown host bridge_container1
[root@5a55638c0ac2 /]# ping bridge_container2
ping: unknown host bridge_container2
[root@5a55638c0ac2 /]# ping -w1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.071 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.074 ms

--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.071/0.072/0.074/0.008 ms
[root@5a55638c0ac2 /]# ping -w1 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.178 ms

--- 172.17.0.3 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.178/0.178/0.178/0.000 ms
[root@5a55638c0ac2 /]# ping -w1 www.baidu.com
PING www.a.shifen.com (14.215.177.38) 56(84) bytes of data.
64 bytes from 14.215.177.38: icmp_seq=1 ttl=127 time=134 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 134.332/134.332/134.332/0.000 ms
[root@5a55638c0ac2 /]#



[root@host31 ~]# docker run -it --network=bridge --name bridge_container2 centos /bin/bash
[root@8e4621500007 /]# ping bridge_container1
ping: unknown host bridge_container1
[root@8e4621500007 /]# ping bridge_container2
ping: unknown host bridge_container2
[root@8e4621500007 /]# ping -w1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.121 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.121/0.121/0.121/0.000 ms
[root@8e4621500007 /]# ping -w1 172.17.0.3
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.072 ms

--- 172.17.0.3 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.072/0.072/0.072/0.000 ms
[root@8e4621500007 /]# ping -w1 www.baidu.com
PING www.a.shifen.com (14.215.177.38) 56(84) bytes of data.

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

[root@8e4621500007 /]#


用户自定义网络

用户自定义网络目前可以创建三种：bridge/overlay/macvlan。本文主要介绍使用bridge方式创建用户自定义网络，这也是使用较为广泛的一种。将会创建一个由三个container组成的bridge的自定义网络，构成如下图所示。
这里写图片描述

创建自定义网络

命令:docker network create –driver bridge isolated_nw
[root@host31 ~]# docker network create --driver bridge isolated_nw
8c3cb606081806f7c23d26773a2acc30cc56b574fe0a2e720661e23781541a74
[root@host31 ~]# 


确认结果
[root@host31 ~]# docker network ls |grep nw
8c3cb6060818        isolated_nw         bridge              local
[root@host31 ~]#
[root@host31 ~]# docker network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "8c3cb606081806f7c23d26773a2acc30cc56b574fe0a2e720661e23781541a74",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
[root@host31 ~]#


生成三个container加入自定义的bridge网络

[root@host31 ~]# docker run -it --network=isolated_nw --name container1 centos /bin/bash
[root@ce52f7298e33 /]#


[root@host31 ~]# docker run -it --network=isolated_nw --name container2 centos /bin/bash
[root@7230fe3506da /]#


[root@host31 ~]# docker run -it --network=isolated_nw --name container3 centos /bin/bash
[root@03f787b1b49b /]#


连接后确认

[root@host31 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
03f787b1b49b        centos              "/bin/bash"         About a minute ago   Up About a minute                       container3
7230fe3506da        centos              "/bin/bash"         About a minute ago   Up About a minute                       container2
ce52f7298e33        centos              "/bin/bash"         About a minute ago   Up About a minute                       container1
8e4621500007        centos              "/bin/bash"         20 minutes ago       Up 20 minutes                           bridge_container2
5a55638c0ac2        centos              "/bin/bash"         21 minutes ago       Up 21 minutes                           bridge_container1
[root@host31 ~]#


docker network inspect

[root@host31 ~]# docker network inspect isolated_nw
[
    {
        "Name": "isolated_nw",
        "Id": "8c3cb606081806f7c23d26773a2acc30cc56b574fe0a2e720661e23781541a74",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1/16"
                }
            ]
        },
        "Internal": false,
        "Containers": {
            "03f787b1b49b7d2ee1078780a195d56e7248e895a1bc200905fc38410e953221": {
                "Name": "container3",
                "EndpointID": "a1700f703e7d27032f8d2f9a2bc93b161c0625a7d5e7c942e08fcd01b68a5a52",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "7230fe3506daad2dd6a2d1e90a37924386da7fa4f099882919397f846de1ea43": {
                "Name": "container2",
                "EndpointID": "fcb31859fa26d112f21b4679647187b4a78978b840f55561faa671868fac63ec",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            },
            "ce52f7298e33b75c8e772466b9b4807a6d63ef46c3a1f33c689ef58d704fa95a": {
                "Name": "container1",
                "EndpointID": "aa3c24e26ed9c84628fa213f8575223085880430c9abfd8760fa581e5efe8fdf",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
[root@host31 ~]#


通过确认，container1-3被赋予了如下IP
container名	IP
container1	172.18.0.2
container2	172.18.0.3
container3	172.18.0.4
IP的连通确认: IP和container名都可以直接使用，default的bridge应该是因为服务发现被disable了所以无法使用。另外此network和default的bridge network的container之间也是不通的。所以network就是为了实现隔离而做出的功能。
[root@03f787b1b49b /]# ping -w1 container1
PING container1 (172.18.0.2) 56(84) bytes of data.
64 bytes from container1.isolated_nw (172.18.0.2): icmp_seq=1 ttl=64 time=0.167 ms

--- container1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.167/0.167/0.167/0.000 ms
[root@03f787b1b49b /]# ping -w2 container2
PING container2 (172.18.0.3) 56(84) bytes of data.
64 bytes from container2.isolated_nw (172.18.0.3): icmp_seq=1 ttl=64 time=0.175 ms
64 bytes from container2.isolated_nw (172.18.0.3): icmp_seq=2 ttl=64 time=0.134 ms

--- container2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1007ms
rtt min/avg/max/mdev = 0.134/0.154/0.175/0.023 ms
[root@03f787b1b49b /]# ping -w1 container3
PING container3 (172.18.0.4) 56(84) bytes of data.
64 bytes from 03f787b1b49b (172.18.0.4): icmp_seq=1 ttl=64 time=0.186 ms

--- container3 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.186/0.186/0.186/0.000 ms
[root@03f787b1b49b /]# ping -w1 172.18.0.2
PING 172.18.0.2 (172.18.0.2) 56(84) bytes of data.
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.124 ms

--- 172.18.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.124/0.124/0.124/0.000 ms
[root@03f787b1b49b /]# ping -w1 www.baidu.com
PING www.a.shifen.com (14.215.177.38) 56(84) bytes of data.
64 bytes from 14.215.177.38: icmp_seq=1 ttl=127 time=160 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 160.208/160.208/160.208/0.000 ms
[root@03f787b1b49b /]# ping -w1 bridge_container2
ping: unknown host bridge_container2
[root@03f787b1b49b /]# ping -w1 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

[root@03f787b1b49b /]#


上述的方式对于一般但节点container个数一般的普通场景可以处理，但是对于集群方式下的网络处理更多则是使用overlay的方式

