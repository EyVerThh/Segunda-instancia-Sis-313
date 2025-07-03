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
      addresses: [192.168.1.102/24]
      routes:
      - to: default
        via: 192.168.1.1
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
const PORT = 3001; // Puerto distinto para APP1

app.use(express.json());

app.get('/', (req, res) => {
  res.send('¡Bienvenido a la API SIS313 APP1!');
});

app.get('/usuarios', (req, res) => {
  res.json([
    { id: 1, nombre: 'Carlos' },
    { id: 2, nombre: 'Ana' }
  ]);
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
192.168.1.102:3001
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
const PORT = 3001;

app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Conexión con pool para MariaDB remota
const pool = mysql.createPool({
  host: '192.168.1.104',
  port: 3306,
  user: 'appcrud',
  password: 'CrudPass123!',
  database: 'biblioteca',
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0
});

app.get('/', (req, res) => {
  pool.query('SELECT * FROM libros', (err, libros) => {
    if (err) {
      console.error('Error consultando la base de datos:', err);
      return res.status(500).send('Error consultando la base de datos: ' + err.message);
    }

    let html = `
      <!DOCTYPE html>
      <html lang="es">
      <head>
        <meta charset="UTF-8" />
        <title>Libros - SIS313</title>
        <style>
          body { font-family: Arial, sans-serif; margin: 40px; }
          table { border-collapse: collapse; width: 100%; margin-top: 20px; }
          th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
          th { background-color: #f2f2f2; }
          form input { padding: 6px; margin-right: 10px; }
          button { padding: 6px 12px; }
          .btn { cursor: pointer; }
          .btn-edit { color: blue; }
          .btn-delete { color: red; }
        </style>
      </head>
      <body>
        <h1>Libros - SIS313</h1>

        <form id="formAgregar">
          <input name="titulo" placeholder="Título del libro" required />
          <input name="autor" placeholder="Autor" required />
          <input name="anio_publicacion" type="number" placeholder="Año" required />
          <input name="precio" type="number" step="0.01" placeholder="Precio" required />
          <button type="submit">Agregar Libro</button>
        </form>

        <table id="tablaLibros">
          <thead>
            <tr>
              <th>ID</th><th>Título</th><th>Autor</th><th>Año</th><th>Precio</th><th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            ${libros.map(l => `
              <tr data-id="${l.id}">
                <td>${l.id}</td>
                <td class="titulo">${l.titulo}</td>
                <td class="autor">${l.autor}</td>
                <td class="anio">${l.anio_publicacion}</td>
                <td class="precio">$${Number(l.precio).toFixed(2)}</td>
                <td>
                  <button class="btn btn-edit" onclick="editar(${l.id})">Editar</button>
                  <button class="btn btn-delete" onclick="borrar(${l.id})">Borrar</button>
                </td>
              </tr>
            `).join('')}
          </tbody>
        </table>

        <script>
          const form = document.getElementById('formAgregar');
          form.addEventListener('submit', async e => {
            e.preventDefault();
            const formData = new FormData(form);
            const data = {
              titulo: formData.get('titulo'),
              autor: formData.get('autor'),
              anio_publicacion: parseInt(formData.get('anio_publicacion')),
              precio: parseFloat(formData.get('precio'))
            };
            const res = await fetch('/agregar', {
              method: 'POST',
              headers: {'Content-Type': 'application/json'},
              body: JSON.stringify(data)
            });
            if (res.ok) {
              form.reset();
              cargarLibros();
            } else {
              alert('Error al agregar libro');
            }
          });

          async function cargarLibros() {
            const res = await fetch('/libros');
            const libros = await res.json();
            const tbody = document.querySelector('#tablaLibros tbody');
            tbody.innerHTML = libros.map(l => \`
              <tr data-id="\${l.id}">
                <td>\${l.id}</td>
                <td class="titulo">\${l.titulo}</td>
                <td class="autor">\${l.autor}</td>
                <td class="anio">\${l.anio_publicacion}</td>
                <td class="precio">$\${Number(l.precio).toFixed(2)}</td>
                <td>
                  <button class="btn btn-edit" onclick="editar(\${l.id})">Editar</button>
                  <button class="btn btn-delete" onclick="borrar(\${l.id})">Borrar</button>
                </td>
              </tr>\`).join('');
          }

          async function borrar(id) {
            if (!confirm('¿Seguro que quieres borrar este libro?')) return;
            const res = await fetch('/borrar/' + id, { method: 'DELETE' });
            if (res.ok) {
              cargarLibros();
            } else {
              alert('Error al borrar libro');
            }
          }

          async function editar(id) {
            const tr = document.querySelector(\tr[data-id="\${id}"]\);
            const tituloTd = tr.querySelector('.titulo');
            const autorTd = tr.querySelector('.autor');
            const anioTd = tr.querySelector('.anio');
            const precioTd = tr.querySelector('.precio');

            const tituloOld = tituloTd.textContent;
            const autorOld = autorTd.textContent;
            const anioOld = anioTd.textContent;
            const precioOld = precioTd.textContent.replace('$','');

            tituloTd.innerHTML = \<input type="text" id="editTitulo" value="\${tituloOld}">\;
            autorTd.innerHTML = \<input type="text" id="editAutor" value="\${autorOld}">\;
            anioTd.innerHTML = \<input type="number" id="editAnio" value="\${anioOld}">\;
            precioTd.innerHTML = \<input type="number" step="0.01" id="editPrecio" value="\${precioOld}">\;

            const btnEdit = tr.querySelector('.btn-edit');
            btnEdit.textContent = 'Guardar';
            btnEdit.onclick = async () => {
              const nuevoTitulo = document.getElementById('editTitulo').value;
              const nuevoAutor = document.getElementById('editAutor').value;
              const nuevoAnio = parseInt(document.getElementById('editAnio').value);
              const nuevoPrecio = parseFloat(document.getElementById('editPrecio').value);
              const res = await fetch('/editar/' + id, {
                method: 'PUT',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({ titulo: nuevoTitulo, autor: nuevoAutor, anio_publicacion: nuevoAnio, precio: nuevoPrecio })
              });
              if (res.ok) {
                cargarLibros();
              } else {
                alert('Error al editar libro');
              }
            };
          }

          cargarLibros();
        </script>
      </body>
      </html>
    `;
    res.send(html);
  });
});

app.get('/libros', (req, res) => {
  pool.query('SELECT * FROM libros', (err, results) => {
    if (err) {
      console.error('Error en /libros:', err);
      return res.status(500).json({ error: err.message });
    }
    res.json(results);
  });
});

app.post('/agregar', (req, res) => {
  const { titulo, autor, anio_publicacion, precio } = req.body;
  pool.query(
    'INSERT INTO libros (titulo, autor, anio_publicacion, precio) VALUES (?, ?, ?, ?)',
    [titulo, autor, anio_publicacion, precio],
    (err) => {
      if (err) {
        console.error('Error en /agregar:', err);
        return res.status(500).json({ error: err.message });
      }
      res.status(200).json({ mensaje: 'Libro agregado' });
    }
  );
});

app.put('/editar/:id', (req, res) => {
  const id = req.params.id;
  const { titulo, autor, anio_publicacion, precio } = req.body;
  pool.query(
    'UPDATE libros SET titulo = ?, autor = ?, anio_publicacion = ?, precio = ? WHERE id = ?',
    [titulo, autor, anio_publicacion, precio, id],
    (err) => {
      if (err) {
        console.error('Error en /editar:', err);
        return res.status(500).json({ error: err.message });
      }
      res.status(200).json({ mensaje: 'Libro actualizado' });
    }
  );
});

app.delete('/borrar/:id', (req, res) => {
  const id = req.params.id;
  pool.query('DELETE FROM libros WHERE id = ?', [id], (err) => {
    if (err) {
      console.error('Error en /borrar:', err);
      return res.status(500).json({ error: err.message });
    }
    res.status(200).json({ mensaje: 'Libro borrado' });
  });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(Servidor API corriendo en http://localhost:${PORT});
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
