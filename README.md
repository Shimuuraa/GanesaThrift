# GanesaThrift — Pasar Barang Bekas & Hibah Mahasiswa ITB

**GanesaThrift** adalah platform web *marketplace* barang bekas layak pakai dan hibah gratis khusus komunitas mahasiswa **Institut Teknologi Bandung (ITB)** (Kampus Ganesha, Jatinangor, dan area kos sekitarnya). Aplikasi ini dirancang dengan konsep **tanpa login (*zero-friction*)** sehingga mahasiswa dapat memasang barang, melakukan *booking*, dan bertransaksi secara *Cash on Delivery* (COD) dalam hitungan detik.

---

## Latar Belakang & Solusi

Setiap pergantian semester atau musim wisuda, banyak mahasiswa ITB memiliki perabot kos, elektronik, alat praktikum, hingga buku diktat TPB yang masih sangat layak pakai namun sulit dipindahkan. Di sisi lain, mahasiswa baru maupun adik tingkat membutuhkan perlengkapan kuliah dengan harga terjangkau.

GanesaThrift menjembatani kebutuhan tersebut melalui:
- **Sirkulasi Barang Kampus:** Mengurangi limbah kos sekaligus membantu mahasiswa menghemat biaya kebutuhan kuliah.
- **Kanal Hibah Gratis:** Memudahkan kakak tingkat mewariskan buku diktat, jas lab, atau perabot secara cuma-cuma kepada adik tingkat yang membutuhkan.
- **Koordinasi COD Terpusat:** Memfokuskan pertemuan pada 5 titik utama kampus dan area kos ITB.

---

## Fitur Utama

### 1. Tayang Instan Tanpa Akun (*Zero-Login Listing*)
- Penjual maupun pemberi hibah tidak perlu membuat akun atau mengingat kata sandi rumit.
- Cukup unggah foto barang (otomatis dikompresi di sisi klien hingga lebar maksimal `900px` agar ringan), isi detail kondisi (skala `1–5`), tentukan harga atau centang **Hibah Gratis**, dan barang langsung tayang di katalog.

### 2. Verifikasi Otomatis Mahasiswa ITB
- Sistem mendeteksi domain email penjual secara *real-time*.
- Penjual yang menggunakan email **`@mahasiswa.itb.ac.id`** atau **`@itb.ac.id`** otomatis memperoleh lencana terverifikasi **Mahasiswa ITB** pada kartu produk dan halaman *booking*.

### 3. Sistem *Booking* & Penahanan Barang 24 Jam
- Pembeli dapat memilih titik COD (lokasi saran penjual, Gerbang Utama ITB Ganesha, atau lokasi kustom) serta menentukan hari dan sesi waktu pertemuan (`Pagi`, `Siang`, `Sore`, `Malam`).
- Setelah dikirim, sistem membuat **Kode Booking** unik (`GT-DDMM-XXXX`), mengubah status barang menjadi **`Dalam Proses`** agar tidak diklaim ganda, dan menyediakan tautan pesan otomatis ke WhatsApp penjual.

### 4. Kontrol Status via PIN Penjual 4-Digit
- Setiap barang dilindungi oleh **4-digit PIN pribadi** yang dibuat penjual saat mengunggah barang.
- Penjual dapat membuka detail barang kapan saja untuk:
  - Menandai barang sebagai **`Sudah Tersalurkan`** setelah COD selesai.
  - Membatalkan *booking* (mengembalikan status ke **`Tersedia`**) apabila calon pembeli berhalangan hadir.

### 5. Sinkronisasi *Real-Time* & Eksplorasi Cepat
- Menggunakan **Supabase Realtime** sehingga setiap barang baru, perubahan status *booking*, maupun barang yang sudah tersalurkan langsung terbarui di layar seluruh pengguna tanpa perlu memuat ulang halaman.
- Dilengkapi pencarian kata kunci, filter 8 kategori, tab cepat (`Terbaru`, `Hibah Gratis`, `Termurah`, `Dekat Ganesha`), fitur simpan **Favorit** (`localStorage`), serta filter **5 Titik COD Kampus** (`Ganesha`, `Jatinangor`, `Cisitu`, `Dago`, `Tubagus Ismail`).

---

## Arsitektur & Teknologi

GanesaThrift dibangun dengan arsitektur *single-page* yang ringan tanpa memerlukan proses *build* kompleks:

| Lapisan | Teknologi | Kegunaan |
| :--- | :--- | :--- |
| **Struktur & UI** | HTML5 & Tailwind CSS (CDN) | Tata letak responsif dengan palet warna kampus (`#2B3D4F`, `#587B8C`, `#CBDCE8`, `#F5EFEA`) |
| **Tipografi & Ikon** | Google Fonts & Lucide Icons | Kombinasi *Plus Jakarta Sans*, *Instrument Serif*, *JetBrains Mono*, serta ikon vektor |
| **Logika Frontend** | Vanilla JavaScript (ES6+) | Manajemen *state*, kompresi gambar *canvas*, filter katalog, dan integrasi WhatsApp |
| **Database & Storage** | Supabase (PostgreSQL + Storage) | Penyimpanan data barang (`items`), bucket foto (`item-photos`), dan *WebSocket Realtime* |
| **Hosting & CI/CD** | GitHub + Vercel | *Deployment* otomatis setiap kali terdapat perubahan pada cabang utama repositori |

---

## Struktur File Repositori

Pastikan seluruh file aset foto berada di direktori utama (*root*) sejajar dengan `index.html`:

```text
GanesaThrift/
├── index.html                      # Halaman utama, antarmuka booking, modal, dan logika aplikasi
├── 1652241026-IMG_0011-01.jpeg     # Foto Hero Slide 1: Air Mancur Kampus ITB Ganesha
├── 1714369519-DSCF0906.jpg         # Foto Hero Slide 2: Wisudawan ITB
├── IMG_2948-feat.jpg               # Foto Hero Slide 3: Kampus ITB Jatinangor
└── README.md                       # Dokumentasi proyek
```

---

## Konfigurasi Backend (Supabase)

Apabila ingin menjalankan proyek ini menggunakan instans Supabase baru, jalankan konfigurasi berikut pada dashboard Supabase:

### 1. Buat Tabel `items` (SQL Editor)
Jalankan perintah SQL berikut untuk membuat tabel beserta kebijakan akses publik dan publikasi *Realtime*:

```sql
CREATE TABLE public.items (
  id BIGINT GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  category TEXT NOT NULL,
  condition INTEGER NOT NULL DEFAULT 4,
  price NUMERIC NOT NULL DEFAULT 0,
  is_free BOOLEAN NOT NULL DEFAULT FALSE,
  location TEXT NOT NULL,
  seller_name TEXT NOT NULL,
  seller_email TEXT NOT NULL,
  seller_wa TEXT NOT NULL,
  pin TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'Tersedia',
  buyer_name TEXT DEFAULT '',
  buyer_wa TEXT DEFAULT '',
  image_url TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Aktifkan Row Level Security (RLS) dengan akses baca, tambah, dan ubah publik
ALTER TABLE public.items ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Publik dapat melihat semua barang"
  ON public.items FOR SELECT
  USING (true);

CREATE POLICY "Publik dapat mengunggah barang baru"
  ON public.items FOR INSERT
  WITH CHECK (true);

CREATE POLICY "Publik dapat memperbarui status booking atau PIN"
  ON public.items FOR UPDATE
  USING (true)
  WITH CHECK (true);

-- Aktifkan pembaruan otomatis pada Supabase Realtime
ALTER PUBLICATION supabase_realtime ADD TABLE public.items;
```

### 2. Buat Storage Bucket `item-photos`
1. Buka menu **Storage** di dashboard Supabase dan klik **New bucket**.
2. Beri nama **`item-photos`** dan aktifkan opsi **Public bucket**.
3. Tambahkan kebijakan (*Policy*) pada bucket `item-photos` agar publik dapat mengunggah (`INSERT`) dan melihat (`SELECT`) file gambar.

### 3. Hubungkan Kredensial di `index.html`
Sesuaikan konstanta `SUPABASE_URL` dan `SUPABASE_ANON_KEY` pada bagian `<script>` di dalam `index.html`:

```javascript
const SUPABASE_URL = 'https://PROJECT_ID_ANDA.supabase.co';
const SUPABASE_ANON_KEY = 'SUPABASE_PUBLISHABLE_OR_ANON_KEY_ANDA';
```

---

## Panduan Deployment (GitHub ke Vercel)

1. Unggah `index.html`, `README.md`, dan ketiga file foto hero (`1652241026-IMG_0011-01.jpeg`, `1714369519-DSCF0906.jpg`, `IMG_2948-feat.jpg`) ke repositori GitHub.
2. Masuk ke dashboard [Vercel](https://vercel.com), pilih **Add New Project**, lalu impor repositori **GanesaThrift**.
3. Biarkan pengaturan *Framework Preset* pada **Other** (situs statis) dan klik **Deploy**.
4. Setiap pembaruan kode atau foto yang di-*commit* ke GitHub akan otomatis ter-*deploy* ke domain Vercel dalam waktu kurang dari satu menit.

---

## Alur Penggunaan Singkat

- **Sebagai Penjual / Pemberi Hibah:**
  1. Klik tombol **+ Jual / Hibah Barang** pada bilah navigasi.
  2. Unggah foto barang, isi deskripsi, pilih titik COD, masukkan email ITB (opsional untuk lencana verifikasi), nomor WhatsApp, serta **4-digit PIN penjual**.
  3. Setelah transaksi COD selesai, buka kartu barang terkait, pilih tab **Kelola Status (PIN Penjual)**, masukkan PIN, dan klik **Tandai Sudah Tersalurkan**.

- **Sebagai Pembeli / Penerima Hibah:**
  1. Cari barang melalui kolom pencarian, kategori, atau titik COD terdekat.
  2. Klik **Booking** (atau **Ambil Gratis** untuk barang hibah), tentukan lokasi dan jadwal pertemuan, lalu isi nama serta nomor WhatsApp aktif.
  3. Simpan **Kode Booking** yang muncul dan klik **Chat penjual via WhatsApp** untuk konfirmasi pertemuan.

---

## Tim & Lisensi

Dikembangkan sebagai inisiatif platform komunitas mahasiswa Institut Teknologi Bandung. Dirilis di bawah lisensi **MIT** — bebas digunakan dan dikembangkan lebih lanjut untuk kebutuhan ekosistem kampus.
