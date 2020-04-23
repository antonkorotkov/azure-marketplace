Sometimes we need to update the Azure Marketplace Octopus VM offering and it's not enough to simply update our [Azure Marketplace](https://github.com/OctopusDeploy/azure-marketplace) git repository. The screenshot below is what this document is referring to. (This process can take a few weeks to go through, be sure to leave as much time as you can)

![image](https://user-images.githubusercontent.com/1627582/80054655-370bd400-8563-11ea-9419-500be1851f5a.png)

What you will need:
-------------------

*   Access to LastPass so that you can get the password for the [**azure-marketplace@octopus.com**](mailto:azure-marketplace@octopus.com) **Microsoft Account** credentials
    
*   Access to the [Devops Mailbox](https://secure.helpscout.net/mailbox/3f6d5cfe44e1afb9/)\- because [**azure-marketplace@octopus.com**](mailto:azure-marketplace@octopus.com) is an alias of [**devops@octopus.com**](mailto:devops@octopus.com) so emails will be forwarded along.
    

Procedure
---------

1.  Test that your changes work by running through the Testing your ARM Template steps.
    
2.  Login to [https://cloudpartner.azure.com/](https://cloudpartner.azure.com/) (Login using [**azure-marketplace@octopus.com**](mailto:azure-marketplace@octopus.com) **Personal Microsoft Account)**
    
3.  Select the Octopus Deploy **Azure Application** offer, (It should be the live one)
    
4.  ![image](https://user-images.githubusercontent.com/1627582/80054677-4ab73a80-8563-11ea-974d-6e13c6ba1fd1.png)

5.  Compress your new marketplace package, it is **Critical** that you put all of your files in the root of the zip file, call this zip file what ever you like
    
6.  ![image](https://user-images.githubusercontent.com/1627582/80054696-586cc000-8563-11ea-9c76-4b077781595b.png)

7.  Upload the new package in the SKU section
    
8.  ![image](https://user-images.githubusercontent.com/1627582/80054710-64588200-8563-11ea-8b45-a7f72a903085.png)

9.  Now click Publish and wait for Microsoft to update the package, which can take 2 weeks if they run into problems.
    

  

Testing your ARM Template
-------------------------

The best way to test that your ARM template works, is to copy the \`Install-OctopusDeploy.ps1\` Powershell script to public blob storage, then specify its URL when you do a deployment.

1.  Install [Azure Powershell](https://docs.microsoft.com/en-us/powershell/azure/install-azurerm-ps?view=azurermps-5.0.0) prerequisites.
    
2.  Clone this [azure-marketplace](https://github.com/OctopusDeploy/azure-marketplace) repository to your local system drive then browse to the directory containing the `mainTemplate.json` file.
    
3.  Using Azure PowerShell, authenticate to Azure and select your desired subscription if applicable.
    
4.  Create an empty resource group
    
5.  Deploy the `mainTemplate.json` ARM Template to the empty resource group you created in step 4.
    

The script below will perform steps 3 to 5 from above:  
Please change the values as necessary to suit your environment, especially the passwords.

```
$ResourceGroupName = "Octopus-marketplace-rg1"
$TemplateFile = ".\mainTemplate.json"

$baseUrl = "https://<StorageAccountofPublicBlob>.blob.core.windows.net/<BlobName>"

$octopusAdminPassword = ConvertTo-SecureString -String "P@ssword!pleasechangeme" -AsPlainText -Force
$sqlAdminPassword = ConvertTo-SecureString -String "P@ssword!pleasechangeme" -AsPlainText -Force
$vmAdminPassword = ConvertTo-SecureString -String "P@ssword!pleasechangeme" -AsPlainText -Force

$ParameterObject = @{
    "octopusDnsName" = '<your desired dns prefix>'
    "licenseFullName" = '<your name>'
    "licenseOrganisationName" = '<your organisation>'
    "licenseEmailAddress" = '<your email address>'
}

Login-AzureRmAccount

Select-AzureRmSubscription -subscriptionId '<SUBSCRIPTION_ID>'

New-AzureRmResourceGroup -Name $ResourceGroupName -Location "australiasoutheast"

New-AzureRmResourceGroupDeployment -Name "octopusdeploy" `
                                   -ResourceGroupName $ResourceGroupName `
                                   -TemplateFile $TemplateFile `
                                   -TemplateParameterObject $ParameterObject `
                                   -octopusAdminPassword $octopusAdminPassword `
                                   -sqlAdminPassword $sqlAdminPassword `
                                   -vmAdminPassword $vmAdminPassword `
                                   -baseUrl $baseUrl
```
