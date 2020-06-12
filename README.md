# Hello Database

API en Express + MongoDB

## Qué vamos a hacer

Una API (_application programming interface_) para comunicarnos con una base de datos de MongoDB. No voy a explicar que es una base de datos, ya tienen una materia entera para eso.

Este es uno de los usos más comunes de Express. Una API nos da una interfaz para comunicar una aplicación web (_frontend_) con una base de datos. Tanto la base de datos como la aplicación de Express se consideran parte del _backend_.

El ejemplo va a ser mínimo, tenemos una base de datos con usuarios y a través de la API podemos obtener esa información en formato JSON. Ya tendremos tiempo de complicar más las cosas.

## Antes de empezar

Para este ejemplo necesitamos lo mismo que en `express-hello-world` y dos cosas más:

- Una cuenta en MongoDB Atlas
- El server de MongoDB instalado en nuestra computadora (para probar localmente).

Cuando instalen el server de MongoDB en Windows pueden omitir la instalación de MongoDB Compass, hay un checkbox en el instalador. No lo vamos a usar.

Verificar que podamos acceder a la _shell_ de MongoDB en la terminal. Si esto no funciona es porque no tienen el ejecutable en el PATH. Tarea de investigación como editar variables de entorno en Windows. Concretamente agregar la ruta `C:\Program Files\MongoDB\Server\4.2\bin` (o algo similar) a la variable de entorno PATH.

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

```js
// index.js

// ahora tambien importamos mongoose
const express  = require('express');
const mongoose = require('mongoose');

// puerto y base de datos
const port = process.env.PORT        || 3000;
const db   = process.env.MONGODB_URI || 'mongodb://localhost/hellodb';

const app = express();

// conexion a la base de datos
mongoose.set('useUnifiedTopology', true);
mongoose.set('useFindAndModify', false);
mongoose
  .connect(db, { useNewUrlParser: true })
  .then(() => {
    console.log(`DB connected @ ${db}`);
  })
.catch(err => console.error(`Connection error ${err}`));

// el server escucha todo
app.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
```

Aparecieron algunas cosas nuevas. En primer lugar importamos una libreria nueva: mongoose. En segundo lugar ademas de la variable que indica el puerto del server tenemos una similar para la base de datos, misma idea, un valor cuando estamos en Heroku y otro cuando ejecutamos localmente. La URI `mongodb://localhost/hellodb` es la base de datos local en nuestra computadora.

Todo lo que está entre el primer `mongoose.set` y el `.catch()` es el _snippet_ de código para conectarnos a una base de datos con mongoose. Un _snippet_ es un pedazo de código que copiamos prácticamente igual en distintas aplicaciones. En resumen son dos llamados a la función `mongoose.set()` para evitar unos _warnings_ (prueben de comentar esas dos líneas). Y un llamado a la función `mongoose.connect()`. El motivo de que lo vean raro es que usa algo llamado **promesas**, que en esencia cumple la misma función que una _callback_.

En `app.listen()` sí usamos una _callback_. Es el segundo argumento de la función. Los dos usos de `console.log()` son para mostrar que todo anduvo bien y que nuestro server esta escuchando en tal puerto y conectado a tal base de datos. Pueden probar la app con `npm start` y van a ver algo similar a esto.

```
$ npm start

> hello-database@1.0.0 start /home/santiago/hello-database
> node index.js

Server listening on port 3000
DB connected @ mongodb://localhost/hellodb
```

## Creando la base de datos

Tenemos que crear una base de datos para trabajar. Lo hacemos mediante un sencillo script usando Mongoose. Las bases de datos de MongoDB guardan la información en colecciones de documentos. Las colecciones pueden ser usuarios, animales, compras o lo que ustedes quieran. Los documentos son objetos de JavaScript. En nuestro caso vamos a tener una colección de usuarios, y el objeto que represente un usuario va a tener esta pinta.

```js
{
  id: 1,
  name: 'Juan',
  mail: 'juan@mail.com',
  birthday: '2000-01-14'
}
```

Cada usuario va a ser un documento en la colección de usuarios. Una colección no es más que un array de objetos con idéntica estructura. Por si todavía no se enteraron la idea acá es que (casi) todo se hace en JavaScript, bases de datos incluídas.

Para que Mongoose interactúe con MongoDB tenemos que crear **modelos**. Un modelo es un objeto que describe una colección, y lo hacemos generalmente en un archivo separado dentro de una carpeta llamada `models`. Así que en la terminal.

```
$ mkdir models
$ cd models
$ touch User.js
```

La convención es usar SnakeCase y nombres en singular para los nombres de los archivos de los modelos. Si tenemos un modelo de animales será `Animal.js`. Si tenemos animales de zoológico será `ZooAnimal.js`. El archivo `models/User.js` quedaría así.

```js
// User.js

// necesitamos importar mongoose
const mongoose = require('mongoose');

// los modelos se crean a partir de un schema
const UserSchema = new mongoose.Schema({
  id: Number,
  name: String,
  mail: String,
  birthday: Date
});
// el schema describe la pinta de un documento de la coleccion

// creamos el modelo llamando a mongoose.model(nombre, schema)
const User = mongoose.model('User', UserSchema);

// hay que exportar el modelo para usarlo despues en otros archivos
module.exports = User;
```

El ejemplo de `User.js` es bien sencillo pero los _schemas_ tienen muchas características más que iremos viendo a medida que las necesitemos. La documentación oficial de Mongoose es la referencia para este tipo de cosas: https://mongoosejs.com/docs/guide.html.

Con el modelo listo nos creamos un script llamado `populatedb.js` en la carpeta del proyecto y copiamos el siguiente código.

```js
// populatedb.js

// necesitamos importar mongoose
const mongoose = require('mongoose');

// importar el modelo de usuario
const User = require('./models/User');

// la URI de la db
const db = 'mongodb://localhost/hellodb';

// array de usuarios para ingresar a la db
const users = [
  {
    id: 1,
    name: 'Juan',
    mail: 'juan@mail.com',
    birthday: '2000-05-24'
  },
  {
    id: 2,
    name: 'Maria',
    mail: 'maria@mail.com',
    birthday: '2000-02-13'
  },
  {
    id: 3,
    name: 'Pedro',
    mail: 'pedro@mail.com',
    birthday: '2000-05-19'
  },
  {
    id: 4,
    name: 'Julia',
    mail: 'julia@mail.com',
    birthday: '1998-03-01'
  }
];

// conectarse a la db
mongoose.set('useUnifiedTopology', true);
mongoose.set('useFindAndModify', false);
mongoose
  .connect(db, { useNewUrlParser: true })
  .then(() => {
    // si nos conectamos con exito mostrar mensajes
    // e insertar los usuarios en el array
    console.log(`DB connected @ ${db}`);
    console.log('Populating DB...');
    User.insertMany(users, (err, users) => {
      if (err) throw err;
      // un mensaje con la cantidad de documentos insertados
      console.log(`${users.length} documents inserted!`);
      // cerramos la conexion cuando terminamos
      mongoose.connection.close();
    });
  })
.catch(err => console.error(`Connection error ${err}`));
```

No importa si no lo entienden del todo, en resumen lo que hace es conectarse a la base de datos y poner cuatro documentos en la colección `users`. Cuando termina se desconecta. Lo pueden ejecutar desde la terminal con `node populatedb.js`.

Chequeamos con la shell de Mongo que el script haya hecho su trabajo.

```
$ mongo
> show databases
admin    0.000GB
config   0.000GB
hellodb  0.000GB
local    0.000GB
> use hellodb
switched to hellodb
> show collections
users
> db.users.find().pretty()
{
  "_id": ObjectId("5ee..."),
  "id": 1,
  "name": "Juan",
  ... etc ...
}
... etc ...
```

Nos debería mostrar los cuatro objetos en la colección. Ya que estamos en la shell de Mongo probemos de agregar un usuario más desde ahí. Después salimos de la shell de Mongo con `exit`.

```
> db.users.insert({ id: 5, name: "Mario", mail: "mario@mail.com", birthday: new Date("1995-04-14") })
WriteResult({ "nInserted" : 1 })
> exit
bye
```

Listo, tenemos base de datos para hacer _queries_.

## Haciendo queries

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
