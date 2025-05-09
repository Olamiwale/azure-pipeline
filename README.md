### This is a CI/CD pipeline that deploys Node.js application to Azure App Service Using Github action
    ________________________________________________________________________________________________________________


##### Step 1: Prepare the local environment, making sure that the .github/workflow/deploy.yml path is in the root folder

/root-folder

|──── app

      |─── server.js
      |─── package.json
      |─── Dockerfile
|───── .github

          |── workflows     
               |── deploy.yml
               
|──── README.md



#### Step 2: Prepare Azure Environment

   Create a resource group

     az group create --name <ResourceGroupName> --location eastus

   Create app service

     az appservice plan create --name <AppServicePlan> --resource-group <ResourceGroup> --sku B1 --location <Region>

   Create a web app on (app service)

    az webapp create --resource-group <ResourceGroupName> --plan <AppServicePlan> --name myApp --runtime "NODE:18-lts"

   Creating registry
   
    az acr create --resource-group <ResourceGroupName> --name <RegistryName> --sku Standard



#### Step 3: Build the Dockerfile: Navigate the Dockfile file on the local machine (vs code) and containerise the app

          FROM node:18-alpine
          WORKDIR /app
          COPY package*.json ./
          RUN npm install
          COPY . .
          EXPOSE 3000
          CMD ["node", "server.js"] 
 

#### Step 4: Create GitHub workflows (deploy.yml)

  Check the code of this repository for a detail workflow
 


#### Step 4: Set Up GitHub Secrets

   Set up azureCredentials (service principal). This is to authenticate with Azure using a Service Principal. This command will generate a JSON file

    az ad sp create-for-rbac --name "myGithubAction" --sdk-auth


   The response should look like this 
   
      ‘{
          "clientId": "6c535cb7-e39506-kgkff-56gg82d7",
          "clientSecret": "TyY8QkkfngnskgfNjyK-ma0K",
          "subscriptionId": "8acg gkfnkkdkkdkgdg09d2f",
          "tenantId": "34f758f58-ksknkng50bdkgns8b29",
          "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
          "resourceManagerEndpointUrl": "https://management.azure.com/",
          "activeDirectoryGraphResourceId": "https://graph.windows.net/",
          "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
          "galleryEndpointUrl": "https://gallery.azure.com/",
          "managementEndpointUrl": "https://management.core.windows.net/"
      } ’

   In the GitHub repository click on settings 
   -> Navigate to the left bar 
   -> secrets and variables 
   -> Action 
   -> New Repository secret 
   -> Add Name and paste the secret into the text box 
   -> Add secret
  

   Add Azure Publish Profile (This is to deploy the application to Azure App Service)

   In the Azure portal click on the menu bar
   -> Select app service 
   -> Click on the application created 
   -> Navigate to Download Publish Profile. Download the file (XML) copy and paste it into the GitHub secret as the second secret.

   Your Download should look like this 

    <?xml version="1.0" encoding="utf-8"?>
      <publishData>
        <publishProfile 
          profileName="briitzApp - Web Deploy" 
          publishMethod="MSDeploy" 
          publishUrl="briitzApp.scm.azurewebsites.net:443" 
          msdeploySite="briitzApp" 
          userName="$briitzApp" 
          userPWD="yourGeneratedPassword" 
          destinationAppUrl="https://briitzApp.azurewebsites.net" 
          SQLServerDBConnectionString="" 
          mySQLDBConnectionString="" 
          hostingProviderForumLink="" 
          controlPanelLink="" >
        <databases/>
    </publishProfile>
    </publishData>


*Ensure the secret_name used in the GitHub action is the same as the name used in the .GitHub/workflow file



#### Step 5:Assigning the Contributor Role. (This is necessary to grant permissions for deployment)

#Run the following command to assign the Contributor role to your service principal:

    az role assignment create --assignee <appId> --role Contributor --scope /subscriptions/<subscriptionId>
 
#Replace <appId> with the Client ID of the service principal.
#Replace <subscriptionId> with your Azure subscription ID.


#### Step 6: Push to Github and run the Pipeline

    git add .
    git commit -m "Set up CI/CD for Azure App Service"
    git push origin main

#### Step 7: Access the web app using this URL

    https://<myApp>.azurewebsites.net
