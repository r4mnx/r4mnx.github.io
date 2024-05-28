---

title: FlipperZero EvilPortal
date: 2023-09-10 07:03 +0800
categories: [hardware]
tags: [hardware, wifi]
pin: true
image:
  path: /assets/img/commons/FlipperZero/EvilPortal/bannerEvilPortal.png
  width: 66
  height: 40
  alt: EvilPortal
---
En este artículo usaremos las aplicaciones `"(ESP32)Evil Portal"` y `"(ESP32)Wifi Marauder"` en nuestro dispositivo **FlipperZero** con una tarjeta **ESP32 Module v1** con el firmware de [RogueMaster](https://github.com/RogueMaster/flipperzero-firmware-wPlugins) 
- **Evil Portal** -> Esta aplicación nos brinda la capacidad de crear una red Wi-Fi `"abierta"` con un portal cautivo, lo que nos permite capturar credenciales de forma efectiva. Cargaremos una plantilla `.html` que simulará un panel de inicio de sesión de varios servicios web.
- **Wifi Marauder** -> Nos permite detectar redes, realizar ataques de desautenticación, capturar handshakes WPA/WPA2, analizar tráfico de red, etc.

Antes de usar estas aplicaciones necesitamos flashear la tarjeta wifi, en mi caso usaré la tarjeta `FlipperZero Module v1`, existen dos opciones, podemos flashear con un unico firmware o incluir dos firmwares con la herramienta `ESP Flasher` para poder usar las dos aplicaciones.
Tengo este [repositorio](https://github.com/r4mnx/FlipperZero) donde recopilo información que pruebo en mis proyectos.

## Dual Boot ESP Flasher
Vamos a incluir firmware en la aplicación `dual boot` para poder utilizar `"Wifi-Marauder"` y `"EvilPortal"` en la misma ESP32. El **FlipperZero** con el que vamos a hacer el `dual boot` tiene instalado el Firmware  [RogueMaster](https://github.com/RogueMaster/flipperzero-firmware-wPlugins)con la antena oficial `Flipper Zero wifi module v1`
- En primer lugar vamos a descargar el firmware dual desde este [repositorio](https://github.com/skizzophrenic/Talking-Sasquach/tree/main/Single%20File%20WiFi%20Board%20Bins/Dual%20Boot) de GitHub, en mi caso usaré este [firmware](https://github.com/skizzophrenic/Talking-Sasquach/raw/main/Single%20File%20WiFi%20Board%20Bins/Dual%20Boot/Dual%20Boot%20Flipper%20or%20WROVER.bin)
- Copiaremos `...WROVER.bin` en el siguiente directorio de nuestro **FlipperZero**
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/DualBoot1.png" alt="Upload .bin">
  <figcaption style="font-style: italic; font-size: smaller;">Files Flash</figcaption>
</figure> 
- Abrimos la aplicación en  nuestro flipper `Apps/GPIO/ESP Flasher`
- Seleccionamos `Manual Flash` y `[ ] Bootloader (0x1000)`. Ahora seleccionamos el firmware **"Dual Boot Flipper or WROVER.bin"**
- Elegimos la opción `[<] FLASH` para comenzar el flash de la tarjeta, al concluir el flash veremos una pantalla similar a la siguiente imagen
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/DualBoot3.png" alt="Done">
  <figcaption style="font-style: italic; font-size: smaller;">Files Flash</figcaption>
</figure> 
### Select Firmware
- Una vez flasheado podemos entrar en la aplicación `ESP Flasher` y seleccionar entre las dos opciones de firmware, `Switch to Firmware A` "EvilPortal" o `Switch to Firmware B` "Marauder" 
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/DualBoot4.png" alt="Select Flash">
  <figcaption style="font-style: italic; font-size: smaller;">Files Flash</figcaption>
</figure>


## Flash Wifi ESP32
- Primero, debemos realizar el proceso de flasheo en la tarjeta ESP32 para poder utilizar la aplicación `"(ESP32)Evil Portal"`. En mi caso, estoy utilizando la tarjeta oficial de FlipperZero Module v1. A continuación, procederemos con el proceso de flasheo:
- Puedes descargar el archivo **wifi_dev_board.zip** desde este [enlace](https://github.com/bigbrodude6119/flipper-zero-evil-portal/releases/download/0.0.2/wifi_dev_board.zip) en el repositorio de [bigbrodude6119](https://github.com/bigbrodude6119/flipper-zero-evil-portal). Una vez que lo hayas descargado, procede a extraer su contenido. Al hacerlo, obtendrás cuatro archivos con extensión .bin.
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/FilesFlash.png" alt="FilesFlash">
  <figcaption style="font-style: italic; font-size: smaller;">Files Flash</figcaption>
</figure> 
- Para conectar nuestra tarjeta ESP32 al PC, procedemos de la siguiente manera: mantenemos presionado el botón `"BOOT"` y, mientras lo hacemos, conectamos el cable.
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/ConectESP.png" alt="ConnectESP">
  <figcaption style="font-style: italic; font-size: smaller;">Connect ESP</figcaption>
</figure> 
- Vamos a [esta web](https://esp.huhn.me/), pulsamos `"Connect"`, seleccionamos el puerto de nuestra ESP32 y hacemos clic en `"Conectar"`. Yo uso el navegador "LibreWolf" y no me reconoce el "FlipperZero", si tienes problemas, puedes abrir la web con "Google Chrome".
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/ConnectChrome.png" alt="ConnectChrome">
  <figcaption style="font-style: italic; font-size: smaller;">Connect Chrome</figcaption>
</figure> 
- Necesitamos referenciar cada archivo.bin a la parte de la memoria de nuestra  ESP32 para que funcione correctamente y pulsamos `"PROGRAM"`.
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/FlashESP32.png" alt="FlashESP32">
  <figcaption style="font-style: italic; font-size: smaller;">Flash ESP32</figcaption>
</figure> 
- <u>0x1000</u> es la dirección de la memoria flash reservada al arranque o bootloader.
- <u>0x8000</u> contiene la información de las particiones de la memoria flash.
- <u>0xE000</u> es una parte que contiene información adicional para el arranque.
- <u>0x0000</u> direccion que contiene la información  necesaria para que llegué a comunicarse la tarjeta con nuestra app `"(ESP32)Evil Portal"`.
- Si todo salió bien obtendremos un `"DONE!"` y tendremos nuestra tarjeta lista para arrancar la app.
## [ESP] Evil Portal
- Vamos a incluir algunos archivos necesarios a nuestro FlipperZero antes de ejecutar la app.
- Descargamos `"ap.config.txt"` y `"index.html"` de este [repo](https://github.com/bigbrodude6119/flipper-zero-evil-portal/tree/d196f60fe8b40f21b6aa296483c03494daf1f946/flipper/apps_data/evil_portal).
- Vamos a descargar también [plantillas](https://github.com/bigbrodude6119/flipper-zero-evil-portal/tree/d196f60fe8b40f21b6aa296483c03494daf1f946/portals) para ejecutar la que más nos convenga.
- Incluimos estos archivos a la siguiente ruta en nuestro FlipperZero.
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/DDFlipper.png" alt="DDFlipper">
  <figcaption style="font-style: italic; font-size: smaller;">Datos FlipperZero</figcaption>
</figure>
- Entramos en nuestro `FlipperZero/Apps/GPIO/[ESP32]Evil Portal` y `Start Portal` para iniciar el servidor con nuestro Portal.
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/AppPortal.png" alt="AppPortal">
  <figcaption style="font-style: italic; font-size: smaller;">App EvilPortal</figcaption>
</figure>
- Si todo va bien, la tarjeta emitirá una luz verde y se creará una wifi `"abierta"` con el nombre indicado en un archivo de configuración "ap.config.txt. 
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/WifiName.png" alt="WifiName">
  <figcaption style="font-style: italic; font-size: smaller;">Nombre Wifi</figcaption>
</figure>   
- Al conectarnos a la wifi `"Google Free Wifi"` que es el nombre de nuestra wifi `"Fake"` nos redirecionará al index.html de nuestra app. 
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/PlantillaPORTAL.png" alt="PlantillaPORTAL">
  <figcaption style="font-style: italic; font-size: smaller;">Plantilla Portal</figcaption>
</figure>
- Esta plantilla es similar al panel de autenficación del servicio WEB, este recogerá los datos introducidos por el user y mandados a nuestro FlipperZero. 
<figure style="text-align: center;">
  <img src="/assets/img/commons/FlipperZero/EvilPortal/LeakDD.png" alt="LeakDD">
  <figcaption style="font-style: italic; font-size: smaller;">Datos Leaked</figcaption>
</figure>
- Podemos modificar el nombre que se mostrará en la wifi cambiando el contenido del archivo de `"ap.config.txt"`. 
- En la carpeta `"portals"` existen varias plantillas , solo tendremos que copiar el file.html de la carpeta "portals" a `"evil_portal"` y cambiar el nombre a `"index.html"`. 
- La siguiente vez que ejecutemos el portal nos cargará el nuevo nombre de la wifi con la nueva plantilla.

Este post es con fines educativos y siempre desde un punto de vista ético. Se agradece que si existe un error se pongan en contacto para corregirlo. 
