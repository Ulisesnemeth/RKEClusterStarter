# RKEClusterStarter

## 1. Instalar RKE bins
```sh
wget https://github.com/rancher/rke/releases/download/v1.4.10/rke_linux-amd64
```
```sh
mv rke_linux-amd64 rke
chmod +x rke
sudo mv rke /usr/local/bin/
```
```sh
rke --version
```

## 2. Instalar docker
```sh
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
```sh
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 3. Agregar usuario al grupo de docker
```sh
sudo usermod -aG docker $(whoami)
```
```sh
su - $(whoami)
```
```sh
docker ps
```

## 4. Crear un archivo de configuracion para el cluster
```sh
nano cluster.yml
```
Aca hay un ejemplo:
```sh
nodes:
    - address: IP_DEL_NODO
      user: USUARIO
      role: 
        - controlplane 
        - etcd
        - worker
```
## 5. Crear una clave ssh
```sh
ssh-keygen
```
```sh
ssh-copy-id -i ~/.ssh/NOMBREDELACLAVE.pub USUARIO@IP_DEL_NODO
```
## 6. Levantar el cluster
```sh
rke up
```
