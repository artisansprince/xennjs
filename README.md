# XennJS - Express.js Minimalist Design Pattern

XennJS adalah design pattern minimalis untuk pengembangan backend menggunakan **Express.js** dan **JavaScript/TypeScript**, yang dirancang khusus untuk memudahkan pembuatan **RESTful API** dengan arsitektur berbasis **MVC** (Model-View-Controller). Fokus utama XennJS adalah menyederhanakan pengembangan dengan struktur yang bersih, modular, dan scalable, sehingga cocok digunakan baik untuk proyek kecil maupun besar.

## Fitur Utama:
- **MVC Architecture**: Memisahkan logika bisnis, data, dan API handling agar lebih mudah dikelola dan di-maintain.
- **TypeScript Support**: Memberikan keamanan tipe dan meningkatkan produktivitas pengembangan dengan fitur TypeScript.
- **Highly Modular**: Setiap komponen seperti model, controller, dan service ditata rapi untuk mendukung pengembangan yang efisien dan scalable.
- **Minimal Setup**: Dibuat dengan filosofi minimalis, tidak membebani dengan konfigurasi berlebihan namun tetap fleksibel.

Cocok untuk kamu yang menginginkan **Express.js** sederhana namun tetap powerful, dengan pola coding yang konsisten dan terstruktur, dan itu didapatkan dalam versi yang ringan dan cepat.

---


# Xennjs Quick Breakdown
Pembahasan sekilas mengenai Xennjs saat mengembangkan **RESTful API** dengan **TypeScript, MariaDb, dan TypeORM**

## **Struktur Folder Lengkap Xennjs**

```plaintext
project-name/
│
├── src/
│   ├── app/
│   │   ├── controllers/
│   │   │   └── UserController.ts
│   │   ├── middlewares/
│   │   │   └── authMiddleware.ts
│   │   ├── models/
│   │   │   └── User.ts
│   │   ├── services/
│   │   │   └── UserService.ts
│   │   └── migrations/
│   │       └── 2024-create-users-table.ts
│   ├── config/
│   │   └── database.ts
│   ├── routes/
│   │   └── api.ts
│   ├── app.ts
│   └── server.ts
├── package.json
├── tsconfig.json
├── .env
└── README.md
```

---

## **Penjelasan File dan Kode Lengkap**

### 1. **Model (app/models/User.ts)**

Model ini mendefinisikan struktur data **User** yang mewakili tabel di database.

```typescript
import { RowDataPacket } from 'mysql2';

export interface User extends RowDataPacket {
  id: number;
  name: string;
  email: string;
  created_at: Date;
  updated_at: Date;
}
```

Dalam hal ini, kita menggunakan `RowDataPacket` dari `mysql2` untuk mengimpor tipe data agar cocok dengan hasil query dari database.

---

### 2. **Service (app/services/UserService.ts)**

Service ini mengelola logika bisnis untuk interaksi dengan database.

```typescript
import { User } from '../models/User';
import { db } from '../config/database';

// Mendapatkan semua pengguna dari database
export const findAllUsers = async (): Promise<User[]> => {
  const [rows] = await db.query<User[]>('SELECT * FROM users');
  return rows;
};

// Menambahkan pengguna baru ke database
export const createUser = async (data: Omit<User, 'id' | 'created_at' | 'updated_at'>): Promise<User> => {
  const [result] = await db.query('INSERT INTO users (name, email) VALUES (?, ?)', [data.name, data.email]);
  const newUser: User = { id: result.insertId, ...data, created_at: new Date(), updated_at: new Date() };
  return newUser;
};
```

Ini mengimpor `db` dari file konfigurasi database dan menggunakan query SQL untuk mengelola data.

---

### 3. **Migrasi (app/migrations/2024-create-users-table.ts)**

File migrasi ini digunakan untuk membuat tabel **users** di database.

```typescript
import { db } from '../config/database';

export const createUsersTable = async () => {
  const query = `
    CREATE TABLE IF NOT EXISTS users (
      id INT AUTO_INCREMENT PRIMARY KEY,
      name VARCHAR(100) NOT NULL,
      email VARCHAR(100) NOT NULL UNIQUE,
      created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
      updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    )
  `;
  await db.query(query);
};
```

Fungsi ini digunakan untuk membuat tabel **users** dengan kolom **id**, **name**, **email**, **created_at**, dan **updated_at**.

---

### 4. **Konfigurasi Database (config/database.ts)**

File ini berisi koneksi ke database MariaDB menggunakan `mysql2`.

```typescript
import dotenv from 'dotenv';
import mysql from 'mysql2/promise';

dotenv.config();

export const db = mysql.createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  waitForConnections: true,
  connectionLimit: 10,
  queueLimit: 0,
});
```

Di sini, kita menggunakan **mysql2/promise** untuk mengelola koneksi database MariaDB dengan pool connection. Jangan lupa tambahkan variabel environment ke dalam file `.env`.

---

### 5. **Environment Variables (.env)**

Pastikan kamu menyimpan detail koneksi database di file `.env`.

```plaintext
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=rootpassword
DB_NAME=mydatabase
PORT=3000
```

---

### 6. **App Setup (app.ts)**

File ini menghubungkan semua bagian aplikasi seperti middleware, route, dan koneksi database.

```typescript
import express, { Application } from 'express';
import apiRoutes from './routes/api';
import { createUsersTable } from './app/migrations/2024-create-users-table';

const app: Application = express();

// Middleware
app.use(express.json());

// Migrasi database
createUsersTable().then(() => console.log('Users table created'));

// API Routes
app.use('/api', apiRoutes);

export default app;
```

Selain menyiapkan routing API, di sini kita juga menjalankan migrasi untuk membuat tabel **users** jika belum ada.

---

### 7. **Routing (routes/api.ts)**

Di sini semua rute API diatur. Misalnya, kita punya rute untuk mendapatkan daftar pengguna dan menambahkan pengguna baru.

```typescript
import { Router } from 'express';
import { getUsers, addUser } from '../app/controllers/UserController';

const router: Router = Router();

router.get('/users', getUsers);
router.post('/users', addUser);

export default router;
```

---

### 8. **Controller (app/controllers/UserController.ts)**

Controller ini bertugas menangani request dan response. Ini menghubungkan request dari user ke service yang berisi logika bisnis.

```typescript
import { Request, Response } from 'express';
import { findAllUsers, createUser } from '../services/UserService';

export const getUsers = async (req: Request, res: Response): Promise<void> => {
  try {
    const users = await findAllUsers();
    res.status(200).json(users);
  } catch (error) {
    res.status(500).json({ message: 'Error retrieving users' });
  }
};

export const addUser = async (req: Request, res: Response): Promise<void> => {
  try {
    const newUser = await createUser(req.body);
    res.status(201).json(newUser);
  } catch (error) {
    res.status(500).json({ message: 'Error creating user' });
  }
};
```

---

### 9. **Middleware (app/middlewares/authMiddleware.ts)**

Ini middleware untuk validasi autentikasi. Bisa ditambahkan di rute yang perlu validasi token.

```typescript
import { Request, Response, NextFunction } from 'express';

export const authMiddleware = (req: Request, res: Response, next: NextFunction): void => {
  const token = req.headers['authorization'];
  if (token === 'Bearer valid-token') {
    next(); // Token valid, lanjutkan ke request berikutnya
  } else {
    res.status(401).json({ message: 'Unauthorized' });
  }
};
```

---

### 10. **Memulai Server (server.ts)**

File ini untuk memulai server.

```typescript
import app from './app';
const PORT = process.env.PORT || 9999;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

### 11. **Install Dependencies**

Install semua dependensi yang diperlukan:

```bash
npm install express dotenv mysql2
npm install -D typescript ts-node-dev @types/node @types/express @types/mysql2
```

---

### 12. **Menjalankan Migrasi dan Server**

1. Jalankan server dalam mode development menggunakan `ts-node-dev`:

```bash
npx ts-node-dev src/server.ts
```

2. Untuk compile TypeScript ke JavaScript:

```bash
npx tsc
```

3. Jalankan hasil compile:

```bash
node dist/server.js
```

---

