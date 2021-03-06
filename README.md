## Docker Install and Usage

### Set Proxy if necessary
 1. Add something followed into file /etc/profile
     
         export http_proxy=http://proxysh.domain.com.cn:80
         export https_proxy=$http_proxy
         export no_proxy=10.0.0.0/8,.domain.com.cn,.domain.intra,localhost       
 2. make file validate
 
         source /etc/profile        
 3. validate your username and password in browser

### Install
 1. execute cmd as follows to intall the latest docker
 
         curl -fsSL https://get.docker.com/ | sh        
 2. check docker version
 
         docker -v       
 3. Set Proxy if necessary when downloading or searching image from DockerHub
 
         add proxy info into docker file /etc/default/docker
        
### Common Commands
 1. check docker images installed
 
         docker images
 2. delete docker iamge
 
         docker rmi -f imageName
        
 3. setup container with parameters -e
 
         docker run -dit -p 5000(hostport):5000(containerport) --name containerName -e "SERVICE_12003_CHECK_TTL=60s" -v /home/share:/mnt imageName [/bin/bash]
        
 4. into container if run by -d
 
         docker exec -it containerName(id) /bin/bash
         docker attach containerName(id)
    
         docker start containerName
 5. delete container
 
         docker rm -f containerName
        
 6. export image tar from image
 
         docker save imageName > imageName.tar
 
 7. import image from image tar
 
         docker load -i imageName.tar
        
 8. create new docker image from base image, example is as follows:
 
         比如要在Ubuntu 14.04基础镜像中安装netopeer后，保存为新的镜像，如下：
         1,启动基础镜像  sudo docker run -t -i ubuntu /bin/bash
         2,进入容器里，安装netopeer，docker attach ubuntu，install netopeer，
         3,容器ID要记住，后面创建新镜像的时候会用到
         4,安装好了之后退出容器exit
         5,创建新镜像，docker commit containerID newImageName
        
 9. create new docker image by dockerfile, example is as folloews:
 
     编写 Dockerfile

          docker build -t newImageName .
        

          FROM java:8-jre
          MAINTAINER moon
          COPY restconf-service-2.0-SNAPSHOT.jar .
          COPY example.yml .
          COPY example.keystore .
          COPY yang ./yang
          CMD java -jar restconf-service-2.0-SNAPSHOT.jar server example.yml
          EXPOSE 8080

    
 10. push image to private registry, example is as folloews:
 
     1. tag
            
             docker tag imageId rd-server:5000/imageName
     2. push
     
             docker push rd-server:5000/imageName
           
 11. solutions to https errors(ubuntu)
Docker与Docker Registry交互默认使用https，然而此处搭建的Docker Registry只提供http服务，所以当和Registry私有仓库交互时会失败，为了解决这个问题需要在启动Docker时配置Registry不安全选项
     
     - modify Docker config file
            
            vim /etc/default/docker

     - add new line as follows, and add ip to no_proxy if necessary:
     
            DOCKER_OPTS="$DOCKER_OPTS --insecure-registry=104.131.173.242:5000"
            
     - centos
            修改/usr/lib/systemd/system/docker.service
            ExecStart=/usr/bin/dockerd --registry-mirror=http://xxxxxxxx.m.daocloud.io  --insecure-registry 192.168.232.25:5000
           
     - restart Docker
     
            sudo service docker restart
            
 12. docker cp
    
     docker cp :用于容器与主机之间的数据拷贝。
     
     语法

         docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
     
         docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

     OPTIONS说明：
     
     -L :保持源目标中的链接
     
     实例

     将主机/aa目录拷贝到容器96f7f14e99ab的/aa目录下。

         docker cp /aa 96f7f14e99ab:/aa/

     将主机/aa目录拷贝到容器96f7f14e99ab中，目录重命名为bb。
 
         docker cp /www/w3cschool 96f7f14e99ab:/bb
     
     将容器96f7f14e99ab的/aa目录拷贝到主机的/tmp目录中。
     
         docker cp  96f7f14e99ab:/aa /tmp/
