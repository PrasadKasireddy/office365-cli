# List to find documents stored with passwords

Author: [Mike Lee (BOSTON)](https://techcommunity.microsoft.com/t5/user/viewprofilepage/user-id/96057)

This PowerShell script that to identify documents stored in SharePoint online that were encrypted with passwords

Disclaimer:
- This PowerShell script is provided "as-is" with no warranties expressed or implied. Use it at your own risk.

Dependencies:
- SharePoint Online Client Components SDK: https://www.microsoft.com/en-us/download/details.aspx?id=42038
- Tested with SharePoint Online Client Components SDK version 16.0.6906.1200
- Parameters: $SiteURL, $ListName, $username

```powershell tab="PowerShell Core"
#Add references to SharePoint client assemblies
[System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SharePoint.Client")
[System.Reflection.Assembly]::LoadWithPartialName("WindowsBase")

#Your SPO Tenant
$SiteURL = "https://tenant.sharepoint.com"

#The name of your document library
$Listname = "Documents"

#The admin account that has access to the library
$username = "admin@tenant.onmicrosoft.com"
$password = Read-Host "Enter Password" -AsSecureString

#Building Context
$ctx = New-Object Microsoft.SharePoint.Client.ClientContext($SiteURL)
$ctx.Credentials = New-Object Microsoft.SharePoint.Client.SharePointOnlineCredentials($userName, $password)
$List = $ctx.Web.Lists.GetByTitle($ListName) 

#CAML Query to recursively look at all items in the library with a 5000 item row limit. 
$camlQuery = New-Object Microsoft.SharePoint.Client.CamlQuery
$camlQuery.ViewXml = @"
<View Scope="RecursiveAll">
<Query>
<OrderBy><FieldRef Name='ID' Ascending='TRUE'/></OrderBy>
</Query>
<RowLimit Paged="TRUE">5000</RowLimit>
</View>
"@

$items = $list.GetItems($camlQuery)
$ctx.Load($items)
$ctx.ExecuteQuery()

#function to read documents

function find-docpasswords($ctx, $FileUrl)
{
#Collect Documents Data
$FileURL = $Item.FieldValues['FileRef']

#Read the files from SharePoint online document library.
$fileInfo = [Microsoft.SharePoint.Client.File]::OpenBinaryDirect($ctx,$FileURL)
$stream = New-Object System.IO.MemoryStream
$fileInfo.Stream.CopyTo($stream)

#Read the first row of bytes as text
$Start = [System.Text.Encoding]::Default.GetString($stream.ToArray()[0000..2000])

# Record files that are password protected
if($Start -match "E.n.c.r.y.p.t.e.d.P.a.c.k.a.g.e")
{
Write-Host "$SiteURL$FileURL -- Is Password Protected" -ForegroundColor Yellow
}
else
{
Write-Host "$SiteURL$FileURL -- Not Password Protected" -ForegroundColor Green
}

$stream.Close()
$fileinfo.Dispose()
$ctx.Dispose()
}

#Run the function to loop through all items in the library and find documents stored with passwords

foreach($item in $items)
{
$fileUrl = $item.FieldValues["fileref"]
find-docpasswords $ctx $fileurl
}
```

You will need a few things to make this works.
- Installed the [SharePoint Online Client Components SDK](https://www.microsoft.com/en-us/download/details.aspx?id=42038)
- Specify the “$SiteURL, $Listname, and $username in the script.

Keywords:

- SharePoint Online