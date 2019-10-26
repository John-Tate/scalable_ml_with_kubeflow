# Scalable ML with Kubernetes and Kubeflow


# Setup

During this workshop we will use Minikube to host a local kubernetes environment, however any kubernetes deployment will do including other local options such as microk8s or cloud options such as GKE (Google), AKS (Azure), or EKS (AWS).



## kubectl

kubectl is a command line interface (CLI) for working with kubernetes clusters. It allows us to manage local **or** remote clusters after we're propperly registered and authenticated with the cluster.



#### Linux

```bash
gunzip ./utils/kubectl_linux/kubectl.gz
chmod +x ./utils/kubectl_linux/kubectl
sudo mv ./utils/kubectl_linux/kubectl /usr/local/bin/kubectl
kubectl version
```

#### macOS


```bash
gunzip ./utils/kubectl_mac/kubectl.gz
chmod +x ./utils/kubectl_mac/kubectl
sudo mv ./utils/kubectl_mac/kubectl /usr/local/bin/kubectl
kubectl version
```

## kfctl

kubectl:kubernetes :: kfctl:kubeflow

kfctl allows us to manage the deployment of kubeflow to our kubernetes cluster



#### Linux

```bash
cd ./utils/kfctl_linux/
tar -xvf kfctl_v0.6.2_linux.tar.gz
sudo mv kfctl /user/local/bin/kfctl
```


#### macOS

```bash
cd ./utils/kfctl_mac/
tar -xvf kfctl_v0.6.2_darwin.tar.gz
sudo mv kfctl /user/local/bin/kfctl
```

alternatively, [download from the releases page](https://github.com/kubeflow/kubeflow/releases/tag/v0.6.2) and add to your path as you see fit


## Minikube Setup


### Install Linux (Ubuntu)

1. Verify the output of this command is non-empty to check if your CPU supports virtualization:

```bash
grep -E --color 'vmx|svm' /proc/cpuinfo
```

2. Ensure you have a hypervisor installed. [VirtualBox](https://www.virtualbox.org/wiki/Downloads) is recomended

3. Install Minikube

local file:

```
gunzip ./utils/linux/minikube.gz
chmod +x ./utils/linux/minikube
sudo install ./utils/linux/minikube /usr/local/bin
```

web:

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube

sudo install minikube /usr/local/bin/

```


### Install macOS


<!-- 
To Do:

## Minikube ISOs + Cache 
-->


# Start Kubernetes Cluster

## Launch Minikube

```
$ minikube start --cpus 4 --memory 8096 --disk-size=40g --kubernetes-version v1.14.0
```


### Local Registry Setup

```
$ minikube addons enable registry
```

#### 1A: Linux
```bash
$ sudo nano /etc/docker/daemon.json
```

add the following and save:

```json
{
  "insecure-registries" : ["localhost:5000"]
}
```

#### 1B: macOS 

* Navigate to your docker preferences 
* Under 'Daemon' add localhost:5000 to the list of insecure registries


#### 2: Port-forward to minikube registry

If you're using the minikube registry addon, we'll need to create a tunnel between our localhost:5000 and the registry hosted on our minikube cluster. We can do this quickly by running a lightweight alpine linux docker image:


In a _new_ terminal, run:
```bash
# load local file for the alpine image w/ socat installed
docker import ./containers/alpine.tar alpine

# run the container 
sudo docker run --rm -it --network=host alpine ash -c "apk add socat && socat TCP-LISTEN:5000,reuseaddr,fork TC$(minikube ip):5000"

```

## Kubeflow Setup

We have to create the _kubeflow-anonymous_ namespace manually due to a small issue with the bootstrap process (should be fixed in future releases)

```
kubectl create namespace kubeflow-anonymous
```

Use existing configuration + cached files
``` 
cd kfdemo
kfctl apply all -V
```

Download and build from web
```bash 
export KFAPP='kfdemo'
export CONFIG='https://raw.githubusercontent.com/kubeflow/kubeflow/v0.6-branch/bootstrap/config/kfctl_k8s_istio.0.6.2.yaml'

kfctl init ${KFAPP} --config=${CONFIG} -V

kubectl create namespace kubeflow-anonymous

cd ${KFAPP}
kfctl generate all -V
kfctl apply all -V

```


## Seldon Setup

### Helm + Tiller

#### Linux
```bash
tar -zxvf ./utils/linux/helm-v2.15.1-linux-amd64.tgz
sudo mv ./utils/linux/helm /usr/local/bin/helm

#verify
helm help
```

#### macOS

Local File:
```bash
tar -zxvf ./utils/mac/helm-v2.15.1-darwin-amd64.tar.gz
sudo mv ./utils/mac/helm /usr/local/bin/helm


#verify
helm help
```


Homebrew
```bash
brew install kubernetes-helm

#verify
helm help
```



### SeldonCore


## Other setup