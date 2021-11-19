REGION_NAME=northeurope
RESOURCE_GROUP=teamResources
SUBNET_NAME=vm-subnet
VNET_NAME=vnet
ACR_NAME=registryhro9346


SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)
	
VERSION=$(az aks get-versions \
    --location $REGION_NAME \
    --query 'orchestrators[?!isPreview] | [-1].orchestratorVersion' \
    --output tsv)	


AKS_CLUSTER_NAME=openhack-$RANDOM

### create admin group
az ad group create --display-name myAKSAdminGroup --mail-nickname myAKSAdminGroup	

### create cluster
az aks create \
--resource-group $RESOURCE_GROUP \
--name $AKS_CLUSTER_NAME \
--vm-set-type VirtualMachineScaleSets \
--node-count 5 \
--load-balancer-sku standard \
--location $REGION_NAME \
--kubernetes-version $VERSION \
--network-plugin azure \
--vnet-subnet-id $SUBNET_ID \
--service-cidr 10.1.0.0/24 \
--dns-service-ip 10.1.0.10 \
--docker-bridge-address 172.17.0.1/16 \
--generate-ssh-keys \
--enable-aad \
--aad-admin-group-object-ids bef5b92d-6427-4f5c-b4cf-69df22d76caa \
--enable-azure-rbac


az aks update \
    --name $AKS_CLUSTER_NAME \
    --resource-group $RESOURCE_GROUP \
    --attach-acr $ACR_NAME

az aks update -g $RESOURCE_GROUP -n openhack-7589 --enable-pod-identity --enable-pod-identity-with-kubenet

az aks get-credentials \
    --resource-group $RESOURCE_GROUP \
    --name $AKS_CLUSTER_NAME


kubectl get nodes

### create namespaces
kubectl create -f namespaces.yaml

kubectl apply -f userprofile.yaml

kubectl get pods -l app=userprofile

kubectl get deployment

kubectl get service

kubectl logs trips-6cd94b589c-9q254

kubectl get pod -o wide

kubectl exec -it tripsviewer-86c7b6c9fc-nqmc7 -- sh

curl -i -X GET 'http://10.2.0.60/api/poi/healthcheck'
curl -i -X GET 'http://10.2.0.40/api/user-java/healthcheck'

git clone https://github.com/darren-ives/containers_artifacts.git