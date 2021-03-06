## WinRM (Windows Remote Management) (T1028)

> WinRM (Windows Remote Management) is Microsoft's implementation of WS-Management in Windows which **allows systems to access or exchange management information across a common network**
>
> -- Wikipedia

*Simply it allows remote access*

WinRM (Remote Powershell) is not enabled by default, to enable it check this [Enable WinRM](https://www.howtogeek.com/117192/how-to-run-powershell-commands-on-remote-computers/)


WinRM is used by attacker for lateral movement 

```powershell
Enter-PSSession -Computername <IP> -Credentials (Get-Credential)
```

or 

```powershell
Invoke-Command -ComputerName 192.168.1.12 -ScriptBlock {get-childitem} -Credential (Get-Credential)
```

also there is a lot of command that has option for remote execution by default
just check if the command has **-Computername** in its option ex: copy-item, get-service

```powershell
$cred = Get-Credential
$session = New-PSSession -ComputerName 192.168.1.12 -Credential $cred
Copy-Item -Path C:\entry.txt -Destination c:\temp\entry.txt -ToSession $session
```



### Detection



Now the fun part, the Detection of execution of Remote Powershell

```mermaid
graph LR
A[Host 192.168.1.6]-->B[Remote 192.168.1.12]
```

#### Artifacts on Host

- Windows Event Log

  - Microsoft-Windows-WinRM/Operational
    - It's not displayed in windows event explorer but it's with other logs in
      c:\windows\system32\winevt\logs
    - EID 6![WinRM-EID6](https://raw.githubusercontent.com/karemfaisal/SMUC/master/WinRM/Misc/WinRM-EID6.PNG)
    - it appears that we connect to 192.168.1.12 using WinRM 

  - This Log Doesn't mean that the connection succeeded, but just record the trial
    if there is not error log after it 
    - EID 142 "WSMan operation CreateShell failed"
    ![EID142](https://raw.githubusercontent.com/karemfaisal/SMUC/master/WinRM/Misc/EID142.PNG)
    
    - EID 161 if Wsman is disabled on the remote server
     ![WinRM-EID161](https://raw.githubusercontent.com/karemfaisal/SMUC/master/WinRM/Misc/WinRM-EID161.PNG)
    - EID 162 if Credentials is wrong
     ![WinRM-EID162](https://raw.githubusercontent.com/karemfaisal/SMUC/master/WinRM/Misc/WinRM-EID162.PNG)


#### Artifacts on Remote

- Powershell log
  - EID 600
    - it will contain HostName=ServerRemoteHost
    - it will contain HostApplication=C:\Windows\system32\\**wsmprovhost.exe** -Embedding
      - wsmprovhost.exe is the process for enable remote commands
       ![WinRM-Remote-EID600](https://raw.githubusercontent.com/karemfaisal/SMUC/master/WinRM/Misc/Powershell-Remote-EID600.PNG)
      - Login at 10:24:45 AM While on Host was 7:24:27 PM (neglect the hour, I just didn't set it correct on both machines, focus on minute and seconds)
      - Here if we investigate the remote and found this, then we know that there is some remote device that tried to connect to this machine
- WinRM 
  - EID 91 (Connection succeeded) *No thing new but it's still an evidence*
   ![WinRM-Remote-EID91](https://raw.githubusercontent.com/karemfaisal/SMUC/master/WinRM/Misc/WinRM-Remote-EID91.PNG)

- Security
  - EID 4624 Logon Type 3
    - it shows the source workstation that made the connection

![Security-EID4624](https://raw.githubusercontent.com/karemfaisal/SMUC/master/WinRM/Misc/Security-EID4624.PNG)



#### Resources

- [WinRM Details](http://www.hurryupandwait.io/blog/a-look-under-the-hood-at-powershell-remoting-through-a-ruby-cross-plaform-lens)  *I didn't discuss any thing from this link, but it's very good link for understanding WinRM process flow*

