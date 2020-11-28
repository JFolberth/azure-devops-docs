---
title: Automating infrastructure deployments in the Cloud with Terraform and Azure Pipelines
description: DevOps CI CD - Use Terraform to manage infrastructure deployment from Azure Pipelines and TFS
ms.topic: tutorial
ms.date: 05/18/2020
monikerRange: '>= tfs-2018'
---

# Use Terraform to manage infrastructure deployment

[!INCLUDE [version-tfs-2018](../includes/version-tfs-2018.md)]

::: moniker range="<= tfs-2018"
[!INCLUDE [temp](../includes/concept-rename-note.md)]
::: moniker-end

[Terraform](https://www.terraform.io/intro/index.html) is a tool for building, changing and versioning infrastructure safely and efficiently. Terraform can manage existing and popular cloud service providers as well as custom in-house solutions.

Configuration files describe to **Terraform** the components needed to run a single application or your entire datacenter. Terraform generates an execution plan describing what it will do to reach the desired state, and then executes it to build the described infrastructure. As the configuration changes, Terraform is able to determine what changed and create incremental execution plans which can be applied.

In this tutorial, you learn about:

> [!div class="checklist"]
> * The structure of a Terraform file
> * Building an application using an Azure CI pipeline
> * Deploying resources using Terraform in an Azure CD pipeline

## Prerequisites

1. A Microsoft Azure account.
1. An Azure DevOps account.
1. Use the [Azure DevOps Demo Generator](https://azuredevopsdemogenerator.azurewebsites.net/?TemplateId=77382&Name=Terraform) to provision the tutorial project on your Azure DevOps organization. This URL automatically selects the Terraform template in the demo generator.

<a name="examine-terraform-file"></a>

## Examine the Terraform file in your source code

This tutorial uses the PartsUnlimited project, which is a sample eCommerce website developed using .NET Core. You will examine the Terraform file that defines the Azure resources required to deploy PartsUnlimited website.

1. Navigate to the project created earlier using the Azure DevOps Demo Generator.

1. In the **Repos** tab of **Azure Pipelines**, select the **terraform** branch. 

    ![Selecting the terraform branch](media/automate-terraform/select-branch.png)

    Make sure that you are now on the terraform branch and Terraform folder is there in the repo.

1. Select the **webapp.tf** file under the **Terraform** folder. Review the code.

	```
	 terraform {
	  required_version = ">= 0.11" 
	 backend "azurerm" {
	  storage_account_name = "__terraformstorageaccount__"
	    container_name       = "terraform"
	    key                  = "terraform.tfstate"
		access_key  ="__storagekey__"
	  features{}
		}
		}
	  provider "azurerm" {
	    version = "=2.0.0"
	features {}
	}
	resource "azurerm_resource_group" "dev" {
	  name     = "PULTerraform"
	  location = "West Europe"
	}
	
	resource "azurerm_app_service_plan" "dev" {
	  name                = "__appserviceplan__"
	  location            = "${azurerm_resource_group.dev.location}"
	  resource_group_name = "${azurerm_resource_group.dev.name}"
	
	  sku {
	    tier = "Free"
	    size = "F1"
	  }
	}
	
	resource "azurerm_app_service" "dev" {
	  name                = "__appservicename__"
	  location            = "${azurerm_resource_group.dev.location}"
	  resource_group_name = "${azurerm_resource_group.dev.name}"
	  app_service_plan_id = "${azurerm_app_service_plan.dev.id}"
	
	}
	```

    **webapp.tf** is a terraform configuration file. Terraform uses its own file format, called HCL (Hashicorp Configuration Language). The structure is similar to YAML. In this example, Terraform will deploy the Azure resource group, app service plan, and app service required to deploy the website. However, since the names of those resources are not yet known, they are marked with tokens that will be replaced with real values during the release pipeline.

    As an added benefit, this Infrastructure-as-Code (IaC) file can be managed as part of source control. You may learn more about working with Terraform and Azure in [this Terraform Basics lab](https://azurecitadel.com/automation/terraform/lab1/).

<a name="build-application"></a>
## YAML Pipeline Definition
The first step to this process will be to define the build/publish stage of the YAML Pipeline.  This stage will take the defined .tf files and publish them as artifacts of the pipeline to be used in our later deploy steps.  As part of this CI the process will be to initiate a Terraform plan to ensure the code being checked in does not violate the current state.

1. Navigatee to Pipelines->Pipelines and select "New Pipeline"
![Selecting New Pipeline](media/automate-terraform/new-pipeline.png)
## Summary

In this tutorial, you learned how to automate repeatable deployments with Terraform on Azure using Azure Pipelines.

## Clean up resources

This tutorial created an Azure DevOps project and some resources in Azure. If you're not going to continue to use these resources, delete them with the following steps:

1. [Delete the Azure DevOps project](../../organizations/projects/delete-project.md) created by the Azure DevOps Demo Generator.

1. All Azure resources created during this tutorial were assigned to either the **PULTerraform** or **terraformrg** resource groups. Deleting those two groups will delete the resources they contain. This can be done via the CLI or portal. The following example shows you how to delete the resource groups using Azure CLI.

    ```azurecli-interactive
    az group delete --name PULTerraform
    az group delete --name terraformrg
	```

## Next steps

> [!div class="nextstepaction"]
> [Terraform with Azure](/azure/developer/terraform/overview)
