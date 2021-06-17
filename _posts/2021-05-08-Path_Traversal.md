---
title: "Path Traversal"
layout: single
excerpt: Una **Vulnerabilidad web** de la cual sacaremos un buen jugo para aprovechar datos sensibles del servidor...
header:
show_date: true
classes: wide
header:
  teaser: "/assets/images/Path_traversal/directory-traversal.svg"
  teaser_home_page: true
  icon: "/assets/images/Path_traversal/directory-traversal.svg"
categories:
  - Web Hacking
tags:
  - pathtraversal
  - hackweb
---

# Path  Traversal

- Que es Path traversal?

**Path traversal** tambien conocido como **Directory Traversal** es una vulnerabilidad de seguridad web que permite a un atacante leer archivos arbitrarios en el servidor que esta ejecutando la aplicacion.En algunos casos esta vulnerabilidad se puede concatenar permitiendo escritura de archivos, modificacion de datos y en ultima instancia hasta tomar el control del servidor.

#### Bueno dejemos la teoria y movamonos a la practica

En lo siguientes apartados se especificara con `I,II,II` y asi sucesivamente cada ejemplo. Haremos uso de la aplicaion ` bWAPP` como medio de aprendizaje.

## Path traversal `I`

![](/assets/images/Path_traversal/1.png)        

```http
http://localhost/bWAPP/directory_traversal_2.php?directory=documents
```

![](/assets/images/Path_traversal/2.png)      

 La URL `directory_traversal_2.php` toma un parametro  `directory` .Los archivos que pertenecen a la carpeta documentos  se almacenan en el disco en la ubicacion **/var/www/html/bWAPP/documents**

Por lo tanto una aplicacion web que no implemente medidas de seguridad contra este tipo de ataques se puede vulnerar la web solicitando otra directorio.

```http
http://localhost/bWAPP/directory_traversal_2.php?directory=documents/../
```

![](/assets/images/Path_traversal/3.png)    

Esto causa que la aplicacion lea desde la siguiente ruta 

`/var/www/html/bWAPP/documents/../`

La secuancia **../** es valida dentro de una ruta de archivo y significa disminuir un nivel en la estructura de directorios.

##### `Nota`

En los sistemas operativos basados en *UNIX* el directorio **/etc/passwd/** es un archivo estandar que contiene detalles de los usuarios que estan registrados en el servidor.

En  *Windows* tanto **../** como **..\\** son secuencias transversales validas y un ejemplo equivalente al de *UNIX* seria

**/windows/win.ini**

## Path traversal `II`

---

Es comun encontrarnos obstaculos para explotar este tipo de vulnerabilidad.Muchas aplicaciones implementan defensa contra este tipo de ataques.

Es posible evadir estos filtros con **variadas tecnicas**

-  A veces es conveniente utilizar la ruta absoluta  `Ej:`

```http
http://localhost/bWAPP/directory_traversal_2.php?directory=/etc/passwd
```



- **Supongamos** que una aplicacion sanetiza la entrada y la we no permita esta secuencia **../**       

   ​                                                                 **¿Como evadirla​?**

```http
http://localhost/bWAPP/directory_traversal_2.php?directory=documents....//....//
```

Asi lo que filtraria la web seria la primera secuencia y por lo tanto evadimos el filtro.

## Path Traversal `III`

En otro variante tambien es posible utilizar  varios encoders no estándares para evadir  la web.

- Utilizar estos encoders `..%c0%af` o  `..%252f`

  **¿** Que significan estos encoders **?** 

  La codificacion **URL** también conocida como codificación porcentual  Ej: "**%xx**"  representa un byte donde cada `X` es un dígito hexadecimal.Por lo tanto **%c0%af**  en una URL corresponde a los bytes **C0  AF** que respectivamente son los bytes 192 y 175 del código `ASCII`.

  Entonces el codigo **ASCII** solo define símbolos para los bytes del `0-127`.Muchas webs de hoy en día utilizan la codificación **UTF-8** siendo los caracteres ASCII codificados como de costumbre con un solo byte.Por  lo tanto algunas librerias  pueden optar por no verificar que el valor del caracter Unicode de dos bytes este en el rango válido.

  Bien, ya explicado de manera resumida lo anterior.

  **¿Por qué este ataque tiene éxito?**

  A menudo sucuede que la sanetizacion de entradas incorrectas y la decodificación de los símbolos Unicode se realiza en diferentes etapas de la aplición.

  Supongamos que una aplicación web no permite el siguiente **path traversal**

  ```http
  http://localhost/bWAPP/directory_traversal_2.php?directory=/../../../../
  ```

  Sin embargo, que pasaría si la decodificación Unicode se realiza después de la verificación que evita el **path traversal**.

  ```http
  http://localhost/bWAPP/directory_traversal_2.php?directory=..%2f..%2f..%2f..%2fetc%2fshadow
  ```

  Como por arte de magia tenemos un ***3312***

  ![SecLists Payloads](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing)  

  - [x] ***H@ppY H@(K1NG!***

  

   

