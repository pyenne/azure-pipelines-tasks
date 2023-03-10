#  Azure App Service Deployment: ARM 

## Overview

The Azure App Service Deployment task is used to update different Azure App Service to deploy [Web Apps](https://azure.microsoft.com/en-in/documentation/articles/app-service-web-overview/) and [WebJobs](https://azure.microsoft.com/en-us/blog/webjobs-goes-into-full-production/) to Azure. The task works on cross platform VSTS agents running Windows, Linux or Mac and uses the underlying deployment technologies of [Web Deploy](https://www.iis.net/downloads/microsoft/web-deploy) and [Kudu REST APIs](https://github.com/projectkudu/kudu/wiki/REST-API).

The task works for [ASP.NET](https://www.visualstudio.com/en-us/docs/release/examples/azure/azure-web-apps-from-build-and-release-hubs), [ASP.NET Core](https://www.visualstudio.com/en-us/docs/release/examples/azure/aspnet-core10-azure-web-apps), PHP, Java, Python and [Node.js](https://www.visualstudio.com/en-us/docs/release/examples/nodejs/node-to-azure-webapps) based web applications.

The task can be used to deploy different Azure App Services like Function App, Web App on Windows, Web App on Linux, Web App for Containers and Azure App Service Environments.

The task is **under development and is available to a limited set of accounts on Visual Studio Team Services (VSTS)**. The [video](https://www.youtube.com/watch?v=uQ2qCmaZ_Ag&feature=youtu.be) describes the features that are available in the task currently.

## Contact Information

Please report a problem at [Developer Community Forum](https://developercommunity.visualstudio.com/spaces/21/index.html) if you are facing problems in making this task work. Also, share feedback about the task, and the new features that you would like to see in it.

## Pre-requisites for the task

The following pre-requisites need to be setup in the target machine(s) for the task to work properly.

##### Azure Web App

The task is used to deploy a Web  project to an existing Azure Web App. The Web App should exist prior to running the task. The Web App can be created from the [Azure portal](https://azure.microsoft.com/en-in/documentation/videos/azure-app-service-web-apps-with-yochay-kiriaty/) and [configured](https://azure.microsoft.com/en-us/documentation/articles/web-sites-configure/) there. Alternatively, the [Azure PowerShell task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzurePowerShell) can be used to run [AzureRM PowerShell scripts](https://msdn.microsoft.com/en-us/library/mt619237.aspx) to provision and configure the Web App.

##### Azure Subscription

To deploy to Azure, an Azure subscription has to be linked to Team Foundation Server or to Visual Studio Team Services using the Services tab in the Account Administration section. Add the Azure subscription to use in the Build or Release Management definition by opening the Account Administration screen (gear icon on the top-right of the screen) and then click on the Services Tab.

Create the [ARM](https://azure.microsoft.com/en-in/documentation/articles/resource-group-overview/) service endpoint, use **'Azure Resource Manager'** endpoint type, for more details follow the steps listed in the link [here](https://go.microsoft.com/fwlink/?LinkID=623000&clcid=0x409).

The task does not work with the Azure Classic service endpoint and it will not list these connections in the parameters in the task.

##### Azure PowerShell

The task needs the Azure PowerShell version to be installed on the automation agent, and that can be done easily using the [Azure PowerShell Installer v1.3.0](https://github.com/Azure/azure-powershell/releases/tag/v1.3.0-March2016).

## Deployment

Based on the type of Azure App Service and VSTS agent, the task chooses a suitable deployment technology. The different deployment technologies used by the task are:
* *Web Deploy* 

* *Kudu REST APIs*

* *Container Registry* 

The task defaults to Web Deploy technology on a Windows Agent when the target is Web App for Windows. On other platforms (for any App service type), the task relies on [Kudu REST APIs](https://github.com/projectkudu/kudu/wiki/REST-API) to deploy the Web App.

### Web Deploy

Web Deploy (msdeploy.exe) is used to deploy the web application to the Azure Web App. Web Deploy Works on a Windows automation agent when the target is a Web App on Windows or Function App. Install it on the agent using the [Microsoft Web Platform Installer](https://www.microsoft.com/web/gallery/install.aspx?appid=wdeploynosmo). The Web Deploy 3.5 needs to be installed without the bundled SQL support. There is no need to choose any custom settings while installing Web Deploy. After installing the Web Deploy is available at C:\\Program Files (x86)\\IIS\\Microsoft Web Deploy V3.

Web Deploy is feature rich and offers options like:
* **Rename locked files\*:**  Rename file which are still in use by the web server by enabling msdeploy flag MSDEPLOY_RENAME_LOCKED_FILES=1 in Azure App Service application settings. The option if set enables msdeploy to rename locked files that are locked during app deployment

* **Remove additional files at destination\*:**  Deletes files on the Azure App Service that have no matching files in the App Service artifact package or folder getting deployed.

* **Exclude files from the App_Data folder\*:**  Prevent files in the App_Data folder (in the artifact package/folder getting deployed) from being deployed to the Azure App Service

* **Additional Web Deploy arguments\*:** Arguments that will be applied when deploying the Azure App Service. Example: -disableLink:AppPoolExtension -disableLink:ContentExtension.For more examples of Web Deploy operation settings, refer [Web Deploy Operation Settings](https://go.microsoft.com/fwlink/?linkid=838471) 


### [Kudu REST APIs](https://github.com/projectkudu/kudu/wiki/REST-API)
Works on a Windows as well as Linux automation agent when the target is a Web App on Windows or Web App on Linux (built-in source) or Function App. The task uses Kudu to copy over files to the Azure App service.

### Container Registry
Works on a Windows as well as Linux automation agent when the target is a Web App for Containers. The task updates the Azure Web App for Containers by setting the right Container registry, repository, image name and tag information. You can also use the task to pass a startup command for the container image.

### Parameters of the task

The task is used to deploy a Web  project to an existing Azure Web App. The mandatory fields are highlighted with a *.

* **Azure Subscription\*:** Select the AzureRM Subscription. If none exists, then click on the **Manage** link, to navigate to the Services tab in the Administrators panel. In the tab click on **New Service Endpoint** and select **Azure Resource Manager** from the dropdown.

* **App Service type\*:** Select the Azure App Service type. The different app types supported are Function App, Web App on Windows, Web App on Linux, Web App for Containers and Azure App Service Environments

* **App Service Name\*:** Select the name of an existing Azure App Service. Enter the name of the Web App if it was provisioned dynamically using the [Azure PowerShell task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzurePowerShell) and [AzureRM PowerShell scripts](https://msdn.microsoft.com/en-us/library/mt619237.aspx).

* **Deploy to Slot:** Select the option to deploy to an existing slot other than the Production slot. Do not select this option if the Web project is being deployed to the Production slot. The Web App itself is the Production slot.

* **Resource Group:** Select the Azure Resource Group that contains the Azure App Service specified above. Enter the name of the Azure Resource Group if has been dynamically provisioned using [Azure Resource Group Deployment task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/DeployAzureResourceGroup) or [Azure PowerShell task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzurePowerShell). This is a required parameter if the option to Deploy to Slot has been selected.

* **Slot:** Select the Slot to deploy the Web project to. Enter the name of the Slot if has been dynamically provisioned using [Azure Resource Group Deployment task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/DeployAzureResourceGroup) or [Azure PowerShell task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzurePowerShell). This is a required parameter if the option to Deploy to Slot has been selected.

* **Virtual Application:** Specify the name of the Virtual Application that has been configured in the Azure portal. The option is not required for deployments to the website root. The Virtual Application should have been [configured](https://azure.microsoft.com/en-us/documentation/articles/web-sites-configure/) prior to deploying the Web project to it using the task.

* **Package or Folder\*:** Location of the Web App zip package or folder on the automation agent or on a UNC path accessible to the automation agent like, \\\\BudgetIT\\Web\\Deploy\\Fabrikam.zip. Predefined system variables and wild cards like, $(System.DefaultWorkingDirectory)\\\***.zip can be also used here.

* **Publish using Web Deploy:** Select the option to use Web Deploy technology on a Windows Agent. On other platforms, the task implicitly relies on [Kudu REST APIs](https://github.com/projectkudu/kudu/wiki/REST-API) to deploy the web App and following Web deploy options are not supported.

* **Parameters File:** (Optional) The parameter file is used to override the default settings in the web deploy zip package file like, the IIS Web application name or the database connection string. This helps in having a single package that can be deployed across dev, test, staging, and production, with a specific parameter file for each environment.

* **Remove Additional Files at Destination:** Select the option to delete the files in the Azure App Service that have no matching files in the Web App zip package. This will ensure that during the Web project deployment any additional files in the Azure App Service are deleted, and the only files in the Azure App Service are the ones in the Web App zip package.
This will also remove all files related to any extension (for example Application Insights) installed on this Azure App Service. To prevent this, enable 'Exclude files from App_Data folder' as well.

* **Exclude Files from the App_Data Folder:** Select the option to prevent files in the App_Data folder from being deployed to the Azure App Service. This is a useful option to select, if a local database or a WebJob has been deployed earlier to the Azure App Service, and they should not be deleted in the subsequent deployments of the Web project.

* **Take Application Offline:** Select the option to take the Azure App Service offline by placing an app_offline.htm file in the root directory of the Web App before the sync operation begins. The file will be removed after the sync operation completes successfully.

* **Additional Arguments:** Additional Web Deploy arguments that will be appended to the MSDeploy command while deploying the Azure Web App like,-disableLink:AppPoolExtension -disableLink:ContentExtension. A useful parameter for enabling and disabling rules and for skipping syncing of certain folders.

* **Generate Web.config:** A standard Web.config will be generated and deployed to Azure App Service if the application does not have one. For example, for [Nodejs application, web.config](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps) will have startup file and iis_node module values. Similarly for Python (Bottle, Django, Flask) the web.config will have details of WSGI handler, Python path etc. The task will generate a new web.config only when the artifact package/folder does not contain an existing web.config. The default values populated by the task can be overriden in the task by using the Web.config parameters field. 

* **Web.config parameters:** Edit values like startup file in the task generated web.config file. The default values populated by the task can be overridden in the task by passing the web.config parameters. This edit feature is **only for the generated web.config**. Feature is useful when [Azure App Service Manage task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzureAppServiceManage) is used to install specific Python version by using extensions or when you want to provide a different startup file for Node.js. 
In  case of Python, the path can be set as an output variable of the [Azure App Service Manage task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/AzureAppServiceManage)  and then set as the Python path in the web.config generated by this deploy task. You can try out this feature by selecting any Python, Nodejs, PHP release definition template.

* **Deployment script:** 
The task provides an option to customize the deployment by providing a script that will run on the Azure App Service once the application artifacts have been copied successfully to the App Service. You can choose to either provide an inline deployment script or point to a script file in your atifact folder. This is very useful when you want to restore your application dependencies on the App service directly. Restoring packages of Node, PHP, Python applications helps in avoiding timeouts when the application dependency results in a large artifact getting copied over from VSTS Agent to Azure app service. An example of this script is:

>@echo off
>if NOT exist requirements.txt (
> echo No Requirements.txt found.
> EXIT /b 0
>)
>if NOT exist "$(PYTHON_EXT)/python.exe" (
> echo Python extension not available >&2
> EXIT /b 1
>)
>echo Installing dependencies
>call "$(PYTHON_EXT)/python.exe" -m pip install -U setuptools
>if %errorlevel% NEQ 0 (
> echo Failed to install setuptools >&2
> EXIT /b 1
>)
>call "$(PYTHON_EXT)/python.exe" -m pip install -r requirements.txt
>if %errorlevel% NEQ 0 (
> echo Failed to install dependencies>&2
 EXIT /b 1
)

* **Image Source:**
App Service on Linux offers two different options to publish your application, one is Custom image deployment (Web App for Containers) and the other is App deployment with a built-in platform image (Web App on Linux). You will see this parameter only when you selected 'Linux Web App' in the App type selection option in the task.

For Web **Web App for Containers** you need to provide the following details:
* *Registry or Namespace:*
A globally unique top-level domain name for your specific registry or namespace. A fully qualified image name will be of the format: '<registry or namespace>/<repository>:<tag>'. For example, 'myregistry.azurecr.io/nginx:latest'.
  
* *Image:*
Image Name of the repository where the container images are stored. A fully qualified image name will be of the format: '<registry or namespace>/<repository>:<tag>'. For example, 'myregistry.azurecr.io/nginx:latest'.

* *Tag:*
Tags are optional, it is the mechanism that registries use to give Docker images a version. A fully qualified image name will be of the format: '<registry or namespace>/<repository>:<tag>'. For example, 'myregistry.azurecr.io/nginx:latest'.
  
* *Startup command:*
Start up command for the container.
  
For Web **Web App on Linux** you need to provide the following details:
* *Runtime stack:* Select the framework and version your web app will run on.
  
* *Startup command:*
Start up command for the app. For example if you are using PM2 process manager for Nodejs then you can specify the PM2 file here.

### Output Variables

* **Web App Hosted URL:** Provide a name, like FabrikamWebAppURL for the variable for the Azure App Service Hosted URL. The variable can be used as $(variableName), like $(FabrikamWebAppURL) to refer to the Hosted URL of the Azure App Service in subsequent tasks like in the [Run Functional Tests task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/RunDistributedTests) or the [Visual Studio Test task](https://github.com/Microsoft/vsts-tasks/tree/master/Tasks/VsTest).


### FAQ
* To ignore SSL error set a Variable of name VSTS_ARM_REST_IGNORE_SSL_ERRORS with value : true in the release definition.
* The task works with the [Azure Resource Manager APIs](https://msdn.microsoft.com/en-us/library/azure/dn790568.aspx) only.
* For avoiding deployment failure with error code ERROR_FILE_IN_USE, in case of .NET apps targeting Web App on Windows, ensure that 'Rename locked files' and 'Take App Offline' are enabled. For zero downtime deployment use slot swap.
* When deploying to an App Service with App Insights configured, if you have enabled ???Remove additional files at destination??? then you also need to enable ???Exclude files from the App_Data folder??? in order to keep App insights extension in safe state. This is required because App Insights continuous web job gets installed into the App_Data folder. 
