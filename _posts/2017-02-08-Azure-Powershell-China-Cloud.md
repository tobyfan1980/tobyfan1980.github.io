---
layout: post
title:  "Working on Azure Powershell for China regions"
date:   2017-02-08 20:00:00 -0800
categories:    Tools
tags:    Azure, Powershell
---

1 -- Login china azure. Login page will be pop up and please login using your china azure credential.

```
Login-AzureRmAccount -EnvironmentName azurechinacloud
```

2 -- Get resource provider api version

```
PS C:\> $chinapowerbi = Get-AzureRmResourceProvider -ProviderNamespace microsoft.powerbi
PS C:\> $chinapowerbi.resourcetypes
ResourceTypeName : workspaceCollections
Locations        : {China North, China East}
ApiVersions      : {2016-01-29}

ResourceTypeName : locations
Locations        : {}
ApiVersions      : {2016-01-29}

ResourceTypeName : locations/checkNameAvailability
Locations        : {China North, China East}
ApiVersions      : {2016-01-29}
```

3 -- If we want to call Azure REST API from our application, this application must be registered to Azure, and use its application ID
to get access token. 


3.1) create application

```
PS C:\> $app = New-AzureRmADApplication -DisplayName "intelab" -HomePage "http://intelab.ilabservice.com" -IdentifierUris "http://intelab.ilabservice.com" -Password "xxxxx-2017"
PS C:\> $app
DisplayName             : intelab
ObjectId                : aa39fb4f-d9ba-4463-xxxx-ca4ebf6f5150
IdentifierUris          : {http://intelab.ilabservice.com}
HomePage                : http://intelab.ilabservice.com
Type                    : Application
ApplicationId           : 8ff92caf-3141-47c5-xxxx-1f00769d2bcb
AvailableToOtherTenants : False
.AppPermissions          :
ReplyUrls               : {}
```

3.2) the app will use its own credential to generate access token (not the azure account credentials). We need create service principle for the application.

```
PS C:\> New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
DisplayName                    Type                           ObjectId
-----------                    ----                           --------
intelab                        ServicePrincipal               bdad8625-60ef-xxxx-99c1-55a1b2f77e8a

PS C:\> New-AzureRmRoleAssignment -RoleDefinitionName Owner -ServicePrincipalName $app.ApplicationId
RoleAssignmentId   : /subscriptions/49537733-xxxx-4e6a-b09e-686ea69a6686/providers/Microsoft.Authorization/roleAssignme
                     nts/9127a2d3-0dae-xxxx-86ee-e77e0d4674e1
Scope              : /subscriptions/49537733-xxxx-4e6a-b09e-686ea69a6686
DisplayName        : intelab
SignInName         :
RoleDefinitionName : Owner
RoleDefinitionId   : 8e3af657-xxxx-443c-xxxx-2fe8c4bcb635
ObjectId           : bdad8625-xxxx-4931-xxxx-55a1b2f77e8a
ObjectType         : ServicePrincipal

```

3.3) To generate access token, we need tenentId of the azure subscription.

```
PS C:\> Get-AzureRmSubscription
SubscriptionName : xxxxxxxx
SubscriptionId   : 49537733-xxxx-4e6a-xxxx-686ea69a6686
TenantId         : ce5605b8-xxxx-xxxx-bd33-19822c98f924
State            : Enabled
```

3.4) generate accesstoken using http requst. Be careful, in body, the "resource" link has to have the backslash at the end.

```
PS C:\> Invoke-RestMethod -Uri https://login.chinacloudapi.cn/{tenentId}/oauth2/token?api-version=1.0 
-Method Post -Body @{"grant_type" = "client_credentials"; "resource" = "https://management.core.chinacloudapi.cn
/";"client_id"="{application id}"; "client_secret" = "{application password}"}

token_type     : Bearer
expires_in     : 3600
ext_expires_in : 0
expires_on     : 1486516056
not_before     : 1486512156
resource       : https://management.core.chinacloudapi.cn/
access_token   : eyJ0eXAiOiJKV1QiL........
```

3.5) using this token, we can query azure resources as we need. The following example get the PowerBI workspacecollections in a subscription.

```
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Bearer", '{access_token}')

Invoke-RestMethod -Uri https://management.chinacloudapi.cn/subscriptions/{subscription_id}/resourceGroups/{resource group name}/providers/Microsoft.PowerBI/workspacecollections?api-version=2016-01-29 -Method Get -Headers $headers
```
