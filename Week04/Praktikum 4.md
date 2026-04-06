# Panduan Praktikum Pertemuan 4: Backend Package & PostgreSQL Integration

Modul ini membahas implementasi backend menggunakan **Golang**, **GORM**, dan **PostgreSQL (Supabase)** dengan pendekatan modular serta dilengkapi dengan **unit testing repository**.

---

## 1. Tujuan Pembelajaran
* Memahami arsitektur modular backend
* Menghubungkan Golang dengan PostgreSQL (Supabase)
* Mengimplementasikan CRUD dengan GORM
* Menggunakan environment variable (.env)
* Melakukan unit testing pada repository

---

## 2. Persiapan Database (Supabase)
Berbeda dengan database NoSQL yang fleksibel, PostgreSQL adalah database relasional yang menyimpan data dalam tabel terstruktur. Kita akan menggunakan **Supabase** sebagai layanan cloud.

1.  **Login/Daftar**: Masuk ke [Supabase](https://supabase.com/) menggunakan akun GitHub.
2.  **Buat Project**: Pilih nama project (misal: `Matkul-PemrogramanIII`) dan simpan password database Kalian.
![image](images/1774929957386-21aac3b2-091e-47a6-8e57-ff44ba836e9d.png)

---

## 3. Inisialisasi Project
Buka terminal di folder kerja Kalian dan Jalankan perintah berikut satu per satu di terminal folder proyek Kalian:

```sh
# 1. Inisialisasi modul (jalankan sekali di awal)
go mod init be_latihan

# 2. Download framework Fiber
go get github.com/gofiber/fiber/v2

# 3. Download GORM (ORM Utama)
go get gorm.io/gorm

# 4. Download Driver PostgreSQL untuk GORM
go get gorm.io/driver/postgres

# 5. Download library untuk membaca file .env
go get github.com/joho/godotenv

# 6. Download library PostgreSQL (diperlukan untuk tipe data Array/pq)
go get github.com/lib/pq

# 7. Rapikan dependensi
go mod tidy
```

---

## 4. Konfigurasi Environment (`.env`)
Jangan pernah menulis password database langsung di dalam kode program. Buat file bernama `.env` di root folder:

Untuk mendapatkan SUPABASE_DSN, ikuti langkah berikut :
1. Buka **Project Settings** > **Cari Tombol Connect**.
![image](images/1774929656285-31e36e44-0493-4d29-b9fb-9a25749a81be.png)
2. Cari bagian **Direct Connection String** > **Connection Method** pilih **Session pooler** > pilih tab **URI** dan copy connection string tersebut.
![image](images/1774931155659-b7d33d03-e599-4989-a76e-f93e3b8bf56d.png)
3. Copy string tersebut (Format: `postgresql://postgres:[YOUR-PASSWORD]@db.xxxx.supabase.co:5432/postgres`).
4. Paste pada file .env yang sudah dibuat sebelumnya. Kemudian ganti `[YOUR-PASSWORD]` dengan password yang kalian buat di tahap 2 saat membuat database di Supabase.
5. Sehingga nanti di file env akan seperti ini : 

```env
SUPABASE_DSN=postgresql://postgres:[YOUR-PASSWORD]@db.xxxx.supabase.co:5432/postgres
```
---

## 5. Struktur Folder Modular
Pastikan folder proyek Kalian tersusun seperti berikut untuk menjaga kerapihan kode:

```text
/be_latihan/
├── main.go
├── .env
├── config/
│   └── database.go
├── model/
│   └── mahasiswa.go
├── repository/
│   └── mahasiswa_repository.go
└──repository_test/
    └── mahasiswa_test.go
```

---

## 6. Implementasi Kode

### A. Konfigurasi Database (`config/database.go`)
Fungsi untuk membuka jalur komunikasi ke Supabase.

```go
package config

import (
	"fmt"
	"log"
	"os"

	"github.com/joho/godotenv"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

var DB *gorm.DB

func InitDB() {
	// coba beberapa kemungkinan lokasi .env
	err := godotenv.Load(".env")
	if err != nil {
		err = godotenv.Load("../.env")
		if err != nil {
			log.Println("⚠️ .env tidak ditemukan di current maupun parent directory")
		}
	}

	dsn := os.Getenv("SUPABASE_DSN")
	if dsn == "" {
		log.Fatal("SUPABASE_DSN tidak ditemukan. Pastikan .env berisi DSN Supabase")
	}

	db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
	if err != nil {
		log.Fatalf("Gagal konek ke database: %v", err)
	}

	DB = db
	fmt.Println("✅ Koneksi ke PostgreSQL (Supabase) BERHASIL")
}

func GetDB() *gorm.DB {
	if DB == nil {
		log.Fatal("DB belum diinisialisasi. Panggil config.InitDB() lebih dulu.")
	}
	return DB
}
```
>   * **InitDB**: Mengambil alamat DSN dari `.env` dan membuka koneksi ke database.

---

### B. Model (`model/mahasiswa.go`)
Mendefinisikan bentuk data yang dikirim dan diterima. Kita menggunakan tag `json` untuk API dan tag `gorm` untuk database.

```go
package model

import "github.com/lib/pq"

type Mahasiswa struct {
	NPM    string         `json:"npm"    gorm:"column:npm;primaryKey;type:varchar(20);not null"`
	Nama   string         `json:"nama"   gorm:"column:nama;type:varchar(100);not null"`
	Prodi  string         `json:"prodi"  gorm:"column:prodi;type:varchar(100);not null"`
	Alamat string         `json:"alamat" gorm:"column:alamat;type:varchar(200)"`
	Email  string         `json:"email"  gorm:"column:email;type:varchar(100)"`
	Hobi   pq.StringArray `json:"hobi"   gorm:"column:hobi;type:text[]"`
}

func (Mahasiswa) TableName() string { return "mahasiswa" }
```

>   * **Struct**: Menentukan kolom apa saja yang ada (NPM, Nama, dll).
>   * **Tag JSON**: Digunakan saat menerima/mengirim data ke Frontend.
>   * **Tag GORM**: Instruksi untuk database (contoh: `primaryKey` agar NPM tidak 

---

### C. Repository (CRUD) (`repository/mahasiswa_repository.go`)
Memisahkan logika akses data dari logika aplikasi.

```go
package repository

import (
	"be_latihan/config"
	"be_latihan/model"
)

// Ambil semua data mahasiswa
func GetAllMahasiswa() ([]model.Mahasiswa, error) {
	var data []model.Mahasiswa
	result := config.GetDB().Find(&data)
	return data, result.Error

}

// Insert mahasiswa baru
func InsertMahasiswa(mhs *model.Mahasiswa) (*model.Mahasiswa, error) {
	result := config.GetDB().Create(mhs)
	return mhs, result.Error
}

// Ambil satu data mahasiswa berdasarkan NPM
func GetMahasiswaByNPM(npm int64) (model.Mahasiswa, error) {
	var mhs model.Mahasiswa
	result := config.GetDB().First(&mhs, "npm = ?", npm)
	return mhs, result.Error
}

// Update data mahasiswa berdasarkan NPM
func UpdateMahasiswa(npm int64, newData *model.Mahasiswa) (*model.Mahasiswa, error) {
	var mhs model.Mahasiswa

	db := config.GetDB()

	if err := db.First(&mhs, "npm = ?", npm).Error; err != nil {
		return nil, err
	}

	if err := db.Model(&mhs).Updates(newData).Error; err != nil {
		return nil, err
	}

	return &mhs, nil
}

// Hapus data mahasiswa berdasarkan NPM
func DeleteMahasiswa(npm int64) error {
	result := config.GetDB().Where("npm = ?", npm).Delete(&model.Mahasiswa{})
	return result.Error
}
```
>   * **Find/First**: Perintah untuk mencari data.
>   * **Create/Save**: Perintah untuk menyimpan data baru atau memperbarui data lama.
>   * **Delete**: Perintah untuk menghapus baris data berdasarkan NPM.
>   * Menggunakan pointer (* atau &) → lebih efisien

---

### D. Unit Testing Repository (`repository_test/mahasiswa_test.go`)
#### Tujuan:
* Menguji CRUD langsung ke database
* Tanpa HTTP / API

### 🔧 Setup Test

```go
func setupTest(t *testing.T) {
	config.InitDB()

	// Auto migrate biar tabel pasti ada
	err := config.GetDB().AutoMigrate(&model.Mahasiswa{})
	if err != nil {
		t.Fatalf("Migration failed: %v", err)
	}
}
```
---

### 🧪 Test Insert

```go
func TestInsertMahasiswa(t *testing.T) {
	setupTest(t)

	npm := time.Now().UnixNano()

	mhs := model.Mahasiswa{
		NPM:    npm,
		Nama:   "Test User",
		Prodi:  "Informatika",
		Alamat: "Bandung",
		Hobi:   []string{"Coding"},
	}

	_, err := repository.InsertMahasiswa(&mhs)
	if err != nil {
		t.Errorf("Insert failed: %v", err)
	}
	fmt.Printf("INSERTED NPM: %d\n", npm)
}
```

> * NPM menggunakan timestamp nanosecond (untuk menghindari duplicate).
> * Memanggil function InsertMahasiswa dari repository.

---

### 🧪 Test Get All

```go
func TestGetAllMahasiswa(t *testing.T) {
	setupTest(t)

	data, err := repository.GetAllMahasiswa()
	if err != nil {
		t.Errorf("GetAll failed: %v", err)
	}

	if len(data) == 0 {
		t.Errorf("Expected data, got empty")
	}
	fmt.Printf("DATA DI TABLE: %+v\n", data)
}
```

---

### 🧪 Test Get By NPM

```go
func TestGetMahasiswaByNPM(t *testing.T) {
	setupTest(t)

	npm := int64(1775410857780493700) // Gunakan NPM yang ada di DB untuk test (disesuaikan)

	mhs, err := repository.GetMahasiswaByNPM(npm)
	if err != nil {
		t.Errorf("GetByNPM failed: %v", err)
	}

	if mhs.NPM != npm {
		t.Errorf("Expected %d, got %d", npm, mhs.NPM)
	}
	fmt.Printf("DATA DITEMUKAN: %+v\n", mhs)
}
```

> Validasi:
> * Data ditemukan
> * NPM sesuai

---

### 🧪 Test Update

```go
func TestUpdateMahasiswa(t *testing.T) {
	setupTest(t)

	npm := int64(1775410857780493700) // Gunakan NPM yang ada di DB untuk test (disesuaikan)

	_, err := repository.UpdateMahasiswa(npm, &model.Mahasiswa{
		Nama:   "New Namez",
		Prodi:  "SI",
		Alamat: "Jakarta",
		Hobi:   []string{"Gaming"},
	})

	if err != nil {
		t.Errorf("Update failed: %v", err)
	}
}
```

---

### 🧪 Test Delete

```go
func TestDeleteMahasiswa(t *testing.T) {
	setupTest(t)

	npm := int64(1775410857780493700) // Gunakan NPM yang ada di DB untuk test (disesuaikan)

	err := repository.DeleteMahasiswa(npm)
	if err != nil {
		t.Errorf("Delete failed: %v", err)
	}
}
```

---
Konsep Penting:
### 🔹 1. `t *testing.T`

`t` adalah object dari package testing yang digunakan untuk:
* logging
* error handling
* kontrol jalannya test”

### 🔹 2. Perbedaan `Errorf` vs `Fatalf`

* `Errorf` → test gagal tapi lanjut
* `Fatalf` → test langsung berhenti

### 🔹 3. Slice (`[]Mahasiswa`)

* Digunakan untuk menampung banyak data, berbeda dengan array karena ukurannya dinamis.
---

## 7. Entry Point (`main.go`)
Menggabungkan semua komponen dan menjalankan fitur **Auto-Migration** untuk membuat tabel otomatis di database.

```go
package main

import (
	"be_latihan/config"
	"be_latihan/model"

	"github.com/gofiber/fiber/v2"
)

func main() {
	app := fiber.New()

	config.InitDB()
	// AutoMigrate membuat tabel berdasarkan Struct secara otomatis
	config.GetDB().AutoMigrate(&model.Mahasiswa{})

	app.Listen(":3000")
}
```
>   * **AutoMigrate**: Secara otomatis membuat tabel di Supabase berdasarkan struktur yang ada di folder **Model**. Kalian tidak perlu membuat tabel secara manual di dashboard database.

---

## 8. Menjalankan & Testing
1.  **Run**: Jalankan perintah `go run main.go`.
2.  **Cek Log**: Pastikan muncul pesan "✅ Koneksi ke PostgreSQL BERHASIL".
3.  **TESTING**:

		```bash
		go test -run ^NAMAFUNCTION$
		```
---

## 9. Tugas Praktikum
* **Modifikasi**: Tambahkan kolom `No HP` pada `struct Mahasiswa` dan jalankan ulang aplikasi lakukan CRUD (Screenshoot).
* **SERTIFIKAT**: Kumpulkan minimal 1 sertifikat kompetensi Golang dari platform yang tersedia di materi.

	### SERTIFIKAT
	* Sertakan sertifikat dari sertifikasi berikut (Minimal 1 sertifikat): 
		* https://www.mygreatlearning.com/academy/learn-for-free/courses/go-programming-language 
		* https://www.mindluster.com/certificate/3394
		* https://www.codecademy.com/learn/learn-go
		* https://www.coursera.org/specializations/google-golang

	### KETENTUAN REPO GITHUB 
	* Repo Wajib Bernama : **PemrogramanIII_NPM**
	* Untuk Praktikum hari ini buat direktori baru bernama **Pertemuan04**, kemudian di dalam folder **Pertemuan04** buat folder **Tugas** yang berisi file project **be_latihan**

	> ### Yang dikumpulkan :
	> * Link Github Kalian.
	> * Screenshoot dari CRUD setelah penambahan kolom Email
	> * Minimal 1 sertifikat kompetensi Golang (Boleh menggunakan link lain selain yang ada di atas)
	> * File-file di atas dikumpulkan dalam 1 file zip.