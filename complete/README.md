# Deploy a Spring Boot application with Open Liberty Operator

It's assumed that there are no additional changes injected to the application container managed by the Open Liberty Operator. This simple sample is to verify the assumption that a Spring Boot application image can run with Open Liberty Operator. 

Following steps below to deploy a Spring Boot application with Open Liberty Operator on Kubernetes.

## Prerequisites

You need to have a Kubernetes cluster and `kubectl` command line tool configured to communicate with your cluster, where the Open Liberty Operator is installed. The easiest way to get started is to leverage [IBM WebSphere Liberty and Open Liberty on Azure Kubernetes Service](https://ms.portal.azure.com/#create/ibm-usa-ny-armonk-hq-6275750-ibmcloud-aiops.20210924-liberty-aksliberty-aks) marketplace offer on Azure.

* Open [IBM WebSphere Liberty and Open Liberty on Azure Kubernetes Service](https://ms.portal.azure.com/#create/ibm-usa-ny-armonk-hq-6275750-ibmcloud-aiops.20210924-liberty-aksliberty-aks).
* Follow the UI instructions to create an AKS cluster with Open Liberty Operator installed.
* Wait until the deployment is completed. It will deploy an AKS cluster, an ACR instance and an Open Liberty Operator running on the AKS cluster.
* Connect to the cluster by running the command retrieved from **Outputs** > **cmdToConnectToCluster**. Also copy the ACR name from **Outputs** > **acrName**.

## Containerize the Spring Boot sample application

Follow steps below to containerize the Spring Boot sample application.

1. Clone this repository.

    ```bash
    git clone https://github.com/majguo/gs-rest-service.git
    cd gs-rest-service/complete
    ```

1. Containerize the application. If you want to directly deploy the application image to AKS cluster, there is a pre-built image `ghcr.io/majguo/gs-rest-service` available on GitHub Package Registry. You can skip this step and directly go to the next section [Deploy the application](#deploy-the-application). Otherwise, you can build and push the image to the ACR deployed in the 1st step.

    ```bash
    mvn clean package
    docker build -t <acr-name>.azurecr.io/gs-rest-service .
    az acr login --name <acr-name>
    docker push <acr-name>.azurecr.io/gs-rest-service
    ```

## Deploy the application

If you have built and pushed the image to the ACR, open file `ola-deployment.yml` and replace the value of property `applicationImage` with `<acr-name>.azurecr.io/gs-rest-service`. Now you can deploy the application by running the following command.

```bash
kubectl apply -f ola-deployment.yml
```

Wait until the application is running. You can check the status by running the following command.

```bash
kubectl get pods
```

After all pods are ready and running, you can retrive the application endpoint by running the following command.

```bash
echo "App endpoint is: http://$(kubectl get svc gs-rest-service -o=jsonpath='{.status.loadBalancer.ingress[0].ip}')/greeting"
```

It takes sometime to create an IP address resource for the service. If you don't see a valid endpoint, retry the above command until you see a valid one.

Open it in a browser. You should see the following message.

```text
{
    "id": 1,
    "content": "Hello, World!"
}
```

## Clean up

To clean up the resources created by this sample, run the following command.

```bash
az group delete --name <resource-group-name> --yes --no-wait
```
