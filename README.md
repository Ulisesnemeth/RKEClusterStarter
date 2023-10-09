# RKEClusterStarter
Ubuntu 23.04
## 1. Instalar RKE bins
```sh
wget https://github.com/rancher/rke/releases/download/v1.4.10/rke_linux-amd64
```
```sh
mv rke_linux-amd64 rke
chmod +x rke
sudo mv rke /usr/local/bin/
```
Para probar si ya podes usar rke:
```sh
rke --version
```

## 2. Instalar docker
Fijate vos

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
Si podes ver la lista de contenedores vacia con tu usuario, segui con el paso 4

## 4. Crear un archivo de configuracion para el cluster
```sh
nano cluster.yml
```

Aca hay un ejemplo:
Reemplaza IP_DEL_NODO con el ipv4 interno del nodo, por ejemplo 192.168.1.80
Reemplaza USUARIO con el usuario que agregaste al grupo docker
```sh
nodes:
    - address: IP_DEL_NODO
      user: USUARIO
      role:
        - controlplane
        - etcd
        - worker
```

## 5. Crea una clave ssh
```sh
ssh-keygen
```
Reemplaza NOMBREDELACLAVE con el nombre de la key, por defecto id_rsa.pub
Reemplaza IP_DEL_NODO con el ipv4 interno del nodo, por ejemplo 192.168.1.80
Reemplaza USUARIO con el usuario que agregaste al grupo docker
```sh
ssh-copy-id -i ~/.ssh/NOMBREDELACLAVE.pub USUARIO@IP_DEL_NODO
```

## 6. Levantar el cluster
```sh
rke up
```

## 7. Intalar Kubectl para gestionar el cluster
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
Para utilizar kubectl necesitas que la variable de entorno KUBECONFIG tenga el directorio del archivo kube_config_cluster.yml, como en el paso 7.1

## 7.1. Ejemplo para ver los nodos del cluster
```sh
export KUBECONFIG=kube_config_cluster.yml
```
```sh
kubectl get nodes
```

## 8. Crear Deployment
Ejemplo:
```sh
nano deployejemplo.yaml
```
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
Ejemplo:
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

## Ya podes enviar un get request al deploy usando la ip del nodo y el puerto 31000
```sh
curl 192.168.1.189:31000
```
