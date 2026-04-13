# Lanjutan Praktikum: Layer Handler, Router, dan Response API

Setelah sebelumnya kita membuat **repository (akses database)**, sekarang kita naik satu level lagi yaitu:

➡️ **Handler → untuk menerima request dari user (HTTP/API)**
➡️ **Router → untuk mengatur endpoint URL**
➡️ **Response → format standar output API**

Jadi alurnya nanti seperti ini:

```
Client (Postman / Frontend)
        ↓
     Router
        ↓
     Handler
        ↓
    Repository
        ↓
     Database
```

---

## A. Response Model (`model/response.go`)

```go
package model

type Response struct {
	Message string      `json:"message"`
	Data    interface{} `json:"data,omitempty"`
	Error   string      `json:"error,omitempty"`
}
```

### Penjelasan:

Kode di atas adalah **template response API** supaya semua output kita rapi dan konsisten.

* `Message` → pesan ke user (contoh: berhasil / gagal)
* `Data` → isi datanya (opsional)
* `Error` → pesan error (opsional)

`omitempty` artinya:
> kalau kosong → tidak akan ditampilkan di JSON

---

## B. Handler (`handler/mahasiswa_handler.go`)

Handler ini adalah **jembatan antara HTTP request dan repository**.

---

### 1. Get All Mahasiswa

```go
func GetAllMahasiswa(c *fiber.Ctx) error {
	data, err := repository.GetAllMahasiswa()
	if err != nil {
		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal mengambil data mahasiswa",
			Error:   err.Error(),
		})
	}

	return c.JSON(model.Response{
		Message: "berhasil mengambil data mahasiswa",
		Data:    data,
	})
}
```

### Penjelasan:

* Memanggil fungsi repository
* Kalau error → kirim status **500**
* Kalau berhasil → kirim data

---

### 2. Get Mahasiswa by NPM

```go
func GetMahasiswaByNPM(c *fiber.Ctx) error {
	npm, err := strconv.ParseInt(c.Params("npm"), 10, 64)
	if err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "npm tidak valid",
			Error:   err.Error(),
		})
	}

	mhs, err := repository.GetMahasiswaByNPM(npm)
	if err != nil {
		if err == gorm.ErrRecordNotFound {
			return c.Status(fiber.StatusNotFound).JSON(model.Response{
				Message: "data mahasiswa tidak ditemukan",
			})
		}

		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal mengambil data mahasiswa",
			Error:   err.Error(),
		})
	}

	return c.JSON(model.Response{
		Message: "berhasil mengambil data mahasiswa",
		Data:    mhs,
	})
}
```
### Penjelasan:

```go
npm, err := strconv.ParseInt(c.Params("npm"), 10, 64)
```

* `c.Params("npm")` → ambil dari URL
* Karena string → diubah ke `int64`

---

```go
if err == gorm.ErrRecordNotFound
```

* Kalau data tidak ada → kirim **404 Not Found**

---

### 3. Insert Mahasiswa

```go
func InsertMahasiswa(c *fiber.Ctx) error {
	var payload model.Mahasiswa
	if err := c.BodyParser(&payload); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "payload tidak valid",
			Error:   err.Error(),
		})
	}

	data, err := repository.InsertMahasiswa(&payload)
	if err != nil {
		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal menambahkan data mahasiswa",
			Error:   err.Error(),
		})
	}

	return c.Status(fiber.StatusCreated).JSON(model.Response{
		Message: "berhasil menambahkan data mahasiswa",
		Data:    data,
	})
}
```
### Penjelasan:
```go
if err := c.BodyParser(&payload); err != nil
```

* Mengambil JSON dari body request
* Diubah jadi struct Go

Contoh JSON:

```json
{
  "npm": "123",
  "nama": "Uki",
  "prodi": "D4 Teknik Informatika"
}
```

---

### 4. Update Mahasiswa
```go
func UpdateMahasiswa(c *fiber.Ctx) error {
	npm, err := strconv.ParseInt(c.Params("npm"), 10, 64)
	if err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "npm tidak valid",
			Error:   err.Error(),
		})
	}

	var payload model.Mahasiswa
	if err := c.BodyParser(&payload); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "payload tidak valid",
			Error:   err.Error(),
		})
	}

	data, err := repository.UpdateMahasiswa(npm, &payload)
	if err != nil {
		if err == gorm.ErrRecordNotFound {
			return c.Status(fiber.StatusNotFound).JSON(model.Response{
				Message: "data mahasiswa tidak ditemukan",
			})
		}

		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal mengubah data mahasiswa",
			Error:   err.Error(),
		})
	}

	return c.JSON(model.Response{
		Message: "berhasil mengubah data mahasiswa",
		Data:    data,
	})
}
```

Alurnya:

1. Ambil NPM dari URL
2. Ambil data baru dari body
3. Kirim ke repository

Jika data tidak ada → 404
Jika error → 500

---

### 5. Delete Mahasiswa

```go
func DeleteMahasiswa(c *fiber.Ctx) error {
	npm, err := strconv.ParseInt(c.Params("npm"), 10, 64)
	if err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "npm tidak valid",
			Error:   err.Error(),
		})
	}

	if err := repository.DeleteMahasiswa(npm); err != nil {
		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal menghapus data mahasiswa",
			Error:   err.Error(),
		})
	}

	return c.JSON(model.Response{
		Message: "berhasil menghapus data mahasiswa",
	})
}
```
### Penjelasan:
```go
repository.DeleteMahasiswa(npm)
```

* Hanya butuh NPM
* Tidak perlu body

---

## C. Router (`router/router.go`)

```go
package router

import (
	"be_latihan/handler"
	"be_latihan/model"

	"github.com/gofiber/fiber/v2"
)

func SetupRoutes(app *fiber.App) {
	app.Get("/", func(c *fiber.Ctx) error {
		return c.JSON(model.Response{
			Message: "API be_latihan aktif",
		})
	})

	mahasiswa := app.Group("/api/mahasiswa")
	mahasiswa.Get("/", handler.GetAllMahasiswa)
	mahasiswa.Get("/:npm", handler.GetMahasiswaByNPM)
	mahasiswa.Post("/", handler.InsertMahasiswa)
	mahasiswa.Put("/:npm", handler.UpdateMahasiswa)
	mahasiswa.Delete("/:npm", handler.DeleteMahasiswa)
}
```

 **Penjelasan Kode:**
 * **app.Group("/api/mahasiswa")**: Digunakan untuk mengelompokkan semua alamat API agar diawali dengan `/api/mahasiswa` (contoh: `localhost:3000/api/mahasiswa`). Ini mempermudah pengaturan versi API dikemudian hari.
 * **api.Method(path, handler)**: Menghubungkan metode HTTP (GET/POST/PUT/DELETE) dan alamat URL dengan fungsi yang sudah dibuat di folder **Handler**.

---

### Endpoint lengkap:

| Method | Endpoint              | Fungsi       |
| ------ | --------------------- | ------------ |
| GET    | `/api/mahasiswa`      | Ambil semua  |
| GET    | `/api/mahasiswa/:npm` | Ambil 1 data |
| POST   | `/api/mahasiswa`      | Insert       |
| PUT    | `/api/mahasiswa/:npm` | Update       |
| DELETE | `/api/mahasiswa/:npm` | Delete       |

---

## D. Main (`main.go`)
Tambahkan di file main.go kalian kode berikut


```go
app.Use(logger.New())
```

Baris kode tersebut berfungsi untuk mendaftarkan Logger Middleware ke dalam aplikasi. Berikut adalah poin-poin utama fungsinya:

* Pencatatan Aktivitas HTTP: Middleware ini secara otomatis mencatat setiap permintaan (request) yang masuk ke server dan tanggapan (response) yang diberikan oleh API.
* Monitoring Real-time: Memungkinkan pengembang memantau lalu lintas API secara langsung melalui terminal/konsol selama masa pengembangan (development).
* Metadata Penting: Data yang ditampilkan biasanya mencakup informasi krusial seperti:
 	* HTTP Method: (GET, POST, PUT, DELETE).
 	* Status Code: (200 OK, 404 Not Found, 500 Internal Server Error).
 	* Latency: Berapa lama waktu yang dibutuhkan server untuk memproses permintaan tersebut.
 	* Client IP: Alamat IP dari pengakses API.
 	* Path: Endpoint atau URL yang diakses.

---
dan kode berikut :

```go
router.SetupRoutes(app)
```

Fungsinya mengaktifkan semua endpoint

---

Sehingga kode lengkapnya seperti berikut :

```go
package main

import (
	"be_latihan/config"
	"be_latihan/model"
	"be_latihan/router"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/logger"
)

func main() {
	app := fiber.New()
	app.Use(logger.New())

	config.InitDB()
	config.GetDB().AutoMigrate(&model.Mahasiswa{})
	router.SetupRoutes(app)

	app.Listen(":3000")
}

```

---

## Contoh Alur Request

Misalnya kalian pakai Postman:

### Request:

```
GET /api/mahasiswa
```

### Flow:

1. Router menerima request
2. Dikirim ke `GetAllMahasiswa`
3. Handler panggil repository
4. Repository ambil dari DB
5. Balik ke handler
6. Dikirim ke client dalam format JSON

---

## Kesimpulan

Setelah bagian ini, project kamu sudah punya:

✅ Database (Supabase + GORM)
✅ Repository (CRUD logic)
✅ Handler (HTTP logic)
✅ Router (endpoint API)
✅ Response standar

Artinya kalian sudah bikin **REST API lengkap**

## Menjalankan & Testing
1.  **Run**: Jalankan perintah `go run main.go`.
2.  **Cek Log**: Pastikan muncul pesan "✅ Koneksi ke PostgreSQL BERHASIL".
3.  **Postman**:
    * **POST** ke `http://localhost:3000/api/mahasiswa` dengan Body JSON data mahasiswa.
	 * Body JSON :
		```JSON
		{
			"npm": 123456778899,
			"nama": "Budi Santoso",
			"prodi": "D4 Teknik Informatika",
			"alamat": "Jl. Raya Kampus No. 123, Bandung",
			"email": "budi.santoso@email.com",
			"hobi": ["Coding", "Berenang", "Membaca"]
		}
		```
    * **GET** ke `http://localhost:3000/mahasiswa` untuk melihat hasilnya.
4.  **Verifikasi**: Buka Dashboard Supabase untuk melihat data yang tersimpan di tabel secara visual.

---
