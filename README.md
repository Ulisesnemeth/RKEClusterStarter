# RKEClusterStarter
Ubuntu 23.04

## 1. Instalar RKE bins
### https://rke.docs.rancher.com/installation
###### RKE is a fast, versatile Kubernetes installer that you can use to install Kubernetes on your Linux hosts. You can get started in a couple of quick and easy steps
#### RKE es un instalador de Kubernetes rapido y versatil que podes usar para instalar Kubernetes en tu servidor de linux.
Descargar bins:
```sh
wget https://github.com/rancher/rke/releases/download/v1.4.10/rke_linux-amd64
```
Hacelo ejecutable:
```sh
mv rke_linux-amd64 rke
chmod +x rke
sudo mv rke /usr/local/bin/
```
Proba si ya podes usar rke:
```sh
rke --version
```

## 2. Instalar docker
https://docs.docker.com/engine/install/ubuntu/

## 3. Agregar usuario al grupo de docker
#### Sino agregas tu usuario al grupo de docker en cada nodo, rke no va a poder acceder a la informacion que necesita para levantar el cluster.
Agrega el usuario:
```sh
sudo usermod -aG docker $(whoami)
```
Deslogueate y logueate para probar el comando docker con el usuario:
```sh
su - $(whoami)
```
Proba el comando docker:
```sh
docker ps
```
Si no podes ver la lista de contenedores vacia con tu usuario, probablemente es porque no existia el grupo de docker en ubuntu, capaz porque instalaste mal docker, volve al paso 2 y hacelo bien.

## 4. Crear un archivo de configuracion para el cluster
En este archivo vas a poner la informacion que necesita rke para levantar el cluster. En este ejemplo uso el minimal yml que esta en la documentacion de rke
##### https://rke.docs.rancher.com/example-yamls#minimal-clusteryml-example
Crea el archivo
```sh
nano cluster.yml
```

Aca hay un ejemplo:
###### Reemplaza IP_DEL_NODO con el ipv4 interno del nodo, por ejemplo 192.168.1.80
###### Reemplaza USUARIO con el usuario que agregaste al grupo docker
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
#### rke usa ssh para comunicarse con los nodos a la hora de levantar el cluster, incluyendo el controlplane.
Crea una clave ssh:
```sh
ssh-keygen
```
Usa este comando para que rke pueda utilizar la clave ssh sin problemas:
###### Reemplaza NOMBREDELACLAVE con el nombre de la key, por defecto id_rsa.pub
###### Reemplaza IP_DEL_NODO con el ipv4 interno del nodo, por ejemplo 192.168.1.80
###### Reemplaza USUARIO con el usuario que agregaste al grupo docker
```sh
ssh-copy-id -i ~/.ssh/NOMBREDELACLAVE.pub USUARIO@IP_DEL_NODO
```

## 6. Levantar el cluster
rke up levanta el cluster usando el archivo cluster.yml, todo lo que hiciste despues de instalar los binarios de rke fue para que este comando no tenga problemas a la hora de levantar los nodos.
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
```
```sh
sudo apt-get install -y kubectl
```
Para utilizar kubectl necesitas que la variable de entorno KUBECONFIG tenga el directorio del archivo kube_config_cluster.yml
Ejemplo para ver los nodos del cluster:
```sh
export KUBECONFIG=kube_config_cluster.yml
```
```sh
kubectl get nodes
```

## 8. Crear Deployment
Sirve para que el cluster sepa cuantos pods usar, que imagen usar y que puerto usar para el API
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
Sirve para exponer el API en un puerto (entre 30000 y 32767)
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

## Para crear un sc
```sh
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```
