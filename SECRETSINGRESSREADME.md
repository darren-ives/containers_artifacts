helm install ingress-nginx ingress-nginx/ingress-nginx --create-namespace --namespace ingress

az aks enable-addons --addons azure-keyvault-secrets-provider --name openhack-7589 --resource-group teamResources

az keyvault create -n openhack-keyvault-7589 -g teamResources -l "North Europe"
az keyvault secret set --vault-name openhack-keyvault-7589 -n server --value sqlserverhro9346.database.windows.net
az keyvault secret set --vault-name openhack-keyvault-7589 -n username --value sqladminhRo9346
az keyvault secret set --vault-name openhack-keyvault-7589 -n password --value tP5nr3Zy6