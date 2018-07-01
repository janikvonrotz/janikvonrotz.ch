---
id: 1902
title: Configure hybrid search results from SharePoint Online in SharePoint on-premise
date: 2014-05-14T07:03:29+00:00
author: Janik von Rotz
layout: post
guid: https://janikvonrotz.ch/?p=1902
permalink: /2014/05/14/configure-hybrid-search-results-from-sharepoint-online-in-sharepoint-on-premise/
dsq_thread_id:
  - "2683850564"
image: /wp-content/uploads/2014/04/SharePoint-Office-365-Hybrid-Solution.jpg
categories:
  - Blog
  - SharePoint
tags:
  - adfs
  - certificate
  - dirsync
  - hybrid
  - identity
  - query
  - results
  - rule
  - search
  - service
  - sharepoint
  - source
  - token
---
*This post of is part of my [Install SharePoint 2013 Three-tier Farm](https://janikvonrotz.ch/projects/install-sharepoint-2013-three-tier-farm/) project.*
*Get the latest version of this article here: [https://gist.github.com/10871110](https://gist.github.com/10871110)*

# Introduction

In this post I'll show you how to get search results from your SharePoint Online in your SharePoint 2013 on-premise search center.
<img src="https://janikvonrotz.ch/wp-content/uploads/2014/04/SharePoint-Hybrid-Outbound-search.jpg" alt="SharePoint Hybrid Outbound search" width="447" height="464" class="aligncenter size-full wp-image-1995" />
<!--more-->
# Requirements

* User synchronisation ActiveDirectory to Office 365 with DirSync
* DirSync password sync or ADFS SSO
* SharePoint Online
* SharePoint 2013 on-premise
  * Enterprise Search service
  * SharePoint Online Management Shell

# Instructions

All configuration will be done either in the Search Administration of the Central Administration or in the PowerShell console of your on-premise SharePoint 2013 server.

# Set up Sever to Server Trust

## Export certificates

To create a server to server trust we need two certificates.

**[certificate name].pfx**: In order to replace the STS certificate, the certificate is needed in Personal Information Exchange (PFX) format including the private key.

**[certificate name].cer**: In order to set up a trust with Office 365 and Windows Azure ACS, the certificate is needed in CER Base64 format.

1. First launch the **Internet Information Services (IIS) Manager**
2. Select your **SharePoint web server** and double-click **Server Certificates**
3. In the **Actions** pane, click **Create Self-Signed Certificate**
4. Enter a name for the certificate and save it with **OK**
5. To export the new certificate in the Pfx format select it and click **Export** in the **Actions** pane
6. Fill the fields and click **OK**
Export to: `C:\[certificate name].pfx`
Password: `[password]`
7. Also we need to export the certificate in the CER Base64 format. For that purpose make a **right-click** on the certificate and click on **View...**
8. Click the **Details** tab and then click **Copy to File**
9. On the Welcome to the Certificate Export Wizard page, click **Next**
10. On the Export Private Key page, click **Next**
11. On the Export File Format page, click **Base-64 encoded X.509** (.CER), and then click **Next**.
12. As file name enter `C:\[certificate name].cer` and then click **Next**
13. Finish the export

## Import the new STS (SharePoint Token Service) certificate

Let's update the certificate on the STS. Configure and run the PowerShell script below on your SharePoint server.

[code lang="ps"]
if(-not (Get-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot; -ErrorAction SilentlyContinue)){Add-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot;}

# set the cerficates paths and password
$PfxCertPath = &quot;c:\[certificate name].pfx&quot;
$PfxCertPassword = &quot;[password]&quot;
$X64CertPath = &quot;c:\[certificate name].cer&quot;

# get the encrypted pfx certificate object
$PfxCert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 $PfxCertPath, $PfxCertPassword, 20

# import it
Set-SPSecurityTokenServiceConfig -ImportSigningCertificate $PfxCert
[/code]

Type **Yes** when prompted with the following message.

> You are about to change the signing certificate for the Security Token Service. Changing the certificate to an invalid, inaccessible or non-existent certificate will cause your SharePoint installation to stop functioning. Refer to the following article for instructions on how to change this certificate: http://go.microsoft.com/fwlink/?LinkID=178475. Are you sure, you want to continue?

Restart IIS so STS picks up the new certificate.

[code lang="ps"]
&amp; iisreset
&amp; net stop SPTimerV4
&amp; net start SPTimerV4
[/code]

Now validate the certificate replacement by running several PowerShell commands and compare their outputs.

[code lang="ps"]
# set the cerficates paths and password
$PfxCertPath = &quot;c:\[certificate name].pfx&quot;
$PfxCertPassword = &quot;[password]&quot;

# get the encrypted pfx certificate object
New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 $PfxCertPath, $PfxCertPassword, 20

# compare the output above with this output
(Get-SPSecurityTokenServiceConfig).LocalLoginProvider.SigningCertificate
[/code]

## Establish the server to server trust

[code lang="ps"]
if(-not (Get-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot; -ErrorAction SilentlyContinue)){Add-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot;}
Import-Module MSOnline 
Import-Module MSOnlineExtended

# set the cerficates paths and password
$PfxCertPath = &quot;c:\[certificate name].pfx&quot;
$PfxCertPassword = &quot;[password]&quot;
$X64CertPath = &quot;c:\[certificate name].cer&quot;

# set the onpremise domain that you added to Office 365
$SPCN = &quot;sharepoint.domain.com&quot; 

# your onpremise SharePoint site url
$SPSite=&quot;http://sharepoint&quot;

# don't change this value
$SPOAppID=&quot;00000003-0000-0ff1-ce00-000000000000&quot;

# get the encrypted pfx certificate object
$PfxCert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 $PfxCertPath, $PfxCertPassword, 20

# get the raw data
$PfxCertBin = $PfxCert.GetRawCertData()

# create a new certificate object
$X64Cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2

# import the base 64 encoded certificate
$X64Cert.Import($X64CertPath)

# get the raw data
$X64CertBin = $X64Cert.GetRawCertData()

# save base 64 string in variable
$CredValue = [System.Convert]::ToBase64String($X64CertBin)

# connect to office 3656
Connect-MsolService

# register the on-premise STS as service principal in Office 365

# add a new service principal
New-MsolServicePrincipalCredential -AppPrincipalId $SPOAppID -Type asymmetric -Usage Verify -Value $CredValue
$MsolServicePrincipal = Get-MsolServicePrincipal -AppPrincipalId $SPOAppID
$SPServicePrincipalNames = $MsolServicePrincipal.ServicePrincipalNames
$SPServicePrincipalNames.Add(&quot;$SPOAppID/$SPCN&quot;)
Set-MsolServicePrincipal -AppPrincipalId $SPOAppID -ServicePrincipalNames $SPServicePrincipalNames

# get the online name identifier
$MsolCompanyInformationID = (Get-MsolCompanyInformation).ObjectID
$MsolServicePrincipalID = (Get-MsolServicePrincipal -ServicePrincipalName $SPOAppID).ObjectID
$MsolNameIdentifier = &quot;$MsolServicePrincipalID@$MsolCompanyInformationID&quot;

# establish the trust from on-premise with ACS (Azure Control Service)

# add a new authenticatio realm
$SPSite = Get-SPSite $SPSite
$SPAppPrincipal = Register-SPAppPrincipal -site $SPSite.rootweb -nameIdentifier $MsolNameIdentifier -displayName &quot;SharePoint Online&quot;
Set-SPAuthenticationRealm -realm $MsolServicePrincipalID

# register the ACS application proxy and token issuer
New-SPAzureAccessControlServiceApplicationProxy -Name &quot;ACS&quot; -MetadataServiceEndpointUri &quot;https://accounts.accesscontrol.windows.net/metadata/json/1/&quot; -DefaultProxyGroup
New-SPTrustedSecurityTokenIssuer -MetadataEndpoint &quot;https://accounts.accesscontrol.windows.net/metadata/json/1/&quot; -IsTrustBroker -Name &quot;ACS&quot;
[/code]

# Add a new result source

To get search results from SharePoint Online we have to add a new result source. Run the following script in a PowerShell ISE session on your SharePoint 2013 on-premise server.
Don't forget to update the settings region

[code lang="ps"]
if(-not (Get-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot; -ErrorAction SilentlyContinue)){Add-PSSnapin &quot;Microsoft.SharePoint.PowerShell&quot;}

# region settings 
$RemoteSharePointUrl = &quot;http://[example].sharepoint.com&quot;
$ResultSourceName = &quot;SharePoint Online&quot;
$QueryTransform = &quot;{searchTerms}&quot;
$Provier = &quot;SharePoint-Remoteanbieter&quot;
# region settings end

$SPEnterpriseSearchServiceApplication = Get-SPEnterpriseSearchServiceApplication
$FederationManager = New-Object Microsoft.Office.Server.Search.Administration.Query.FederationManager($SPEnterpriseSearchServiceApplication)
$SPEnterpriseSearchOwner = Get-SPEnterpriseSearchOwner -Level Ssa  

$ResultSource = $FederationManager.GetSourceByName($ResultSourceName, $SPEnterpriseSearchOwner)
if(!$ResultSource){
    Write-Host &quot;Result source does not exist. Creating...&quot;
    $ResultSource = $FederationManager.CreateSource($SPEnterpriseSearchOwner)
}

$ResultSource.Name = $ResultSourceName
$ResultSource.ProviderId = $FederationManager.ListProviders()[$Provier].Id
$ResultSource.ConnectionUrlTemplate = $RemoteSharePointUrl
$ResultSource.CreateQueryTransform($QueryTransform)
$ResultSource.Commit()
[/code]

## Add a new query rule

1. In the Search Administration click on **Query Rules**
2. Select **Local SharePoint** as Result Source
3. Click **New Query Rule**
4. Enter a Rule name f.g. Search results from SharePoint Online
5. Expand the **Context** section
6. Under **Query is performed on these sources** click on **Add Source**
7. Select your SharePoint Online result source
8. In the **Query Conditions** section click on **Remove Condition**
9. In the **Actions** section click on **Add Result Block**
10. As **title** enter **Results for "{subjectTerms}" from SharePoint Online**
11. In the **Search this Source** dropdown select your SharePoint Online result source
12. Select 3 in the **Items** dropdown
13. Expand the **Settings** section and select **"More" link goes to the following URL**
14. In the box below enter this Url **https://[example].sharepoint.com/search/pages/results.aspx?k={subjectTerms}**
15. Select **This block is always shown above core results** and click the OK button
16. Save the new query rule

# Source

[Display hybrid search results in SharePoint Server 2013](http://technet.microsoft.com/en-us/library/dn197173.aspx)  
[Office 365-Configure Hybrid Search with Directory Synchronization –Password Sync](http://blogs.msdn.com/b/spses/archive/2013/10/22/office-365-configure-hybrid-search-with-directory-synchronization.aspx)  
[Office 365-Configure Hybrid Search with Directory Synchronization –Password Sync –Part2](http://blogs.msdn.com/b/spses/archive/2014/01/05/office-365-configure-hybrid-search-with-directory-synchronization-password-sync-part2.aspx)