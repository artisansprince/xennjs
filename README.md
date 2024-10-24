# XennJS - Express.js Minimalist Design Pattern

XennJS adalah design pattern minimalis untuk pengembangan backend menggunakan **Express.js** dan **JavaScript/TypeScript**. XennJS dirancang untuk memudahkan pengembangan **RESTful API** menggunakan arsitektur **MVC** (Model-View-Controller), yang membuat aplikasi menjadi modular, scalable, dan mudah di-maintain. Pada versi ini, XennJS menggunakan **Sequelize** sebagai ORM untuk mengelola interaksi database.

## Fitur Utama:
- **MVC Architecture**: Memisahkan logika bisnis, data, dan API handling agar lebih mudah dikelola dan di-maintain.
- **TypeScript Support**: Memberikan keamanan tipe dan meningkatkan produktivitas pengembangan dengan fitur TypeScript.
- **Sequelize ORM**: Abstraksi database yang lebih mudah dikelola dan mendukung berbagai database relasional (MySQL, PostgreSQL, SQLite, dll.).
- **Highly Modular**: Setiap komponen seperti model, controller, dan service ditata rapi untuk mendukung pengembangan yang efisien dan scalable.
- **Minimal Setup**: Konfigurasi minimalis dengan filosofi sederhana namun tetap fleksibel.

---

## Struktur Folder Lengkap XennJS

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
│   │       └── 240101-create-users-table.ts
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

## Setup XennJS

### 1. **Instalasi Dependency**

Pertama, kamu perlu menginisialisasi proyek Node.js dan menginstall beberapa dependencies dasar. Berikut cara melakukannya:

1. **Inisialisasi Proyek**:

   Buka terminal dan jalankan perintah berikut untuk menginisialisasi proyek:

   ```bash
   npm init -y
   ```

2. **Instalasi Dependency**:

   Install dependency yang dibutuhkan seperti **Express.js**, **Sequelize**, dan **TypeScript**:

   ```bash
   npm install express sequelize mysql2 dotenv
   ```

3. **Instalasi Dependency untuk Development** (TypeScript, nodemon, dll):

   Untuk memastikan proyek kamu bisa menggunakan TypeScript dan dijalankan otomatis saat development, install juga **TypeScript** dan beberapa tool seperti **nodemon** untuk auto-restart server saat file diubah:

   ```bash
   npm install --save-dev typescript @types/node @types/express nodemon ts-node
   ```

---

### 2. **Membuat File `package.json`**

File ini akan berisi script yang bisa kamu gunakan untuk menjalankan aplikasi di mode **development** dan **production**. Jika belum terbuat, tambahkan di dalam file `package.json` seperti ini:

```json
{
  "name": "xennjs",
  "version": "1.0.0",
  "main": "dist/server.js",
  "scripts": {
    "start": "node dist/server.js",
    "dev": "nodemon --watch 'src/**/*.ts' --exec 'ts-node' src/server.ts",
    "build": "tsc"
  },
  "dependencies": {
    "express": "^4.18.2",
    "sequelize": "^6.25.3",
    "mysql2": "^3.2.0",
    "dotenv": "^16.0.3"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^20.0.0",
    "@types/express": "^4.17.0",
    "nodemon": "^3.0.0",
    "ts-node": "^10.0.0"
  }
}
```

- **Script penting**:
  - `start`: untuk menjalankan server di mode **production** (setelah build).
  - `dev`: untuk menjalankan server di mode **development** dengan **nodemon** (auto-restart saat file berubah).
  - `build`: untuk compile TypeScript menjadi JavaScript.

---

### 3. **Membuat File `tsconfig.json`**

File ini digunakan untuk mengatur konfigurasi TypeScript, seperti lokasi input-output file dan rule untuk kompilasi.

Buat file `tsconfig.json` di root proyek:

```json
{
  "compilerOptions": {
    "target": "ES6",
    "module": "commonjs",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

- `rootDir`: Direktori sumber kode TypeScript (`src`).
- `outDir`: Direktori hasil build dalam JavaScript (`dist`).

---

### 4. **Setup Folder dan File**

Buat struktur folder seperti berikut:

```
/xennjs
│
├── /src
│   └── ... # isi folder src seperti struktur folder diatas
|
├── tsconfig.json
├── package.json
├── .env
└── node_modules
```

Isi masing-masing file seperti yang sudah dijelaskan sebelumnya.

---

### 5. **File .env**

Tambahkan file `.env` di root untuk menyimpan variabel environment seperti konfigurasi database:

```env
# App port
PORT=3000

# Database
DB_DIALECT=mysql
DB_HOST=localhost
DATABASE_NAME=your_database
DATABASE_USERNAME=root
DATABASE_PASSWORD=
```

---

## **Penjelasan File dan Kode Lengkap**

### 1. **Model (app/models/User.ts)**

Model ini mendefinisikan struktur data **User** yang mewakili tabel di database. Menggunakan **Sequelize** untuk mendefinisikan model User yang di-mapping ke tabel **users**.

```typescript
import { DataTypes, Model } from 'sequelize';
import { sequelize } from '../config/database';

class User extends Model {
  public id!: number;
  public name!: string;
  public email!: string;
  public readonly createdAt!: Date;
  public readonly updatedAt!: Date;
}

User.init(
  {
    id: {
      type: DataTypes.INTEGER.UNSIGNED,
      autoIncrement: true,
      primaryKey: true,
    },
    name: {
      type: DataTypes.STRING(100),
      allowNull: false,
    },
    email: {
      type: DataTypes.STRING(100),
      allowNull: false,
      unique: true,
    },
    createdAt: {
      type: DataTypes.DATE,
      defaultValue: DataTypes.NOW,
    },
    updatedAt: {
      type: DataTypes.DATE,
      defaultValue: DataTypes.NOW,
    },
  },
  {
    sequelize,
    tableName: 'users',
    timestamps: true, // Untuk createdAt dan updatedAt
  }
);

export default User;
```

---

### 2. **Service (app/services/UserService.ts)**

Service ini bertugas untuk mengelola logika bisnis terkait operasi CRUD pada tabel **users** menggunakan Sequelize.

```typescript
import User from '../models/User';

// Mendapatkan semua pengguna dari database
export const findAllUsers = async (): Promise<User[]> => {
  return await User.findAll();
};

// Menemukan pengguna berdasarkan ID
export const findUserById = async (id: number): Promise<User | null> => {
  return await User.findByPk(id);
};

// Menambahkan pengguna baru ke database
export const createUser = async (data: Omit<User, 'id' | 'createdAt' | 'updatedAt'>): Promise<User> => {
  return await User.create(data);
};

// Memperbarui pengguna berdasarkan ID
export const updateUser = async (id: number, data: Partial<User>): Promise<[number, User[]]> => {
  return await User.update(data, { where: { id } });
};

// Menghapus pengguna berdasarkan ID
export const deleteUser = async (id: number): Promise<number> => {
  return await User.destroy({ where: { id } });
};
```

Pada service ini, kita menggunakan fungsi-fungsi dari Sequelize untuk mempermudah interaksi dengan database, seperti `findAll()`, `findByPk()`, `create()`, `update()`, dan `destroy()`.

---

### 3. **Migrasi (app/migrations/240101-create-users-table.ts)**

File migrasi digunakan untuk membuat tabel **users** di database.

```typescript
import { QueryInterface, DataTypes } from 'sequelize';

export const up = async (queryInterface: QueryInterface): Promise<void> => {
  await queryInterface.createTable('users', {
    id: {
      type: DataTypes.INTEGER.UNSIGNED,
      autoIncrement: true,
      primaryKey: true,
    },
    name: {
      type: DataTypes.STRING(100),
      allowNull: false,
    },
    email: {
      type: DataTypes.STRING(100),
      allowNull: false,
      unique: true,
    },
    createdAt: {
      type: DataTypes.DATE,
      defaultValue: DataTypes.NOW,
    },
    updatedAt: {
      type: DataTypes.DATE,
      defaultValue: DataTypes.NOW,
    },
  });
};

export const down = async (queryInterface: QueryInterface): Promise<void> => {
  await queryInterface.dropTable('users');
};
```

Migrasi ini akan membuat tabel **users** dengan kolom **id**, **name**, **email**, **createdAt**, dan **updatedAt**.

---

### 4. **Controller (app/controllers/UserController.ts)**

Controller ini bertanggung jawab untuk menangani request dan response, serta menghubungkan request dari user ke service yang berisi logika bisnis.

```typescript
import { Request, Response } from 'express';
import { findAllUsers, findUserById, createUser, updateUser, deleteUser } from '../services/UserService';

// Mendapatkan semua pengguna
export const getUsers = async (req: Request, res: Response): Promise<void> => {
  try {
    const users = await findAllUsers();
    res.status(200).json(users);
  } catch (error) {
    res.status(500).json({ message: 'Error retrieving users' });
  }
};

// Mendapatkan pengguna berdasarkan ID
export const getUserById = async (req: Request, res: Response): Promise<void> => {
  try {
    const user = await findUserById(+req.params.id);
    if (user) {
      res.status(200).json(user);
    } else {
      res.status(404).json({ message: 'User not found' });
    }
  } catch (error) {
    res.status(500).json({ message: 'Error retrieving user' });
  }
};

// Membuat pengguna baru
export const addUser = async (req: Request, res: Response): Promise<void> => {
  try {
    const newUser = await createUser(req.body);
    res.status(201).json(newUser);
  } catch (error) {
    res.status(500).json({ message: 'Error creating user' });
  }
};

// Memperbarui pengguna berdasarkan ID
export const updateUserById = async (req: Request, res: Response): Promise<void> => {
  try {
    const [updated] = await updateUser(+req.params.id, req.body);
    if (updated) {
      const updatedUser = await findUserById(+req.params.id);
      res.status(200).json(updatedUser);
    } else {
      res.status(404).json({ message: 'User not found' });
    }
  } catch (error) {
    res.status(500).json({ message: 'Error updating user' });
  }
};

// Menghapus pengguna berdasarkan ID
export const deleteUserById = async (req: Request, res: Response): Promise<void> => {
  try {
    const deleted = await deleteUser(+req.params.id);
    if (deleted) {
      res.status(200).json({ message: 'User deleted' });
    } else {
      res.status(404).json({ message: 'User not found' });
    }
  } catch (error) {
    res.status(500).json({ message: 'Error deleting user' });
  }
};
```

Controller ini menghubungkan request HTTP seperti **GET**, **POST**, **PUT**, dan **DELETE** dengan fungsi CRUD yang ada di service.

---

### 5. **Konfigurasi Database (config/database.ts)**

File konfigurasi ini digunakan untuk mengatur koneksi ke database menggunakan Sequelize.

```typescript
import { Sequelize } from 'sequelize';
require('dotenv').config(); // Memuat variabel dari .env

// Masukkan variabel env ke dalam variabel JavaScript
const dbName = process.env.DATABASE_NAME;
const dbUser = process.env.DATABASE_USERNAME;
const dbPassword = process.env.DATABASE_PASSWORD || ''; // Jika password kosong di .env, tetap gunakan string kosong
const dbHost = process.env.DB_HOST || 'localhost'; // Default ke 'localhost' jika DB_HOST tidak ada di .env
const dbDialect = process.env.DB_DIALECT || 'mysql'; // Default ke 'mysql' jika DB_DIALECT tidak ada di .env

// Inisialisasi Sequelize dengan variabel
export const sequelize = new Sequelize(dbName, dbUser, dbPassword, {
  host: dbHost,
  dialect: dbDialect,
  logging: false,
});

```

---

### 6. **Routing (routes/api.ts)**

Routing untuk menghubungkan endpoint HTTP ke controller.

```typescript
import { Router } from 'express';
import { getUsers, getUserById, addUser, updateUserById, deleteUserById } from '../app/controllers/UserController';

const router = Router();

router.get('/users', getUsers);
router.get('/users/:id', getUserById);
router.post('/users', addUser);
router.put('/users/:id', updateUserById);
router.delete('/users/:id', deleteUserById);

export default router;
```

---

### 7. **Entry Point (app.ts)**

File ini menginisialisasi aplikasi Express.js dan menghubungkan ke routing.

```typescript
import express from 'express';
import apiRoutes from './routes/api';
import { sequelize } from './config/database';

const app = express();

app.use(express.json()); // Untuk membaca body JSON
app.use('/api', apiRoutes);

// Sinkronisasi model ke database
sequelize.sync().then(() => {
  console.log('Database connected and models synced.');
});

export default app;

```


---

### 8. **Server (server.ts)**

File ini menginisialisasi aplikasi Express.js dan menghubungkan ke routing.

```typescript
import app from './app';

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

---

### 9. **.env File**

File **.env** untuk menyimpan konfigurasi environment, seperti port dan database credentials.

```plaintext
# App port
PORT=3000

# Database
DB_DIALECT=mysql
DB_HOST=localhost
DATABASE_NAME=your_database
DATABASE_USERNAME=root
DATABASE_PASSWORD=
```

---

Dengan struktur dan penjelasan di atas, yang merupakan contoh XennJS **CRUD** pada **user** menggunakan **Sequelize ORM**. Setiap komponen memiliki tanggung jawab yang jelas, memudahkan pengembangan aplikasi scalable.

---

## Menjalankan XennJS


### **Mode Development**:

Untuk mode development, cukup jalankan perintah ini. Ini akan menggunakan **nodemon** dan otomatis merestart server jika ada perubahan pada file.

```bash
npm run dev
```

### **Build untuk Production**:

Kompilasi TypeScript ke JavaScript dengan menjalankan:

```bash
npm run build
```

Hasil build akan ada di folder `dist/`. Setelah build selesai, jalankan server di mode **production**:

```bash
npm start
```

---

Sekarang XennJS sudah berjalan baik di mode **development** maupun **production**.


