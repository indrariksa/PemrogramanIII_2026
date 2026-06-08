# Praktikum Pertemuan 11

## Autentikasi JWT: Golang Fiber + GORM + React Router

Pada Praktikum 11 ini kita menambahkan **autentikasi JWT** agar endpoint mahasiswa hanya bisa diakses oleh user yang sudah login dan memiliki role `admin`.

# Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa mampu:

- Membuat fitur **register** dan **login** di backend Golang Fiber
- Menyimpan password dengan **bcrypt**
- Membuat dan memverifikasi token **JWT**
- Melindungi endpoint CRUD Mahasiswa dengan middleware JWT
- Mengirim token otomatis dari React menggunakan Axios interceptor
- Membuat halaman **Login**, **Register**, **PrivateRoute**, dan **Logout**
- Memahami perbedaan response `401 Unauthorized` dan `403 Forbidden`

---

# Alur Praktikum

```text
Register user
      |
      v
Password di-hash dengan bcrypt
      |
      v
Login username + password
      |
      v
Backend membuat JWT
      |
      v
React menyimpan token ke localStorage
      |
      v
Axios mengirim Authorization: Bearer <token>
      |
      v
Middleware backend mengecek token dan role admin
      |
      v
CRUD Mahasiswa bisa diakses
```

---

# Bagian 1 - Backend

## STEP 1 - Install Dependency JWT

Masuk ke folder backend:

```bash
go get github.com/golang-jwt/jwt/v5
```

Project ini sudah memiliki `golang.org/x/crypto`, sehingga bcrypt dapat langsung dipakai. Jika belum ada, jalankan:

```bash
go get golang.org/x/crypto/bcrypt
go mod tidy
```

---

## STEP 2 - Tambahkan JWT Secret di `.env`

Edit file `.env` backend:

```env
SUPABASE_DSN=postgresql://username:password@host:5432/postgres
JWT_SECRET=praktikum-jwt-secret-123456789
```

`JWT_SECRET` digunakan untuk menandatangani token. Pada project nyata, isi secret harus panjang dan tidak boleh dibagikan.

---

## STEP 3 - Buat Model User

File: `be_latihan/model/user.go`

```go
package model

type User struct {
	ID       string `json:"id" gorm:"column:id;type:uuid;default:gen_random_uuid();primaryKey"`
	Username string `json:"username" gorm:"column:username;type:varchar(50);uniqueIndex;not null"`
	Password string `json:"-" gorm:"column:password;type:varchar(255);not null"`
	Role     string `json:"role" gorm:"column:role;type:varchar(20);not null;default:admin"`
}

func (User) TableName() string { return "users" }

type AuthRequest struct {
	Username string `json:"username"`
	Password string `json:"password"`
	Role     string `json:"role"`
}

type AuthUserResponse struct {
	ID       string `json:"id"`
	Username string `json:"username"`
	Role     string `json:"role"`
}

type LoginResponse struct {
	Token string           `json:"token"`
	User  AuthUserResponse `json:"user"`
}
```

Penjelasan:

- `User` adalah tabel database untuk akun login.
- `ID` memakai UUID string agar cocok dengan kolom `id` Supabase/PostgreSQL yang biasanya bertipe `uuid`.
- `Password` memakai `json:"-"` agar hash password tidak ikut tampil di response.
- `AuthRequest` dipakai untuk body register dan login.
- `LoginResponse` berisi token JWT dan data user.

---

## STEP 4 - AutoMigrate Tabel User

File: `be_latihan/main.go`

Cari:

```go
config.GetDB().AutoMigrate(&model.Mahasiswa{})
```

Ubah menjadi:

```go
config.GetDB().AutoMigrate(&model.Mahasiswa{}, &model.User{})
```

Dengan ini GORM akan membuat tabel `users` jika belum ada.

Tambahkan juga `AllowHeaders` pada CORS agar browser boleh mengirim header `Authorization`:

```go
app.Use(cors.New(cors.Config{
	AllowOrigins: strings.Join(config.GetAllowedOrigins(), ","),
	AllowMethods: "GET,POST,PUT,DELETE,OPTIONS",
	AllowHeaders: "Origin,Content-Type,Accept,Authorization",
}))
```

---

## STEP 5 - Buat Helper Password

File: `be_latihan/pkg/password/hasher.go`

```go
package password

import "golang.org/x/crypto/bcrypt"

func HashPassword(plainPassword string) (string, error) {
	hashedPassword, err := bcrypt.GenerateFromPassword([]byte(plainPassword), 12)
	return string(hashedPassword), err
}

func CheckPasswordHash(plainPassword, hashedPassword string) bool {
	err := bcrypt.CompareHashAndPassword([]byte(hashedPassword), []byte(plainPassword))
	return err == nil
}
```

Penjelasan:

- `HashPassword` mengubah password biasa menjadi hash bcrypt.
- `CheckPasswordHash` membandingkan password login dengan hash di database.
- Password yang disimpan di database bukan password asli.

---

## STEP 6 - Buat Repository User

File: `be_latihan/repository/user_repository.go`

```go
package repository

import (
	"be_latihan/config"
	"be_latihan/model"
)

func FindUserByUsername(username string) (model.User, error) {
	var user model.User
	result := config.GetDB().First(&user, "username = ?", username)
	return user, result.Error
}

func InsertUser(user *model.User) (*model.User, error) {
	result := config.GetDB().Create(user)
	return user, result.Error
}
```

Penjelasan:

- `FindUserByUsername` dipakai saat login.
- `InsertUser` dipakai saat register.
- Validasi username unik dibantu oleh `uniqueIndex` di model.

---

## STEP 7 - Buat Middleware JWT

File: `be_latihan/config/middleware/jwt.go`

```go
package middleware

import (
	"be_latihan/model"
	"log"
	"os"
	"strings"
	"time"

	"github.com/gofiber/fiber/v2"
	"github.com/golang-jwt/jwt/v5"
)

type JWTClaims struct {
	Username string `json:"username"`
	Role     string `json:"role"`
	jwt.RegisteredClaims
}

func getJWTSecret() []byte {
	secret := os.Getenv("JWT_SECRET")
	if secret == "" {
		log.Println("⚠️ JWT_SECRET tidak ditemukan di .env")
	}
	return []byte(secret)
}

func GenerateJWT(user model.User, duration time.Duration) (string, error) {
	claims := JWTClaims{
		Username: user.Username,
		Role:     user.Role,
		RegisteredClaims: jwt.RegisteredClaims{
			Subject:   user.Username,
			IssuedAt:  jwt.NewNumericDate(time.Now()),
			ExpiresAt: jwt.NewNumericDate(time.Now().Add(duration)),
		},
	}

	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString(getJWTSecret())
}

func JWTProtected(requiredRole string) fiber.Handler {
	return func(c *fiber.Ctx) error {
		authHeader := c.Get("Authorization")
		if authHeader == "" {
			return c.Status(fiber.StatusUnauthorized).JSON(model.Response{
				Message: "authorization header wajib diisi",
			})
		}

		tokenString := strings.TrimPrefix(authHeader, "Bearer ")
		if tokenString == authHeader {
			return c.Status(fiber.StatusUnauthorized).JSON(model.Response{
				Message: "format token harus Bearer <token>",
			})
		}

		token, err := jwt.ParseWithClaims(tokenString, &JWTClaims{}, func(token *jwt.Token) (interface{}, error) {
			return getJWTSecret(), nil
		})
		if err != nil || !token.Valid {
			return c.Status(fiber.StatusUnauthorized).JSON(model.Response{
				Message: "token tidak valid atau sudah expired",
			})
		}

		claims, ok := token.Claims.(*JWTClaims)
		if !ok {
			return c.Status(fiber.StatusUnauthorized).JSON(model.Response{
				Message: "claims token tidak valid",
			})
		}

		if requiredRole != "" && claims.Role != requiredRole {
			return c.Status(fiber.StatusForbidden).JSON(model.Response{
				Message: "user tidak memiliki akses untuk fitur ini",
			})
		}

		c.Locals("username", claims.Username)
		c.Locals("role", claims.Role)
		return c.Next()
	}
}
```

Penjelasan:

- `GenerateJWT` membuat token yang berlaku selama durasi tertentu.
- `JWTProtected("admin")` hanya mengizinkan user dengan role `admin`.
- `401` artinya belum login/token salah.
- `403` artinya sudah login, tetapi role tidak boleh mengakses fitur.

---

## STEP 8 - Buat Handler Register dan Login

File: `be_latihan/handler/auth_handler.go`

```go
package handler

import (
	"be_latihan/config/middleware"
	"be_latihan/model"
	"be_latihan/pkg/password"
	"be_latihan/repository"
	"strings"
	"time"

	"github.com/gofiber/fiber/v2"
	"gorm.io/gorm"
)

func Register(c *fiber.Ctx) error {
	var payload model.AuthRequest
	if err := c.BodyParser(&payload); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "payload tidak valid",
			Error:   err.Error(),
		})
	}

	payload.Username = strings.TrimSpace(payload.Username)
	payload.Role = strings.TrimSpace(payload.Role)
	if payload.Role == "" {
		payload.Role = "admin"
	}

	if payload.Username == "" || payload.Password == "" {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "username dan password wajib diisi",
		})
	}

	hashedPassword, err := password.HashPassword(payload.Password)
	if err != nil {
		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal membuat hash password",
			Error:   err.Error(),
		})
	}

	user := model.User{
		Username: payload.Username,
		Password: hashedPassword,
		Role:     payload.Role,
	}

	data, err := repository.InsertUser(&user)
	if err != nil {
		return c.Status(fiber.StatusConflict).JSON(model.Response{
			Message: "username sudah digunakan atau data tidak valid",
			Error:   err.Error(),
		})
	}

	return c.Status(fiber.StatusCreated).JSON(model.Response{
		Message: "register berhasil",
		Data: model.AuthUserResponse{
			ID:       data.ID,
			Username: data.Username,
			Role:     data.Role,
		},
	})
}

func Login(c *fiber.Ctx) error {
	var payload model.AuthRequest
	if err := c.BodyParser(&payload); err != nil {
		return c.Status(fiber.StatusBadRequest).JSON(model.Response{
			Message: "payload tidak valid",
			Error:   err.Error(),
		})
	}

	user, err := repository.FindUserByUsername(strings.TrimSpace(payload.Username))
	if err != nil {
		if err == gorm.ErrRecordNotFound {
			return c.Status(fiber.StatusUnauthorized).JSON(model.Response{
				Message: "username atau password salah",
			})
		}

		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal mencari user",
			Error:   err.Error(),
		})
	}

	if !password.CheckPasswordHash(payload.Password, user.Password) {
		return c.Status(fiber.StatusUnauthorized).JSON(model.Response{
			Message: "username atau password salah",
		})
	}

	token, err := middleware.GenerateJWT(user, 2*time.Hour)
	if err != nil {
		return c.Status(fiber.StatusInternalServerError).JSON(model.Response{
			Message: "gagal membuat token",
			Error:   err.Error(),
		})
	}

	return c.JSON(model.Response{
		Message: "login berhasil",
		Data: model.LoginResponse{
			Token: token,
			User: model.AuthUserResponse{
				ID:       user.ID,
				Username: user.Username,
				Role:     user.Role,
			},
		},
	})
}
```

---

## STEP 9 - Tambahkan Route Auth dan Proteksi Route Mahasiswa

File: `be_latihan/router/router.go`

```go
package router

import (
	"be_latihan/config/middleware"
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

	app.Post("/register", handler.Register)
	app.Post("/login", handler.Login)

	mahasiswa := app.Group("/api/mahasiswa", middleware.JWTProtected("admin"))
	mahasiswa.Get("/", handler.GetAllMahasiswa)
	mahasiswa.Get("/:npm", handler.GetMahasiswaByNPM)
	mahasiswa.Post("/", handler.InsertMahasiswa)
	mahasiswa.Put("/:npm", handler.UpdateMahasiswa)
	mahasiswa.Delete("/:npm", handler.DeleteMahasiswa)
}
```

---

## STEP 10 - Testing Backend dengan Postman

Jalankan backend:

```bash
go run main.go
```

### A. Register Admin

Endpoint:

```text
POST http://127.0.0.1:3000/register
```

Body:

```json
{
  "username": "admin",
  "password": "admin123",
  "role": "admin"
}
```

### B. Register User Biasa

```json
{
  "username": "user",
  "password": "user123",
  "role": "user"
}
```

### C. Login Admin

Endpoint:

```text
POST http://127.0.0.1:3000/login
```

Body:

```json
{
  "username": "admin",
  "password": "admin123"
}
```

Response akan berisi token:

```json
{
  "message": "login berhasil",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": 1,
      "username": "admin",
      "role": "admin"
    }
  }
}
```

### D. Akses Mahasiswa Tanpa Token

```text
GET http://127.0.0.1:3000/api/mahasiswa
```

Hasil:

```json
{
  "message": "authorization header wajib diisi"
}
```

Status: `401 Unauthorized`

### E. Akses Mahasiswa dengan Token

Tambahkan header:

```text
Authorization: Bearer <token-admin>
```
Copy token dari login
![image](images/token.png)

Kemudian paste seperti gambar di bawah
![image](images/bearer-token.png)

Jika token valid dan role `admin`, data mahasiswa akan tampil.

![image](images/berhasil-get.png)

### F. Login Role User

Jika login memakai role `user`, lalu akses `/api/mahasiswa`, hasilnya:

```json
{
  "message": "user tidak memiliki akses untuk fitur ini"
}
```

Status: `403 Forbidden`
---


# Penutup

Pada praktikum ini kita sudah:

- Menambahkan autentikasi JWT di backend Golang Fiber.
- Menyimpan password dengan bcrypt.
- Membuat endpoint register dan login.
- Melindungi endpoint CRUD Mahasiswa berdasarkan role `admin`.
- Menambahkan login, register, private route, token interceptor, dan logout di React.

## Pengumpulan
- Push ke direktori Pertemuan11/Praktikum
- Screenshoot dari postman ketika register, login, dan data berhasil ditampilkan (Push ke direktori Pertemuan11/Praktikum/Hasil)