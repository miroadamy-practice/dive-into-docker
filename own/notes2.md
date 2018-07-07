# Chapter 6 - Docker in the real world (cont)

See ../src/06-docker-in-the-real-world/09-linking-containers-with-docker-networks

```
➜  09-linking-containers-with-docker-networks git:(master) cat Dockerfile
FROM python:2.7-alpine

RUN mkdir /app
WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

LABEL maintainer="Nick Janetakis <nick.janetakis@gmail.com>" \
      version="1.0"

CMD flask run --host=0.0.0.0 --port=5000

---


➜  09-linking-containers-with-docker-networks git:(master) docker image build -t web2 .

....

```

## Networks

```
➜  09-linking-containers-with-docker-networks git:(master) docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
93c36d6ebe90        bridge              bridge              local
71df6df239f2        common_default      bridge              local
e42c83d93fd5        docker_default      bridge              local
91569667bd1d        host                host                local
ceec667820ae        none                null                local

➜  09-linking-containers-with-docker-networks git:(master) docker network inspect 93c36d6ebe90
[
    {
        "Name": "bridge",
        "Id": "93c36d6ebe904625c5867ed075d48236d4bf25aef3466c7c266bc7c9f652fd9d",
        "Created": "2018-06-30T11:59:40.930867915Z",
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
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
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

➜  09-linking-containers-with-docker-networks git:(master) docker network ls -q
93c36d6ebe90
71df6df239f2
e42c83d93fd5
91569667bd1d
ceec667820ae


➜  09-linking-containers-with-docker-networks git:(master) docker inspect `docker network ls -q` | jq '.[].Name'
"bridge"
"common_default"
"docker_default"
"host"
"none"

➜  09-linking-containers-with-docker-networks git:(master) docker inspect `docker network ls -q` | jq '.[] | .Name, .IPAM.Config'
"bridge"
[
  {
    "Subnet": "172.17.0.0/16",
    "Gateway": "172.17.0.1"
  }
]
"common_default"
[
  {
    "Subnet": "172.18.0.0/16",
    "Gateway": "172.18.0.1"
  }
]
"docker_default"
[
  {
    "Subnet": "172.19.0.0/16",
    "Gateway": "172.19.0.1"
  }
]
"host"
[]
"none"
[]

➜  09-linking-containers-with-docker-networks git:(master) docker inspect `docker network ls -q` | jq '.[] | .Name, .IPAM.Config[].Subnet'
"bridge"
"172.17.0.0/16"
"common_default"
"172.18.0.0/16"
"docker_default"
"172.19.0.0/16"
"host"
"none"
```

Check - running containers in network

```
➜  diveintodocker git:(master) ✗ docker network inspect bridge | jq '.[].Containers'
{
  "6368e90791312b341d110ecba1182f7b9503240eefca020fad9bb11a34b46243": {
    "Name": "web2",
    "EndpointID": "c30990eb30badcfea827cac4d188bf728568f84d5296631ffc6185d00dff8b45",
    "MacAddress": "02:42:ac:11:00:02",
    "IPv4Address": "172.17.0.2/16",
    "IPv6Address": ""
  }
}

```

Run 2 containers

I can NOT reach internal IP from mac

```
➜  diveintodocker git:(master) ✗ curl 172.17.0.2:5000
^C

```

I can reach from inside network

```
➜  diveintodocker git:(master) ✗ docker network inspect bridge | jq '.[].Containers'
{
  "6368e90791312b341d110ecba1182f7b9503240eefca020fad9bb11a34b46243": {
    "Name": "web2",
    "EndpointID": "c30990eb30badcfea827cac4d188bf728568f84d5296631ffc6185d00dff8b45",
    "MacAddress": "02:42:ac:11:00:02",
    "IPv4Address": "172.17.0.2/16",
    "IPv6Address": ""
  },
  "b53a99a8a682cf13de5853886cd93638928398cbb40fdbbb3da95f37671903b3": {
    "Name": "web2a",
    "EndpointID": "b2d9df1858a7fa08710ef0920e1a32b525a8eec9f46d8010ee43e10409c47103",
    "MacAddress": "02:42:ac:11:00:03",
    "IPv4Address": "172.17.0.3/16",
    "IPv6Address": ""
  }
}

➜  diveintodocker git:(master) ✗ docker container ls -s

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES               SIZE
b53a99a8a682        web2                "/bin/sh -c 'flask r…"   3 seconds ago       Up 17 seconds       0.0.0.0:5001->5000/tcp   web2a               298kB (virtual 84.6MB)
6368e9079131        web2                "/bin/sh -c 'flask r…"   2 minutes ago       Up 3 minutes        0.0.0.0:5000->5000/tcp   web2                298kB (virtual 84.6MB)

➜  diveintodocker git:(master) ✗ docker exec -it b53a99a8a682 sh
/app # wget 172.17.0.2:5000
Connecting to 172.17.0.2:5000 (172.17.0.2:5000)
wget: server returned error: HTTP/1.0 500 INTERNAL SERVER ERROR

## => because no Redis


```

Start redis

```
➜  diveintodocker git:(master) ✗ docker container run -itd --rm -p 6379:6379 --name redis redis:3.2-alpine
232ab4d24b54a4aac9d1cb8b56c3b7f63de451c5081937fa88fb7baa97103a93


➜  diveintodocker git:(master) ✗ docker network inspect bridge | jq '.[].Containers'
{
  "232ab4d24b54a4aac9d1cb8b56c3b7f63de451c5081937fa88fb7baa97103a93": {
    "Name": "redis",
    "EndpointID": "ca4f9f7a59d85adefb1dddd575bd0d86a73375633ba433a5b065a5be0867f484",
    "MacAddress": "02:42:ac:11:00:04",
    "IPv4Address": "172.17.0.4/16",
    "IPv6Address": ""
  },
  "6368e90791312b341d110ecba1182f7b9503240eefca020fad9bb11a34b46243": {
    "Name": "web2",
    "EndpointID": "c30990eb30badcfea827cac4d188bf728568f84d5296631ffc6185d00dff8b45",
    "MacAddress": "02:42:ac:11:00:02",
    "IPv4Address": "172.17.0.2/16",
    "IPv6Address": ""
  },
  "b53a99a8a682cf13de5853886cd93638928398cbb40fdbbb3da95f37671903b3": {
    "Name": "web2a",
    "EndpointID": "b2d9df1858a7fa08710ef0920e1a32b525a8eec9f46d8010ee43e10409c47103",
    "MacAddress": "02:42:ac:11:00:03",
    "IPv4Address": "172.17.0.3/16",
    "IPv6Address": ""
  }
}

```

=> does not work when I start the web2 / web2a before redis

=> does not work even if I start REDIS first

Containers can see each other

```
➜  diveintodocker git:(master) ✗ docker exec -it redis cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.4	232ab4d24b54

➜  diveintodocker git:(master) ✗ docker exec -it web2 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	9ff96555959b

➜  diveintodocker git:(master) ✗ docker exec -it web2 ping 172.17.0.4
PING 172.17.0.4 (172.17.0.4): 56 data bytes
64 bytes from 172.17.0.4: seq=0 ttl=64 time=0.171 ms
64 bytes from 172.17.0.4: seq=1 ttl=64 time=0.146 ms
^C
--- 172.17.0.4 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.146/0.158/0.171 ms

➜  diveintodocker git:(master) ✗ docker exec -it redis ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2): 56 data bytes
64 bytes from 172.17.0.2: seq=0 ttl=64 time=0.093 ms
64 bytes from 172.17.0.2: seq=1 ttl=64 time=1.212 ms
^C
--- 172.17.0.2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.093/0.652/1.212 ms

```

## Create own network to do DNS

```
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker network create --driver bridge firstnet
1edc7315b6ad27a3f9dc69880fc0b96ba0e0607f5d65e6273086bd83a5812723
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
93c36d6ebe90        bridge              bridge              local
71df6df239f2        common_default      bridge              local
e42c83d93fd5        docker_default      bridge              local
1edc7315b6ad        firstnet            bridge              local
91569667bd1d        host                host                local
ceec667820ae        none                null                local
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker network inspect firstnet
[
    {
        "Name": "firstnet",
        "Id": "1edc7315b6ad27a3f9dc69880fc0b96ba0e0607f5d65e6273086bd83a5812723",
        "Created": "2018-07-07T17:01:00.8849662Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
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
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

Run both IN that net

```
➜  diveintodocker git:(master) ✗ docker container run -itd --rm -p 6379:6379 --name redis --net firstnet redis:3.2-alpine
d3d8774f3d61d68246e662b67aff7b4e1f159c824384d7954eb6019713a94c66

➜  09-linking-containers-with-docker-networks git:(master) ✗ docker container run -itd --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --name web2 --net firstnet web2
1979375e945353d3ada942b91d622f514cfa50b26db97e3632931b7baa1aebf1


➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
6%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
7%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
8%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
9%

----

➜  09-linking-containers-with-docker-networks git:(master) ✗ docker exec -it redis cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.20.0.3	d3d8774f3d61
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker exec -it redis ping web2
PING web2 (172.20.0.2): 56 data bytes
64 bytes from 172.20.0.2: seq=0 ttl=64 time=0.087 ms
64 bytes from 172.20.0.2: seq=1 ttl=64 time=0.147 ms
^C
--- web2 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.087/0.117/0.147 ms
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker exec -it web2 ping redis
PING redis (172.20.0.3): 56 data bytes
64 bytes from 172.20.0.3: seq=0 ttl=64 time=0.216 ms
64 bytes from 172.20.0.3: seq=1 ttl=64 time=0.143 ms
64 bytes from 172.20.0.3: seq=2 ttl=64 time=0.147 ms
^C
--- redis ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.143/0.168/0.216 ms


➜  09-linking-containers-with-docker-networks git:(master) ✗ docker exec -it redis redis-cli
127.0.0.1:6379> KEYS *
1) "web2_counter"
127.0.0.1:6379> INCRBY web2_counter 30000
(integer) 30009
127.0.0.1:6379>
➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
30010%


```

## ?? Why do they see each other ONLY in custom network ?

```
10713  docker container run -itd --rm -p 6379:6379 --name redis redis:3.2-alpine
10714  docker container run -itd --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --name web2  web2
10715  docker network ls
10716  docker network inspect bridge
10717  docker exec -it web2 ping redis
10718  docker exec -it redis ping web2


➜  09-linking-containers-with-docker-networks git:(master) ✗ docker exec -it web2 ping redis
ping: bad address 'redis'
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker exec -it redis ping web2
ping: bad address 'web2'


### VS

docker container run -itd --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --name web2 --net firstnet web2
docker container run -itd --rm -p 6379:6379 --name redis --net firstnet redis:3.2-alpine

curl localhost:5000

docker exec -it redis cat /etc/hosts
docker exec -it redis ping web2
docker exec -it web2 ping redis

docker network inspect firstnet


➜  09-linking-containers-with-docker-networks git:(master) ✗ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "93c36d6ebe904625c5867ed075d48236d4bf25aef3466c7c266bc7c9f652fd9d",
        "Created": "2018-06-30T11:59:40.930867915Z",
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
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0cb2eaa1a44197bcb376e64ea58be47547f14370a5551f13d71579263cd69ca4": {
                "Name": "web2",
                "EndpointID": "0bccd22e52e86f5e794b4f8115242177b7fb398cfe0ae05d26ab7730e8898cd6",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "e68c509575f267f6c22b9e0eaa862e170d5c731b5d85c9be0505543e29274b2e": {
                "Name": "redis",
                "EndpointID": "ad7fb300ad3c670f4680a9ecc6cc90e3d734b41d3a9880c36273b8ea94f5bf0f",
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


VS

➜  09-linking-containers-with-docker-networks git:(master) ✗ docker network inspect firstnet
[
    {
        "Name": "firstnet",
        "Id": "1edc7315b6ad27a3f9dc69880fc0b96ba0e0607f5d65e6273086bd83a5812723",
        "Created": "2018-07-07T17:01:00.8849662Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.20.0.0/16",
                    "Gateway": "172.20.0.1"
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
            "1979375e945353d3ada942b91d622f514cfa50b26db97e3632931b7baa1aebf1": {
                "Name": "web2",
                "EndpointID": "f8a974089484f9aa3900c8935138e9a369ba7b8f4d4cdfa837bd40b1198764c7",
                "MacAddress": "02:42:ac:14:00:02",
                "IPv4Address": "172.20.0.2/16",
                "IPv6Address": ""
            },
            "d3d8774f3d61d68246e662b67aff7b4e1f159c824384d7954eb6019713a94c66": {
                "Name": "redis",
                "EndpointID": "7828973fa72ae4e665dca21d3d44e102753ce8e8b9520f4576d08711daf26919",
                "MacAddress": "02:42:ac:14:00:03",
                "IPv4Address": "172.20.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

==>
https://docs.docker.com/v17.09/engine/userguide/networking/#the-default-bridge-network

Containers connected to the default bridge network can communicate with each other by IP address. Docker does not support automatic service discovery on the default bridge network. If you want containers to be able to resolve IP addresses by container name, you should use user-defined networks instead.


## Named volumes

- special area

docker volume ls

docker volume create web2_redis

docker container run -itd --rm -p 6379:6379 --name redis --net firstnet -v web2_redis:/data redis:3.2-alpine
docker container run -itd --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --name web2 --net firstnet web2

```
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker container run -itd --rm -p 6379:6379 --name redis --net firstnet -v web2_redis:/data redis:3.2-alpine

6d0af64156ec26850084a1e08349c3b6c2ed2298211e7923da99b8b00edc7b3e
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker container run -itd --rm -p 5000:5000 -e FLASK_APP=app.py -e FLASK_DEBUG=1 -v $PWD:/app --name web2 --net firstnet web2

34b59ce84b01a9139ec8a30c85a6e06856fa02838c135b15d00c538990d50760
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
34b59ce84b01        web2                "/bin/sh -c 'flask r…"   17 seconds ago      Up 2 seconds        0.0.0.0:5000->5000/tcp   web2
6d0af64156ec        redis:3.2-alpine    "docker-entrypoint.s…"   37 seconds ago      Up 22 seconds       0.0.0.0:6379->6379/tcp   redis
➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
1%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
2%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
3%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
4%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
5%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
6%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker stop redis
redis
➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
  "http://www.w3.org/TR/html4/loose.dtd">
<html>
  <head>
    <title>ConnectionError: Error -2 connecting to redis:6379. Name does not resolve. // Werkzeug Debugger</title>
    <link rel="stylesheet" href="?__debugger__=yes&amp;cmd=resource&amp;f=style.css"
        type="text/css">

 
 ... DELETED ...
        

ConnectionError: Error -2 connecting to redis:6379. Name does not resolve.

-->
➜  09-linking-containers-with-docker-networks git:(master) ✗ docker container run -itd --rm -p 6379:6379 --name redis --net firstnet -v web2_redis:/data redis:3.2-alpine

ffab649a559dd4fb039ff01db830130d8499655a1e4e9921f3ecc590c35055c1
➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
7%                                                                                                                                                                                            ➜  09-linking-containers-with-docker-networks git:(master) ✗ curl localhost:5000
8%
```

## Sharing data between containers

=> not touching docker host


