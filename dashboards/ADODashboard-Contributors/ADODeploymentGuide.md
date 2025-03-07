# CCO Azure DevOps Contributions Dashboard

### _Navigation_

- [CCO Azure DevOps Contributions Dashboard](#cco-azure-devops-contributions-dashboard)
    - [_Navigation_](#navigation)
  - [Overview](#overview)
  - [Infrastructure](#infrastructure)
    - [Deployment](#deployment)
      - [Pre-requisites](#pre-requisites)
      - [Backend Deployment](#backend-deployment)
  - [Dashboard](#dashboard)

## Overview

As part of the Continuous Cloud Optimization solution, a dashboard is included to track the contributions made to a Azure DevOps repository. The objective is to monitor not only the cloud environment, but also all the resources used for its design, deployment and maintenance. This dashboard allows you to monitor different metrics such as:
- Number of Projects
- Number of open/closed pull requests
- Average pull requests per day
- Comparison between number of open vs closed pull requests over the last months
- Branches created over the last months
- ...

An important note about this dashboard is that **this dashboard can be published in the PowerBI online service with auto refresh enabled**. The difference with the current versions of the other CCO dashboards is that, for this one, no dynamic queries are being done directly from the PowerBI file, meaning that it can be published and consumed directly from the [PowerBI online](https://docs.microsoft.com/en-us/power-bi/create-reports/desktop-upload-desktop-files) service.

## Infrastructure

This dashboard requires an infrastructure being deployed in Azure. The infrastructure consists of a Powershell Function App, an Application Insights for monitoring and a Storage Account where results from the Azure DevOps REST API calls will be stored in different tables. The following diagram represents the infrastructure to be deployed.

![GitHub Dashboard Architecture](/install/images/github-dashboard-architecture.png)

### Deployment

As part of this solution we offer you already the required [bicep](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) template that will deploy and connect the architecture presented previously.

#### Pre-requisites

In order to successfully user the deploy.bicep and workflow provided, you will need to have:
- This repository forked in your own environment.
- An Azure subscription. If you don't have one you can create one for free using this [link](https://azure.microsoft.com/en-us/free/search/?OCID=AID2200258_SEM_069a8abd963111ebbd21e8d33199249f:G:s&ef_id=069a8abd963111ebbd21e8d33199249f:G:s&msclkid=069a8abd963111ebbd21e8d33199249f). If you already have an Azure tenant but you want to create a new subscription you can follow the instructions [here](https://docs.microsoft.com/en-us/azure/cost-management-billing/manage/create-subscription#:~:text=On%20the%20Customers%20page%2C%20select%20the%20customer.%20In,page%2C%20select%20%2B%20Add%20to%20create%20a%20subscription.).
- A [resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) already created.
- A service principal with Owner permissions in your subscription. You will need owner permissions because as part of the architecture you will be creating a Managed Identity that will require a role assignment to save the retrieved data in the Storage Account. You can create your service principal with Contributor rights by executing the following commands:
    ```sh
    az ad sp create-for-rbac --name "<<service-principal-name>>" --role "Contributor" --output "json"
    ```
- A secret in your GitHub repository with the name `AZURE_CREDENTIALS`. You can user the output from the previous command to generate this secret. The format of the secret should be:
    ```json
    {
    "clientId": "<client_id>",
    "ClientSecret": "<client_secret>",
    "SubscriptionId": "<subscription_id>",
    "TenantId": "<tenant_id>"
    }
    ```
- Another secret in your Azure DevOps repository with the name `PAT`. This will be store the value of a PAT token you will need to generate with the following permissions:
    | Scope | Permission |
    |-------| ---------- |
    | Code | Read |
    | Graph | Read |
    | Identity | Read |
    | Project and Team | Read |

- In the [local.settings.json](./src/local.settings.json) file, update the values for the `organization`, `resourceGroup` and `storageAccount` with the names you want to configure in your environment. Also, make sure that these names match the values in the [deploy.bicep](./infrastructure/deploy.bicep) file for the same resources.

    > Note: The **organization** correponds to the ADO organization from where the information needs to be retrieved.

#### Backend Deployment

In the [infrastructure](./infrastructure/) folder you will find a `deploy.bicep` file which is the template that will be used to deploy the infrastructure. Please, go ahead and update the first two parameters (`name` and `staname`) with your unique values. **Name** will be used to compose the name of all resources except for the storage account, which will leverage the **staname**.

In the [src](./src/) folder you can find the source code that will be deployed in the Function App once the infrastructure is ready. Basically you will deploy two endpoints:
- **InitializeTables**: you will need to run this endpoint once manually to initialize the Storage Account with the required tables and collect all the data history available in the Azure DevOps API.
- **ADODailySync**: this endpoint will be automatically executed in a daily basis and will add more data to the already created storage account tables. If you don't want a daily execution you can update the cron expression in the `function.json` file under the [ADO DailySync folder](./src/ADOs/ADODailySync/).

Finally, if you go to the root folder of the repository you will find the [workflows folder](/.github/workflows/) under the `.github` folder. There you can locate the workflow that you will have to execute to deploy the backend of the dashboard. The only parameter you will need to setup manually while triggering the workflow in the `resourceGroupName` that you created earlier.

Now you are ready to deploy your backend in your environment:
![deploy-backend](/install/images/ado-run-workflow.png)

After successfully deploying the backend go to the Azure portal and manually rung the `InitializeTables` endpoint. Make sure you see the tables in your Storage Account before moving forward.

![storage-tables](/install/images/ado-storage-tables.png)

## Dashboard

With the previous backend deployed, you can now download the [ADOContributions v1.0.pbit](./ADOContributions%20v1.0.pbit) and execute it locally. You will be asked to enter:
- The Storage Account name of the Storage Account you deployed.
![Storage Account Name](/install/images/ado-storage-account.png)
- The Storage account access key.

After that you will be able to monitor your contributions!

![Ado Contributions](/install/images/Ado-contributions-dashboard.png)