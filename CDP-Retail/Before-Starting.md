
# Azure Synapse Analytics WWI Setup Guide

**Contents**

<!-- TOC -->

- [Azure Synapse Analytics WWI setup guide](#azure-synapse-analytics-wwi-setup-guide)
  - [Requirements](#requirements)
  - [Before Starting](#before-starting)
    - [Task 1: Create a Power BI Workspace](#task-1-create-a-power-bi-workspace)
    - [Task 2: Create a Power BI Streaming Dataset](#task-2-create-a-power-bi-streaming-dataset)
    - [Task 3: Create a resource group in Azure](#task-3-create-a-resource-group-in-azure)
    - [Task 4: Create Azure Synapse Analytics workspace](#task-4-create-azure-synapse-analytics-workspace)
    - [Task 5: Download artifacts](#task-5-download-artifacts)
    - [Task 6: Establish a user context](#task-6-establish-a-user-context)
    - [Task 7: Run environment setup PowerShell script](#task-7-run-environment-setup-powershell-script)
    - [Task 8: Location Analytics Streaming Dataset Setup](#task-8-location-analytics-streaming-dataset-setup)
    - [Task 9: Twitter Analytics Streaming Dataset Setup](#task-9-twitter-analytics-streaming-dataset-setup)
    - [Task 10: Twitter Analytics Report](#task-10-twitter-analytics-report)
    - [Task 11: Location Analytics Real-Time Report](#task-11-location-analytics-real-time-report)

<!-- /TOC -->

## Requirements

1. An Azure Account with the ability to create an Azure Synapse Workspace
2. Make sure the following resource providers are registered for your Azure Subscription.  

* Microsoft.Sql
* Microsoft.Synapse
* Microsoft.StreamAnalytics
* Microsoft.EventHub  

See [further documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types#azure-portal) for more information on registerting resource providers on the Azure Portal.

3. A Power BI Pro or Premium account to host Power BI reports, dashboards, and configuration of streaming datasets.
4. A Twitter developer account (Optional for Twitter Real-Time Power BI Report)

## Before starting

### Task 1: Create a Power BI Workspace

1. Sign in into the [Power BI Portal](https://powerbi.microsoft.com/en-us/) using your Azure credentials.

2. Select **Workspaces** from the left menu and select **Create a workspace** to create a new Power BI Workspace for your new environment.

![Left hamburger menu on Power BI Portal is shown. Workspaces collection is open. Create a workspace button is highligted.](media/powerbi-workspace-create.png)

3. Name your workspace **CDP** and select **Save** to finish creating the workspace.

![Workspace creation screen is open. The name for the workspace is filled as CDP. The Save button is highlighted.](media/powerbi-workspace-create-2.png)

3. Once your workspace is created you will need to take note of the **workspace Id**. You can find the workspace Id for your workspace in the URL bar in your browser as shown in the screenshot below. Finally, select **skip** to close the Welcome wizard on the page.

![The newly created workspace is open. The workspace id in the URL is highlighted. The skip button for the welcome wizard is selected.](media/powerbi-workspace-id.png)

### Task 2: Create a Power BI Streaming Dataset

1. Sign in into the [Power BI Portal](https://powerbi.microsoft.com/en-us/) using your Azure credentials.

2. Go to your workspace and select **Create** 

 ![Create menu is open and the streaming dataset option is highlighted.](media/powerbi-create-streaming-dataset.png)

3. Select **API** and select **Next** to continue. 

 ![New streaming dataset options are listed. API option is selected and the Next button is highligted.](media/powerbi-streamingdataset-api.png)
 
4. Enter **locationstream** as your **dataset name** and enter the field names and types listed below:

| Field Name       | Type     |
|------------------|----------|
| Category         | Text     |
| CustomerSegment  | Text     |
| DeptID           | Number   |
| DeptName         | Text     |
| EntryTime        | DateTime |
| ItemName         | Text     |
| Price            | Number   |
| Qty              | Number   |
| Sentiment        | Text     |
| StoreID          | Number   |
| StoreName        | Text     |
| TransactionID    | Number   |
| VisitorId        | Number   |
| VisitTime        | Number   |
| VisitType        | Text     |

 ![Streaming dataset schema configuration window is shown. Dataset name and sample configuration is highlighted.](media/powerbi-streamingdataset-schema.png)
 
Make sure **Historic data analysis** is enabled. Select **Create** to proceed.

 ![Dataset schema is shown. Historic data analysis option is enabled. The Create button is highlighted.](media/powerbi-streamingdataset-historic.png)
 
5. Take note of the given **Push URL** to be used during the following setup tasks. Select **Done** to close the window.

 ![Dataset creation message is shown. The Push URL is highligted.](media/powerbi-streamingdataset-endpoint.png)

### Task 3: Create a resource group in Azure

> If you have a CloudLabs environment, please skip this step and use the resource group you are provided.

1. Log into the [Azure Portal](https://portal.azure.com) using your Azure credentials.

2. On the Azure Portal home screen, select the **+ Create a resource** tile.

    ![A portion of the Azure Portal home screen is displayed with the + Create a resource tile highlighted.](media/bhol_createaresource.png)

3. In the **Search the Marketplace** text box, type **Resource group** and press the **Enter** key.

    ![On the new resource screen Resource group is entered as a search term.](media/bhol_searchmarketplaceresourcegroup.png)

4. Select the **Create** button on the **Resource group** overview page.

5. On the **Create a resource group** screen, select your desired Subscription and Region. For Resource group, enter **Synapse-WWI-Lab**, then select the **Review + Create** button.

    ![The Create a resource group form is displayed populated with Synapse-MCW as the resource group name.](media/bhol_resourcegroupform.png)

6. Select the **Create** button once validation has passed.

### Task 4: Create Azure Synapse Analytics workspace

1. Deploy the workspace through the following Azure ARM template (press the button below):

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2FAzure-Analytics-and-AI-Engagement%2Fmain%2FCDP-Retail%2Fartifacts%2Fenvironment-setup%2Fautomation%2F00-asa-workspace-core.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png" /></a>

2. On the **Custom deployment** form fill in the fields described below.

* **Subscription**: Select your desired subscription for the deployment.
* **Resouce group**: Select the **Synapse-WWI-Lab** resource group you previously created.
* **Unique Suffix**: This unique suffix will be used naming resources that will created as part of your deployment. Make sure you follow correct Azure [Resource naming](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#resource-naming) conventions.
* **SQL Administrator Login Password**: Provide a strong password for the SQLPool that will be created as part of your deployment. [Visit here](https://docs.microsoft.com/en-us/sql/relational-databases/security/password-policy?view=sql-server-ver15#password-complexity) to read about password rules in place. Your password will be needed during the next steps. Make sure you have your password noted and secured.
* **Location**: The datacenter where your Azure Synapse environment will be created.
* **Streaming Dataset**: The name of your Twitter PowerBI Dataset that will be automatically created by Azure Stream Analytics. Feel free to leave the default value.
* **Pbi Worksapce Id**: The Power BI workspace ID for your default workspace on the Power BI Portal. 
* **Streaming Url**: The **Push Url** you received at the end of Task 1 in the Setup Guide.
* **Event Hub Name**: The name of the event hub that will be used to dump tweets recevied from Twitter through an Azure Function. Feel free to leave the default value.
* **Asa name**: The name of the Azure Stream Analytics Job that will be created to read data from Event Hubs and send to a Power BI streaming dataset.

    ![The Custom deployment form is displayed with sample values filled in.](media/bhol_customdeploymentform.png)
  
    > **Important**: The `location` field under 'Settings' will list the Azure regions where Azure Synapse Analytics (Preview) is available as of July 2020. This will help you find a region where the service is available without being limited to where the resource group is defined.

3. Check the **I agree to the terms and conditions stated above**, then select the **Purchase** button. The provisioning of your deployment resources will take approximately 13 minutes. **Wait** until provisioning successfully completes before continuing. You will need the resources in place before running the scripts below.

    > **Note**: You may experience a deployment step failing in regards to Role Assignment. This error may safely be ignored.

### Task 5: Download artifacts

1. In the Azure Portal, open the Azure Cloud Shell by selecting its icon from the right side of the top toolbar.

    ![A portion of the Azure Portal taskbar is displayed with the Cloud Shell icon highlighted.](media/bhol_azurecloudshellmenu.png)

    > **Note**: If you are prompted to choose a shell, select **Powershell**, and if asked to create a storage account for the Cloud Shell, agree to have it created.
    
    ![Cloud Shell storage account creation screen is shown. Create storage button is selected.](media/cloud-shell-storage.png)

2. In the Cloud Shell window, enter the following command to clone the repository files.

    ```PowerShell
    git clone https://github.com/microsoft/Azure-Analytics-and-AI-Engagement.git synapse-wwi
    ```
    
    ![The Azure Portal with Cloud shell opened. Git clone command is typed into the cloud shell terminal ready for execution.](media/cloud-shell-git-clone.png)

3. Keep the Cloud Shell open.

### Task 6: Establish a user context

1. In the Cloud Shell, execute the following command:

    ```cli
    az login
    ```

2. A message will be displayed asking you to open a new tab in your web browser, navigate to [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) and enter the code you have been given for authentication.

   ![A message is displayed indicating to enter an authentication code on the device login page.](media/bhol_devicelogin.png)

   ![A dialog is shown requesting the entry of a code.](media/bhol_clicodescreen.png)

3. Once complete, you may close the tab from the previous step and return to the Cloud Shell.

   ![The JSON result showing the subscription details.](media/shell-login-result.png)

### Task 7: Run environment setup PowerShell script

When executing the script below, it is important to let the scripts run to completion. Some tasks may take longer than others to run. When a script completes execution, you will be returned to a command prompt. The total runtime of all steps in this task will take approximately 15 minutes.

1. In the Cloud Shell, change the current directory to the **automation** folder of the cloned repository by executing the following:

    ```PowerShell
    cd './synapse-wwi/CDP-Retail/artifacts/environment-setup/automation'
    ```

2. Execute the **01-environment-setup.ps1** script by executing the following command:

    ```PowerShell
    ./01-environment-setup.ps1
    ```

    You will be prompted to specify the size of the data you want to populate into the Sales table. You can either chose the small data size with 3 million records, or the large data size with 30 billion records. Choosing 30 billion records will have the script scale your SQL Pool to DW3000c during data transfer, which might take up to 4 hours. 
    
    ![The Azure Cloud Shell window is displayed with the choices of different data sizes that can be loaded into the environment by the script.](media/setup-guide-data-size.png)
    
    You may be prompted to enter the name of your desired Azure Subscription. You can copy and paste the value from the list to select one.   

    You will also be prompted for the following information for this script:

    | Prompt |
    |--------|
    | Enter the SQL Administrator password you used in the deployment |

    ![The Azure Cloud Shell window is displayed with a sample of the output from the preceding command.](media/bhol_sampleshelloutput.png)
    
    Select the resource group you selected during Task 3.2. This will make sure automation runs against the correct environment you provisioned in Azure.
    
    ![The Azure Cloud Shell window is displayed with a selection of resource groups the user owns.](media/setup-resource-group-selection.png)

    You will be asked to go through another round of device login for the Power BI Gateway access. Repeat the steps in Task 4.2 and 4.3 to complete the process.
    
    Finally, you will be prompted for the default Power BI workspace Id for your account.
    ![The default Power BI Workspace ID is requested on the console.](media/setup-powerbi-workspace-id.png)

    During the execution of the automation script you may be prompted to approve installations from PS-Gallery. Please approve to proceed with the automation.   

    ![The Azure Cloud Shell window is displayed with a sample of the output from the preceding command.](media/untrusted-repo.png)
    
3. Sign in into the [Power BI Portal](https://powerbi.microsoft.com/en-us/) using your Azure credentials.

4. From the hamburger menu select **Workspaces** to access the list of workspaces available to you. Select your workspace.

![The workspaces button from the hamburger menu is selected to list workspaces available.](media/powerbi_workspace_selection.png)

5. Select the **Settings** icon from the top right bar, and select **Settings** again to navigate to the settings page.

![The settings button on the Power BI portal clicked and the Settings selection on the context menu selected.](media/powerbi_settings_menu.png)

6. Select **datasets** tab to access the list of datasets available. Then select `2-Billion Rows Demo` dataset to access its settings. From the settings page open **Data source credentials** and select **Edit credentials**.

![The datasets tab is selected. From the list of datasets 2-Billion Rows Demo is selected. Edit credentials will be selected next. ](media/powerbi_datasource_credentials.png)

7. Select **Microsoft Account** for the **Authentication method** and select **Sign In** to complete the process.

![From the list of authentication methods Microsoft Account is picked. The sign in button is selected. ](media/powerbi_datasource_credentials-update.png)

### Task 8: Location Analytics Streaming Dataset Setup

1. Log into the [Azure Portal](https://portal.azure.com) using your Azure credentials.

2. On the Azure Portal home screen, go to **Search** and search for **locfunction**. Once the azure function is found select it to proceed.

![Azure Portal is open. Search box is used to search for loc. Result shows a function app with a name that starts with locfunction. ](media/setup-location-function.png)

3. Select **Functions** from the left menu to list the functions available in the function app. Next, select the function named **Start**.

![Functions tab is selected in the function app and a function named start is highligted.](media/setup-location-function-start.png)

4. In the **Overview** tab select **Get Function Url** and select **Copy to clipboard** to copy the Url for the function into your clipboard.

![Get function url option is selected and copy to clipboard button is highlighted.](media/setup-location-function-uri.png)

5. Open a new browser tab, paste the url and navigate to function endpoint to start location analytics data generator.

![A new browser tab is navigated to the location function endpoint and the page shows a message stating the location analytics is started](media/setup-location-analytics-start.png)

6. Once the Azure Function starts you can start building Power BI real-time reports for your new Power BI dataset. The name of the dataset is the value you provided during Task 2.4 of this setup guide.

![A new locationstream dataset is highligted on the Power BI portal under the datasets collection listing page.](media/setup-powerbi-locationstream-dataset.png)

### Task 9: Twitter Analytics Streaming Dataset Setup

1. Log into the [Azure Portal](https://portal.azure.com) using your Azure credentials.

2. On the Azure Portal home screen, go to **Search** and search for **tweets**. If you provided a different name for you Azure Stream Analytics Job use the same name for your search term.

![Azure Portal is open. Search box is used to search for tweets. Result shows an Azure Stream Analytics Job with a name that starts with tweets. ](media/setup-asa-start.png)

3. Select **Start** to start the Azure Stream Analytics Job.

![The stream analytics job is open and the start button is highlighted. ](media/setup-asa-start-selected.png)

4. Once the job starts gathering data you can start building Power BI real-time reports for your new Power BI dataset that will be create by the Azure Stream Analytics job. The name of the dataset is based on the value you provided for the configuration parameter named **Streaming dataset** during template deployment.

### Task 10: Twitter Analytics Report

1. Launch [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) on your machine.

2. **Sign in** using your Power BI account credentials.

![Power BI Desktop is open. Account sign-in menu is on screen. Sign-in with a different account is highlighted.](media/powerbi-desktop-signin.png)

3. Select **Get Data** from the **Home** toolbar. Select **Power BI datasets** to proceed.

![Get Data menu is opened. Power BI datasets option is highligted](media/powerbi-desktop-dataset-import.png)

4. Select **Tweetsout** from the list of datasets. The name of the dataset is based on the value you provided for the configuration parameter named **Streaming dataset** during template deployment. It might be different in your case if you have changed the default value during deployment.

> Please note, it make take several minutes for this dataset to show up on the list. The Stream Analytics job you started in the previous task sends data to Power BI. There is an initial delay before the dataset initially appears after receiving the initial load of streaming data. If you do not see it listed, select Cancel to close this dialog, then re-open it.

![Dataset selection screen is open. Tweetsout dataset is selected. Create button is highlighted.](media/powerbi-desktop-tweetsout.png)

5. To create **Tweets by City Visualization**, which is the count of neutral sentiment tweets over a geographical region, select the map icon from the Visualization tray.

![The map visualization is selected.](media/powerbi-desktop-twitter.png)

6. Select **city**, **sentiment** and **tweet** fields to be included into the report. Move the **Tweet** field into the **Size** section.

![City, Sentiment and Tweet fields are selected. Location, legend and Size settings are set.](media/powerbi-desktop-twitter-report.png)

7. Save your report locally, and then publish it to your workspace to be used during your demo. _This is the workspace you created in Task 1_.

### Task 11: Location Analytics Real-Time Report

1. Launch [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) on your machine.

2. **Sign in** using your Power BI account credentials.

![Power BI Desktop is open. Account sign-in menu is on screen. Sign-in with a different account is highlighted.](media/powerbi-desktop-signin.png)

3. Select **Get Data** from the **Home** toolbar. Select **Power BI datasets** to proceed.

![Get Data menu is opened. Power BI datasets option is highligted](media/powerbi-desktop-dataset-import.png)

4. Select **locationstream** from the list of datasets. The name of the dataset is the value you provided during Task 2.4 of this setup guide. It might be different in your case if you picked a different name.

![Dataset selection screen is open. locationstream dataset is selected. Create button is highlighted.](media/powerbi-desktop-location-dataset.png)

5. To create a **Lined and stacked column Chart** for Avg Visit Time and Visitors by customer segment and department with the help of real-time data, select the map icon from the Visualization tray.

![The Lined and stacked column Chart visualization is selected.](media/powerbi-desktop-location-chart.png)

6. Select **CustomerSegment**, **DeptName**, **VisitorId** and **VisitTime** fields to be included into the report. 

![CustomerSegment, DeptName, VisitorId and VisitTime fields are selected.](media/powerbi-desktop-location-fields.png)

7. Move **VisitTime** to Column Values and select to use its **average** value. Move **VisitorId** to Line values and select its **count** value to be used in the report.

![Column values are set to the average of visittime. Line values are set to the count of VisitorIds.](media/powerbi-desktop-location-field-setting.png)

8. Save and publish your report to your workspace to be used during your demo.









