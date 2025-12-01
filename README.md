# 🤖 Kontrol Gerak Simultan Robot Arm & Conveyor dengan FreeRTOS pada ESP32-S3

---

Proyek ini merancang sistem kendali lengan robot dan conveyor yang dapat bekerja secara paralel dengan memanfaatkan dual-core ESP32-S3 dan fitur multitasking FreeRTOS. Seluruh komponen dikendalikan real-time tanpa saling mengganggu sehingga pergerakan lebih stabil dan respon sensor lebih cepat.

Fokus utama proyek ini adalah penerapan **Semaphore**, **Queue**, **Mutex**, dan **Interrupt (ISR)** sebagai fondasi komunikasi antar-task yang aman dan efisien.

---

## 🎯 Tujuan Proyek

1. Mengatur robot arm dan conveyor berjalan paralel pada dua core berbeda.
2. Menangani sinyal sensor secara cepat menggunakan ISR + Binary Semaphore.
3. Mengirim nilai kecepatan conveyor melalui Queue untuk mencegah race condition.
4. Mengamankan akses OLED menggunakan Mutex.
5. Menjaga sistem tetap stabil meskipun banyak task aktif simultan.

---

## ⚙️ Perangkat yang Digunakan

| No | Perangkat / Komponen     | Fungsi                     | Jenis  | Core Digunakan |
| -- | ------------------------ | -------------------------- | ------ | -------------- |
| 1  | ESP32-S3 DevKitC         | Mikrokontroler utama       | MCU    | Core 0 & 1     |
| 2  | Sensor IR / Limit Switch | Deteksi objek → memicu ISR | Input  | ISR → Core 1   |
| 3  | Stepper + Driver         | Menggerakkan conveyor      | Output | Core 0/1       |
| 4  | Servo Motor              | Menggerakkan robot arm     | Output | Core 1         |
| 5  | OLED SSD1306 (I2C)       | Menampilkan status sistem  | Output | Core 0         |

---

## 🧩 Komponen FreeRTOS

| Komponen         | Digunakan Untuk         | Alasan                           |
| ---------------- | ----------------------- | -------------------------------- |
| Binary Semaphore | Sensor → Robot Arm      | 1 sinyal = 1 aksi, no event loss |
| Queue            | Kecepatan conveyor      | Aman antar-core                  |
| Mutex            | Menjaga akses OLED/I2C  | Tidak terjadi tabrakan penulisan |
| ISR              | Deteksi objek real-time | Eksekusi cepat, tanpa blocking   |

---

## 🦾 Fungsi Utama Sistem
###  1️⃣ Robot Arm Task – Core 1 

* Menunggu sinyal dari sensor lewat Binary Semaphore.
* Satu sinyal memicu satu siklus gerakan.
* Menggunakan portMAX_DELAY supaya event tidak terlewat.

### 2️⃣ Conveyor Task 

* Menerima nilai kecepatan melalui Queue.
* Data selalu utuh karena dikirim per paket.
* Tidak memakai polling, sehingga beban CPU lebih ringan.

### 3️⃣ OLED Task – Core 0 

* Menampilkan informasi kondisi robot dan conveyor.
* Akses dilindungi Mutex agar tidak rebutan.
* Timeout 100 ms untuk menghindari potensi deadlock.
  
###  4️⃣ ISR Sensor 

* Aktif saat mendeteksi sinyal FALLING edge.
* Hanya menjalankan xSemaphoreGiveFromISR() agar tetap cepat.
* Disimpan di IRAM untuk mempercepat eksekusi ISR.
  
---

## 📁 Struktur Proyek

```
/include/
  Header dan definisi task

/lib/
  Modul servo, sensor, stepper

/src/
  main.cpp  ← kode utama FreeRTOS

platformio.ini
README.md
```

---

Video Demo dapat diakses pada tautan berikut : 
https://drive.google.com/file/d/1TP4ffca7xjXOGnQ75S1rGTs7wljzyM2K/view

---
