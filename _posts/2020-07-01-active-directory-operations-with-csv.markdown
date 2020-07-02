---
layout: post
title:  "Simple Active Directory operations with Powershell and csv"
date:   2020-07-01 12:30:00 +0300
categories: oim active_directory powershell
---
To make specific user opeartions on Active Directory and pulling that change into OIM the following script can be used.
After an operation completed for a user, I change a dummy attribute (like "comment" attribute) of a user to an arbitrary value. 
Then I run *Active Directory User Target Recon* scheduled job with a *Filter* parameter of *equalTo('comment','OK')* 

# **Add/Remove Users to/from the Groups**

The input csv file (should be delimited with ;) includes 3 columns:
- **sam**   : sAMAccountName attribute of user
- **dnAdd** : DN of group where user will be added to
- **dnRmv** : DN of group where user will be removed from

<img src="{{site.baseurl}}/assets/img/oim/csv_demo_grp.png">

```powershell
$oper = Import-Csv -Path C:\Scripts\IDMBatchOperations\list.csv -Delimiter ";" -Encoding UTF8

foreach ($o in $oper) {
    $userSAM = ""
    $userSAM = $o.sam
    $currAdd = $o.dnAdd
    $currRmv = $o.dnRemove
    if ($currAdd -ne "") {
        $groupcheck = $null
        $groupcheck = Get-ADUser -Identity $userSAM -Properties memberof | where {$_.memberof -like "*$currAdd*"}
        If ($groupcheck -eq $null) {
            Add-ADGroupMember -Identity $currAdd -Members $userSAM
            Write-Host "Added: $userSAM -> $currAdd" -ForegroundColor Green
            Write-Host "-"
            Set-ADUser -Identity $userSAM -Replace @{comment='OK'}
        }
    }
    if ($currRmv -ne "") {
        $groupcheck = $null
        $groupcheck = Get-ADUser -Identity $userSAM -Properties memberof | where {$_.memberof -like "*$currRmv*"}
        If ($groupcheck -ne $null) {
            Remove-ADGroupMember -Identity $currRmv -Members $userSAM -Confirm:$false
            Write-Host "Removed: $userSAM -> $currRmv" -ForegroundColor Yellow
            Write-Host "-"
            Set-ADUser -Identity $userSAM -Replace @{comment='OK'}
        }
    }
}
```

# **Move User to Another OU**

The input csv file (should be delimited with ;) includes 3 columns:
- **sam**   : sAMAccountName attribute of user
- **ouMove** : OU where user will be moved to

<img src="{{site.baseurl}}/assets/img/oim/csv_demo_ou.png">

```powershell
$oper = Import-Csv -Path C:\Scripts\IDMBatchOperations\list2.csv -Delimiter ";" -Encoding UTF8

foreach ($o in $oper) {
    $userSAM = ""
    $userSAM = $o.sam
    $currMove = $o.ouMove
	
	Get-ADUser $userSAM | Move-ADObject -TargetPath $currMove
	Write-Host "Moved: $userSAM -> $currMove" -ForegroundColor Green
	Write-Host "-"
    Set-ADUser -Identity $userSAM -Replace @{comment='OK'}
}
```


