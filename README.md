﻿# Notas de preparación para el OSCP 

**Año : 2021**

![oscp](https://blog.nivel4.com/wp-content/uploads/2019/01/try-harder.png)

**Curso: Penetration Testing with Kali Linux (PEN-200)**

## Contenido
- [Linux](#Linux)
  - [Reverse shells](#reverse-shells)  
  - [Mejorando nuestra shell](#Spawning-a-TTY-shell)   
  - [Files]  
  - Bypassing restricted shells (rbash , rzsh,etc)    
- [Windows](#Windows)
- 
- [Reverse Shell]

Linux 
================================
Técnicas para usar en sistemas basados en UNIX



  ```bash 
      python3 -c 'import pty;pty.spawn("/bin/bash")' 
      script /dev/null -c bash
  ```
     
  
3. Windows
