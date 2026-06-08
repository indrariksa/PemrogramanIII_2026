# Praktikum Pertemuan 12

# Bagian Frontend

## STEP 1 - Buat Service Auth

File: `my-fe/src/services/auth.js`

```js
import axios from "axios";

const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;
const AUTH_BASE_URL = API_BASE_URL?.replace(/\/api\/?$/, "");
const TOKEN_KEY = "token";
const USER_KEY = "user";

if (!AUTH_BASE_URL) {
  throw new Error("VITE_API_BASE_URL belum diatur di file .env");
}

const authApi = axios.create({
  baseURL: AUTH_BASE_URL,
  headers: {
    Accept: "application/json",
  },
});

export function getToken() {
  return localStorage.getItem(TOKEN_KEY);
}

export function getUser() {
  const rawUser = localStorage.getItem(USER_KEY);

  if (!rawUser) return null;

  try {
    return JSON.parse(rawUser);
  } catch {
    return null;
  }
}

export function saveAuthSession({ token, user }) {
  localStorage.setItem(TOKEN_KEY, token);
  localStorage.setItem(USER_KEY, JSON.stringify(user));
}

export function clearAuthSession() {
  localStorage.removeItem(TOKEN_KEY);
  localStorage.removeItem(USER_KEY);
}

export async function login({ username, password }) {
  const response = await authApi.post("/login", { username, password });
  return response.data?.data ?? response.data;
}

export async function register({ username, password, role }) {
  const response = await authApi.post("/register", { username, password, role });
  return response.data?.data ?? response.data;
}
```

Pastikan file `.env` frontend berisi:

```env
VITE_API_BASE_URL=http://127.0.0.1:3000/api
VITE_API_TIMEOUT=10000
```

---

## STEP 2 - Update Axios Service Mahasiswa

File: `my-fe/src/services/api.js`

Tambahkan import:

```js
import { clearAuthSession, getToken } from "./auth";
```

Tambahkan interceptor setelah `axios.create`:

```js
api.interceptors.request.use((config) => {
  const token = getToken();

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      clearAuthSession();
      window.location.href = "/login";
    }

    return Promise.reject(error);
  },
);
```

Penjelasan:

- Request interceptor menambahkan token ke semua request mahasiswa.
- Response interceptor menghapus token dan redirect ke login jika token tidak valid.
- Error `403` tidak redirect, tetapi ditampilkan oleh halaman lewat SweetAlert/error text.

---

## STEP 3 - Buat PrivateRoute

File: `my-fe/src/routes/PrivateRoute.jsx`

```jsx
import { Navigate, Outlet } from "react-router-dom";
import { getToken } from "../services/auth";

export default function PrivateRoute() {
  if (!getToken()) {
    return <Navigate to="/login" replace />;
  }

  return <Outlet />;
}
```

---

## STEP 4 - Buat Halaman Login

File: `my-fe/src/pages/LoginPage.jsx`

```jsx
import { useEffect, useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import Swal from "sweetalert2";
import Button from "../components/atoms/Button";
import TextInput from "../components/atoms/TextInput";
import FormField from "../components/molecules/FormField";
import { getToken, login, saveAuthSession } from "../services/auth";

export default function LoginPage() {
  const navigate = useNavigate();
  const [form, setForm] = useState({ username: "", password: "" });
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (getToken()) {
      navigate("/dashboard", { replace: true });
    }
  }, [navigate]);

  const handleChange = (event) => {
    const { name, value } = event.target;
    setForm((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (event) => {
    event.preventDefault();

    try {
      setLoading(true);
      const data = await login(form);
      saveAuthSession(data);
      await Swal.fire({
        title: "Berhasil",
        text: "Login berhasil.",
        icon: "success",
        confirmButtonText: "OK",
      });
      navigate("/dashboard", { replace: true });
    } catch (error) {
      await Swal.fire({
        title: "Gagal",
        text: error.response?.data?.message || "Username atau password salah.",
        icon: "error",
        confirmButtonText: "Tutup",
      });
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="flex min-h-screen items-center justify-center bg-slate-100 px-4 py-8">
      <section className="w-full max-w-md rounded-lg border border-slate-200 bg-white p-6 shadow-sm">
        <div className="mb-6 text-center">
          <p className="text-sm font-semibold uppercase text-blue-600">
            Praktikum 11
          </p>
          <h1 className="mt-1 text-2xl font-bold text-slate-900">Login JWT</h1>
          <p className="mt-2 text-sm text-slate-600">
            Masuk untuk mengakses dashboard dan CRUD mahasiswa.
          </p>
        </div>

        <form onSubmit={handleSubmit} className="space-y-4">
          <FormField label="Username" htmlFor="username">
            <TextInput
              id="username"
              name="username"
              value={form.username}
              onChange={handleChange}
              placeholder="username"
              autoComplete="username"
              required
            />
          </FormField>

          <FormField label="Password" htmlFor="password">
            <TextInput
              id="password"
              name="password"
              type="password"
              value={form.password}
              onChange={handleChange}
              placeholder="password"
              autoComplete="current-password"
              required
            />
          </FormField>

          <Button type="submit" className="w-full" disabled={loading}>
            {loading ? "Memproses..." : "Login"}
          </Button>
        </form>

        <p className="mt-5 text-center text-sm text-slate-600">
          Belum punya akun?{" "}
          <Link to="/register" className="font-semibold text-blue-600 hover:text-blue-700">
            Register
          </Link>
        </p>
      </section>
    </main>
  );
}
```

---

## STEP 5 - Buat Halaman Register

File: `my-fe/src/pages/RegisterPage.jsx`

```jsx
import { useState } from "react";
import { Link, useNavigate } from "react-router-dom";
import Swal from "sweetalert2";
import Button from "../components/atoms/Button";
import SelectInput from "../components/atoms/SelectInput";
import TextInput from "../components/atoms/TextInput";
import FormField from "../components/molecules/FormField";
import { register } from "../services/auth";

export default function RegisterPage() {
  const navigate = useNavigate();
  const [form, setForm] = useState({
    username: "",
    password: "",
    role: "admin",
  });
  const [loading, setLoading] = useState(false);

  const handleChange = (event) => {
    const { name, value } = event.target;
    setForm((prev) => ({ ...prev, [name]: value }));
  };

  const handleSubmit = async (event) => {
    event.preventDefault();

    try {
      setLoading(true);
      await register(form);
      await Swal.fire({
        title: "Berhasil",
        text: "Register berhasil. Silakan login.",
        icon: "success",
        confirmButtonText: "OK",
      });
      navigate("/login");
    } catch (error) {
      await Swal.fire({
        title: "Gagal",
        text: error.response?.data?.message || "Register gagal.",
        icon: "error",
        confirmButtonText: "Tutup",
      });
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="flex min-h-screen items-center justify-center bg-slate-100 px-4 py-8">
      <section className="w-full max-w-md rounded-lg border border-slate-200 bg-white p-6 shadow-sm">
        <div className="mb-6 text-center">
          <p className="text-sm font-semibold uppercase text-blue-600">
            Praktikum 11
          </p>
          <h1 className="mt-1 text-2xl font-bold text-slate-900">Register User</h1>
          <p className="mt-2 text-sm text-slate-600">
            Buat akun untuk mencoba autentikasi JWT dan role.
          </p>
        </div>

        <form onSubmit={handleSubmit} className="space-y-4">
          <FormField label="Username" htmlFor="username">
            <TextInput
              id="username"
              name="username"
              value={form.username}
              onChange={handleChange}
              placeholder="username"
              autoComplete="username"
              required
            />
          </FormField>

          <FormField label="Password" htmlFor="password">
            <TextInput
              id="password"
              name="password"
              type="password"
              value={form.password}
              onChange={handleChange}
              placeholder="password"
              autoComplete="new-password"
              required
            />
          </FormField>

          <FormField label="Role" htmlFor="role">
            <SelectInput id="role" name="role" value={form.role} onChange={handleChange}>
              <option value="admin">admin</option>
              <option value="user">user</option>
            </SelectInput>
          </FormField>

          <Button type="submit" className="w-full" disabled={loading}>
            {loading ? "Menyimpan..." : "Register"}
          </Button>
        </form>

        <p className="mt-5 text-center text-sm text-slate-600">
          Sudah punya akun?{" "}
          <Link to="/login" className="font-semibold text-blue-600 hover:text-blue-700">
            Login
          </Link>
        </p>
      </section>
    </main>
  );
}
```

---

## STEP 6 - Update Routing App

File: `my-fe/src/App.jsx`

```jsx
import { Navigate, Route, Routes } from "react-router-dom";
import AppLayout from "./components/layout/AppLayout";
import DashboardPage from "./pages/DashboardPage";
import LoginPage from "./pages/LoginPage";
import MahasiswaDetailPage from "./pages/MahasiswaDetailPage";
import MahasiswaFormPage from "./pages/MahasiswaFormPage";
import MahasiswaListPage from "./pages/MahasiswaListPage";
import RegisterPage from "./pages/RegisterPage";
import PrivateRoute from "./routes/PrivateRoute";

export default function App() {
  return (
    <Routes>
      <Route path="/" element={<Navigate to="/dashboard" replace />} />
      <Route path="/login" element={<LoginPage />} />
      <Route path="/register" element={<RegisterPage />} />

      <Route element={<PrivateRoute />}>
        <Route element={<AppLayout />}>
          <Route path="/dashboard" element={<DashboardPage />} />
          <Route path="/mahasiswa" element={<MahasiswaListPage />} />
          <Route path="/mahasiswa/new" element={<MahasiswaFormPage mode="create" />} />
          <Route path="/mahasiswa/:npm" element={<MahasiswaDetailPage />} />
          <Route
            path="/mahasiswa/:npm/edit"
            element={<MahasiswaFormPage mode="edit" />}
          />
        </Route>
      </Route>

      <Route path="*" element={<p>Halaman tidak ditemukan</p>} />
    </Routes>
  );
}
```

Penjelasan:

- `/login` dan `/register` adalah route public.
- `/dashboard` dan `/mahasiswa` masuk ke `PrivateRoute`.
- Jika belum login, user diarahkan ke `/login`.
- Jika sudah login, halaman dashboard dan CRUD dapat dibuka.

---

## STEP 7 - Tambahkan Logout di Header

File: `my-fe/src/components/layout/Header.jsx`

```jsx
import { useNavigate } from "react-router-dom";
import Swal from "sweetalert2";
import { clearAuthSession, getUser } from "../../services/auth";
import Button from "../atoms/Button";

export default function Header({ pageTitle, onToggleSidebar }) {
  const navigate = useNavigate();
  const user = getUser();

  const handleLogout = async () => {
    const result = await Swal.fire({
      title: "Logout?",
      text: "Sesi login akan dihapus dari browser.",
      icon: "question",
      showCancelButton: true,
      confirmButtonText: "Ya, logout",
      cancelButtonText: "Batal",
      confirmButtonColor: "#dc2626",
    });

    if (result.isConfirmed) {
      clearAuthSession();
      navigate("/login", { replace: true });
    }
  };

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

        <div className="flex items-center gap-2">
          <span className="hidden rounded-full bg-slate-100 px-3 py-1 text-xs font-medium text-slate-600 sm:inline-flex">
            {user?.username ?? "User"} ({user?.role ?? "-"})
          </span>
          <span className="rounded-full bg-gradient-to-r from-blue-100 to-indigo-100 px-3 py-1 text-xs font-medium text-blue-700 md:text-sm">
            {pageTitle}
          </span>
          <Button
            type="button"
            variant="danger"
            className="px-3 py-1 text-xs"
            onClick={handleLogout}
          >
            Logout
          </Button>
        </div>
      </div>
    </header>
  );
}
```

Catatan: di project asli tombol mobile boleh tetap memakai simbol yang sudah ada. Yang penting fungsi logout ditambahkan.

---

# Cara Menjalankan

## Backend

```bash
go mod tidy
go run main.go
```

## Frontend

```bash
npm install
npm run dev
```

Buka:

```text
http://localhost:5173
```

---

# Skenario Pengujian

1. Buka `/dashboard` tanpa login.
2. Aplikasi harus redirect ke `/login`.
3. Buka `/register`.
4. Buat akun:

```json
{
  "username": "admin2",
  "password": "admin123",
  "role": "admin"
}
```

5. Login dengan akun admin.
6. Masuk ke halaman Mahasiswa.
7. Data mahasiswa harus tampil.
8. Coba tambah, edit, hapus data.
9. Klik logout.
10. Setelah logout, akses dashboard harus kembali redirect ke login.
11. Buat akun role `user`, login, lalu buka halaman mahasiswa.
12. Backend akan mengembalikan `403 Forbidden` (Error: user tidak memiliki akses untuk fitur ini) karena route mahasiswa hanya untuk `admin`.

---

## Pengumpulan Praktikum
- Push ke direktori Pertemuan12/Praktikum
- Screenshoot dari frontend ketika register, login, dan data berhasil ditampilkan pada role admin, data tidak berhasil ditampilkan pada role user (Push ke direktori Pertemuan12/Praktikum/Hasil)

# Latihan Mandiri

1. Tampilkan pesan khusus saat status `403 Forbidden`, misalnya: "Akun Anda bukan admin".

    ![image](images/tugas-1.png)

2. Tambahkan fitur ubah password.
    ![image](images/tugas-2.png)

3. Tambahkan halaman profil yang menampilkan username dan role dari localStorage.
    ![image](images/tugas-3.png)

4. Tambahkan tombol "Lihat Token" untuk menampilkan token JWT di modal SweetAlert.
    ![image](images/tugas-4.png)

- Catatan: Gambar di atas hanya digunakan sebagai referensi. Mahasiswa diperbolehkan mengembangkan dan menyesuaikan tampilan antarmuka (UI) sesuai dengan kreativitas, kemampuan, dan pemahaman masing-masing, selama seluruh fitur yang diminta telah diimplementasikan dengan baik.
---

## Pengumpulan Latihan Mandiri
- Push ke direktori Pertemuan12/Tugas
- Screenshoot dari 4 tugas di atas (Push ke direktori Pertemuan12/Praktikum/Tugas)
---
