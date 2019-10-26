# Launch and configue a Kubernetes cluster on Azure


## Setup a Cluster

1. Create a resource group

2. Create an Azure Container Registry

3. Create a service account for your cluster
    - Use AzureAD to register an application
    - Give the account ACR push/pull 
    - Give the account a 'Contributor' role in your resource group

4. Create a Kubernetes Cluster

```
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 2 \
    --service-principal $SA_CLIENT_ID \
    --client-secret $SA_CLIENT_SECRET \
    --generate-ssh-keys
 ```


## Register Kubectl with Azure

```
az aks get-credentials -n $CLUSTER_NAME -g $RESOURCE_GROUP_NAME
```


## (AFTER KF DEPLOYMENT) Port Forward the dashboard

```
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```