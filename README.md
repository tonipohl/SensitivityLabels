# Working with Microsoft 365 SensitivityLabels

Currently application permissions aren’t supported when updating assigned labels in Graph. While individual users can apply and manage sensitivity labels, applications do not have the capability to update these labels. This restriction means third-party applications cannot assign sensitivity labels. The current workaround requires user assigned permissions. Martina Grom and Toni Pohl inform about this topic and a workaround here.

These samples are Azure Logic Apps that use the Microsoft Graph to work with sensitivity labels in Microsoft 365. The Logic Apps are triggered by HTTP requests and use the Graph to get and assign sensitivity labels to Microsoft Teams and SharePoint sites. 

## Graph and SharePoint API requests

Basically the flows here do the following:

### 01-GetLabels.json

Get all sensitivity labels from the Graph and write them to a table in an Azure Storage Account.
  - Read all labels: 
~~~
https://graph.microsoft.com/beta/security/informationProtection/sensitivityLabels
~~~
  - Documentation: https://learn.microsoft.com/en-us/graph/api/security-informationprotection-list-sensitivitylabels?view=graph-rest-beta&viewFallbackFrom=graph-rest-1.0

### 02-GetTeams

Reads all Microsoft teams, and the assigned Sensitivity Labels, from the Graph and writes them to a table.

  - Get all teams: 
~~~
https://graph.microsoft.com/v1.0/groups?$filter=(groupTypes/any('s:s eq Unified')&$select=id,displayName,assignedLabels,groupTypes
~~~
  - Get group details, like the SPO site address: 
~~~ 
https://graph.microsoft.com/v1.0/groups/[id]/sites/root?$select=id,webUrl,name
~~~

### 03-GetSites

Reads all SharePoint sites (that are not Personal), and the assigned Sensitivity Labels, from the Graph and the SharePoint APO and writes them to a table.
Note: The user used in the connection must have access to all sites.

  - Read all non-personal SharePoint sites: 
~~~  
https://graph.microsoft.com/v1.0/sites/getAllSites?$filter=isPersonalSite eq false
~~~
  - Documentation: https://learn.microsoft.com/en-us/graph/api/site-getallsites?view=graph-rest-1.0&tabs=http
  - Get the site address with SharePoint REST API: 
~~~
[site-webUrl]/_api/site
~~~

### 04-AssignTeams

Assigns a Sensitivity Label to a Microsoft Team. The Logic App reads the Teams from the table and assigns the label to the team. Additionally, the operation is logged in a table.

  - Get the team: 
~~~
https://graph.microsoft.com/v1.0/groups?$filter=startswith(displayName,'Team Paris')&$select=id,displayName,assignedLabels
~~~
  - Patch the team: 
~~~
PATCH 
https://graph.microsoft.com/v1.0/groups/[teamid]
{
  "assignedLabels": [
    {
      "labelId": "[labelid]"
    }
  ]
}
~~~

### 05-AssignSites

Assigns a Sensitivity Label to a SharePoint site. The Logic App reads the site from the table and assigns the label to the site. Additionally, the operation is logged in a table.

  - Get the site: 
~~~
https://graph.microsoft.com/v1.0/sites/getAllSites?$filter=isPersonalSite eq false and startswith(displayName,'Viva Home')
~~~
  - Patch the site:
~~~
PATCH 
https://tpe5.sharepoint.com/sites/VivaHome/_api/site
{
"SensitivityLabelId": "[labelid]"
}
~~~

## Azure resources

The Logic Apps use the following Azure resources:
- Azure Storage Account: To store the labels and the Teams and Sites information.
- Azure Key Vault: To store the Graph credentials.
- Entra ID App: To authenticate the Logic App with the Graph.
- Azure Logic Apps: To run the flows.
- Service Account: To connect the Logic Apps actions with M365 resources