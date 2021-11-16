### Create a bridged docker network
docker network create --driver bridge openhack-net

### Create SQL server image on the new network
docker run --network "openhack-net" -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<SQLPASS>" -p 1433:1433 --name sqlopenhack -h sqlopenhack -d mcr.microsoft.com/mssql/server:2017-latest

### Enter bash shell
docker exec -it sqlopenhack "bash"

### Enter sql cli from the bash shell
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "<SQLPASS>"

### Create required table
CREATE DATABASE mydrivingDB
GO

### Login to repository
az acr login --name registryhro9346

### Populate data
docker run --network openhack-net -e SQLFQDN=sqlopenhack -e SQLUSER=sa -e SQLPASS=<SQLPASS> -e SQLDB=mydrivingDB registryhro9346.azurecr.io/dataload:1.0

### Run POI
docker run -d -p 8080:80 --network openhack-net --name poi -e "SQL_PASSWORD=<SQLPASS>" -e "SQL_SERVER=sqlopenhack" -e "SQL_USER=sa" -e "ASPNETCORE_ENVIRONMENT=Local" tripinsights/poi:1.0
  
Test: http://localhost:8080/api/poi

### Push image to Azure Container Registry
docker build -f Dockerfile -t "tripinsights/userprofile:1.0" .
 docker tag tripinsights/userprofile:1.0 registryhro9346.azurecr.io/tripinsights/userprofile:1.0
 docker push registryhro9346.azurecr.io/tripinsights/userprofile:1.0

### Create secret from file
kubectl apply -f=openhack-secrets.yaml




