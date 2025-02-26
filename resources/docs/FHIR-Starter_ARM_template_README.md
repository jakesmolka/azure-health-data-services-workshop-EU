# FHIR-Starter Quickstart Bicep/ARM template  

## Introduction 

The FHIR-Starter Quickstart [Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/overview) (ARM) template deploys an [Azure Health Data Services workspace](https://docs.microsoft.com/en-us/azure/healthcare-apis/workspace-overview), [FHIR service](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/overview), [FHIR-Proxy](https://github.com/microsoft/fhir-proxy), and [FHIR Loader](https://github.com/microsoft/fhir-loader) for learner environments in the [Azure Health Data Services Workshop](https://github.com/microsoft/azure-health-data-services-workshop).

## Deploy AHDS workspace, FHIR service, FHIR-Proxy, and FHIR Loader

To begin, **CTRL+click** (Windows or Linux) or **CMD+click** (Mac) on the **Deploy to Azure** button below to open the deployment form in a new browser tab.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2Ffhir-starter%2Fmain%2Fquickstarts%2Fdeployfhirtrain.json)

The ARM/Bicep template will deploy the following components:
+ [AHDS workspace](https://docs.microsoft.com/en-us/azure/healthcare-apis/workspace-overview)
+ [FHIR service](https://docs.microsoft.com/en-us/azure/healthcare-apis/fhir/overview)
+ [FHIR-Proxy](https://github.com/microsoft/fhir-proxy)
+ [FHIR Loader](https://github.com/microsoft/fhir-loader)

> __Important:__ In order to successfully deploy resources with this ARM template, the user must have [Owner](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#owner) rights for the [Resource Group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) where the components are deployed. Before running the ARM template, it is recommended to create a new resource group first and check to make sure that you have Owner permissions for that resource group. Once you confirm that you have Owner rights, then choose that resource group in the dropdown menu when you fill out the deployment form (see the image in Step 1 #2 below).

## Step 1 - Initial deployment 

1. Select or fill in the parameter values (see image below). 

+ Enter a custom **Deployment Prefix**. This prefix will be prepended to the names of all created resources ("trn05" is shown as an example prefix).

+ Make sure to select the "true" values as shown. 

2. Click **Review + create** when ready, and then click **Create** on the next page. 

<img src="./images/ARM_template_config4.png" height="420"> 

__Note:__ Deployment of **FHIR service**, **FHIR-Proxy**, and **FHIR Loader** typically takes 20 minutes.

### Deployed Components
When the deployment finishes, you should see these components in your resource group. 


Name              | Type                 |  Purpose                               
------------------|----------------------|----------------------------------------
[prefix]**hdsws**  | PaaS | **AHDS Workspace** - managed PaaS
[prefix]**hdsws/fhirtrn**  | PaaS | **FHIR service** - managed FHIR service
[prefix]**pxyfa** | Function App | **FHIR-Proxy** - filters FHIR data input/output 
[prefix]**ldrfa** | Function App | **FHIR Loader** - ingest FHIR data
[prefix]**asp**   | App Service Plan | Shared by FHIR-Proxy and FHIR Loader function apps
[prefix]**expsa** | Storage account      | Blob storage for FHIR service `$export` operation
[prefix]**funsa** | Storage account      | Storage for FHIR-Proxy function app
[prefix]**impsa** | Storage account      | Storage account for FHIR Loader
[prefix]**kv**    | Key Vault            | Stores secrets and configuration settings
[prefix]**la**    | Log Analytics Workspace  | Logs the activity of deployed components
[prefix]**ldrai** | Application Insights | Monitors FHIR Loader application
[prefix]**pxyai** | Application Insights | Monitors FHIR-Proxy application
[prefix]**ldrtopic** | Event Grid System Topic | Triggers processing of FHIR bundles placed in the **"impsa"** storage account
[prefix]**rc**    | Redis Cache  | Required by FHIR-Proxy modules, Consent Opt Out

### Data Flow

<img src="./images/FHIR-TRAINING-ARM-DIAGRAM-FS.png" height="410">

__Note:__ [Postman](https://www.getpostman.com/) is shown as an example REST client. If you are interested in setting up Postman to connect with FHIR service, please see [here](./Postman_FHIR_service_README.md) after completing steps 2 and 3 below. 

## Step 2 - Complete FHIR-Proxy Authentication 
Once the ARM template finishes the initial deployment, additional steps are necessary to set up authentication for the FHIR-Proxy function app. 

1. In the Azure Portal, navigate to the FHIR-Proxy function app in your resource group. The name of the FHIR-Proxy function app ends with **"pxyfa"**.
<img src="./images/FHIR-PROXY-AUTH1.png" height="410">

2. Select the function app and select **Authentication**.
<img src="./images/FHIR-PROXY-AUTH2.png" height="410">

3. Click **Add identity provider**.
<img src="./images/FHIR-PROXY-AUTH3.png" height="410">

4. Select **Microsoft**.
<img src="./images/FHIR-PROXY-AUTH4.png" height="410">

5. Configure basic settings as follows. The **Allow unauthenticated access** button should remain checked as this will make the FHIR service [Capability Statement](https://www.hl7.org/fhir/capabilitystatement.html) generally available. Click **Next Permissions** at the bottom. 
<img src="./images/FHIR-PROXY-AUTH5a.png" height="410">

6. Once in the **Permissions** tab, click **Add**.
<img src="./images/FHIR-PROXY-AUTH6.png" height="410">

At this point, the FHIR-Proxy application registration is complete. 

## Step 3 - Configure App Roles and API Permissions 

Further configuration is necessary to define **App Roles and API Permissions** for FHIR-Proxy. 

1. In the **Authentication** blade, click on the link next to the Microsoft identity provider. This will open the Azure AD blade.
<img src="./images/FHIR-PROXY-AUTH7.png" height="410">

2. Click on **Manifest**.
<img src="./images/FHIR-PROXY-AUTH8.png" height="410">

3. Populate the **appRoles** element with the data in the `fhirproxyroles.json` file available [here](../deploy/fhirproxyroles.json).
<img src="./images/FHIR-PROXY-AUTH9.png" height="410">

4. After pasting the contents of the `fhirproxyroles.json` file, the **appRoles** element should look something like shown below. Click **Save**.
<img src="./images/FHIR-PROXY-AUTH10.png" height="410">

5. Select the **API permissions** blade. Then click **+Add a permission**.
<img src="./images/FHIR-PROXY-AUTH11.png" height="410">

6. Select **APIs my organization uses**.
<img src="./images/FHIR-PROXY-AUTH12.png" height="310">

7. Filter the results to "Azure Healthcare APIs". Click on **Azure Healthcare APIs**.
<img src="./images/FHIR-PROXY-AUTH13.png" height="310">

8. Select the **user_impersonation** permission box and click **Add permissions** at the bottom.
<img src="./images/FHIR-PROXY-AUTH14.png" height="410">

9. Verify the **API permissions**.
<img src="./images/FHIR-PROXY-AUTH15.png" height="310">

10. Click **Grant admin consent** (blue checkmark).

11. Verify that the **App roles** were created successfully.
<img src="./images/FHIR-PROXY-AUTH16.png" height="310">
