# PSTI Tasks

PSTI Tasks es una aplicación para gestionar tareas, desplegada en Heroku y construida con NodeJS, Express, MongoDB y Vue.

## Requisitos

### En Línea
- Cuenta en [GitHub](https://github.com) (opcional)
- Cuenta en [MongoDB Cloud](https://cloud.mongodb.com)
- Cuenta en [Heroku](https://heroku.com)

### Instalados
- [Postman](https://www.postman.com) (opcional)

Asegúrate de tener instalados y configurados los siguientes:

```bash
$ git --version
$ heroku --version
$ npm --version
$ node --version
```

## Inicialización del Proyecto

Crea una carpeta para el proyecto e inicializa el proyecto:

```bash
$ mkdir todo-tasks
$ cd todo-tasks
$ npm init
```

Ingresa los valores solicitados y cambia el "entry point" de `index.js` a `server.js`. El archivo `package.json` se generará con la información del proyecto.

## Estructura de Archivos

En la carpeta `todo-tasks`, crea los siguientes archivos:

- `.env.dist`: Ejemplo de archivo `.env` para variables de entorno.
- `.gitignore`: Ignora archivos o carpetas en el versionamiento.
- `api.js`: Definición de la API para la lista de tareas.
- `index.html`: Interfaz para administrar la lista de tareas.
- `package.json`: Configuración del proyecto.
- `server.js`: Servidor web.
- `task_schema.js`: Esquema de una tarea.

## Hola Mundo en Express

En `server.js`, agrega el siguiente código:

```javascript
const express = require('express');
const port = process.env.PORT || 3000;
const app = express();

app.listen(port, function () {
    console.log("Server is listening at port: " + port);
});

app.get('/', function (req, res) {
    res.send("hello world");
});
```

Instala Express y ejecuta el servidor:

```bash
$ cd todo-tasks
$ npm install express
$ npm start
```

Visita http://localhost:3000 y verás "hello world" en el navegador.

## Creando la API de Tareas en MongoDB

### Conexión a la Base de Datos

Instala Mongoose y conecta con la base de datos en `api.js`:

```bash
$ npm install mongoose@6.10.0
```

En `api.js`:

```javascript
const mongoose = require("mongoose");
const express = require("express");
const router = express.Router();
const query = "mongodb+srv://<user>:<password>@<cluster-url>/<dbname>?retryWrites=true&w=majority";
const db = (query);

mongoose.Promise = global.Promise;

mongoose.connect(db, {
    useNewUrlParser: true,
    useUnifiedTopology: true
}, function (error) {
    if (error) {
        console.log("Error!" + error);
    } else {
        console.log("Connected to the database successfully");
    }
});

module.exports = router;
```

Importa `api.js` en `server.js` y úsalo en la ruta `/api`:

```javascript
const api = require('./api');
app.use('/api', api);
```

Ejecuta `npm start` y verifica que se conecte a la base de datos exitosamente.

## Parseando Mensajes con Body-Parser

En `server.js`, usa `body-parser` para manipular mensajes en el body:

```javascript
const bodyParser = require('body-parser');
app.use(bodyParser.json());
```

## Definiendo la Estructura de una Tarea

Crea `task_schema.js`:

```javascript
const mongoose = require('mongoose');

const TaskSchema = new mongoose.Schema({
    TaskId: Number,
    Name: String,
    Deadline: Date,
});

module.exports = mongoose.model('task', TaskSchema, 'Tasks');
```

En `api.js`, importa el esquema de la tarea:

```javascript
const TaskModel = require('./task_schema');
```

## Creando, Consultando, Actualizando y Eliminando Tareas

### Crear una Tarea

En `api.js`, agrega la ruta para crear tareas:

```javascript
router.post('/create-task', function (req, res) {
    let task_id = req.body.TaskId;
    let name = req.body.Name;
    let deadline = req.body.Deadline;

    let task = {
        TaskId: task_id,
        Name: name,
        Deadline: deadline
    }
    var newTask = new TaskModel(task);

    newTask.save(function (err, data) {
        if (err) {
            console.log(err);
            res.status(500).send("Internal error\n");
        }
        else {
            res.status(200).send("OK\n");
        }
    });
});
```

Prueba creando una tarea:

```bash
$ curl -i -X POST -H "Content-Type: application/json" -d '{"TaskId": 123, "Name":"Estudiar para el quiz", "Deadline": "2020-12-01"}' http://localhost:3000/api/create-task
```

### Consultar Todas las Tareas

Agrega la ruta para consultar todas las tareas:

```javascript
router.get('/all-tasks', function (req, res) {
    TaskModel.find(function (err, data) {
        if

 (err) {
            res.status(500).send("Internal error\n");
        }
        else {
            res.status(200).send(data);
        }
    });
});
```

Prueba consultando todas las tareas:

```bash
$ curl -i -X GET -H "Content-Type: application/json" http://localhost:3000/api/all-tasks
```

### Actualizar una Tarea

Agrega la ruta para actualizar una tarea:

```javascript
router.post('/update-task', function (req, res) {
    TaskModel.updateOne({ TaskId: req.body.TaskId }, {
        Name: req.body.Name,
        Deadline: req.body.Deadline
    }, function (err, data) {
        if (err) {
            res.status(500).send("Internal error\n");
        } else {
            res.status(200).send("OK\n");
        }
    });
});
```

Prueba actualizando una tarea:

```bash
$ curl -i -X POST -H "Content-Type: application/json" -d '{"TaskId": 123, "Name":"Estudiar para el quiz MODIFICADO", "Deadline": "2020-12-02"}' http://localhost:3000/api/update-task
```

### Eliminar una Tarea

Agrega la ruta para eliminar una tarea:

```javascript
router.delete('/delete-task', function (req, res) {
    TaskModel.deleteOne({ TaskId: req.body.TaskId }, function (err, data) {
        if (err) {
            res.status(500).send("Internal error\n");
        } else {
            res.status(200).send("OK\n");
        }
    });
});
```

Prueba eliminando una tarea:

```bash
$ curl -i -X DELETE -H "Content-Type: application/json" -d '{"TaskId": 123}' http://localhost:3000/api/delete-task
```

## Variables de Entorno

Para mayor seguridad y confidencialidad, utiliza variables de entorno. Instala `node-env-file`:

```bash
$ cd todo-tasks
$ npm install node-env-file
```

En `server.js`:

```javascript
let environment = null;

if (!process.env.ON_HEROKU) {
    console.log("Cargando variables de entorno desde archivo");
    const env = require('node-env-file');
    env(__dirname + '/.env');
}

environment = {
    DBMONGOUSER: process.env.DBMONGOUSER,
    DBMONGOPASS: process.env.DBMONGOPASS,
    DBMONGOSERV: process.env.DBMONGOSERV,
    DBMONGO: process.env.DBMONGO,
};

var query = 'mongodb+srv://' + environment.DBMONGOUSER + ':' + environment.DBMONGOPASS + '@' + environment.DBMONGOSERV + '/' + environment.DBMONGO + '?retryWrites=true&w=majority';
```

Crea un archivo `.env` con la información de conexión a MongoDB y no lo compartas. Puedes crear un archivo `.env.dist` con la estructura para compartir sin datos sensibles.

**.env.dist:**

```
DBMONGO=YOUR_DB_NAME
DBMONGOPASS=YOUR_USER_PASSWORD
DBMONGOSERV=YOUR_CLUSTER_SERVER
DBMONGOUSER=YOUR_USER
```

**.env:**

```
DBMONGO=mydatabase
DBMONGOPASS=mypassword
DBMONGOSERV=cluster.mongodb.net
DBMONGOUSER=myuser
```

Asegúrate de que al ejecutar el servidor, se muestre "Connected to the database successfully".