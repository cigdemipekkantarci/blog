---
layout: post
title:  "Export Tableau Reports to Filesystem / Encrypt admin password"
date:   2020-07-04 11:00:00 +0300
categories: tableau windows_server powershell
---
The following simple PS script can be used to export tableau reports as a png file to a location in filesystem. The process is scripted and runs as a Windows scheduler task, also tableau admin account password is encrypted for security purposes.

# **Get credential (one time event)**

```powershell
$credential = Get-Credential 
```

<img src="{{site.baseurl}}/assets/img/tableau/get-cred.png">

# **Store encrypted admin password in encrypted_pw.txt file**

```powershell
$credential.Password | ConvertFrom-SecureString | Set-Content 'C:\Program Files\Tableau\files\encrypted_pw.txt' 
```

<img src="{{site.baseurl}}/assets/img/tableau/encr-passwd.png">

# **Create credential object**

```powershell
$pw = Get-Content 'C:\Program Files\Tableau\files\encrypted_pw.txt' | ConvertTo-SecureString

$credential = New-Object System.Management.Automation.PsCredential("tableau.sa",$pw) 
```

# **Login to Tableau**

```powershell
cd 'C:\Program Files\Tableau\Tableau Server\packages\bin.20182.18.0627.2230'

tabcmd login -s http://tableauserver.example.com.tr -u 

```

# **Export to filesystem**

```powershell
tabcmd export "GKK/report_1" --png -f "\\fs01\GKK\gkk_report1.png"
exit 
```

# **Complete script**

You can put the following script anywhere you remember, then we will call this on Windows sceduler. In my case the location is `C:\Program Files\Tableau\files\gkk_rapor\gkk_all_exports.ps1`

```powershell
$pw = Get-Content 'C:\Program Files\Tableau\files\encrypted_pw.txt' | ConvertTo-SecureString
$credential = New-Object System.Management.Automation.PsCredential("tableau.sa",$pw)

cd 'C:\Program Files\Tableau\Tableau Server\packages\bin.20182.18.0627.2230'
tabcmd login -s http://tableauserver.example.com.tr -u $credential.GetNetworkCredential().Username -p $credential.GetNetworkCredential().Password

#gkk_report1
tabcmd export "GKK/report_1" --png -f "\\fs01\GKK\gkk_report1.png"
#gkk_report1
tabcmd export "GKK/report_2" --png -f "\\fs01\GKK\gkk_report2.png"
exit 
```

# **Run PowerShell script as Windows Scheduler Task**

<img src="{{site.baseurl}}/assets/img/tableau/sch-task1.png">

<img src="{{site.baseurl}}/assets/img/tableau/sch-task2.png">

<img src="{{site.baseurl}}/assets/img/tableau/sch-task3.png">

<img src="{{site.baseurl}}/assets/img/tableau/sch-task4.png">

Program/script: 
```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Add arguments: -noprofile -executionpolicy unrestricted -noninteractive -file "C:\Program Files\Tableau\files\gkk_rapor\gkk_all_exports.ps1"
```