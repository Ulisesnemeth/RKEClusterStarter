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

## 7. Intalar Kubectl para gestionar
```sh
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl
```
```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```sh
# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```sh
sudo apt-get update
sudo apt-get install -y kubectl
```

## 7.1. Ejemplo para ver los nodos del cluster
```sh
export KUBECONFIG=kube_config_cluster.yml
```
```sh
kubectl get nodes
```

## 8. Crear Deployment
```sh
nano deployejemplo.yaml
```
Ejemplo
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
```
```sh
kubectl apply -f deployejemplo.yaml
```
## 9. Crear Service para exponer el deployment
```sh
nano serviceejemplo.yaml
```
```sh
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 31000 
  selector:
    app: hello-world
```
```sh
kubectl apply -f serviceejemplo.yaml
```





