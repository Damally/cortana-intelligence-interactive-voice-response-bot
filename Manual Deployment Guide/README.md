The solution guide below provides a full set of instructions on how to put together a intelligent multi language call center solution using Microsoft Cognitive Services and Cortana Intelligence Suite.

# Solution Guide

## Abstract

The guide is designed for a typical call center scenario of a hypothetical
company called Contoso Insurance. Contoso Insurance provides claims,
loans and grant services to its customers and customers typically call
the company to enquire about the status of these services. Contoso
Insurance plans to automate their call center for this scenario and
reduce the need for human operators.

The solution utilizes the Speech to Text and LUIS API from Microsoft’s
Cognitive Services offering. It supports two languages, English and
Chinese.

This document explains how to build the solution piece by piece. The
manual process gives an implementer an inside view on how the solution
is built and an understanding of each of the components.


## Requirements

This section contains required accounts and software you will need to
create this solution.

1.  Windows workstation

2.  Clone of this repository.

3.  A Microsoft Azure subscription.

4.  A network connection

5.  [Microsoft Visual
    Studio 2015](https://www.visualstudio.com/post-download-vs?sku=community&clcid=0x409&downloadrename=true)

6.  [SQL Server Management
    Studio](https://msdn.microsoft.com/en-us/library/mt238290.aspx) or
    another similar tool to access a SQL server database.

## Architecture

The image in this section shows the overall architecture of the Call
Center solution, the remainder of this document describes it in detail.

![](images/arch-diag.png)

The solution could be divided into following four steps

1.  Capture and recognize what a user says using the Speech APIs 

2.  Understand user’s intent using LUIS API 

3.  Generate an appropriate response by querying the database.

4.  Speak the response to the user using the Speech APIs.

### Setup Steps

The remainder of this document walks the reader through the creation of
many different Cortana Intelligence Suite services with the end result
of replicating the architecture defined previously.

As there are several services, it is suggested to group these services
under a single Azure Resource Group. For more information about Azure
Resource Groups, read [*here*](https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/)

Similarly, we want to use a common name for the different services we
are creating. The remainder of this document will use the assumption
that the base service name is:

callcentersolution\[UI\]\[n\]

Where \[UI\] is the user's initials and n is a random integer that you
choose. Characters must be entered lowercase. Several services, such as
Azure Storage, require a unique name for the storage account across a
region and hence this format should provide the user with a unique
identifier.

So for example, Steven X. Smith might use a base service name of
***callcentersolutionsxs01***

Before proceeding further make sure you have downloaded the solution
package zip file with all the resources.

### 1. Create a new Azure Resource Group

-   Navigate to ***portal.azure.com*** and log in to your account.

-   On the left tab click ***Resource Groups***

-   In the resource groups page that appears, click Add

-   Provide a name ***callcentersolution\[UI\]\[n\]-rg***

-   Set the location to **Central US**

-  Select **Pin to Dashboard** option

-  Click ***Create***


### 2. Create Cognitive Services account for Language Understanding Intelligent Service (LUIS)

-   Navigate to ***portal.azure.com*** and log in to your account.

-  On the left tab click ***New***

-   Search for ***Cognitive Services APIs,*** select it and click **create**

-  In the create screen, for Account name, enter ***callcentersolution\[UI\]\[n\]-luis***

-  Select the correct subscription

-   For API type, select **Language Understanding Intelligent Service**

-   For Location, select **West US** (In future it would be location
    independent i.e. global)

-   For Pricing, select **S0 Standard** tier. (You can change the tier
    later based on your usage)

-   For Resource group, select ***Use Existing*** and then select ***callcentersolution\[UI\]\[n\]-rg***

-   Open Legal terms and click I Agree.

-   Click ***Create.***

-   It would take a minute or two for Cognitive Services API account to be created, navigate to the dashboard on Azure Portal.

-   On the dashboard, click on your resource group,
    ***callcentersolution\[UI\]\[n\]-rg*** and then click on the newly
    created LUIS service account, ***callcentersolution\[UI\]\[n\]-luis***

-   Go to Settings &gt; Keys, make a note of Key 1.



### 3. Create Cognitive Services account for Bing Speech API

-   Navigate to ***portal.azure.com*** and log in to your account.

-   On the left tab click ***New***

-   Search for ***Cognitive Services APIs,*** select it and click
    **create**

-   For Account name, enter ***callcentersolution\[UI\]\[n\]-speech***

-   Select the correct subscription

-   For API type, select **Bing Speech API**

-   For Location, select **West US**.

-   For Pricing, select **Standard**.

-   For Resource group, select ***callcentersolution\[UI\]\[n\]-rg***

-   Open Legal terms and click I Agree.

-   Click ***Create***

-   Go to the dashboard of your Azure account.

-   On the dashboard, click on your resource group,
    ***callcentersolution\[UI\]\[n\]-rg*** and then click on the newly
    created Speech service account,
    ***callcentersolution\[UI\]\[n\]-speech***

-   Go to Settings &gt; Keys, make a note of Key 1 and Key 2.

### 4. Create two LUIS applications for English and Chinese

We will first create LUIS application that understands intents for
English Speaker.

-   Navigate to <https://www.luis.ai> and login with a Microsoft account.

-   Click the **My Applications** tab on the top right.

-   Click on the **New App** button and select the **Import existing application** option.

-   Navigate to LUIS models folder in the solution package, select the **ContosoInsurance-CallCenter-English.json** file.

-   Provide an optional name for your app, by default it would pick the name of the imported json file as the app name.

-   Click Import, this would take few minutes to complete.

-   On the newly created application page, click on the **Train** button on the bottom left. This step will also take a few minutes.

-   When training has completed, a **Publish** button will appear in the left sidebar. Click the **Publish** button to load a publishing options pane, then click the **Publish as web service** button at the upper-right of the pane.

-   Close the Publish dialog and click on the App Settings button and make a note of the **App Id**.

_Next, we will create LUIS Application that understands intent for Chinese speaker. Follow the exact same steps as above, except select the  **ContosoInsurance-CallCenter-Chinese.json** file instead of the json file for English. Publish the app and make a note of the App Id_

Now that we have LUIS applications created for English and Chinese, we need to add our LUIS Account key that we obtained from the Azure portal to these applications. 
 
-   In the [LUIS portal](luis.ai), click on your account name on top right and select **My Settings**.

-   Under My Settings, go to Subscription Keys section and add the **LUIS API Account Key** that you saved in the previous step and click on **Add Key** button.

-   Make sure the key shows up in Assigned Key section for your applications.

### 5. Azure SQL Server and Database

We need to create an Azure SQL Database to store customer information and status of their claims. The solution would look customers up in this database.

-   Navigate to ***portal.azure.com*** and login in to your account.

-   On the left tab click ***New&gt;Data and Storage&gt;SQL Database***

-   Enter the name ***callcentersolution\[UI\]\[n\]-db*** for the database name

-   Resource group: Previously created
    ***callcentersolution\[UI\]\[n\]-rg***

-   Select blank database for ***source***

-   Under Server click the arrow and choose ***Create a new server***

    -   Name: **callcentersolution\[UI\]\[n\]-server**

    -   Enter in an administrator account name and password and save it
        to the table below.

    -   Choose **Central US** as the location to keep the resource in
        the same region as the rest of the services.

    -   Leave other fields to be default.

    -   Click OK

-   Click ***Create***

-   Wait for the database and server to be created.

-   Go to the dashboard of your Azure account.

-   On the dashboard, click on your resource group,
    ***callcentersolution\[UI\]\[n\]-rg*** and then click on the newly created SQL Server in the list of resources.

-   Under ***Settings*** for the new server, click ***Firewall*** and create a rule called ***open*** with the IP range of 0.0.0.0 to 255.255.255.255. This will allow you to access the database from your desktop. Click ***Save.***

    **NOTE:** This firewall rule is not recommended for production level systems but for this solution it is acceptable. You will want to set this rule to the IP range of your secure system.

-   Click on the SQL Server Database that you just created, under
    properties tab click on “**show database connection strings**”.

-   Save the connection string for ADO.NET along with username and password

-   Launch [*SQL Server Management Studio*](https://msdn.microsoft.com/en-us/library/mt238290.aspx) (SSMS), or a similar tool, and connect to the database with the information you recorded previously. Following instructions are for SSMS

    - NOTE: The server name in most tools will require the full name: 
    **callcentersolution\[UI\]\[n\]-server.database.windows.net,1433**

    -   For Authentication, select SQL Server Authentication and enter the
    Login and Password

    -   Click on the ***callcentersolution*\[UI\]\[n\]-*db*** that you created on the server.

    -   Right click on the database and select ***New Query***.

    -   Copy and execute the SQL script located in the package directory
    ***SQL Script\\databasescript.sql to*** create the necessary table for the solution <br/>
     NOTE: If you get permission denied error, close SSMS and try running it as administrator (Right Click &gt; Run as administrator).

### 6. Build the source code with new credentials.

-   You will need Visual Studio to work with code, you can install it from [here](https://www.visualstudio.com/post-download-vs?sku=community&clcid=0x409&downloadrename=true). The version of the Visual Studio should not matter, the guide has been tested on Visual Studio 2015.

-   Navigate to the CI-Callcenterdemo folder and double click on the solution file (.sln extension), this will open
    the solution in Visual Studio.

-   Open **App.config** file and fill in the values from the above tables under appSettings section. Following are the values you need to change

```
<!-- This is the appID for the English Version of LUIS Model -->
<add key="luisAppID" value="CHANGE_ME" />

<!-- This is the appID for the Chinese Version of LUIS Model -->
<add key="luisAppIDChinese" value="CHANGE_ME" />

<!-- This is the key of the LUIS API Cognitive services account that you want    to be charged, this will be under Cognitive Services in your subscription -->
<add key="luisAPIAccountKey" value="CHANGE_ME" />

<!-- This is the primary key of the Speech API Cognitive services account that you want to be charged, it will be under Cognitive Services in your subscription                                                         -->
<add key="speechAPIAccountKey" value="CHANGE_ME" />

<!-- This is the secondary key of the Speech API Cognitive services account that you want to be charged, this will be under Cognitive Services in your subscription -->
<add key="speechAPISecondaryAccountKey" value="CHANGE_ME" />

<!-- Azure SQL Connection String, you can get it from portal.azure.com -->
<add key="dbConnectionString" value="CHANGE_ME"/>    
```

-  To build your code, select Debug from Solution Configurations dropdown and select x64 for solution platforms dropdown. If you don't see x64 in the drop down, click on the Configuration Manager in solution platforms dropdown (next to the green start icon). In the Configuration Manager window click on the Active solution platform dropdown, select new and then add x64 as one of the solution platform.

-  Click on the green start icon.

-   You should now have a working solution.

-    To Test the solution, you need to have working speaker and microphone
      - Dial 132307, you will hear a telephone ring.
      - When the system prompts, ask the question - " What is the status of my claim?"
      - When prompted for CRN, say "56788"

      **NOTE: Please watch the solution video (SolutionVideo-English.mp4) to see it in action.**

### 7.  Publishing the solution

-   Click Build and then select “Publish CI\_CallCenterDemo”

-   This will open the publish wizard.

-   On Publish wizard, select the location on your machine where you want to publish the solution.

-  Click Next, and select From CD-ROM or DVD-ROM.

-   On the next screen, select “The application will not check for updates”

-   Click Next and then select Finish.

-   The publish folder will have an .exe file, you can install the solution locally using this file.

    **NOTE: If the publish step fails with an error about path or file name being too long, try copying the [source code](https://github.com/Azure/cortana-intelligence-call-center-solution/tree/master/Manual%20Deployment%20Guide/CI-Callcenterdemo) to a folder with shorter path on disk. In C#, fully qualified file name must be less than 260 characters, and the directory name must be less than 248 characters.**
