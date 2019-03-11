# Portainer

## 获取image

docker pull portainer/portainer

## 启动 portainer
docker run -d --name portainer -p 9000:9000 --restart always -v /var/run/docker.sock:/var/run/docker.sock -v e:\portainer_data:/data portainer/portainer
