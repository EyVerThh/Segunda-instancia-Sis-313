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

