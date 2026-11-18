# Sistem Pemesanan Tiket Bus Antarkota: Exception Handling

Pengembangan proyek UTS Pemrograman Berorientasi Objek (studi kasus **Pemesanan Tiket Bus Antarkota**) dengan menambahkan penanganan pengecualian (exception handling) di Python.

## 1. Identitas

**Kelompok:** (isi nomor kelompok)

|             Nama             |     NIM    |       Akun Git          |
| I Komang Nanda Anggara Putra | 2501020046 | komangnandaanggara-hue  |
|         Muhamad Afif         | 2501020064 | muhamadafif2303202-code |
| Tehillah Kim Theodore Kajeng | 2501020057 |       tehiilah          |

## 2. Deskripsi Sistem

Sistem digunakan oleh penumpang dan petugas operator. Operator memiliki bus eksekutif (konfigurasi kursi 2-1, tarif 1,5 kali tarif dasar) dan bus ekonomi (2-2). Operator membuat jadwal keberangkatan dengan menunjuk satu bus. Penumpang memilih jadwal, memilih satu atau beberapa kursi, lalu membuat pemesanan. Sistem memeriksa ketersediaan kursi, menolak pemesanan yang melanggar aturan bisnis, menghitung tarif sesuai jenis bus, dan menghitung total biaya.

Kelas domain: `Bus`, `BusEksekutif`, `BusEkonomi`, `FasilitasMixin`, `Kursi`, `Penumpang`, `JadwalKeberangkatan`, `Pemesanan`.

## 3. Aturan Bisnis dan Exception

| No | Aturan bisnis | Class / method yang menegakkan | Exception |
|----|---------------|-------------------------------|-----------|
| 1 | Kursi yang sudah dipesan tidak boleh dipesan lagi | `Kursi.pesan()`, `Pemesanan.__init__` | `KursiTidakTersediaError` |
| 2 | Nomor kursi harus ada pada bus di jadwal tersebut | `JadwalKeberangkatan.cari_kursi()` | `KursiTidakDitemukanError` |
| 3 | Nama tidak boleh kosong dan NIK harus 16 digit angka | setter `Penumpang.nama`, `Penumpang.__init__`, `Penumpang.dari_teks()` | `DataPenumpangTidakValidError` |
| 4 | Tarif dasar harus lebih dari nol | setter `JadwalKeberangkatan.tarif_dasar` | `TarifTidakValidError` |
| 5 | Pemesanan minimal satu kursi dan tanpa nomor kursi ganda | `Pemesanan.__init__` | `PemesananTidakValidError` |
| 6 | Kapasitas bus harus bilangan bulat lebih dari nol | setter `Bus.kapasitas` | `KapasitasBusTidakValidError` |

## 4. Class Diagram Revisi

![Class Diagram](docs/class-diagram.png)

Keterangan: kotak berwarna adalah hierarki custom exception (stereotype `«exception»`). Panah merah putus-putus `«raises»` menunjukkan method yang memunculkan exception tersebut. Notasi visibilitas: `+` public, `#` protected, `-` private.

## 5. Daftar Custom Exception

| Exception | Induk | Atribut konteks | Dimunculkan saat | Method yang memunculkan | Penanganan di program utama |
|-----------|-------|-----------------|------------------|-------------------------|-----------------------------|
| `TiketBusError` | `Exception` | - | Base exception aplikasi | - | `except TiketBusError` sebagai penangkap umum |
| `KursiTidakTersediaError` | `TiketBusError` | `nomor` | Kursi sudah terisi | `Kursi.pesan()`, `Pemesanan.__init__` | Pesan kursi bentrok |
| `KursiTidakDitemukanError` | `TiketBusError` | `nomor` | Nomor kursi tidak ada di bus | `cari_kursi()` (`raise ... from`) | Pesan kursi tidak ada |
| `DataPenumpangTidakValidError` | `TiketBusError` | `field` | Nama kosong, NIK salah, format teks salah | setter `nama`, `__init__`, `dari_teks()` (`raise ... from`) | Pesan data salah beserta field-nya |
| `TarifTidakValidError` | `TiketBusError` | `nilai` | Tarif dasar kurang dari atau sama dengan nol | setter `tarif_dasar` | Penolakan di blok `try` terpisah |
| `PemesananTidakValidError` | `TiketBusError` | `daftar_nomor` | Daftar kursi kosong atau ganda | `Pemesanan.__init__` | Ditangkap oleh `except TiketBusError` |
| `KapasitasBusTidakValidError` | `TiketBusError` | `nilai` | Kapasitas bus bukan bilangan bulat positif | setter `Bus.kapasitas` | Penolakan di blok `try` terpisah |

Teknik yang dipakai:

- **EAFP:** `cari_kursi()` langsung mengambil dari dictionary lalu menangani `KeyError`; `dari_teks()` langsung meng-unpack hasil `split` lalu menangani `ValueError`.
- **`raise ... from`:** kedua method di atas mengubah exception bawaan menjadi custom exception tanpa kehilangan penyebab aslinya.
- **Re-raise:** `buat_pemesanan()` di `main.py` mencatat log lalu memunculkan ulang exception dengan `raise`.
- **Validasi dulu sebelum mengubah state:** `Pemesanan` memeriksa semua kursi sebelum memesan, sehingga pemesanan yang gagal tidak meninggalkan kursi setengah terpesan.

## 6. Struktur Repository

```
.
├── README.md
├── docs/
│   ├── class-diagram.png
│   ├── class-diagram.dot      (sumber diagram, dirender dengan Graphviz)
│   └── output.txt
└── src/
    ├── exceptions.py          (hierarki custom exception)
    ├── models.py              (class domain)
    └── main.py                (program utama)
```

## 7. Cara Menjalankan

Membutuhkan Python 3.8 atau lebih baru, tanpa library tambahan.

```bash
python src/main.py
```

Contoh output (potongan):

```
1. Pemesanan valid
  [OK] PSN-001: Budi, jadwal JD01, kursi 1, 2
  Total bayar: Rp600,000
  Kursi kosong tersisa: 4
2. Kursi sudah dipesan
  [LOG] pemesanan ditolak: KursiTidakTersediaError
  [GAGAL] Kursi bentrok (kursi 2): Kursi 2 sudah dipesan
  Kursi kosong tersisa: 4
```

Output lengkap ada di [docs/output.txt](docs/output.txt).

## 8. Pembagian Peran

|           Anggota            |                     Peran                      |
| I Komang Nanda Anggara Putra | (mis. `exceptions.py` dan tabel aturan bisnis) |
|         Muhamad Afif         |               (mis. `models.py`)               |
| Tehillah Kim Theodore Kajeng |   (mis. `main.py`, class diagram, dan README)  |
