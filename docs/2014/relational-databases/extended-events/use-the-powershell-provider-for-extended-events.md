---
title: "Use the PowerShell Provider for Extended Events | Microsoft Docs"
ms.custom: ""
ms.date: "06/13/2017"
ms.prod: "sql-server-2014"
ms.reviewer: ""
ms.technology: xevents
ms.topic: conceptual
helpviewer_keywords: 
  - "PowerShell [SQL Server], xevent"
  - "extended events [SQL Server], PowerShell"
  - "PowerShell [SQL Server], extended events"
ms.assetid: 0b10016f-a479-4444-a484-46cb4677cf64
author: rothja
ms.author: jroth
manager: craigg
---
# Use the PowerShell Provider for Extended Events
  You can manage [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] Extended Events by using the [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] PowerShell provider. The XEvent subfolder is available under the SQLSERVER drive. You can access the folder by using either of the following methods:  
  
-   At a command prompt, type `sqlps`, and then press ENTER. Type `cd xevent`, and then press ENTER. From there, you can use the **cd** and `dir` commands (or **Set-Location** and **Get-Childitem** cmdlets) to navigate to the server name and instance name.  
  
-   In Object Explorer, expand the instance name, expand **Management**, right-click **Extended Events**, and then click **Start PowerShell**. This starts PowerShell in the following path:  
  
     PS SQLSERVER:\XEvent\\*ServerName*\\*InstanceName*>  
  
    > [!NOTE]  
    >  You can start PowerShell from any node under **Extended Events**. For example, you can right-click **Sessions**, and then click **Start PowerShell**. This starts PowerShell one level deeper, at the Sessions folder.  
  
 You can browse the XEvent folder tree to view existing Extended Events sessions and their associated events, targets and predicates. For example, from the PS SQLSERVER:\XEvent\\*ServerName*\\*InstanceName*> path, if you type `cd sessions`, press ENTER, type `dir`, and then press ENTER, you can see the list of sessions that are stored on that instance. You can also view whether the session is running (and if this is the case, for how long), and whether the session is configured to start when the instance starts.  
  
 To view the events, their predictates, and the targets that are associated with a session, you can change directories to the session name, and then view either the events or targets folder. For example, to view the events and their predicates that are associated with the default system health session, from the PS SQLSERVER:\XEvent\\*ServerName*\\*InstanceName*\Sessions> path, type `cd system_health\events,` press ENTER, type `dir`, and then press ENTER.  
  
 The [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] PowerShell provider is a powerful tool that you can use to create, alter, and manage Extended Events sessions. The following section provides some basic examples of using PowerShell scripts with Extended Events.  
  
## Examples  
 In the following examples, note the following:  
  
-   The scripts must be run from the PS SQLSERVER:\\> prompt (available by typing `sqlps` at a command prompt).  
  
-   The scripts use the default instance of [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)].  
  
-   The scripts must be saved with a .ps1 extension.  
  
-   The PowerShell execution policy must allow the script to run. To set the execution policy, use the **Set-Executionpolicy** cmdlet. (For more information, type `get-help set-executionpolicy -detailed`, and then press ENTER.)  
  
 The following script creates a new session that is named 'TestSession'.  
  
```powershell
#Script for creating a session.  
cd XEvent  
$h = hostname  
cd $h  
  
#Use the default instance.  
$store = dir | where {$_.DisplayName -ieq 'default'}  
$session = New-Object Microsoft.SqlServer.Management.XEvent.Session -ArgumentList $store, "TestSession"  
$event = $session.AddEvent("sqlserver.file_written")  
$event.AddAction("package0.callstack")  
$session.Create()  
```  
  
 The following script adds the ring buffer target to the session that was created in the previous example. (This example shows the use of the `Alter` method. Be aware that you can add the target when you first create the session.)  
  
```powershell
#Script to alter a session.  
cd XEvent  
$h = hostname  
cd $h  
cd DEFAULT\Sessions  
  
#Used to find the specified session.  
$session = dir | where {$_.Name -eq 'TestSession'}  
  
#Add the ring buffer target and call the Alter method.  
$session.AddTarget("package0.ring_buffer")  
$session.Alter()  
```  
  
 The following script creates a new session that uses a predicate expression. In this case, the session collects information for when the c:\temp.log file is written to (through the sqlserver.file_written event).  
  
```powershell
#Script for creating a session.  
cd XEvent  
$h = hostname  
cd $h  
  
#Use the default instance.  
$store = dir | where {$_.DisplayName -ieq 'default'}  
$session = New-Object Microsoft.SqlServer.Management.XEvent.Session -ArgumentList $store, "TestSession2"  
$event = $session.AddEvent("sqlserver.file_written")  
  
#Construct a predicate "equal_i_unicode_string(path, N'c:\temp.log')".  
$column = $store.SqlServerPackage.EventInfoSet["file_written"].DataEventColumnInfoSet["path"]  
$operand = new-object Microsoft.SqlServer.Management.XEvent.PredOperand -argumentlist $column  
$value = new-object Microsoft.SqlServer.Management.XEvent.PredValue -argumentlist "c:\temp.log"  
$compare = $store.Package0Package.PredCompareInfoSet["equal_i_unicode_string"]  
$predicate = new-object Microsoft.SqlServer.Management.XEvent.PredFunctionExpr -ArgumentList $compare, $operand, $value  
$event.SetPredicate($predicate)  
$session.Create()  
```  
  
## Security  
 To create, alter, or drop an Extended Events session, you must have the ALTER ANY EVENT SESSION permission.  
  
## See Also  
 [SQL Server PowerShell](../../powershell/sql-server-powershell.md)   
 [Use the system_health Session](use-the-ssms-xe-profiler.md)   
 [Extended Events Tools](extended-events-tools.md)  
