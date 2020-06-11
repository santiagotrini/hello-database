# Hello Database

API en Express + MongoDB

## Qué vamos a hacer

Una API (_application programming interface_) para comunicarnos con una base de datos de MongoDB. No voy a explicar que es una base de datos, ya tienen una materia entera para eso.

Este es uno de los usos más comunes de Express. Una API nos da una interfaz para comunicar una aplicación web (_frontend_) con una base de datos. Tanto la base de datos como la aplicación de Express se consideran parte del _backend_.

El ejemplo va a ser mínimo, tenemos una base de datos con usuarios y a través de la API podemos obtener esa información en formato JSON. Ya tendremos tiempo de complicar más las cosas.

## Antes de empezar

Para este ejemplo necesitamos lo mismo que en `express-hello-world` más:

- Una cuenta en MongoDB Atlas
- El server de MongoDB instalado en nuestra computadora (para probar localmente).

Verificar que podamos acceder a la _shell_ de MongoDB en la terminal.

```
$ mongo --version
MongoDB shell version v4.2.7
... etc ...
$ mongo
> exit
bye
```

Si al ejecutar mongo nos recibe un prompt de Mongo (el `>`) va todo bien. Pueden salir con exit.

## Creando el proyecto

Esto ya lo vimos, en resumen:

```
$ mkdir hello-database
$ cd hello-database
$ git init
$ npm init -y
$ echo node_modules > .gitignore
$ npm install express
$ touch index.js
```

Recuerden que `npm install` requiere Internet. Agregamos una nueva librería para conectarnos y trabajar con bases de datos de MongoDB.

```
$ npm install mongoose
```

Agregamos a `scripts` en el `package.json` la propiedad `"start": "node index.js"` (lo mismo que hicimos en `express-hello-world`).

## Mandamos JavaScript

En `index.js` le decimos a nuestra app que se conecte a la base de datos.
