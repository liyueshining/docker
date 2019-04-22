# Docker Registry

## precondiftion
docker installed

## Get Registry
```
docker pull registry:2
```

## Start Registry

```
docker run -d -p 5000:5000 -v /home/moon/registry/data:/var/lib/registry --privileged=true --restart=always --name registry registry:2
```
> directory /home/moon/registry/data is a local self-defined directory on host.

## Config docker client
* centos系统：
   1.Modify file  /usr/lib/systemd/system/docker.service，add  --insecure-registry $HOSTIP:5000 to below:
    ```
    ExecStart=/usr/bin/dockerd ......... --insecure-registry $HOSTIP:5000
    ```
    $HOSTIP is the host ip, please replace with real ip address.  
    
   2.Validate config and restart Docker：
   ```
    systemctl daemon-reload
    systemctl restart docker
    ```
    
## How to use：
 1. add tag
    ```    
    docker tag registry:2 172.16.0.90:5000/registry:2
    ```
 2. push image
    ```
    docker push 172.16.0.90:5000/registry:2
    ```
 3. pull image
    ```
    docker pull 172.16.0.90:5000/registry:2
