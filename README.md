# XennJS - Express.js Minimalist Design Pattern

XennJS adalah design pattern minimalis untuk pengembangan backend menggunakan **Express.js** dan **JavaScript**. XennJS dirancang untuk memudahkan pengembangan **RESTful API** menggunakan arsitektur **MVC** (Model-View-Controller), yang membuat aplikasi menjadi modular, scalable, dan mudah di-maintain. Pada versi ini, XennJS menggunakan **mysql2** untuk mengelola interaksi database.

## Fitur Utama:
- **MVC Architecture**: Memisahkan logika bisnis, data, dan API handling agar lebih mudah dikelola dan di-maintain.
- **Highly Modular**: Setiap komponen seperti model, controller, dan service ditata rapi untuk mendukung pengembangan yang efisien dan scalable.
- **Minimal Setup**: Konfigurasi minimalis dengan filosofi sederhana namun tetap fleksibel.

---


## **Tree Folder**
```
xennjs/
│
├── app/
│   ├── middleware/
│   │   └── adminMiddleware.js
│   ├── models/
│   │   ├── adminModel.js
│   │   ├── categoryModel.js
│   │   └── productModel.js
│   ├── controllers/
│   │   ├── adminController.js
│   │   ├── categoryController.js
│   │   └── productController.js
│   ├── routes/
│   │   ├── adminRoutes.js
│   │   ├── categoryRoutes.js
│   │   ├── productRoutes.js
│   │   └── main.js
│
├── config/
│   └── dbconnect.js
│
├── migrations/
│   ├── migrate-up.js
│   └── migrate-down.js
│
├── app.js
├── server.js
├── .env
├── package.json
└── package-lock.json

```

---

### **1. File Terluar**

#### **File `package.json`**
```json
{
  "name": "xennjs",
  "version": "1.0.0",
  "description": "An Express MVC application with MySQL2",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "migrate-up": "node migrations/migrate-up.js",
    "migrate-down": "node migrations/migrate-down.js"
  },
  "dependencies": {
    "dotenv": "^16.1.4",
    "express": "^4.18.2",
    "mysql2": "^3.6.0",
    "nodemon": "^3.0.1",
    "jsonwebtoken": "^9.0.0"
  }
}

```

---

#### **File `.env`**
```env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=password
DB_NAME=mydatabase
PORT=9988

JWT_SECRET=your_jwt_secret_key

```

---

#### **File `server.js`**
```javascript
const app = require('./app');
const dotenv = require('dotenv');

dotenv.config();

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

---

#### **File `app.js`**
```javascript
const express = require('express');
const mainRoutes = require('./app/routes/main');

const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.use('/', mainRoutes);

module.exports = app;
```

---

### **2. Konfigurasi Database**

#### **File `config/dbconnect.js`**
```javascript
const mysql = require('mysql2');
const dotenv = require('dotenv');

dotenv.config();

const pool = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
});

module.exports = pool.promise();
```

---

### **3. Model**

#### **File `app/models/adminModel.js`**
```javascript
const db = require('../../config/dbconnect');

exports.findAdminByUsername = async (username) => {
  const [rows] = await db.query('SELECT * FROM admin WHERE username = ?', [username]);
  return rows[0];
};
```

---

#### **File `app/models/categoryModel.js`**
```javascript
const db = require('../../config/dbconnect');

exports.getAllCategories = async () => {
  const [rows] = await db.query('SELECT * FROM categories ORDER BY created_at DESC');
  return rows;
};

exports.getCategoryById = async (id) => {
  const [rows] = await db.query('SELECT * FROM categories WHERE id = ?', [id]);
  return rows[0];
};

exports.createCategory = async (name) => {
  const [result] = await db.query('INSERT INTO categories (name) VALUES (?)', [name]);
  return result.insertId;
};

exports.updateCategory = async (id, name) => {
  await db.query('UPDATE categories SET name = ? WHERE id = ?', [name, id]);
};

exports.deleteCategory = async (id) => {
  await db.query('DELETE FROM categories WHERE id = ?', [id]);
};
```

---

#### **File `app/models/productModel.js`**
```javascript
const db = require('../../config/dbconnect');

exports.getAllProducts = async () => {
  const [rows] = await db.query(`
    SELECT products.*, categories.name AS category_name 
    FROM products 
    LEFT JOIN categories ON products.category_id = categories.id 
    ORDER BY products.created_at DESC
  `);
  return rows;
};

exports.getProductById = async (id) => {
  const [rows] = await db.query(`
    SELECT products.*, categories.name AS category_name 
    FROM products 
    LEFT JOIN categories ON products.category_id = categories.id 
    WHERE products.id = ?
  `, [id]);
  return rows[0];
};

exports.createProduct = async (name, categoryId, price) => {
  const [result] = await db.query(
    'INSERT INTO products (name, category_id, price) VALUES (?, ?, ?)',
    [name, categoryId, price]
  );
  return result.insertId;
};

exports.updateProduct = async (id, name, categoryId, price) => {
  await db.query(
    'UPDATE products SET name = ?, category_id = ?, price = ? WHERE id = ?',
    [name, categoryId, price, id]
  );
};

exports.deleteProduct = async (id) => {
  await db.query('DELETE FROM products WHERE id = ?', [id]);
};
```

---

### **4. Controller**

#### **File `app/controllers/adminController.js`**
```javascript
const AdminModel = require('../models/adminModel');
const jwt = require('jsonwebtoken');
const dotenv = require('dotenv');

dotenv.config();

exports.login = async (req, res) => {
  const { username, password } = req.body;

  try {
    const admin = await AdminModel.findAdminByUsername(username);

    if (!admin || admin.password !== password) {
      return res.status(401).json({ message: 'Invalid username or password' });
    }

    const token = jwt.sign({ id: admin.id, username: admin.username, role: 'admin' }, process.env.JWT_SECRET, {
      expiresIn: '1h',
    });

    res.json({ message: 'Login successful', token });
  } catch (err) {
    res.status(500).json({ message: 'An error occurred during login', error: err.message });
  }
};

exports.getDashboard = (req, res) => {
  res.json({ message: 'Welcome to the admin dashboard!', user: req.user });
};

```

---

#### **File `app/controllers/categoryController.js`**
```javascript
const CategoryModel = require('../models/categoryModel');

exports.getCategories = async (req, res) => {
  const categories = await CategoryModel.getAllCategories();
  res.json(categories);
};

exports.getCategory = async (req, res) => {
  const { id } = req.params;
  const category = await CategoryModel.getCategoryById(id);

  if (category) {
    res.json(category);
  } else {
    res.status(404).json({ message: 'Category not found' });
  }
};

exports.createCategory = async (req, res) => {
  const { name } = req.body;
  const id = await CategoryModel.createCategory(name);
  res.status(201).json({ message: 'Category created successfully', id });
};

exports.updateCategory = async (req, res) => {
  const { id } = req.params;
  const { name } = req.body;

  await CategoryModel.updateCategory(id, name);
  res.json({ message: 'Category updated successfully' });
};

exports.deleteCategory = async (req, res) => {
  const { id } = req.params;

  await CategoryModel.deleteCategory(id);
  res.json({ message: 'Category deleted successfully' });
};
```

---

#### **File `app/controllers/productController.js`**
```javascript
const ProductModel = require('../models/productModel');

exports.getProducts = async (req, res) => {
  const products = await ProductModel.getAllProducts();
  res.json(products);
};

exports.getProduct = async (req, res) => {
  const { id } = req.params;
  const product = await ProductModel.getProductById(id);

  if (product) {
    res.json(product);
  } else {
    res.status(404).json({ message: 'Product not found' });
  }
};

exports.createProduct = async (req, res) => {
  const { name, categoryId, price } = req.body;
  const id = await ProductModel.createProduct(name, categoryId, price);
  res.status(201).json({ message: 'Product created successfully', id });
};

exports.updateProduct = async (req, res) => {
  const { id } = req.params;
  const { name, categoryId, price } = req.body;

  await ProductModel.updateProduct(id, name, categoryId, price);
  res.json({ message: 'Product updated successfully' });
};

exports.deleteProduct = async (req, res) => {
  const { id } = req.params;

  await ProductModel.deleteProduct(id);
  res.json({ message: 'Product deleted successfully' });
};
```

---

### **5. Middleware**

#### **File `app/middleware/adminMiddleware.js`**

```javascript
const jwt = require('jsonwebtoken');
const dotenv = require('dotenv');

dotenv.config();

exports.verifyAdmin = (req, res, next) => {
  const token = req.headers['authorization']?.split(' ')[1]; // Mengambil token dari header Authorization

  if (!token) {
    return res.status(401).json({ message: 'Access denied. No token provided.' });
  }

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET); // Verifikasi token
    if (decoded.role !== 'admin') {
      return res.status(403).json({ message: 'Access denied. Not an admin.' });
    }

    req.user = decoded; 
    next(); 
  } catch (err) {
    res.status(400).json({ message: 'Invalid token.' });
  }
};


```

### **6. Routes**

#### **File `app/routes/adminRoutes.js`**
```javascript
const express = require('express');
const adminController = require('../controllers/adminController');
const { verifyAdmin } = require('../middleware/adminMiddleware');

const router = express.Router();

router.post('/login', adminController.login); 


router.use(verifyAdmin);

router.get('/dashboard', adminController.getDashboard); 

module.exports = router;

```

---

#### **File `app/routes/categoryRoutes.js`**
```javascript
const express = require('express');
const { getCategories, getCategory, createCategory, updateCategory, deleteCategory } = require('../controllers/categoryController');
const { verifyAdmin } = require('../middleware/adminMiddleware');

const router = express.Router();

router.get('/', getCategories);
router.get('/:id', getCategory);
router.post('/create', verifyAdmin, createCategory);
router.put('/edit/:id', verifyAdmin, updateCategory);
router.delete('/delete/:id', verifyAdmin, deleteCategory);

module.exports = router;
```

---

#### **File `app/routes/productRoutes.js`**
```javascript
const express = require('express');
const { getProducts, getProduct, createProduct, updateProduct, deleteProduct } = require('../controllers/productController');
const { verifyAdmin } = require('../middleware/adminMiddleware');

const router = express.Router();

router.get('/', getProducts);
router.get('/:id', getProduct);
router.post('/create', verifyAdmin, createProduct);
router.put('/edit/:id', verifyAdmin, updateProduct);
router.delete('/delete/:id', verifyAdmin, deleteProduct);

module.exports = router;
```

---

#### **File `app/routes/main.js`**
```javascript
const express = require('express');
const adminRoutes = require('./adminRoutes');
const categoryRoutes = require('./categoryRoutes');
const productRoutes = require('./productRoutes');

const router = express.Router();

router.use('/admin', adminRoutes);
router.use('/categories', categoryRoutes);
router.use('/products', productRoutes);

router.get('/', (req, res) => {
  res.send('Welcome to the Express MVC app!');
});

module.exports = router;
```

---


### **7. File Migrasi**

#### **File `migrations/migrate-up.js`**
```javascript
const db = require('../config/dbconnect');

const migrateUp = async () => {
  try {
    await db.query(`
      CREATE TABLE IF NOT EXISTS admin (
        id INT AUTO_INCREMENT PRIMARY KEY,
        username VARCHAR(255) NOT NULL UNIQUE,
        password VARCHAR(255) NOT NULL
      )
    `);

    await db.query(`
      CREATE TABLE IF NOT EXISTS categories (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL UNIQUE,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);

    await db.query(`
      CREATE TABLE IF NOT EXISTS products (
        id INT AUTO_INCREMENT PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        category_id INT,
        price DECIMAL(10, 2) NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE SET NULL
      )
    `);

    await db.query(`
      INSERT INTO admin (username, password)
      VALUES ('admin', 'password')
    `);

    console.log('Migration up completed successfully!');
  } catch (err) {
    console.error('Error during migration up:', err);
  } finally {
    db.end();
  }
};

migrateUp();
```

---

#### **File `migrations/migrate-down.js`**
```javascript
const db = require('../config/dbconnect');

const migrateDown = async () => {
  try {
    await db.query('DROP TABLE IF EXISTS products');
    await db.query('DROP TABLE IF EXISTS categories');
    await db.query('DROP TABLE IF EXISTS admin');

    console.log('Migration down completed successfully!');
  } catch (err) {
    console.error('Error during migration down:', err);
  } finally {
    db.end();
  }
};

migrateDown();
```

---




























