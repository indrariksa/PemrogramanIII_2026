# 📘 Praktikum Pertemuan 7

## Frontend: Axios + Dashboard Layout + Sidebar Responsive (React + Tailwind)

# Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa mampu:

* Menggunakan **Axios** untuk mengambil data dari API (lebih ringkas dari `fetch`)
* Mengelola konfigurasi API dengan **environment variable** (`.env`)
* Menyusun layout aplikasi dengan komponen **Header, Sidebar, Footer**
* Membuat navigasi halaman sederhana (**Dashboard** dan **Mahasiswa**)
* Membuat tabel data mahasiswa dengan fitur:
  * **Search semua field**
  * **Refresh data**
* Menerapkan struktur komponen sederhana berbasis **atomic** (atoms, molecules, layout)

---

# Prasyarat

1. Project React dari praktikum pertemuan 6 sudah berjalan.
2. Backend API mahasiswa sudah aktif.
3. Tailwind sudah terpasang dan aktif.

---

# Konsep Dasar: Kenapa Axios?

Dengan `fetch`, kita biasanya perlu cek status response manual (`res.ok`) dan parsing JSON sendiri.

Dengan Axios:

* Parsing JSON otomatis
* Error handling lebih nyaman (`error.response?.data`)
* Bisa buat instance reusable (`baseURL`, `timeout`, `headers`)

Contoh singkat:

```js
const response = await api.get("/mahasiswa");
const data = response.data;
```

---

# Alur Aplikasi Pertemuan 7

```text
Sidebar (Dashboard / Mahasiswa)
      ↓ pilih menu
App state (activePage)
      ↓
Render halaman sesuai menu
      ├─ DashboardPage (data statis)
      └─ MahasiswaPage (Axios GET + tabel + search + refresh)
```

---

# STEP 1 — Install Axios

```bash
npm install axios
```

---

# STEP 2 — Konfigurasi Environment Variable

Buat file `.env`:

```env
VITE_API_BASE_URL=http://127.0.0.1:3000/api
VITE_API_TIMEOUT=10000
```

Buat juga `.env.example` (template untuk dibagikan ke repo):

```env
VITE_API_BASE_URL=http://127.0.0.1:8000/api
VITE_API_TIMEOUT=10000
```

Masukkan kode berikut ke file `.gitignore` (agar `.env` tidak ikut ter-push ke repository):

```bash
.env
```

> Catatan: setelah mengubah `.env`, **restart** `npm run dev`.

---

Penjelasan :

Pada step ini, kita mengatur **environment variable** yang digunakan untuk menyimpan konfigurasi penting aplikasi, seperti alamat API dan timeout request.

File `.env` digunakan untuk menyimpan nilai konfigurasi yang bisa berbeda-beda tergantung environment (development, staging, production). Pada kasus ini:

* `VITE_API_BASE_URL` berisi alamat backend API yang akan diakses oleh frontend
* `VITE_API_TIMEOUT` menentukan batas waktu maksimal request sebelum dianggap gagal

Penggunaan environment variable sangat penting karena:

* Menghindari hardcode URL di dalam kode
* Memudahkan perubahan konfigurasi tanpa mengubah source code
* Lebih aman saat bekerja dalam tim atau deployment

File `.env.example` dibuat sebagai template agar developer lain tahu variabel apa saja yang harus diisi tanpa harus membagikan file `.env` asli.

Kemudian, `.env` dimasukkan ke dalam `.gitignore` agar tidak ikut ter-upload ke repository. Ini penting karena file `.env` sering berisi konfigurasi sensitif.

Terakhir, setiap kali kita mengubah isi `.env`, kita harus melakukan restart pada server React (`npm run dev`) agar perubahan tersebut terbaca oleh aplikasi.

---

# STEP 3 — Buat Service Axios

File: `src/services/api.js`

```js
import axios from "axios";

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;
const API_TIMEOUT = Number(import.meta.env.VITE_API_TIMEOUT ?? 10000);

if (!API_BASE_URL) {
  throw new Error("VITE_API_BASE_URL belum diatur di file .env");
}

const api = axios.create({
  baseURL: API_BASE_URL,
  timeout: API_TIMEOUT,
  headers: {
    Accept: "application/json",
  },
});

export async function getMahasiswa() {
  try {
    const response = await api.get("/mahasiswa");
    const payload = response.data;
    const mahasiswa = Array.isArray(payload) ? payload : payload?.data;

    if (!Array.isArray(mahasiswa)) {
      throw new Error("Format response tidak valid");
    }

    return mahasiswa;
  } catch (error) {
    const message =
      error.response?.data?.message || error.message || "Gagal mengambil data";
    throw new Error(message);
  }
}
```

Penjelasan :

Pada step ini, kita membuat **service Axios** yang berfungsi sebagai penghubung antara frontend React dengan backend API.

Pertama, kita mengimpor library Axios untuk melakukan HTTP request. Kemudian kita mengambil konfigurasi dari file `.env`, yaitu `VITE_API_BASE_URL` sebagai alamat API dan `VITE_API_TIMEOUT` sebagai batas waktu request. Penggunaan `.env` ini penting agar URL API tidak ditulis langsung di kode (hardcode), sehingga lebih fleksibel jika nanti berpindah environment.

Selanjutnya dilakukan validasi: jika `API_BASE_URL` belum diatur, maka aplikasi akan menampilkan error. Ini bertujuan agar developer langsung tahu bahwa konfigurasi belum lengkap.

Setelah itu, kita membuat instance Axios menggunakan `axios.create()`. Dengan cara ini, kita tidak perlu menuliskan ulang `baseURL`, `timeout`, dan `headers` setiap kali melakukan request. Semua konfigurasi sudah terpusat di satu tempat.

Fungsi `getMahasiswa` digunakan untuk mengambil data dari endpoint `/mahasiswa`. Data yang diterima dari API bisa memiliki dua kemungkinan format: langsung array atau dibungkus dalam object (`{ data: [...] }`). Oleh karena itu, kita lakukan pengecekan agar tetap fleksibel terhadap berbagai format response backend.

Kemudian dilakukan validasi lagi untuk memastikan data yang diterima benar-benar berupa array. Jika tidak sesuai, maka akan dianggap error.

Terakhir, bagian `catch` digunakan untuk menangani error. Pesan error diambil dari response backend jika tersedia, atau menggunakan pesan default jika tidak ada. Dengan cara ini, aplikasi menjadi lebih robust dan user mendapatkan informasi error yang lebih jelas.

---

# STEP 4 — Struktur Komponen (Atomic Sederhana)

Contoh struktur:

```text
src/
 ├─ components/
 │   ├─ atoms/
 │   │   └─ SidebarNavButton.jsx
 │   ├─ molecules/
 │   │   ├─ SidebarBrandCard.jsx
 │   │   └─ SidebarInfoCard.jsx
 │   └─ layout/
 │       ├─ Header.jsx
 │       ├─ Sidebar.jsx
 │       └─ Footer.jsx
 └─ pages/
     ├─ DashboardPages.jsx
     └─ MahasiswaPages.jsx
```

---

# STEP 5 — Layout Utama (Header + Sidebar + Footer)

## `src/components/layout/Header.jsx`

```jsx id="l1z2ax"
export default function Header({ pageTitle, onToggleSidebar }) {
  return (
    <header className="border-b border-slate-200 bg-white/80 text-slate-800 backdrop-blur">
      <div className="flex w-full items-center justify-between px-4 py-4 md:px-6">
        <button
          type="button"
          onClick={onToggleSidebar}
          className="mr-3 rounded-lg border border-slate-300 bg-white px-2 py-1 text-xl leading-none shadow-sm md:hidden"
          aria-label="Buka menu"
        >
          ☰
        </button>

        <h1 className="text-lg font-semibold md:text-xl">Praktikum React</h1>

        <span className="rounded-full bg-gradient-to-r from-blue-100 to-indigo-100 px-3 py-1 text-xs font-medium text-blue-700 md:text-sm">
          {pageTitle}
        </span>
      </div>
    </header>
  );
}
```

---

## `src/components/layout/Sidebar.jsx`

```jsx id="o3k1zp"
import SidebarNavButton from "../atoms/SidebarNavButton";
import SidebarBrandCard from "../molecules/SidebarBrandCard";
import SidebarInfoCard from "../molecules/SidebarInfoCard";

const menuItems = [
  {
    id: "dashboard",
    label: "Dashboard",
    description: "Ringkasan data kelas",
    icon: "🏠",
  },
  {
    id: "mahasiswa",
    label: "Mahasiswa",
    description: "Daftar data mahasiswa",
    icon: "🎓",
  },
];

export default function Sidebar({
  activePage,
  isOpen,
  onClose,
  onSelectPage,
}) {
  return (
    <>
      {isOpen && (
        <button
          type="button"
          aria-label="Tutup menu"
          onClick={onClose}
          className="fixed inset-0 z-30 bg-slate-900/40 md:hidden"
        />
      )}

      <aside
        className={`fixed inset-y-0 left-0 z-40 flex h-full w-72 flex-col border-r border-white/10 bg-gradient-to-b from-slate-950 via-slate-900 to-blue-950 p-4 shadow-2xl transition-transform duration-300 md:translate-x-0 ${
          isOpen ? "translate-x-0" : "-translate-x-full"
        }`}
      >
        <div className="mb-4 flex items-center justify-between">
          <p className="text-xs font-semibold uppercase tracking-widest text-slate-300">
            Main Navigation
          </p>
          <button
            type="button"
            onClick={onClose}
            className="rounded-md bg-white/10 px-2 py-1 text-xs text-slate-200 md:hidden"
          >
            Tutup
          </button>
        </div>

        <SidebarBrandCard />

        <ul className="mt-5 space-y-3">
          {menuItems.map((item) => (
            <li key={item.id}>
              <SidebarNavButton
                icon={item.icon}
                label={item.label}
                description={item.description}
                isActive={activePage === item.id}
                onClick={() => onSelectPage(item.id)}
              />
            </li>
          ))}
        </ul>

        <div className="mt-auto">
          <SidebarInfoCard />
        </div>
      </aside>
    </>
  );
}
```

---

## `src/components/layout/Footer.jsx`

```jsx id="b2t4nm"
export default function Footer() {
  return (
    <footer className="mt-auto border-t border-slate-200 bg-white/80 backdrop-blur">
      <div className="w-full px-4 py-3 text-center text-sm text-slate-500 md:px-6">
        Praktikum Pemrograman III - React & Axios
      </div>
    </footer>
  );
}
```

---
## `src/components/molecules/SidebarBrandCard.jsx`

```jsx
export default function SidebarBrandCard() {
  return (
    <div className="rounded-2xl border border-white/10 bg-white/5 p-4">
      <div className="flex items-center gap-3">
        <div className="inline-flex h-10 w-10 items-center justify-center rounded-xl bg-gradient-to-br from-blue-400 to-indigo-500 text-lg font-bold text-white">
          P3
        </div>

        <div>
          <p className="text-sm font-semibold text-white">Pemrograman III - Web Service</p>
          <p className="text-xs text-slate-300">Praktikum React</p>
        </div>
      </div>
    </div>
  );
}
```

- Molecule kartu branding sidebar (identitas matakuliah).
---
## `src/components/molecules/SidebarInfoCard.jsx`

```jsx
export default function SidebarInfoCard() {
  return (
    <div className="rounded-2xl border border-blue-300/20 bg-gradient-to-br from-blue-500/20 to-indigo-500/20 p-4">
      <p className="text-xs font-semibold uppercase tracking-wide text-blue-200">
        Info Kelas
      </p>
      <p className="mt-2 text-sm font-medium text-white">
        Kelas D4 TI 2C
      </p>
    </div>
  );
}
```
- Molecule kartu informasi tambahan di bagian bawah sidebar.
---

## `src/components/atoms/SidebarNavButton.jsx`

```jsx
export default function SidebarNavButton({
  icon,
  label,
  description,
  isActive,
  onClick,
}) {
  return (
    <button
      type="button"
      onClick={onClick}
      className={`group w-full rounded-xl border px-3 py-3 text-left transition ${
        isActive
          ? "border-blue-400/60 bg-gradient-to-r from-blue-500 to-indigo-500 text-white shadow-lg shadow-blue-900/30"
          : "border-white/10 bg-white/5 text-slate-200 hover:border-blue-300/40 hover:bg-white/10"
      }`}
    >
      <div className="flex items-center gap-3">
        <span
          className={`inline-flex h-9 w-9 items-center justify-center rounded-lg text-lg ${
            isActive
              ? "bg-white/25 text-white"
              : "bg-slate-800/70 text-slate-100 group-hover:bg-slate-700"
          }`}
        >
          {icon}
        </span>

        <div className="min-w-0">
          <p className="truncate text-sm font-semibold">{label}</p>
          <p
            className={`truncate text-xs ${
              isActive ? "text-blue-100" : "text-slate-400"
            }`}
          >
            {description}
          </p>
        </div>
      </div>
    </button>
  );
}
```
- Atom tombol navigasi sidebar.
- Menerima props: `icon`, `label`, `description`, `isActive`, `onClick`.
- Menangani style active/inactive.
---

## `src/App.jsx`

```jsx id="z7l9qw"
import { useState } from "react";
import Header from "./components/layout/Header";
import Sidebar from "./components/layout/Sidebar";
import Footer from "./components/layout/Footer";
import MahasiswaPage from "./pages/MahasiswaPages";
import DashboardPage from "./pages/DashboardPages";

export default function App() {
  const [activePage, setActivePage] = useState("dashboard");
  const [isSidebarOpen, setIsSidebarOpen] = useState(false);

  const pageTitle = activePage === "dashboard" ? "Dashboard" : "Data Mahasiswa";

  return (
    <div className="min-h-screen bg-slate-100 text-slate-800">
      <Sidebar
        activePage={activePage}
        isOpen={isSidebarOpen}
        onClose={() => setIsSidebarOpen(false)}
        onSelectPage={(pageId) => {
          setActivePage(pageId);
          setIsSidebarOpen(false);
        }}
      />

      <div className="flex min-h-screen flex-col md:pl-72">
        <Header
          pageTitle={pageTitle}
          onToggleSidebar={() => setIsSidebarOpen((prev) => !prev)}
        />

        <main className="flex-1 p-3 sm:p-4 md:p-6">
          <div className="w-full rounded-2xl border border-slate-200 bg-white p-4 shadow-sm md:p-6">
            {activePage === "dashboard" ? <DashboardPage /> : <MahasiswaPage />}
          </div>
        </main>

        <Footer />
      </div>
    </div>
  );
}
```

---

Penjelasan :

Pada step ini, kita membangun **layout utama aplikasi** yang terdiri dari Header, Sidebar, dan Footer. Tujuannya adalah agar aplikasi memiliki struktur yang rapi, konsisten, dan mudah dikembangkan.

Komponen **Header** berfungsi sebagai bagian atas aplikasi yang menampilkan judul utama dan nama halaman aktif. Selain itu, terdapat tombol menu (ikon ☰) yang digunakan untuk membuka sidebar pada tampilan mobile. Ini penting agar aplikasi tetap responsif di berbagai ukuran layar.

Komponen **Sidebar** digunakan sebagai navigasi utama aplikasi. Di dalamnya terdapat menu seperti Dashboard dan Mahasiswa. Sidebar ini dibuat dinamis menggunakan array `menuItems`, sehingga mudah menambah menu baru tanpa mengubah banyak kode. Sidebar juga mendukung tampilan mobile dengan fitur slide (muncul dari kiri) serta backdrop (area gelap di belakang) untuk meningkatkan pengalaman pengguna.

Komponen **Footer** adalah bagian bawah aplikasi yang menampilkan informasi sederhana. Walaupun sederhana, komponen ini membantu menjaga konsistensi layout.

Pada file **App.jsx**, semua komponen layout digabungkan menjadi satu kesatuan. Di sini kita menggunakan state:

* `activePage` untuk menentukan halaman yang sedang ditampilkan
* `isSidebarOpen` untuk mengontrol sidebar pada mobile

Ketika user memilih menu di sidebar, nilai `activePage` akan berubah, dan React akan merender halaman yang sesuai (Dashboard atau Mahasiswa). Ini merupakan konsep penting dalam React, yaitu **rendering berdasarkan state**.

Selain itu, layout dibuat menggunakan Flexbox sehingga:

* Sidebar berada di kiri
* Konten utama di kanan
* Footer tetap di bawah

Dengan struktur ini, aplikasi menjadi:

* Lebih terorganisir
* Mudah dikembangkan
* Siap untuk ditambahkan halaman baru di masa depan


---

# STEP 6 — Halaman Dashboard (Statis)

File: `src/pages/DashboardPages.jsx`

```jsx
const stats = [
  { label: "Total Mahasiswa", value: "17" },
  { label: "Total Pertemuan", value: "7" },
  { label: "Mahasiswa Hadir", value: "16" },
];

const announcements = [
  "Praktikum React dimulai minggu ke-6.",
  "Tugas 6 dikumpulkan paling lambat hari Minggu.",
  "Gunakan API lokal untuk latihan GET data.",
];

export default function DashboardPage() {
  return (
    <div className="space-y-5">
      <h2 className="text-xl font-bold">Dashboard</h2>

      <div className="grid gap-3 sm:grid-cols-3">
        {stats.map((item) => (
          <div key={item.label} className="rounded-lg border bg-slate-50 p-4">
            <p className="text-sm text-slate-500">{item.label}</p>
            <p className="mt-1 text-2xl font-bold text-blue-600">{item.value}</p>
          </div>
        ))}
      </div>

      <div className="rounded-lg border p-4">
        <h3 className="mb-2 text-sm font-semibold uppercase text-slate-500">
          Pengumuman
        </h3>

        <ul className="space-y-2 text-sm text-slate-700">
          {announcements.map((item) => (
            <li key={item} className="rounded-md bg-slate-50 px-3 py-2">
              {item}
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

Isi dashboard:

* 3 kartu statistik sederhana
* daftar pengumuman statis

Tujuan: mahasiswa memahami pemisahan halaman dan layout dashboard dasar.

---

# STEP 7 — Halaman Mahasiswa: Tabel + Search + Refresh

File: `src/pages/MahasiswaPages.jsx`

```jsx
import { useEffect, useState } from "react";
import { getMahasiswa } from "../services/api";

export default function Mahasiswa() {
  const [mahasiswa, setMahasiswa] = useState([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [error, setError] = useState("");
  const [search, setSearch] = useState("");

  const filteredMahasiswa = mahasiswa.filter((mhs) => {
    const keyword = search.trim().toLowerCase();

    if (!keyword) return true;

    return Object.values(mhs).some((value) =>
      String(value ?? "")
        .toLowerCase()
        .includes(keyword)
    );
  });

  useEffect(() => {
    let isMounted = true;

    const loadInitialData = async () => {
      try {
        if (isMounted) {
          setError("");
        }

        const data = await getMahasiswa();

        if (isMounted) {
          setMahasiswa(data);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    loadInitialData();

    return () => {
      isMounted = false;
    };
  }, []);

  const handleRefresh = async () => {
    try {
      setRefreshing(true);
      setError("");
      const data = await getMahasiswa();
      setMahasiswa(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setRefreshing(false);
    }
  };

  if (loading) return <p className="text-center">Loading...</p>;

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-bold">Daftar Mahasiswa</h2>

      <p className="text-sm text-gray-600">
        Total Mahasiswa:{" "}
        <span className="bg-blue-100 text-blue-700 px-2 py-1 rounded-md font-semibold">
          {mahasiswa.length}
        </span>
      </p>

      <div className="flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between">
        <input
          type="text"
          placeholder="Cari semua data mahasiswa..."
          value={search}
          onChange={(event) => setSearch(event.target.value)}
          className="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm outline-none focus:border-blue-500 sm:max-w-sm"
        />

        <button
          type="button"
          onClick={handleRefresh}
          disabled={refreshing}
          className="rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white transition hover:bg-blue-700 disabled:cursor-not-allowed disabled:opacity-70"
        >
          {refreshing ? "Refreshing..." : "Refresh Data"}
        </button>
      </div>

      {error && <p className="text-sm text-red-500">Error: {error}</p>}

      <div className="overflow-x-auto rounded-lg border">
        <table className="min-w-full text-left text-sm">
          <thead className="bg-gray-300 border-b text-gray-700 uppercase text-xs">
            <tr>
              <th className="px-4 py-3 border">No</th>
              <th className="px-4 py-3 border">NPM</th>
              <th className="px-4 py-3 border">Nama / Prodi</th>
              <th className="px-4 py-3 border">Email</th>
              <th className="px-4 py-3 border">Alamat</th>
            </tr>
          </thead>
          <tbody className="divide-y">
            {filteredMahasiswa.length > 0 ? (
              filteredMahasiswa.map((mhs, index) => (
                <tr key={mhs.npm} className="hover:bg-blue-50">
                  <td className="px-4 py-3 border">{index + 1}</td>
                  <td className="px-4 py-3 border">{mhs.npm}</td>
                  <td className="px-4 py-3 border">
                    <div className="font-medium">{mhs.nama}</div>
                    <div className="text-gray-500 text-xs">{mhs.prodi}</div>
                  </td>
                  <td className="px-4 py-3 text-gray-600 border">{mhs.email}</td>
                  <td className="px-4 py-3 text-gray-500 border">{mhs.alamat}</td>
                </tr>
              ))
            ) : (
              <tr>
                <td colSpan="5" className="px-4 py-4 text-center text-slate-500">
                  Data tidak ditemukan.
                </td>
              </tr>
            )}
          </tbody>
        </table>
      </div>
    </div>
  );
}
```

Fitur utama:

1. Load data awal dari API Axios
2. Search semua field menggunakan:
3. Tombol Refresh Data untuk ambil ulang data API
4. Loading, error, dan empty-state tetap ditampilkan

Penjelasan :

Pada step ini, kita membangun halaman **Daftar Mahasiswa** yang menjadi inti dari praktikum karena menggabungkan beberapa konsep penting dalam React, yaitu pengambilan data dari API, state management, filtering (search), dan interaksi user (refresh).

Pertama, kita menggunakan beberapa state seperti `mahasiswa`, `loading`, `refreshing`, `error`, dan `search`. State `mahasiswa` digunakan untuk menyimpan data dari API, sedangkan `loading` digunakan saat pertama kali data diambil. State `refreshing` digunakan khusus saat tombol refresh ditekan agar user mendapatkan feedback visual. `error` digunakan untuk menampilkan pesan jika terjadi kesalahan, dan `search` digunakan untuk menyimpan input pencarian dari user.

Data mahasiswa diambil dari backend menggunakan fungsi `getMahasiswa` yang sudah dibuat pada STEP sebelumnya. Proses ini dilakukan di dalam `useEffect`, sehingga otomatis dijalankan saat komponen pertama kali ditampilkan. Setelah data berhasil diambil, data disimpan ke state dan ditampilkan dalam tabel.

Fitur **search semua kolom** dibuat dengan cara mengambil semua nilai dari setiap object mahasiswa menggunakan `Object.values()`, kemudian dicek apakah ada yang mengandung keyword pencarian. Dengan pendekatan ini, user bisa mencari berdasarkan nama, NPM, email, alamat, atau field lainnya tanpa perlu membuat filter khusus satu per satu.

Selain itu, terdapat tombol **Refresh Data** yang memungkinkan user mengambil ulang data terbaru dari API tanpa perlu reload halaman. Saat tombol ditekan, state `refreshing` akan aktif sehingga tombol berubah menjadi “Refreshing...” sebagai feedback ke user.

Tabel digunakan untuk menampilkan data mahasiswa secara terstruktur. Jika hasil pencarian kosong, maka akan ditampilkan pesan “Data tidak ditemukan”, sehingga user tetap mendapatkan informasi yang jelas.

Secara keseluruhan, pada step ini kalian belajar:

* Mengambil data dari API menggunakan Axios
* Mengelola state di React
* Menggunakan `useEffect` untuk lifecycle data
* Membuat fitur pencarian dinamis
* Menangani loading dan error
* Meningkatkan user experience dengan fitur refresh

---

# STEP 8 — Jalankan Project

```bash
npm run dev
```

Pastikan:

1. Menu sidebar bisa pindah halaman
2. Dashboard tampil normal
3. Halaman Mahasiswa menampilkan data API
4. Search dan refresh berjalan

---

# Latihan Mandiri

1. Tambahkan 1 menu baru di sidebar: **Data Diri**
2. Buat halaman statis untuk menu tersebut
3. Pelajari penggunaan layout di React

---

# Penutup

Pada praktikum ini kita sudah:

* Migrasi dari `fetch` ke **Axios**
* Menata struktur UI lebih rapi (atomic + layout)
* Menambah pengalaman pengguna lewat sidebar responsive, dashboard, search, dan refresh

## Pengumpulan

* Push ke direktori `Pertemuan07/Praktikum`
* Screenshot:
  * Dashboard
  * Halaman Mahasiswa
  * Search aktif di Halaman Mahasiswa
  * Halaman Data Diri
