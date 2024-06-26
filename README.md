# SISTEM-BISNIS-TOKO-BUAH

## Entity Relationship diagram Toko Buah
![WhatsApp Image 2024-06-24 at 23 22 08](https://github.com/Febytrinita/SISTEM-BISNIS-TOKO-BUAH/assets/168648613/6f4b058b-8bdb-48d5-8246-188b79bf9c76)

## Relasi
1. Pesanan dan pelanggan (one to many)
   Relasi ini menunjukkan bahwa satu pelanggan dapat melakukan banyak pesanan. Hal ini diimplementasikan dengan menghubungkan kolom id_pelanggan pada tabel Pelanggan dengan kolom id_pelanggan pada tabel Pesanan.
2. Pesanan dan Produk (many to many)
   Relasi ini menunjukkan bahwa satu pesanan dapat memuat banyak produk, dan satu produk dapat dipesan dalam banyak pesanan. Hal ini diimplementasikan dengan menggunakan tabel perantara ItemPesanan yang menghubungkan tabel Pesanan dan Produk.
3. Produk dan ItemPesanan (one to one)
   Relasi ini menunjukkan bahwa satu item pesanan hanya dapat memuat satu produk, dan satu produk hanya dapat dipesan dalam satu item pesanan. Hal ini diimplementasikan dengan menghubungkan kolom id_produk pada tabel Produk dengan kolom id_produk pada tabel ItemPesanan.
   
# Deskripsi Toko Buah

## Latar Belakang 
Pada jaman modern seperti sekarang banyak toko buah yang mulai menggunakan sistem informasi untuk mengelola bisnis mereka. Penggunaan teknologi informasi memungkinkan manajemen stok, transaksi penjualan, dan data pelanggan dilakukan dengan lebih efisien dan akurat. Sistem informasi yang baik dapat membantu toko buah dalam mengoptimalkan operasional, mengurangi kesalahan manusia, dan meningkatkan kepuasan pelanggan.

## Tujuan
Tujuan utama dari proyek ini adalah untuk merancang dan mengimplementasikan sistem basis data yang dapat mendukung operasional toko buah secara efisien. Sistem ini dirancang untuk:
Mengelola stok buah secara otomatis.
Mencatat transaksi penjualan secara terperinci. 
Menyediakan laporan penjualan yang komprehensif.
Mempermudah proses backup dan replikasi data untuk memastikan integritas dan ketersediaan data.

## Lingkup Proyek
Lingkup proyek ini mencakup:
Perancangan dan implementasi basis data untuk toko buah.
Implementasi proses trigger dan view untuk mengelola stok buah.
Penerapan operasi agregat dan indeks untuk meningkatkan performa query.
Penyusunan berbagai jenis query untuk mendukung analisis data dan operasional toko.
Strategi backup dengan metode dump.
Implementasi replikasi basis data untuk memastikan ketersediaan data.

## Sfesifikasi Teknis Toko Buah
Basis Data: MySQL
Bahasa Pemrograman: SQL untuk operasi basis data
Indexing: Indeks diterapkan pada kolom buah_id di tabel detail_transaksi untuk mempercepat query yang sering digunakan, seperti pencarian dan join.
View: View buah_tersedia digunakan untuk menyederhanakan query yang kompleks dengan menampilkan daftar buah yang masih tersedia untuk dijual.
Trigger: Trigger update_stok_trigger digunakan untuk otomatisasi pengurangan stok buah setiap kali terjadi transaksi penjualan.

## Skema Basis Data

CREATE DATABASE db_TokoBuah;
USE db_TokoBuah;

CREATE TABLE buah (
    buah_id INT PRIMARY KEY,
    nama_buah VARCHAR(100) NOT NULL,
    jenis_buah VARCHAR(50),
    harga DECIMAL(10, 2),
    stok INT
);

INSERT INTO buah (buah_id, nama_buah, jenis_buah, harga, stok)
VALUES (1, 'Apel', 'Tropis', 5000.00, 100),
       (2, 'Jeruk', 'Agrum', 3000.00, 150),
       (3, 'Mangga', 'Subtropis', 7000.00, 50),
       (4, 'Pisang', 'Tropis', 4000.00, 80);

CREATE TABLE transaksi (
    transaksi_id INT PRIMARY KEY,
    tanggal_transaksi DATE,
    nama_pelanggan VARCHAR(100)
);

INSERT INTO transaksi (transaksi_id, tanggal_transaksi, nama_pelanggan)
VALUES (1, '2024-06-01', 'riri'),
       (2, '2024-06-02', 'ruru');

CREATE TABLE detail_transaksi (
    transaksi_id INT,
    buah_id INT,
    jumlah INT,
    PRIMARY KEY (transaksi_id, buah_id),
    FOREIGN KEY (transaksi_id) REFERENCES transaksi(transaksi_id),
    FOREIGN KEY (buah_id) REFERENCES buah(buah_id)
);

INSERT INTO detail_transaksi (transaksi_id, buah_id, jumlah)
VALUES (1, 1, 2),
       (1, 2, 3),
       (2, 3, 1),
       (2, 4, 2);
       
       
       
DELIMITER //

CREATE TRIGGER update_stok_trigger
AFTER INSERT ON detail_transaksi
FOR EACH ROW
BEGIN
    UPDATE buah
    SET stok = stok - NEW.jumlah
    WHERE id = NEW.buah_id;
END//
DELIMITER ;

CREATE VIEW buah_tersedia AS
SELECT b.buah_id, b.nama_buah, b.jenis_buah, b.harga, b.stok
FROM buah b
WHERE b.stok > 0;

-- OPERASI AGREGAT
SELECT jenis_buah, SUM(dt.jumlah) AS total_penjualan
FROM buah b
INNER JOIN detail_transaksi dt ON b.buah_id = dt.buah_id
GROUP BY jenis_buah;

-- IMPLEMENTASI INDEX
CREATE INDEX idx_buah_id ON detail_transaksi (buah_id);

-- LEFT JOIN
SELECT b.nama_buah, COALESCE(SUM(dt.jumlah), 0) AS total_penjualan
FROM buah b
LEFT JOIN detail_transaksi dt ON b.buah_id = dt.buah_id
GROUP BY b.nama_buah;

-- INNER JOIN
SELECT t.transaksi_id, t.tanggal_transaksi, t.nama_pelanggan, b.nama_buah, dt.jumlah
FROM transaksi t
INNER JOIN detail_transaksi dt ON t.transaksi_id = dt.transaksi_id
INNER JOIN buah b ON dt.buah_id = b.buah_id;

-- SUBQUERRY
SELECT nama_buah, stok
FROM buah
WHERE stok < (SELECT AVG(stok) FROM buah);

-- HAVING
SELECT jenis_buah, SUM(dt.jumlah) AS total_penjualan
FROM buah b
INNER JOIN detail_transaksi dt ON b.buah_id = dt.buah_id
GROUP BY jenis_buah
HAVING SUM(dt.jumlah) > 5;

-- WILDCARD
SELECT *
FROM buah
WHERE nama_buah LIKE '%apel%';




