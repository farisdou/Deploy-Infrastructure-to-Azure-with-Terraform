# Deploy Infrastructure to Azure with Terraform

Video Implentation:


---
## Goal
- Deploying a sample website to Azure, leveraging the power of Terraform to automate the process efficiently.
- Develop a solid understanding of key concepts and gain confidence in deploying a websites using this powerful combination.
- The backend.sh has azure cli commands to create the remote backend for terraform statefile.
---
## 📚 What I Learned
- ✔️ Understanding the fundamentals of infrastructure-as-code and its benefits
- ✔️ Setting up development environment with Azure and Terraform
- ✔️ Defining infrastructure as code with Terraform configuration files (HCL)
- ✔️ Creating and configuring Azure resources using Terraform modules
- ✔️ Managing secrets and environment variables securely
- ✔️ Best practices for maintaining and updating your Terraform Code
---
## Steps

- ### Introduction & Requirements
- ### Terraform & how it works
- ### Installing Azure CLI and Terraform
- ### Writing Terraform Code
- ### Deploying infrastructure to Azure
- ### Part 2: Advanced Terraform Concepts
- ### Remote Backend for Terraform State
- ### Variables in Terraform
- ### Terraform Destroy
- ### Conclusion
---
### Introduction
What is Terraform?
Terraform was developed by Hashicorp. It is a configuration orchestration tool that is incredible for provisioning, adjusting and destroying the virtual server environments. It is available both as a DevOps-as-a-Service enterprise-grade from Hashicorp and as an open-source solution, which allows you to work with a variety of Cloud Service Providers to create multi-cloud ecosystems.
![image](https://github.com/FarisDou/Deploy-Infrastructure-to-Azure-with-Terraform/assets/109401839/18fd70b4-5ef6-4490-8512-081bd1df80bf)

Go to Hashicorp terraform website and download terraform that matches your operating system. 
In my case, I am using windows on a 64bit CPU, once downloaded extract the file and move it to a location you can find it later. Personally, I moved it to "C:/Terraform" so it is easy to remember. 

Afterwards, open your start menu and type, "env" for "edit system environment variable" window and open that. |
Select " Environment Variable" 
Under system variables select "Path", Edit, and include your new terraform directory. 
![path](https://cdn.discordapp.com/attachments/1235529568056512553/1251614036400013312/image.png?ex=666f37e8&is=666de668&hm=ef177a0c29c7156f5db935419c1cc53d4272afadff8a72d34d3da641fdc1563c&)

Once that is done, go to Command Prompt and you can verify it is installed with the command, "terraform -v".

![image](https://github.com/FarisDou/Deploy-Infrastructure-to-Azure-with-Terraform/assets/109401839/064f25e5-b60d-4e94-8bd6-adabb1a91f15)
Verified we have terraform installed on windows.

### Pre-Requisites | Installing Azure CLI and Terraform
[Download .NET 8.0 SDK](https://download.visualstudio.microsoft.com/download/pr/b6f19ef3-52ca-40b1-b78b-0712d3c8bf4d/426bd0d376479d551ce4d5ac0ecf63a5/dotnet-sdk-8.0.302-win-x64.exe)

[Command Lines Install Azure](https://learn.microsoft.com/en-us/powershell/azure/install-azps-windows?view=azps-12.0.0&tabs=powershell&pivots=windows-psgallery)

[MSI Azure CLI Download](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli#install-or-update)

- Get-ExecutionPolicy -List
- Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
- Install-Module -Name Az -Repository PSGallery -Force
- Update-Module -Name Az -Force
- Connect-AzAccount -Tenant {ID}

![Connect](https://cdn.discordapp.com/attachments/1235529568056512553/1251644024683761775/image.png?ex=666f53d5&is=666e0255&hm=98af49c9718a1d000e1733726e6392e3620dd3d540ffcf28690a1ca3298453ad&)


### Writing Terraform Code, Main.tf
```provider "azurerm" {
    features {}
}

# Create a resource group
resource "azurerm_resource_group" "resource_group" {
    name = "rg-terraform-demo"
    location = "eastus"
}

# Create a Storage Account
resource "azurerm_storage_account" "storage_account" {
    name = "terraform-azure-houston-4"
    resource_group_name = azurerm_resource_group.resource_group.name
    location = azurerm_resource_group.resource_group.location
    account_tier ="Standard"
    account_replication_type = "LRS"
    account_kind = "StorageV2" # terraform 

    static_website {
      index_document = "index.html"
    }
}

# Add index.html file
resource "azurerm_storage_blob" "blob" {
    name = "index.html"
    storage_account_name = azurerm_storage_account.storage_account.name
    storage_container_name = "$web" 
    type = "Block"
    content_type = "text/html"
    source_content = "<h1> Hello World, this is a website that was used to deploy terraform within azure </h1>"
}
```
### Deploying infrastructure to Azure & Backend

Within terminal, use the command, "terraform init"
![init](https://cdn.discordapp.com/attachments/1086819278965194792/1251611214794985513/image.png?ex=666f3547&is=666de3c7&hm=fcd1b58906343ff166ef6f758121e08defc8cbb47ef65e23859349601fc81812&)

``````python
az group create --name tf-state-rg --location eastus
``````
Then run to create the storage account, 

 ``````python
 az storage account create --name tftxsa1 --location eastus --resource-group tf-state-rg
 ``````

If the line of code dont work above with the error,

"(SubscriptionNotFound) Subscription XXXX-XXXX-XXX-XXX-XXXX was not found.
Code: SubscriptionNotFound
Message: Subscription XXXX-XXXX-XXX-XXX-XXXX was not found. "

The "Subscription not found" error usually occurs in Azure Site Recovery when the Azure subscription associated with the Azure Site Recovery service cannot be found or accessed.

![Resource_Provider](https://cdn.discordapp.com/attachments/1235529568056512553/1251655562157096970/f9b053d2-ebee-40cb-a59d-93420d028b7b.png?ex=666f5e94&is=666e0d14&hm=fff322930787207218b68d2ff5029b759534acda3063bc68e49436584311299b&)

Get Account Key, 

```ACCOUNT_KEY=$(az storage account keys list --resource-group tf-state-rg --account-name tftxsa1 --query '[0].value' -o tsv)```

 Then to create the storage container, 

``` az storage container create --account-name tftxsa1 --name tfstatecon --public-access off --account-key $ACCOUNT_KEY```

---

- ### Part 2: Advanced Terraform Concepts & Variables
Setting Variables from Main.tf, create a variable.tf file and determine variables. 

``` variable "location" {
  description = "The Azure Region in which all resources in this project should be created."
}

variable "resource_group_name" {
  description = "The name of the resource group in which to create the storage account."
}

variable "storage_account_name" {
  description = "The name of the storage account to create."
}

variable "source_content" {
  description = "The content of the index.html file."
}

variable "index_document" {
  description = "The name of the index document."
} 
```
 After, give those variables values in a new file, dev.tfvars; 

 ``` 
 location = "eastus"
resource_group_name = "tf-state-rg"
storage_account_name = "terraformazurefaris01"
index_document = "index.html"
source_content = "<h1> Salute, this website is deployed using terraform."
``` 

So when we replace the variables in main.tf the code is cleaner. 

``` 
# Create a resource group
resource "azurerm_resource_group" "tf-state-rg" {
    name = var.resource_group_name
    location = var.location
}

# Create a Storage Account
resource "azurerm_storage_account" "storage_account" {
    name = var.storage_account_name
    resource_group_name = azurerm_resource_group.tf-state-rg.name
    account_tier ="Standard"
    location = var.location
    account_replication_type = "LRS"
    account_kind = "StorageV2"

    static_website {
      index_document = var.index_document
    }
}

# Add index.html file
resource "azurerm_storage_blob" "tfstatecon" {
    name = var.index_document
    storage_account_name = azurerm_storage_account.storage_account.name
    storage_container_name = "$web" 
    type = "Block"
    content_type = "text/html"
    source_content = var.source_content
``` 


- ### Alternative Path
- Upload terraform files into azure and init terraform, plan,
![image](https://github.com/FarisDou/Deploy-Infrastructure-to-Azure-with-Terraform/assets/109401839/311d96c9-4ced-4157-9711-f0f43567f7e9)

terrafrom apply 
enter values
accept actions: yes

- ### Terraform Destroy
- ### Conclusion

