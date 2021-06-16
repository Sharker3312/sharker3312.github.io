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

---

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

Algunas aplicaciones 