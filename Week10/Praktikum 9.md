# 📘 Praktikum Pertemuan 9

## Frontend: CRUD Mahasiswa + Routing Persisten + Atomic Design + Responsive + SweetAlert2

Dokumen ini berisi **semua kode lengkap** agar hasil akhir **sama persis** dengan project `my-fe`. Jika mahasiswa masih berada di Praktikum 7 lalu mengikuti Praktikum 9 ini **langkah demi langkah**, hasilnya akan sama dengan project `my-fe` saat ini.

# Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa mampu:

- Mengimplementasikan **CRUD Mahasiswa** (GET detail, POST, PUT, DELETE)
- Menerapkan **React Router** agar refresh tidak mengubah halaman aktif
- Menyusun komponen dengan **Atomic Design** (atoms, molecules, organisms, layout, pages)
- Menampilkan **SweetAlert2** untuk notifikasi insert, update, delete
- Membuat **UI responsive** untuk mobile, tablet, dan desktop

---

# Prasyarat

1. Praktikum 7 sudah berjalan (Axios + layout + sidebar).
2. Backend API Mahasiswa aktif.
3. Tailwind CSS sudah terpasang.

---

# Ringkasan Fitur yang Akan Dibuat

1. **List Mahasiswa** (GET list + search + refresh)
2. **Detail Mahasiswa** (GET detail + kartu ringkasan)
3. **Tambah Mahasiswa** (POST + SweetAlert)
4. **Edit Mahasiswa** (PUT + SweetAlert)
5. **Hapus Mahasiswa** (DELETE + SweetAlert konfirmasi)
6. **Routing React**
7. **Responsive + Atomic Design**

---

# Struktur Folder

```text
src/
 ├─ components/
 │  ├─ atoms/
 │  │  ├─ Button.jsx
 │  │  ├─ TextInput.jsx
 │  │  ├─ SelectInput.jsx
 │  │  └─ SidebarNavButton.jsx
 │  ├─ molecules/
 │  │  ├─ FormField.jsx
 │  │  ├─ PageTitle.jsx
 │  │  ├─ SidebarBrandCard.jsx
 │  │  └─ SidebarInfoCard.jsx
 │  ├─ organisms/
 │  │  ├─ MahasiswaTable.jsx
 │  │  └─ MahasiswaForm.jsx
 │  └─ layout/
 │     ├─ AppLayout.jsx
 │     ├─ Header.jsx
 │     ├─ Sidebar.jsx
 │     └─ Footer.jsx
 ├─ pages/
 │  ├─ DashboardPage.jsx
 │  ├─ MahasiswaListPage.jsx
 │  ├─ MahasiswaDetailPage.jsx
 │  ├─ MahasiswaFormPage.jsx
 │  ├─ MahasiswaPages.jsx
 │  └─ MahasiswaPagesTugas.jsx
 ├─ services/
 │  └─ api.js
 ├─ App.jsx
 ├─ main.jsx
 ├─ index.css
 └─ App.css
```

---

# STEP 1 — Install Dependencies

```bash
npm install react-router-dom sweetalert2
```

---

# STEP 2 — Entry Point & Routing

## `src/main.jsx`
**Tujuan:** mengaktifkan React Router di root aplikasi.

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import "./index.css";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
);
```

**Penjelasan singkat:**
- `BrowserRouter` membuat URL berubah sesuai halaman dan tetap konsisten saat refresh.
- `StrictMode` membantu mendeteksi potensi masalah saat development.
- `App` adalah root component yang berisi seluruh routing aplikasi.

## `src/App.jsx`
**Tujuan:** definisi routing utama (persisten saat refresh).

```jsx
import { Navigate, Route, Routes } from "react-router-dom";
import AppLayout from "./components/layout/AppLayout";
import DashboardPage from "./pages/DashboardPage";
import MahasiswaDetailPage from "./pages/MahasiswaDetailPage";
import MahasiswaFormPage from "./pages/MahasiswaFormPage";
import MahasiswaListPage from "./pages/MahasiswaListPage";

export default function App() {
  return (
    <Routes>
      <Route element={<AppLayout />}>
        <Route path="/" element={<Navigate to="/dashboard" replace />} />
        <Route path="/dashboard" element={<DashboardPage />} />
        <Route path="/mahasiswa" element={<MahasiswaListPage />} />
        <Route path="/mahasiswa/add" element={<MahasiswaFormPage mode="create" />} />
        <Route path="/mahasiswa/:npm" element={<MahasiswaDetailPage />} />
        <Route
          path="/mahasiswa/:npm/edit"
          element={<MahasiswaFormPage mode="edit" />}
        />
      </Route>
      <Route path="*" element={<p>Halaman tidak ditemukan</p>} />
    </Routes>
  );
}
```

**Penjelasan singkat:**
- `Routes` berisi daftar route dan komponen halaman yang ditampilkan.
- `Navigate` mengarahkan `/` langsung ke `/dashboard`.
- `:npm` pada path menjadi parameter dinamis untuk detail dan edit mahasiswa.
- `AppLayout` membungkus seluruh halaman agar header/sidebar/footer tetap tampil.

---

# STEP 3 — Atomic Design

## A. Atoms

### `Button.jsx`
**Tujuan:** tombol reusable dengan variasi warna (primary, secondary, ghost, danger).

```jsx
const VARIANT_STYLES = {
  primary: "bg-blue-600 text-white hover:bg-blue-700",
  secondary: "border border-slate-300 bg-white text-slate-700 hover:bg-slate-50",
  ghost: "text-slate-600 hover:bg-slate-100",
  danger: "bg-red-600 text-white hover:bg-red-700",
};

export default function Button({
  type = "button",
  variant = "primary",
  className = "",
  disabled,
  children,
  ...props
}) {
  return (
    <button
      type={type}
      disabled={disabled}
      className={`inline-flex items-center justify-center rounded-lg px-4 py-2 text-sm font-medium transition disabled:cursor-not-allowed disabled:opacity-70 ${
        VARIANT_STYLES[variant] ?? VARIANT_STYLES.primary
      } ${className}`}
      {...props}
    >
      {children}
    </button>
  );
}
```

**Penjelasan singkat:**
- `VARIANT_STYLES` menyimpan variasi warna agar konsisten di seluruh aplikasi.
- `variant` menentukan gaya tombol tanpa perlu menulis class berulang.
- `disabled` membuat tombol tidak bisa diklik dan tampil lebih redup.

### `TextInput.jsx`
**Tujuan:** input teks dengan style konsisten & state disabled.

```jsx
export default function TextInput({ className = "", ...props }) {
  return (
    <input
      {...props}
      className={`w-full rounded-lg border border-slate-300 px-3 py-2 text-sm outline-none transition focus:border-blue-500 focus:ring-2 focus:ring-blue-100 disabled:cursor-not-allowed disabled:bg-slate-100 disabled:text-slate-500 ${className}`}
    />
  );
}
```

**Penjelasan singkat:**
- Semua props (placeholder, value, onChange, dll) diteruskan lewat `{...props}`.
- Class Tailwind memastikan fokus input jelas dan responsif.

### `SelectInput.jsx`
**Tujuan:** dropdown/select yang style-nya konsisten dengan TextInput.

```jsx
export default function SelectInput({ className = "", children, ...props }) {
  return (
    <select
      {...props}
      className={`w-full rounded-lg border border-slate-300 px-3 py-2 text-sm outline-none transition focus:border-blue-500 focus:ring-2 focus:ring-blue-100 disabled:cursor-not-allowed disabled:bg-slate-100 disabled:text-slate-500 ${className}`}
    >
      {children}
    </select>
  );
}
```

**Penjelasan singkat:**
- Mirip `TextInput`, tetapi memakai elemen `<select>`.
- Menjaga tampilan dropdown konsisten dengan input lain.

### `SidebarNavButton.jsx`
**Tujuan:** tombol menu sidebar yang aktif sesuai route (NavLink).

```jsx
import { NavLink } from "react-router-dom";

export default function SidebarNavButton({ icon, label, description, to, onClick }) {
  return (
    <NavLink
      to={to}
      onClick={onClick}
      className={({ isActive }) =>
        `group block w-full rounded-xl border px-3 py-3 text-left transition ${
          isActive
            ? "border-blue-400/60 bg-gradient-to-r from-blue-500 to-indigo-500 text-white shadow-lg shadow-blue-900/30"
            : "border-white/10 bg-white/5 text-slate-200 hover:border-blue-300/40 hover:bg-white/10"
        }`
      }
    >
      {({ isActive }) => (
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
      )}
    </NavLink>
  );
}
```

**Penjelasan singkat:**
- `NavLink` otomatis memberi status `isActive` untuk menu yang sedang dibuka.
- Class diganti dinamis agar menu aktif terlihat menonjol.
- `icon`, `label`, `description` membuat menu lebih informatif.

---

## B. Molecules

### `FormField.jsx`
**Tujuan:** kombinasi label + input + error.

```jsx
export default function FormField({ label, htmlFor, error, children }) {
  return (
    <div className="space-y-1">
      {label && (
        <label htmlFor={htmlFor} className="text-sm font-medium text-slate-700">
          {label}
        </label>
      )}
      {children}
      {error && <p className="text-xs text-red-500">{error}</p>}
    </div>
  );
}
```

**Penjelasan singkat:**
- `label` terhubung ke input lewat `htmlFor` agar aksesibilitas lebih baik.
- Jika ada `error`, pesan tampil di bawah input.

### `PageTitle.jsx`
**Tujuan:** judul halaman + deskripsi + slot action.

```jsx
export default function PageTitle({ title, description, actions }) {
  return (
    <div className="flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between">
      <div>
        <h2 className="text-xl font-bold">{title}</h2>
        {description && <p className="text-sm text-slate-600">{description}</p>}
      </div>
      {actions && <div className="flex flex-wrap gap-2">{actions}</div>}
    </div>
  );
}
```

**Penjelasan singkat:**
- `actions` digunakan untuk meletakkan tombol seperti “Tambah Mahasiswa”.
- Layout fleksibel agar tetap rapi di layar kecil maupun besar.

---

## C. Organisms

### `MahasiswaTable.jsx`
**Tujuan:** tabel data mahasiswa + tombol aksi (Detail, Edit, Hapus).

```jsx
import { Link } from "react-router-dom";

export default function MahasiswaTable({ data, onDelete }) {
  const baseActionClass =
    "inline-flex items-center rounded-md border px-3 py-1 text-xs font-semibold transition hover:shadow-sm";

  return (
    <div className="overflow-x-auto rounded-lg border">
      <table className="min-w-full text-left text-sm">
        <thead className="bg-gray-300 border-b text-gray-700 uppercase text-xs">
          <tr>
            <th className="px-4 py-3 border">No</th>
            <th className="px-4 py-3 border">NPM</th>
            <th className="px-4 py-3 border">Nama / Prodi</th>
            <th className="px-4 py-3 border">Email</th>
            <th className="px-4 py-3 border">Alamat</th>
            <th className="px-4 py-3 border">Aksi</th>
          </tr>
        </thead>
        <tbody className="divide-y">
          {data.length > 0 ? (
            data.map((mhs, index) => (
              <tr key={mhs.npm} className="hover:bg-blue-50">
                <td className="px-4 py-3 border">{index + 1}</td>
                <td className="px-4 py-3 border">{mhs.npm}</td>
                <td className="px-4 py-3 border">
                  <div className="font-medium">{mhs.nama}</div>
                  <div className="text-gray-500 text-xs">{mhs.prodi}</div>
                </td>
                <td className="px-4 py-3 text-gray-600 border">{mhs.email}</td>
                <td className="px-4 py-3 text-gray-500 border">{mhs.alamat}</td>
                <td className="px-4 py-3 border">
                  <div className="flex flex-wrap gap-2">
                    <Link
                      to={`/mahasiswa/${mhs.npm}`}
                      className={`${baseActionClass} border-blue-200 bg-blue-50 text-blue-700 hover:bg-blue-100`}
                    >
                      Detail
                    </Link>
                    <Link
                      to={`/mahasiswa/${mhs.npm}/edit`}
                      className={`${baseActionClass} border-amber-200 bg-amber-50 text-amber-700 hover:bg-amber-100`}
                    >
                      Edit
                    </Link>
                    <button
                      type="button"
                      onClick={() => onDelete?.(mhs.npm)}
                      className={`${baseActionClass} border-red-200 bg-red-50 text-red-700 hover:bg-red-100`}
                    >
                      Hapus
                    </button>
                  </div>
                </td>
              </tr>
            ))
          ) : (
            <tr>
              <td colSpan="6" className="px-4 py-4 text-center text-slate-500">
                Data tidak ditemukan.
              </td>
            </tr>
          )}
        </tbody>
      </table>
    </div>
  );
}
```

**Penjelasan singkat:**
- `Link` dipakai agar navigasi detail/edit tetap menggunakan routing React.
- `baseActionClass` menyamakan gaya tombol aksi.
- `onDelete?.(mhs.npm)` memanggil handler hapus jika disediakan.

### `MahasiswaForm.jsx`
**Tujuan:** form tambah/edit mahasiswa dengan validasi NPM & dropdown Prodi.

```jsx
import Button from "../atoms/Button";
import SelectInput from "../atoms/SelectInput";
import TextInput from "../atoms/TextInput";
import FormField from "../molecules/FormField";

const PRODI_OPTIONS = [
  "S2 Manajemen Logistik",
  "S1 Manajemen Logistik",
  "S1 Manajemen Rekayasa",
  "S1 Manajemen Transportasi",
  "S1 Bisnis Digital",
  "S1 Sains Data",
  "D4 Akuntansi",
  "D4 Teknik Informatika",
  "D4 Logistik Bisnis",
  "D4 Akuntansi Keuangan",
  "D4 Manajemen Perusahaan",
  "D3 Administrasi Logistik",
  "D3 Manajemen Pemasaran",
  "D3 Teknik Informatika",
];

export default function MahasiswaForm({
  form,
  errors,
  onChange,
  onSubmit,
  onCancel,
  submitLabel,
  disableNpm,
  isSubmitting,
}) {
  const handleChange = (event) => {
    const { name, value } = event.target;
    if (name === "npm") {
      onChange(name, value.replace(/\D/g, ""));
      return;
    }
    onChange(name, value);
  };

  return (
    <form onSubmit={onSubmit} className="space-y-4">
      <div className="grid gap-4 md:grid-cols-2">
        <FormField label="NPM" htmlFor="npm" error={errors?.npm}>
          <TextInput
            id="npm"
            name="npm"
            inputMode="numeric"
            pattern="\d*"
            value={form.npm}
            onChange={handleChange}
            placeholder="Masukkan NPM"
            disabled={disableNpm}
          />
        </FormField>

        <FormField label="Nama" htmlFor="nama" error={errors?.nama}>
          <TextInput
            id="nama"
            name="nama"
            value={form.nama}
            onChange={handleChange}
            placeholder="Masukkan nama"
          />
        </FormField>

        <FormField label="Prodi" htmlFor="prodi" error={errors?.prodi}>
          <SelectInput
            id="prodi"
            name="prodi"
            value={form.prodi}
            onChange={handleChange}
          >
            <option value="">Pilih prodi</option>
            {PRODI_OPTIONS.map((option) => (
              <option key={option} value={option}>
                {option}
              </option>
            ))}
          </SelectInput>
        </FormField>

        <FormField label="Email" htmlFor="email" error={errors?.email}>
          <TextInput
            id="email"
            name="email"
            type="email"
            value={form.email}
            onChange={handleChange}
            placeholder="nama@email.com"
          />
        </FormField>
      </div>

      <FormField label="Alamat" htmlFor="alamat" error={errors?.alamat}>
        <TextInput
          id="alamat"
          name="alamat"
          value={form.alamat}
          onChange={handleChange}
          placeholder="Masukkan alamat lengkap"
        />
      </FormField>

      <div className="flex flex-wrap justify-end gap-2">
        <Button type="button" variant="secondary" onClick={onCancel}>
          Kembali
        </Button>
        <Button type="submit" disabled={isSubmitting}>
          {isSubmitting ? "Menyimpan..." : submitLabel}
        </Button>
      </div>
    </form>
  );
}
```

**Penjelasan singkat:**
- `handleChange` khusus NPM hanya menerima angka (`replace(/\D/g, "")`).
- `disableNpm` dipakai saat edit agar NPM tidak berubah.

---

## D. Layout

### `Sidebar.jsx`
**Tujuan:** sidebar navigasi dengan NavLink.

```jsx
import SidebarNavButton from "../atoms/SidebarNavButton";
import SidebarBrandCard from "../molecules/SidebarBrandCard";
import SidebarInfoCard from "../molecules/SidebarInfoCard";

const menuItems = [
  {
    id: "dashboard",
    label: "Dashboard",
    description: "Ringkasan data kelas",
    icon: "🏠",
    to: "/dashboard",
  },
  {
    id: "mahasiswa",
    label: "Mahasiswa",
    description: "Daftar data mahasiswa",
    icon: "🎓",
    to: "/mahasiswa",
  },
];

export default function Sidebar({ isOpen, onClose }) {
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
                to={item.to}
                onClick={onClose}
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

**Penjelasan singkat:**
- Overlay muncul pada mobile agar sidebar bisa ditutup saat area kosong diklik.
- `menuItems` memudahkan menambah menu baru tanpa menulis ulang komponen.
- `onClose` menutup sidebar setelah menu dipilih di layar kecil.

### `AppLayout.jsx`
**Tujuan:** layout utama dengan sidebar, header, footer, dan Outlet.

```jsx
import { Outlet, useLocation } from "react-router-dom";
import { useMemo, useState } from "react";
import Header from "./Header";
import Sidebar from "./Sidebar";
import Footer from "./Footer";

const PAGE_TITLES = {
  "/dashboard": "Dashboard",
  "/mahasiswa": "Data Mahasiswa",
};

export default function AppLayout() {
  const [isSidebarOpen, setIsSidebarOpen] = useState(false);
  const location = useLocation();

  const pageTitle = useMemo(() => {
    if (location.pathname.startsWith("/mahasiswa")) {
      return "Data Mahasiswa";
    }

    return PAGE_TITLES[location.pathname] ?? "Praktikum";
  }, [location.pathname]);

  return (
    <div className="min-h-screen bg-slate-100 text-slate-800">
      <Sidebar isOpen={isSidebarOpen} onClose={() => setIsSidebarOpen(false)} />

      <div className="flex min-h-screen flex-col md:pl-72">
        <Header
          pageTitle={pageTitle}
          onToggleSidebar={() => setIsSidebarOpen((prev) => !prev)}
        />

        <main className="flex-1 p-3 sm:p-4 md:p-6">
          <div className="w-full rounded-2xl border border-slate-200 bg-white p-4 shadow-sm md:p-6">
            <Outlet />
          </div>
        </main>

        <Footer />
      </div>
    </div>
  );
}
```

**Penjelasan singkat:**
- `Outlet` adalah tempat halaman aktif dirender oleh React Router.
- `pageTitle` berubah sesuai URL agar header dinamis.
- Layout dibuat responsif dengan `md:pl-72` untuk memberi ruang sidebar.

---

## E. Pages

### `MahasiswaListPage.jsx`
**Tujuan:** list mahasiswa + search + refresh + delete dengan SweetAlert2.

```jsx
import { useEffect, useMemo, useState } from "react";
import { Link } from "react-router-dom";
import Swal from "sweetalert2";
import Button from "../components/atoms/Button";
import TextInput from "../components/atoms/TextInput";
import PageTitle from "../components/molecules/PageTitle";
import MahasiswaTable from "../components/organisms/MahasiswaTable";
import { deleteMahasiswa, getMahasiswa } from "../services/api";

export default function MahasiswaListPage() {
  const [mahasiswa, setMahasiswa] = useState([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [error, setError] = useState("");
  const [search, setSearch] = useState("");

  const filteredMahasiswa = useMemo(() => {
    const keyword = search.trim().toLowerCase();

    if (!keyword) return mahasiswa;

    return mahasiswa.filter((mhs) =>
      Object.values(mhs).some((value) =>
        String(value ?? "")
          .toLowerCase()
          .includes(keyword)
      )
    );
  }, [mahasiswa, search]);

  useEffect(() => {
    let isMounted = true;

    const loadInitialData = async () => {
      try {
        const data = await getMahasiswa();

        if (isMounted) {
          setMahasiswa(data);
          setError("");
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

  const handleDelete = async (npm) => {
    const result = await Swal.fire({
      title: "Hapus data mahasiswa?",
      text: "Data yang dihapus tidak dapat dikembalikan.",
      icon: "warning",
      showCancelButton: true,
      confirmButtonText: "Ya, hapus",
      cancelButtonText: "Batal",
      confirmButtonColor: "#dc2626",
    });

    if (!result.isConfirmed) return;

    try {
      setError("");
      await deleteMahasiswa(npm);
      setMahasiswa((prev) => prev.filter((item) => item.npm !== npm));
      await Swal.fire({
        title: "Berhasil",
        text: "Data mahasiswa berhasil dihapus.",
        icon: "success",
        confirmButtonText: "OK",
      });
    } catch (err) {
      setError(err.message);
      await Swal.fire({
        title: "Gagal",
        text: err.message || "Gagal menghapus data.",
        icon: "error",
        confirmButtonText: "Tutup",
      });
    }
  };

  if (loading) return <p className="text-center">Loading...</p>;

  return (
    <div className="space-y-4">
      <PageTitle
        title="Daftar Mahasiswa"
        description="Kelola data mahasiswa, tambahkan, edit, dan lihat detail."
        actions={
          <Link to="/mahasiswa/add">
            <Button type="button">Tambah Mahasiswa</Button>
          </Link>
        }
      />

      <p className="text-sm text-gray-600">
        Total Mahasiswa:{" "}
        <span className="bg-blue-100 text-blue-700 px-2 py-1 rounded-md font-semibold">
          {mahasiswa.length}
        </span>
      </p>

      <div className="flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between">
        <TextInput
          type="text"
          placeholder="Cari semua data mahasiswa..."
          value={search}
          onChange={(event) => setSearch(event.target.value)}
          className="sm:max-w-sm"
        />

        <Button
          type="button"
          variant="secondary"
          onClick={handleRefresh}
          disabled={refreshing}
        >
          {refreshing ? "Refreshing..." : "Refresh Data"}
        </Button>
      </div>

      {error && <p className="text-sm text-red-500">Error: {error}</p>}

      <MahasiswaTable data={filteredMahasiswa} onDelete={handleDelete} />
    </div>
  );
}
```

**Penjelasan singkat:**
- `useMemo` mencegah filter ulang jika data belum berubah.
- `SweetAlert2` digunakan untuk konfirmasi hapus dan notifikasi sukses/gagal.
- `refreshing` memberi umpan balik saat data diambil ulang.

### `MahasiswaDetailPage.jsx`
**Tujuan:** detail mahasiswa lebih menarik dengan card & avatar.

```jsx
import { useEffect, useState } from "react";
import { Link, useParams } from "react-router-dom";
import Button from "../components/atoms/Button";
import PageTitle from "../components/molecules/PageTitle";
import { getMahasiswaDetail } from "../services/api";

export default function MahasiswaDetailPage() {
  const { npm } = useParams();
  const [mahasiswa, setMahasiswa] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    let isMounted = true;

    const loadDetail = async () => {
      try {
        const data = await getMahasiswaDetail(npm);

        if (isMounted) {
          setMahasiswa(data);
          setError("");
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

    loadDetail();

    return () => {
      isMounted = false;
    };
  }, [npm]);

  if (loading) return <p className="text-center">Loading...</p>;

  if (error) return <p className="text-center text-red-500">Error: {error}</p>;

  if (!mahasiswa) {
    return <p className="text-center text-slate-500">Data tidak ditemukan.</p>;
  }

  const initials =
    mahasiswa.nama
      ?.split(" ")
      .filter(Boolean)
      .slice(0, 2)
      .map((word) => word[0].toUpperCase())
      .join("") || "M";

  return (
    <div className="space-y-4">
      <PageTitle
        title="Detail Mahasiswa"
        description="Informasi lengkap mahasiswa."
      />

      <div className="rounded-2xl border bg-gradient-to-br from-blue-50 via-white to-indigo-50 p-5">
        <div className="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
          <div className="flex items-center gap-4">
            <div className="flex h-14 w-14 items-center justify-center rounded-full bg-blue-600 text-lg font-bold text-white">
              {initials}
            </div>
            <div>
              <p className="text-xs uppercase text-slate-500">NPM</p>
              <p className="text-sm font-semibold text-slate-800">
                {mahasiswa.npm}
              </p>
              <h3 className="mt-1 text-lg font-semibold text-slate-900">
                {mahasiswa.nama}
              </h3>
              <span className="mt-2 inline-flex rounded-full bg-blue-100 px-3 py-1 text-xs font-semibold text-blue-700">
                {mahasiswa.prodi}
              </span>
            </div>
          </div>
          <div className="flex flex-wrap gap-2">
            <Link to={`/mahasiswa/${mahasiswa.npm}/edit`}>
              <Button type="button" variant="secondary">
                Edit
              </Button>
            </Link>
            <Link to="/mahasiswa">
              <Button type="button" variant="ghost">
                Kembali
              </Button>
            </Link>
          </div>
        </div>
      </div>

      <div className="grid gap-4 sm:grid-cols-2">
        <div className="rounded-xl border bg-white p-4 shadow-sm">
          <p className="text-xs uppercase text-slate-500">Email</p>
          <p className="mt-1 text-sm font-semibold text-slate-800">
            {mahasiswa.email}
          </p>
        </div>
        <div className="rounded-xl border bg-white p-4 shadow-sm">
          <p className="text-xs uppercase text-slate-500">Prodi</p>
          <p className="mt-1 text-sm font-semibold text-slate-800">
            {mahasiswa.prodi}
          </p>
        </div>
        <div className="rounded-xl border bg-white p-4 shadow-sm sm:col-span-2">
          <p className="text-xs uppercase text-slate-500">Alamat</p>
          <p className="mt-1 text-sm font-semibold text-slate-800">
            {mahasiswa.alamat}
          </p>
        </div>
      </div>
    </div>
  );
}
```

**Penjelasan singkat:**
- `initials` diambil dari nama untuk avatar sederhana.
- Card dan badge membuat detail lebih mudah dibaca.

### `MahasiswaFormPage.jsx`
**Tujuan:** halaman form tambah/edit + SweetAlert2.

```jsx
import { useEffect, useState } from "react";
import { useNavigate, useParams } from "react-router-dom";
import Swal from "sweetalert2";
import PageTitle from "../components/molecules/PageTitle";
import MahasiswaForm from "../components/organisms/MahasiswaForm";
import {
  createMahasiswa,
  getMahasiswaDetail,
  updateMahasiswa,
} from "../services/api";

const EMPTY_FORM = {
  npm: "",
  nama: "",
  prodi: "",
  email: "",
  alamat: "",
};

export default function MahasiswaFormPage({ mode }) {
  const isEdit = mode === "edit";
  const { npm } = useParams();
  const navigate = useNavigate();
  const [form, setForm] = useState(EMPTY_FORM);
  const [errors, setErrors] = useState({});
  const [error, setError] = useState("");
  const [loading, setLoading] = useState(isEdit);
  const [saving, setSaving] = useState(false);

  useEffect(() => {
    let isMounted = true;

    const loadDetail = async () => {
      if (!isEdit) {
        setLoading(false);
        return;
      }

      try {
        const data = await getMahasiswaDetail(npm);

        if (isMounted) {
          setForm({
            npm: data?.npm ? String(data.npm) : "",
            nama: data?.nama ?? "",
            prodi: data?.prodi ?? "",
            email: data?.email ?? "",
            alamat: data?.alamat ?? "",
          });
          setError("");
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

    loadDetail();

    return () => {
      isMounted = false;
    };
  }, [isEdit, npm]);

  const handleChange = (field, value) => {
    setForm((prev) => ({
      ...prev,
      [field]: value,
    }));
  };

  const validateForm = () => {
    const nextErrors = {};
    const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/;

    if (!form.npm.trim()) nextErrors.npm = "NPM wajib diisi";
    if (form.npm && !/^\d+$/.test(form.npm)) {
      nextErrors.npm = "NPM harus berupa angka";
    }
    if (!form.nama.trim()) nextErrors.nama = "Nama wajib diisi";
    if (!form.prodi.trim()) nextErrors.prodi = "Prodi wajib diisi";
    if (!form.email.trim()) nextErrors.email = "Email wajib diisi";
    if (!form.alamat.trim()) nextErrors.alamat = "Alamat wajib diisi";
    if (form.email && !emailPattern.test(form.email)) {
      nextErrors.email = "Email tidak valid";
    }

    setErrors(nextErrors);
    return Object.keys(nextErrors).length === 0;
  };

  const handleSubmit = async (event) => {
    event.preventDefault();

    if (!validateForm()) return;

    try {
      setSaving(true);
      setError("");
      const payload = {
        ...form,
        npm: Number(form.npm),
      };

      if (isEdit) {
        await updateMahasiswa(npm, payload);
      } else {
        await createMahasiswa(payload);
      }

      await Swal.fire({
        title: "Berhasil",
        text: isEdit
          ? "Data mahasiswa berhasil diperbarui."
          : "Data mahasiswa berhasil ditambahkan.",
        icon: "success",
        confirmButtonText: "OK",
      });

      navigate("/mahasiswa");
    } catch (err) {
      setError(err.message);
      await Swal.fire({
        title: "Gagal",
        text: err.message || "Terjadi kesalahan.",
        icon: "error",
        confirmButtonText: "Tutup",
      });
    } finally {
      setSaving(false);
    }
  };

  if (loading) return <p className="text-center">Loading...</p>;

  return (
    <div className="space-y-4">
      <PageTitle
        title={isEdit ? "Edit Mahasiswa" : "Tambah Mahasiswa"}
        description="Lengkapi form berikut untuk menyimpan data."
      />

      {error && <p className="text-sm text-red-500">Error: {error}</p>}

      <MahasiswaForm
        form={form}
        errors={errors}
        onChange={handleChange}
        onSubmit={handleSubmit}
        onCancel={() => navigate(-1)}
        submitLabel={isEdit ? "Simpan Perubahan" : "Tambah Mahasiswa"}
        disableNpm={isEdit}
        isSubmitting={saving}
      />
    </div>
  );
}
```

**Penjelasan singkat:**
- `validateForm` memastikan data wajib terisi dan email valid.
- `Number(form.npm)` mengubah NPM menjadi angka agar sesuai backend.
- `Swal.fire` memberi feedback setelah simpan data.

### `MahasiswaPages.jsx` (Legacy dari Praktikum 7) biarkan saja jangan dihapus
**Tujuan:** versi lama list mahasiswa tanpa Atomic Design (disimpan sebagai referensi).

---

## F. Services (API)

### `api.js`
**Tujuan:** semua komunikasi API Mahasiswa (GET, POST, PUT, DELETE).

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

function normalizeError(error, fallback = "Terjadi kesalahan") {
  return error.response?.data?.message || error.message || fallback;
}

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
    throw new Error(normalizeError(error, "Gagal mengambil data"));
  }
}

export async function getMahasiswaDetail(npm) {
  try {
    const response = await api.get(`/mahasiswa/${npm}`);
    return response.data?.data ?? response.data;
  } catch (error) {
    throw new Error(normalizeError(error, "Gagal mengambil detail"));
  }
}

export async function createMahasiswa(payload) {
  try {
    const response = await api.post("/mahasiswa", payload);
    return response.data;
  } catch (error) {
    throw new Error(normalizeError(error, "Gagal menambah data"));
  }
}

export async function updateMahasiswa(npm, payload) {
  try {
    const response = await api.put(`/mahasiswa/${npm}`, payload);
    return response.data;
  } catch (error) {
    throw new Error(normalizeError(error, "Gagal memperbarui data"));
  }
}

export async function deleteMahasiswa(npm) {
  try {
    const response = await api.delete(`/mahasiswa/${npm}`);
    return response.data;
  } catch (error) {
    throw new Error(normalizeError(error, "Gagal menghapus data"));
  }
}
```

**Penjelasan singkat:**
- `axios.create` membuat instance API dengan `baseURL` dan `timeout`.
- Fungsi `get/create/update/delete` dibuat terpisah agar mudah dipanggil di halaman.
- `normalizeError` menyederhanakan pesan error agar lebih ramah pengguna.
---

# Catatan Validasi & Data

- **NPM wajib angka** → input difilter `value.replace(/\D/g, "")` dan dikirim sebagai `Number(form.npm)`.
- **Email valid** → regex `^[^\s@]+@[^\s@]+\.[^\s@]{2,}$` (contoh valid: `usman@hotmail.id`, `usman@gmail.com`).

---

# Menjalankan Project

```bash
npm install
npm run dev
```

---

# Latihan Mandiri (Tugas di Rumah)

1. Tambahkan fitur **filter prodi** (dropdown di halaman list) dan tampilkan hasil filter di tabel.
2. Tambahkan **tombol reset** untuk menghapus keyword pencarian dan filter prodi.
3. Tambahkan tombol kembali ke dashboard di halaman Tabel Mahasiswa.

---

# Penutup (Kesimpulan)

Pada praktikum ini kita sudah:

- Mengubah navigasi menjadi **React Router** agar refresh tetap di halaman yang sama.
- Menerapkan **Atomic Design** agar komponen mudah dipakai ulang.
- Mengimplementasikan **CRUD lengkap** dengan notifikasi SweetAlert2.
- Membuat tampilan **responsif** di berbagai ukuran layar.

## Pengumpulan

- Push ke direktori `Week10/Praktikum`.
- Screenshot:
  - Halaman Dashboard
  - Halaman Daftar Mahasiswa
  - Halaman Detail Mahasiswa
  - Form Tambah/Edit (salah satu)
  - SweetAlert saat insert, update, delete berhasil
