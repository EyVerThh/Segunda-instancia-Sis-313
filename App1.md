
# **APP1**
<div align="justify">

**Objetivo de app1** 

La aplicación app1 dentro del proyecto es una instancia del servicio, brinda funciones de CRUD, es decir que nos deja ver los datos almaecnados en nuestra bd, hace consultas a la bd para poder actualizar, eliminar o agregas datos de en nuestra bd, en este caso libros y otros datos en ellos como su precio, autor, etc. Los servicios de la app2 son los mismos ya que que estas estan conectadas con el balanceador, y a la bd

## **Configuración de IP estatica**

* **Cambio de IP dinámica a estatica:**
```bash
 sudo nano /etc/netplan/50-cloud-init.yaml
   ```
Dentro del archivo /etc/...
```bash
 network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.105.11/24]
      routes:
      - to: default
        via: 192.168.105.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

   ```

## **Instalación de Nginx:**
* **Instalación de Nginx:**
```bash
   sudo apt update && sudo apt install nginx
   ```

* **Verificación del Estado del Servicio Nginx:**
```bash
   sudo systemctl status nginx
   ```

* **Instalación de módulos:**
```bash
 sudo apt install php8.3-fpm php8.3-mbstring php8.3-curl php8.3-xml php8.3-mysql php8.3-zip
```

## **node.js con nvm**

* **instalación**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

```bash
\. "$HOME/.nvm/nvm.sh"
```
```bash
nvm install 22
```
```bash
node -v
```
respuesta: v22.16.0

```bash
nvm current
```
respuesta: v22.16.0

```bash
npm -v
```
respuesta: 11.4.2
## **express js**

* **carpeta para la aplicación:**

```bash
mkdir myapp1
cd myapp1
```
* **subcarpeta para el código de la API:**
Se la crea dentro de la carpeta myapp1
```bash
mkdir api
cd api/
```
* **Instalación de express js:**
```bash
npm init
```
* **Configuracion de express js:**

```bash
package name: (api)
version: (1.0.0)
description: API Backend SIS313
entry point: (index.js)
test command:
git repository:
keywords: API, SIS313
author: SIS313
license: (ISC)
About to write to /home/app1/myapp1/api/package.json:

{
  "name": "api",
  "version": "1.0.0",
  "description": "API Backend SIS313",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "API",
    "SIS313"
  ],
  "author": "SIS313",
  "license": "ISC"
}


Is this OK? (yes) y
```

* **Verificación:**
Dentro de /myapp1/api se ejecuta:
```bash
ls
```
con ls se deveria mostrar package.json, que es donde se guarda los archivos que permiten trabajar con nodejs y expressjs de la aplicación, usados en index.js
* **Añadir extensiones dentro del proyecto:**
```bash
npm install -g npm@11.4.2
```

## **prueba:**
dentro de index.js
```bash
nano index.js
```
Dentro se haya este código:
```bash
const express = require('express');
const app = express();
const PORT = 3000; // Puerto distinto para APP1

app.use(express.json());

app.get('/', (req, res) => {
  res.send('¡Bienvenido a la API SIS313 APP1!');
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(Servidor APP1 corriendo en http://localhost:${PORT});
});
```
Para hacer correr la API, se ejecuta:
```bash
node index.js
```
si todo sale bien sale: Servidor API corriendo en http://localhost:3001

para ver si manda el mensaje de prueba podemos poner en el navegador:
```bash
192.168.1.102:3000
```

si ya se tiene el balanceador se puede probar con:
```bash
sis313.final #dns del balanceador
```
# **Aplicación 1 (con la BD creada)**
# **Cliente de mariadb:**
* **extensiones de cliente para mariadb:**
```bash
npm install mysql2
```

* **extensiones de para usar express:**
```bash
npm install express
```
# **CRUD DE BIBLIOTECA**
```bash
const express = require('express');
const mysql = require('mysql2');
const app = express();
const PORT = 3000;

app.use(express.urlencoded({ extended: true }));
app.use(express.json());

const pool = mysql.createPool({
  host: '192.168.105.13',
  user: 'eyverth',
  password: '1234',
  database: 'crud_db',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

app.get('/', (req, res) => {
  pool.query('SELECT * FROM items', (err, items) => {
    if (err) {
      console.error('Error consultando la base de datos:', err);
      return res.status(500).send('Error consultando la base de datos: ' + err.message);
    }

    let html = `
      <h1 style="color:blue;">App1 - Servidor Azul 📘</h1>
      <h2>Gestión de Items</h2>
      <form method="POST" action="/agregar">
        <input name="nombre" placeholder="Nombre del item" required />
        <button type="submit">Agregar Item</button>
      </form>
      <ul>
        ${items.map(item => `
          <li>
            ID: ${item.id}, Nombre: ${item.nombre}
            <form method="POST" action="/borrar/${item.id}" style="display:inline;">
              <button type="submit">Eliminar</button>
            </form>
          </li>
        `).join('')}
      </ul>
    `;
    res.send(html);
  });
});

app.post('/agregar', (req, res) => {
  const { nombre } = req.body;
  pool.query('INSERT INTO items (nombre) VALUES (?)', [nombre], (err) => {
    if (err) {
      console.error('Error al agregar:', err);
      return res.status(500).send('Error al agregar item.');
    }
    res.redirect('/');
  });
});

app.post('/borrar/:id', (req, res) => {
  const { id } = req.params;
  pool.query('DELETE FROM items WHERE id = ?', [id], (err) => {
    if (err) {
      console.error('Error al borrar:', err);
      return res.status(500).send('Error al borrar item.');
    }
    res.redirect('/');
  });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Servidor App1 corriendo en http://localhost:${PORT}`);
});
```

volvemos a ejecutar node index.js y tendria que devolver: 
- Servidor API corriendo en http://localhost:3001
Conectado a la base de datos

## **Verificación**
al ejecuar dentro del balanceador:
```bash
sis313.final 
```
ya deveria esta visible el CRUD
</div>
