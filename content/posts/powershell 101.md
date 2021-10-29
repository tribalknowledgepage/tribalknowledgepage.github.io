---
author:
  name: "Piyush Verma"
date: 2021-10-29
linktitle: PowerShell 101
type:
- post
- posts
title: PowerShell 101
draft: false
toc: true
images:
tags:
  - PowerShell
weight: 10
---

Commonly used PowerShell commands:

Get the PowerShell Version
```powershell
 $PSVersionTable 
 ```
List help about Get-Help in a readable format:
```powershell 
Get-Help Get-Help | more
```

List cmdlets which contain the keyword 'service' in them:
```powershell
Get-Command -CommandType Cmdlet | ?{$_.Name -match "service"}
```
List all commands and their respective details which contain the keyword 'command' in them:

```powershell
Get-Command -CommandType All -ShowCommandInfo | ?{$_.Name -match "command"}
```

Wilcards can be used to match as well. Example:
```powershell
Get-Command -CommandType All | ?{$_.Name -match "serv*"}
```

Count the number of commands in the output: 

```powershell
(Get-Command -CommandType All | ?{$_.Name -match "command"}).count
```

List all cmdlets with PKI as source

```powershell
Get-Command -CommandType Cmdlet | ?{$_.Source -match "PKI"}
```

List all commands in a module by running:
```powershell
Get-Command -Module <module name>
```

