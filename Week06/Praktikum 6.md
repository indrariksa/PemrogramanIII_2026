# 📘 Praktikum Pertemuan 6

## Frontend: Mengambil Data dari API (React + Fetch)

# Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa mampu:

* Memahami konsep komunikasi frontend dan backend
* Menggunakan Fetch API untuk mengambil data dari backend Golang
* Menampilkan data dalam bentuk tabel menggunakan React
* Mengelola state (loading, error, data)
* Menyusun struktur project frontend yang rapi dan scalable

---

# Konsep Dasar

Frontend (React) berkomunikasi dengan backend (Golang API) menggunakan HTTP request.

Jenis request:

* **GET** → mengambil data
* **POST** → menambah data
* **PUT** → update data
* **DELETE** → hapus data

Pada praktikum ini kita fokus ke:
**GET data mahasiswa dari API**

---

# Alur Data

```
React (Frontend)
      ↓ fetch()
API Golang (Backend)
      ↓
Database
      ↓
Response JSON
      ↓
React tampilkan ke UI
```

---

# STEP 1 — Buat Project React (Vite)

```bash
npm create vite@latest my-fe
```

Isi:

* Project name → `my-fe`
* Framework → React
* Variant → JavaScript

Tunggu instalasi hingga selesai, seharusnya akan otomatis muncul
`http://localhost:5173`

![image](images/step-1.png)

![image](images/step-1(2).png)

Jika tidak muncul otomatis, masuk ke folder:

```bash
cd my-fe
npm install
```

Jalankan:

```bash
npm run dev
```

Jika muncul `http://localhost:5173` → berhasil

---

# STEP 2 — Install Tailwind CSS

```bash
npm install -D tailwindcss@3 postcss autoprefixer
```

### Penjelasan:

* `-D` → install sebagai dev dependency
* `tailwindcss` → framework CSS utility
* `postcss` → untuk proses CSS
* `autoprefixer` → menambahkan prefix browser otomatis

---

```bash
npx tailwindcss init -p
```

### Penjelasan:

* Membuat:

  * `tailwind.config.js`
  * `postcss.config.js`

---

# STEP 3 — Konfigurasi Tailwind
Edit `tailwind.config.js`:
```js
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### Penjelasan:

* `content` → menentukan file mana yang akan diproses Tailwind
* Tailwind hanya generate CSS yang dipakai (efisien)

---

# STEP 4 — Setup CSS
Edit `src/index.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Penjelasan:

* `base` → reset CSS
* `components` → komponen bawaan
* `utilities` → class seperti `bg-blue-500`, dll

---
Pastikan di `main.jsx`:
```jsx
import './index.css'
```

### Penjelasan:

* Import CSS ke React agar Tailwind aktif

---

# STEP 5 — Test Tailwind
Edit `App.jsx`:
```jsx
import './App.css'

export default function App() {
  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center">
      <div className="bg-white p-10 rounded-2xl shadow-xl text-center">
        <h1 className="text-3xl font-bold text-blue-600">
          React + Tailwind 🚀
        </h1>
      </div>
    </div>
  );
}
```
Jalankan Lagi

```bash
npm run dev
```

Sekarang Tailwind sudah aktif.
![image](images/step-5.png)
### Penjelasan:

* `className` → pengganti `class` di React
* `min-h-screen` → tinggi full layar
* `flex items-center justify-center` → center content
* `shadow-xl` → efek bayangan

---

# STEP 6 — Buat Service API

File:

```
src/services/api.js
```

```js
export async function getMahasiswa() {
  const res = await fetch("http://127.0.0.1:3000/api/mahasiswa");
  const payload = await res.json();

  if (!res.ok) {
    throw new Error(payload?.message || "Gagal mengambil data");
  }

  const mahasiswa = Array.isArray(payload) ? payload : payload?.data;

  if (!Array.isArray(mahasiswa)) {
    throw new Error("Format response tidak valid");
  }

  return mahasiswa;
}
```

### Penjelasan detail:

* `export async function getMahasiswa()`
  → membuat fungsi async agar bisa pakai `await`

* `fetch(...)`
  → request ke backend Golang

* `await res.json()`
  → convert response ke JSON

* `res.ok`
  → cek status HTTP (200–299)

* `throw new Error(...)`
  → kirim error ke React jika gagal

* `Array.isArray(...)`
  → memastikan data berbentuk array

Kenapa penting?
Supaya UI tidak langsung bergantung ke API (clean architecture)

---

# STEP 7 — Buat Halaman Mahasiswa

File:

```
src/pages/MahasiswaPages.jsx
```
Kode lengkap:
```jsx
import { useEffect, useState } from "react";
import { getMahasiswa } from "../services/api";

export default function Mahasiswa() {
  const [mahasiswa, setMahasiswa] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    getMahasiswa()
    .then(setMahasiswa)
    .catch((err) => setError(err.message))
    .finally(() => setLoading(false));
  }, []);

  if (loading) return <p className="text-center">Loading...</p>;

  if (error) return <p className="text-center text-red-500">Error: {error}</p>;

  return (
    <div className="max-w-8xl mx-auto p-6">
      <h2 className="text-xl font-bold mb-4">Daftar Mahasiswa</h2>
      
      <div className="overflow-hidden border rounded-lg">
        <table className="w-full text-sm text-left">
          <thead className="bg-gray-300 border-b text-gray-700 uppercase text-xs">
            <tr>
              <th className="px-4 py-3 border">Nama / Prodi</th>
              <th className="px-4 py-3 border">Email</th>
              <th className="px-4 py-3 border">Alamat</th>
            </tr>
          </thead>
          <tbody className="divide-y">
            {mahasiswa.map((mhs) => (
              <tr key={mhs.npm} className="hover:bg-blue-50">
                <td className="px-4 py-3 border">
                  <div className="font-medium">{mhs.nama}</div>
                  <div className="text-gray-500 text-xs">{mhs.prodi}</div>
                </td>
                <td className="px-4 py-3 text-gray-600 border">{mhs.email}</td>
                <td className="px-4 py-3 text-gray-500 border">{mhs.alamat}</td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```
---
### Penjelasan:

```jsx
import { useEffect, useState } from "react";
import { getMahasiswa } from "../services/api";
```

* `useState` → menyimpan data
* `useEffect` → menjalankan kode saat pertama render

---
### Penjelasan:
```jsx
const [mahasiswa, setMahasiswa] = useState([]);
const [loading, setLoading] = useState(true);
const [error, setError] = useState("");
```

* `mahasiswa` → data dari API
* `loading` → status loading
* `error` → pesan error

---
### Penjelasan:
```jsx
useEffect(() => {
    getMahasiswa()
    .then(setMahasiswa)
    .catch((err) => setError(err.message))
    .finally(() => setLoading(false));
  }, []);
```

* `[]` → hanya jalan sekali saat pertama render
* `.then()` → jika berhasil
* `.catch()` → jika error
* `.finally()` → selalu dijalankan

---
### Penjelasan:
```jsx
if (loading)
  return <p className="text-center">Loading...</p>;
```
* Tampilkan loading saat request berjalan

---
### Penjelasan:
```jsx
if (error)
  return <p className="text-center text-red-500">Error: {error}</p>;
```
* Tampilkan error jika gagal

---
### Penjelasan:
```jsx
{mahasiswa.map((mhs, index) =>  (
```

* `map()` → looping array
* `mhs` → data mahasiswa
* `index` → index

---
### Penjelasan:
```jsx
<td>{i + 1}</td>
```
* Nomor urut (mulai dari 1)

---

# STEP 8 — Panggil di App.jsx

```jsx
import MahasiswaPages from "./pages/MahasiswaPages";

export default function App() {
  return (
    <div className="min-h-screen bg-gray-100 p-10">
      <MahasiswaPages />
    </div>
  );
}
```

### Penjelasan:

* Import komponen
* Render `<MahasiswaPages />` ke halaman utama

---

# STEP 9 — Jalankan

```bash
npm run dev
```

---

# ⚠️ Penting: CORS Error

Jika muncul:

```
Access to fetch has been blocked by CORS policy
```
![image](images/cors-error.png)

### Penjelasan:

Browser memblokir request karena beda origin

---

### Solusi di Golang (Fiber):
Buat file `cors.go` di dalam folder `config` kemudian isi kodenya
```go
package config

var allowedOrigins = []string{
	"http://localhost:5173",
}

func GetAllowedOrigins() []string {
	return allowedOrigins
}
```

Kemudian pada file `main.go` tambahkan kode berikut
```go
import "github.com/gofiber/fiber/v2/middleware/cors"

// Basic CORS
app.Use(cors.New(cors.Config{
	AllowOrigins:     strings.Join(config.GetAllowedOrigins(), ","),
	AllowMethods:     "GET,POST,PUT,DELETE,OPTIONS",
}))
```
Coba cek lagi sehingga hasilnya seperti berikut
![image](images/hasil.png)

---

# Latihan Mandiri

1. Tambahkan kolom **NPM** pada tabel

2. Tampilkan **jumlah total mahasiswa** di atas tabel
  
    Contoh:
    ```
    Total Mahasiswa: 2
    ```
---

# Penutup

Pada praktikum ini kita sudah:

* Menghubungkan React dengan Golang API
* Mengambil data dari backend
* Menampilkan data ke UI

## Pengumpulan
- Push ke direktori Pertemuan06/Praktikum
- Screenshoot dari frontend ketika tabel berhasil ditampilkan (Push ke direktori Pertemuan06/Praktikum/Hasil)

---


# TUGAS DI RUMAH

1. Tambahkan fitur **Search (pencarian)**

    - Pencarian harus bisa mencari berdasarkan:
      - Nama
      - Prodi
      - Email
      - Alamat
      - NPM

    Hint:
    - Gunakan `useState` untuk menyimpan keyword
    - Gunakan `.filter()` untuk menyaring data

---

2. Tambahkan tombol **Refresh Data**

    - Ketika tombol diklik → data diambil ulang dari API

![image](images/tugas.png)
---
