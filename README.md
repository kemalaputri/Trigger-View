# 🧋 Sistem Database Toko Minuman (MySQL)
![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)
![Database](https://img.shields.io/badge/Database-Relational-blue?style=for-the-badge)

> Sistem database untuk manajemen toko minuman menggunakan **MySQL**  

👨‍💻 Kelompok 2 Anggota :
- Kemala Putri Oktaviani (16)
- Muhammad Satria Nabil (21)
- Salatin Nibras Bama Kerti (32)

---

## 📌 Deskripsi Project

Database **toko_minuman** digunakan untuk:

- ✅ Mengelola data produk minuman
- ✅ Mengelola data pelanggan
- ✅ Mencatat transaksi pembelian
- ✅ Mengelola pembayaran
- ✅ Mencatat log aktivitas otomatis (trigger)
- ✅ Menyediakan laporan menggunakan VIEW

---

# 🗄️ 1. Setup Database

```sql
CREATE DATABASE toko_minuman;
USE toko_minuman;
```

---

# 📊 2. Struktur Tabel

---

## 🧋 Tabel `produk`

| Field        | Tipe Data       | Keterangan        |
|--------------|----------------|------------------|
| id_produk    | INT (PK, AI)   | ID produk        |
| nama_produk  | VARCHAR(100)   | Nama minuman     |
| harga        | INT            | Harga produk     |
| stok         | INT            | Stok tersedia    |

```sql
CREATE TABLE produk (
    id_produk INT AUTO_INCREMENT PRIMARY KEY,
    nama_produk VARCHAR(100),
    harga INT,
    stok INT
);
```

---

## 👤 Tabel `pelanggan`

| Field          | Tipe Data       | Keterangan        |
|----------------|----------------|------------------|
| id_pelanggan   | INT (PK, AI)   | ID pelanggan     |
| nama_pelanggan | VARCHAR(100)   | Nama pelanggan   |

```sql
CREATE TABLE pelanggan (
    id_pelanggan INT AUTO_INCREMENT PRIMARY KEY,
    nama_pelanggan VARCHAR(100)
);
```

---

## 🧾 Tabel `transaksi`

| Field          | Tipe Data       | Keterangan                  |
|----------------|----------------|----------------------------|
| id_transaksi   | INT (PK, AI)   | ID transaksi               |
| id_produk      | INT (FK)       | Relasi ke produk           |
| id_pelanggan   | INT (FK)       | Relasi ke pelanggan        |
| jumlah         | INT            | Jumlah beli                |
| total_harga    | INT            | Total harga otomatis       |

```sql
CREATE TABLE transaksi (
    id_transaksi INT AUTO_INCREMENT PRIMARY KEY,
    id_produk INT,
    id_pelanggan INT,
    jumlah INT,
    total_harga INT,
    FOREIGN KEY (id_produk) REFERENCES produk(id_produk),
    FOREIGN KEY (id_pelanggan) REFERENCES pelanggan(id_pelanggan)
);
```

---

## 💳 Tabel `pembayaran`

| Field         | Tipe Data       | Keterangan           |
|--------------|----------------|---------------------|
| id_bayar     | INT (PK, AI)   | ID pembayaran       |
| id_transaksi | INT (FK)       | Relasi transaksi    |
| jumlah_bayar | INT            | Jumlah bayar        |

```sql
CREATE TABLE pembayaran (
    id_bayar INT AUTO_INCREMENT PRIMARY KEY,
    id_transaksi INT,
    jumlah_bayar INT,
    FOREIGN KEY (id_transaksi) REFERENCES transaksi(id_transaksi)
);
```

---

## 📝 Tabel `log_aktivitas`

| Field       | Tipe Data       | Keterangan        |
|------------|----------------|------------------|
| id_log     | INT (PK, AI)   | ID log           |
| keterangan | VARCHAR(255)   | Aktivitas sistem |

```sql
CREATE TABLE log_aktivitas (
    id_log INT AUTO_INCREMENT PRIMARY KEY,
    keterangan VARCHAR(255)
);
```

---

# 📝 3. Insert Data

```sql
INSERT INTO produk (nama_produk, harga, stok) VALUES
('Es Kopi Susu', 18000, 20),
('Matcha Latte', 22000, 15),
('Thai Tea', 15000, 25),
('Cokelat Creamy', 20000, 10);

INSERT INTO pelanggan (nama_pelanggan) VALUES
('Kemala'),
('Bama'),
('Nabil');
```

---

# ⚙️ 4. Trigger (Otomatisasi Database)

---

## 🔹 Hitung Total Harga Otomatis

```sql
DELIMITER $$

CREATE TRIGGER trg_total_harga
BEFORE INSERT ON transaksi
FOR EACH ROW
SET NEW.total_harga = (
    SELECT harga * NEW.jumlah
    FROM produk
    WHERE id_produk = NEW.id_produk
);

$$
DELIMITER ;
```

---

## 🔹 Mengurangi Stok

```sql
DELIMITER $$

CREATE TRIGGER trg_kurangi_stok
AFTER INSERT ON transaksi
FOR EACH ROW
BEGIN
    UPDATE produk
    SET stok = stok - NEW.jumlah
    WHERE id_produk = NEW.id_produk;
END$$

DELIMITER ;
```

---

## 🔹 Log Transaksi

```sql
DELIMITER $$

CREATE TRIGGER trg_log_transaksi
AFTER INSERT ON transaksi
FOR EACH ROW
BEGIN
    INSERT INTO log_aktivitas(keterangan)
    VALUES (
        CONCAT('Transaksi baru: Produk ID ', NEW.id_produk,
        ', Pelanggan ID ', NEW.id_pelanggan,
        ', Jumlah ', NEW.jumlah,
        ', Total ', NEW.total_harga)
    );
END$$

DELIMITER ;
```

---

## 🔹 Log Stok Habis

```sql
DELIMITER $$

CREATE TRIGGER trg_stok_habis
AFTER UPDATE ON produk
FOR EACH ROW
BEGIN
    IF NEW.stok = 0 THEN
        INSERT INTO log_aktivitas(keterangan)
        VALUES (CONCAT('Stok habis untuk Produk ID ', NEW.id_produk));
    END IF;
END$$

DELIMITER ;
```

---

## 🔹 Mengembalikan Stok Saat Delete

```sql
DELIMITER $$

CREATE TRIGGER trg_kembalikan_stok
AFTER DELETE ON transaksi
FOR EACH ROW
BEGIN
    UPDATE produk
    SET stok = stok + OLD.jumlah
    WHERE id_produk = OLD.id_produk;
END$$

DELIMITER ;
```

---

## 🔹 Log Pembayaran

```sql
DELIMITER $$

CREATE TRIGGER trg_log_pembayaran
AFTER INSERT ON pembayaran
FOR EACH ROW
BEGIN
    INSERT INTO log_aktivitas(keterangan)
    VALUES (
        CONCAT('Pembayaran berhasil: ID Transaksi ',
        NEW.id_transaksi, ', Jumlah ', NEW.jumlah_bayar)
    );
END$$

DELIMITER ;
```

---

# 🧾 5. Insert Transaksi & Pembayaran

```sql
INSERT INTO transaksi (id_produk, id_pelanggan, jumlah) VALUES
(1,1,2),
(2,2,1),
(3,3,5);

INSERT INTO pembayaran (id_transaksi, jumlah_bayar) VALUES
(1,36000),
(2,22000),
(3,75000);
```

---

# 📊 6. VIEW (Laporan)

---

## 🔹 Detail Transaksi

```sql
CREATE VIEW view_transaksi_detail AS
SELECT t.id_transaksi,
       p.nama_produk,
       c.nama_pelanggan,
       t.jumlah,
       t.total_harga
FROM transaksi t
JOIN produk p ON t.id_produk = p.id_produk
JOIN pelanggan c ON t.id_pelanggan = c.id_pelanggan;
```

---

## 🔹 Stok Produk

```sql
CREATE VIEW view_stok_produk AS
SELECT id_produk, nama_produk, stok
FROM produk;
```

---

# 📈 7. Hasil Output

---

## 🔹 Data Transaksi

| id | produk | pelanggan | jumlah | total |
|----|--------|----------|--------|--------|
| 1 | 1 | 1 | 2 | 36000 |
| 2 | 2 | 2 | 1 | 22000 |
| 3 | 3 | 3 | 5 | 75000 |

---

## 🔹 Data Produk

| id | nama | harga | stok |
|----|------|-------|------|
| 1 | Es Kopi Susu | 18000 | 18 |
| 2 | Matcha Latte | 22000 | 14 |
| 3 | Thai Tea | 15000 | 20 |
| 4 | Cokelat Creamy | 20000 | 10 |

---

## 🔹 Log Aktivitas

| id | keterangan |
|----|------------|
| 1 | Transaksi baru... |
| 2 | Transaksi baru... |
| 3 | Transaksi baru... |
| 4 | Pembayaran berhasil... |

---

## 🔹 View Transaksi

| transaksi | produk | pelanggan | jumlah | total |
|----------|--------|-----------|--------|--------|
| 1 | Es Kopi Susu | Kemala | 2 | 36000 |
| 2 | Matcha Latte | Bama | 1 | 22000 |
| 3 | Thai Tea | Nabil | 5 | 75000 |

---

# 🎯 Kesimpulan

- Database sudah menggunakan **relasi antar tabel**
- Trigger membantu otomatisasi sistem
- View mempermudah laporan
- Sistem sudah siap digunakan untuk skala sederhana

---


# 👨‍💻 Dibuat Oleh
**© 2026 - Kelompok 2**
