# Golang Project Assignment

| Nama              | NRP        | Kelas  |
|-------------------|------------|--------|
| Wan Sabrina Mayzura | 5025211023 | PBKK D |

## Daftar Isi
1. [Deskripsi Proyek](#deskripsi-proyek)
2. [Penjelasan main.go](#penjelasan-main-go)
    - [Imports](#imports)
    - [Konstanta dan Variabel](#konstanta-dan-variabel)
    - [Struct UserData](#struct-userdata)
    - [Fungsi Utama (main)](#fungsi-utama-main)
    - [Fungsi Pendukung](#fungsi-pendukung)
3. [Cara Memisahkan Package helper](#cara-memisahkan-package-helper)
4. [Cara Menjalankan](#cara-menjalankan)

---

## Deskripsi Proyek

Proyek ini adalah aplikasi sederhana untuk pemesanan tiket konferensi yang dibangun menggunakan bahasa Go (Golang). Aplikasi ini menangani pemesanan tiket, validasi input pengguna, operasi secara bersamaan (concurrency), dan pengiriman tiket secara asinkron.


## Penjelasan main.go

### Imports

```go
import (
	"booking-app/helper"
	"fmt"
	"sync"
	"time"
)
```
- **"booking-app/helper"**: Mengimpor fungsi bantuan dari package `helper`.
- **"fmt"**: Menyediakan fungsi input-output standar.
- **"sync"**: Mengelola operasi bersamaan dengan `WaitGroup` untuk sinkronisasi.
- **"time"**: Digunakan untuk simulasi delay saat mengirim tiket.

### Konstanta dan Variabel

```go
const conferenceTickets int = 50
var remainingTickets uint = 50
var conferenceName = "Go Conference"
var bookings = make([]UserData, 0)
var wg sync.WaitGroup
```
- **`conferenceTickets`**: Menyimpan total tiket yang tersedia.
- **`remainingTickets`**: Melacak sisa tiket yang tersedia (awal sama dengan `conferenceTickets`).
- **`conferenceName`**: Menyimpan nama konferensi.
- **`bookings`**: Slice dari struct `UserData` untuk menyimpan data pemesanan.
- **`wg`**: Variabel `WaitGroup` untuk mengelola goroutine.

### Struct UserData

```go
type UserData struct {
	firstName       string
	lastName        string
	email           string
	numberOfTickets uint
}
```
- **Struct `UserData`**: Mendefinisikan informasi pemesanan pengguna, termasuk nama depan, nama belakang, email, dan jumlah tiket yang dipesan.

### Fungsi Utama (main)

```go
func main() { /* ... */ }
```

- `greetUsers()`: Menampilkan pesan selamat datang dan informasi konferensi.
- `getUserInput()`: Meminta pengguna memasukkan detail pribadi dan jumlah tiket yang ingin dipesan.
- `ValidateUserInput()`: Memvalidasi input pengguna dengan memanggil fungsi dari package `helper`.
- **Konfirmasi Pemesanan**: Jika validasi berhasil, maka:
  - Memanggil `bookTicket()` untuk memperbarui data pemesanan.
  - ```wg.Add(1)``` Untuk memulai goroutine baru dengan `sendTicket()` untuk pengiriman tiket secara asinkron.
  - Menampilkan nama depan pengguna yang sudah memesan.
  - Memeriksa apakah tiket sudah habis.
- Penanganan Error: Jika validasi gagal, menampilkan pesan error yang sesuai.
- `wg.Wait()`: Menunggu semua goroutine selesai sebelum keluar dari program.

### Fungsi Pendukung

#### printFirstNames()

```go
func printFirstNames() []string {
	firstNames := []string{}

	for _, booking := range bookings {
		firstNames = append(firstNames, booking.firstName)
	}
	return firstNames
}
```
- `firstNames := []string{}`: Inisialisasi slice untuk menyimpan nama depan pengguna.
- `for _, booking := range bookings`: Iterasi melalui semua pemesanan yang ada.
- `firstNames = append(firstNames, booking.firstName)`: Menambahkan nama depan ke slice `firstNames`.
- `return firstNames`: Mengembalikan slice yang berisi nama depan pengguna yang telah memesan.

#### getUserInput()

```go
func getUserInput() (string, string, string, uint) {
	var firstName string
	var lastName string
	var email string
	var userTickets uint

	fmt.Println("Enter Your First Name: ")
	fmt.Scanln(&firstName)

	fmt.Println("Enter Your Last Name: ")
	fmt.Scanln(&lastName)

	fmt.Println("Enter Your Email: ")
	fmt.Scanln(&email)

	fmt.Println("Enter number of tickets: ")
	fmt.Scanln(&userTickets)

	return firstName, lastName, email, userTickets
}
```
- `var firstName, lastName, email, userTickets`: Mendeklarasikan variabel untuk menyimpan input pengguna.
- `fmt.Scanln(&firstName)`: Menerima input untuk nama depan pengguna.
- `fmt.Scanln(&lastName)`: Menerima input untuk nama belakang pengguna.
- `fmt.Scanln(&email)`: Menerima input untuk email pengguna.
- `fmt.Scanln(&userTickets)`: Menerima input untuk jumlah tiket yang dipesan.
- `return`: Mengembalikan input pengguna.

#### greetUsers()

```go
func greetUsers() {
	fmt.Printf("Welcome to %v booking application.\n", conferenceName)
	fmt.Printf("We have total of %v tickets and %v are still available.\n", conferenceTickets, remainingTickets)
	fmt.Println("Get your tickets here to attend")
}
```
- `fmt.Printf`: Menampilkan pesan selamat datang, total tiket, dan sisa tiket yang tersedia.
- `fmt.Println`: Menginstruksikan pengguna untuk memesan tiket.

#### bookTicket()

```go
func bookTicket(userTickets uint, firstName string, lastName string, email string) {
	remainingTickets = remainingTickets - userTickets

	var userData = UserData{
		firstName:       firstName,
		lastName:        lastName,
		email:           email,
		numberOfTickets: userTickets,
	}

	bookings = append(bookings, userData)

	fmt.Printf("Thank you %v %v for booking %v tickets.\n", firstName, lastName, userTickets)
	fmt.Printf("%v tickets remaining for %v\n", remainingTickets, conferenceName)
}
```
- `remainingTickets = remainingTickets - userTickets`: Mengurangi sisa tiket sesuai jumlah yang dipesan.
- `userData := UserData{...}`: Membuat instance dari struct `UserData` dengan data pengguna.
- `bookings = append(bookings, userData)`: Menambahkan data pengguna ke slice `bookings`.
- `fmt.Printf`: Menampilkan pesan konfirmasi pemesanan dan sisa tiket.

#### sendTicket()

```go
func sendTicket(userTickets uint, firstName string, lastName string, email string) {
	time.Sleep(10 * time.Second)
	var ticket = fmt.Sprintf("%v tickets for %v %v", userTickets, firstName, lastName)
	fmt.Println("-------------------")
	fmt.Printf("Sending ticket:\n %v \nto email address %v\n", ticket, email)
	fmt.Println("-------------------")
	wg.Done()
}
```
- `time.Sleep(10 * time.Second)`: Menunda eksekusi selama 10 detik untuk mensimulasikan pengiriman tiket.
- `fmt.Sprintf`: Membuat string yang berisi detail tiket.
- `fmt.Printf`: Menampilkan detail tiket yang dikirim.
- `wg.Done()`: Menandakan goroutine selesai.

#### ValidateUserInput()

```go
func ValidateUserInput(firstName string, lastName string, email string, userTickets uint, remainingTickets uint) (bool, bool, bool) {
	isValidName := len(firstName) >= 2 && len(lastName) >= 2
	isValidEmail := strings.Contains(email, "@")
	isValidTicketNumber := userTickets > 0 && userTickets <= remainingTickets
	return isValidName, isValidEmail, isValidTicketNumber
}
```
- `len(firstName) >= 2 && len(lastName) >= 2`: Mengecek apakah nama depan dan nama belakang memenuhi syarat panjang minimal.
- `strings.Contains(email, "@")`: Mengecek apakah email mengandung karakter "@".
- `userTickets > 0 && userTickets <= remainingTickets`: Memastikan jumlah tiket yang dipesan valid.
- `return`: Mengembalikan hasil validasi dalam bentuk boolean.

## Cara Memisahkan Package helper

1. Buat direktori `helper` di dalam proyek.
2. Buat file `helper.go` di dalam direktori tersebut.
3. Tambahkan fungsi yang ingin dipisahkan (seperti `ValidateUserInput()`) ke dalam file ini.
4. Pastikan menggunakan `package helper` di bagian atas file.
5. Impor package ini di file utama (`main.go`) dengan menambahkan:
   ```go
   import "booking-app/helper"
   ```

## Cara Menjalankan

1. Pastikan sudah menginstal Go (versi 1.16 atau lebih baru).
2. Clone repositori ini:
   ```bash
   git clone https://github.com/wansabrina/Go-Project.git
   ```
3. Masuk ke direktori proyek:
   ```bash
   cd Go-Project
   ```
4. Jalankan aplikasi:
   ```bash
   go run .
   ```