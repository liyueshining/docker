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
 
         curl -sSL https://get.docker.com/ | sh        
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
         ```bash
          docker build -t newImageName .
        ```
        
        ```bash
          FROM java:8-jre
          MAINTAINER moon
          COPY restconf-service-2.0-SNAPSHOT.jar .
          COPY example.yml .
          COPY example.keystore .
          COPY yang ./yang
          CMD java -jar restconf-service-2.0-SNAPSHOT.jar server example.yml
          EXPOSE 8080
        ```
    
 10. push image to private registry, example is as folloews:
 
         1. tag
         
            ```bash
               docker tag imageId rd-server:5000/imageName
            ```
         2. push
         
           ```bash
               docker push rd-server:5000/imageName
            ```
 11. solutions to https errors
     
     - modify Docker config file
            
            vim /etc/default/docker

     - add new line as follows:
     
            DOCKER_OPTS="$DOCKER_OPTS --insecure-registry=104.131.173.242:5000"
     - restart Docker
     
            sudo service docker restart
