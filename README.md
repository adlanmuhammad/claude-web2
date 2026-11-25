# Day Cycle Background

Background hero animasi **malam → pagi → siang → sore → malam**, dari 4 gambar ilustrasi jendela (sudah di-flip horizontal).

| File | Isi |
|---|---|
| `index.html` | Demo siap pakai, self-contained (SVG + CSS + JS inline, tanpa request tambahan) |
| `flipped/*.jpg` | 4 gambar asli yang sudah di-flip horizontal. Hanya referensi, **tidak dipakai** oleh halaman |

## Kenapa tidak pakai gambar

Keempat ilustrasi punya bentuk yang identik, hanya warnanya yang berbeda. Jadi gambar di-trace ulang menjadi vektor:

- **Bentuk disimpan sekali** (`<defs>`), lalu dipakai ulang oleh 4 layer suasana lewat `<use>`, masing-masing dengan palet warna/gradiennya sendiri.
- Lantai dan pantulan cahaya di lantai sudah dihapus. Tembok bukan lagi bagian gambar, melainkan warna background.
- Hasilnya tajam di resolusi apa pun (termasuk 4K/retina) dan jauh lebih ringan:

| | Ukuran |
|---|---|
| 4 JPG asli | 420 KB |
| `index.html` (semua suasana + bulan + burung + WebGL + CSS + JS + demo) | 85 KB, **≈28 KB gzip** |

## Struktur & cara kerja

Above the fold terdiri dari 3 lapisan:

1. **Tembok digambar WebGL, satu shader dengan pemandangan.** Canvas WebGL selebar hero menggambar tembok dan pemandangan dalam satu clock, dengan interpolasi OKLab per piksel. Cahaya seolah masuk dari jendela: tembok dekat jendela berubah sedikit lebih dulu, dan sedikit setelah langit. Warnanya tetap token HSL yang sama (`data-walls`). Untuk first paint dan fallback tanpa WebGL, tembok memakai background CSS `hsl(var(--dc-h) var(--dc-s) var(--dc-l))`; transisi CSS-nya dimatikan saat WebGL aktif.

   | Suasana | `--dc-h` | `--dc-s` | `--dc-l` |
   |---|---|---|---|
   | Malam | 231 | 55% | 17% |
   | Pagi | 258 | 31% | 24% |
   | Siang | 218 | 48% | 29% |
   | Sore | 261 | 32% | 19% |

2. **Tirai, kusen, dan jendela = 1 SVG statis** berwarna putih (atau hitam) dengan transparansi. Karena transparan, warnanya otomatis mengikuti warna tembok di belakangnya. Kusen sengaja dibuat sama dengan warna tembok.
3. **Pemandangan di luar jendela dirender WebGL.** Tiap suasana tetap digambar sebagai SVG, lalu dirender sekali menjadi tekstur GPU (ketajaman sesuai layar, maks 2× DPR). Saat transisi, shader mencampur warna tiap piksel di ruang warna **OKLab** (perseptual), jadi warna di tengah transisi tetap bersih, tidak keruh seperti crossfade biasa. Ada juga **gelombang cahaya**: saat fajar dan senja perubahan dimulai dari cakrawala lalu menyebar, sedangkan menuju siang dan malam langit berubah dari atas. Dithering menghilangkan *banding* pada gradasi gelap. GPU hanya bekerja selama transisi; saat diam tidak ada render sama sekali. Perangkat yang lambat otomatis menurunkan resolusi canvas selama transisi saja, lalu kembali tajam penuh. Kalau WebGL tidak tersedia (atau konteksnya hilang), halaman otomatis memakai crossfade SVG.

**Batas bawah section = ujung bawah tirai** di semua ukuran layar. Tinggi gambar jendela+tirai selalu sama dengan tinggi hero (`100svh`). Di layar lebar gambar rata kanan dan sisa ruang di kiri adalah warna tembok. Di layar sempit gambar digeser agar jendela tetap di tengah (`--dc-focus-x`).

Detail animasi:

- **Hemat memori:** dengan WebGL, semua layer SVG pemandangan `display:none` dan hanya ada 1 canvas. Di mode fallback, saat diam hanya 1 layer yang dirender dan saat transisi maksimal 2.
- **First paint tetap cepat:** scene malam (SVG) langsung tampil; WebGL disiapkan setelahnya dan mengambil alih tanpa kedipan.
- **Bulan** adalah layer terpisah yang di-*mask* hanya ke area langit, sehingga bisa tertutup gunung, awan, dan kusen jendela. Lintasannya **setengah lingkaran dari kiri ke kanan**:
  - sore → malam: muncul dari balik gunung di kiri, lalu naik ke posisinya di gambar malam;
  - selama malam: terus bergerak pelan melintasi puncak busur;
  - malam → pagi: turun di kanan dan tenggelam di balik gunung.

  Kecepatannya (hampir) konstan. Sabit bulan tetap tegak sepanjang lintasan.
- **Bintang:** yang statis ada di layer malam, 5 bintang berkelip ada di layer kecil terpisah.
- **Burung:** 3 siluet elang terbang dari kanan ke kiri, setiap kali halaman dimuat dan sebelum tiap pergantian suasana. Siluet dan kepakan sayapnya di-trace dari video referensi: 14 frame vektor dalam satu sprite (≈5 KB), diputar dengan `transform` + `steps()`, jadi tetap dijalankan GPU tanpa repaint. Burung terbang di depan awan, tapi tetap lewat di belakang kusen dan tirai.
- **Elemen pendukung ikut bergeser:** titik aksen dan latar teks di layar ponsel (memakai angka HSL tembok yang sama) berubah sepanjang transisi.
- **Otomatis pause** saat hero di-scroll keluar layar atau tab tidak aktif.
- **`prefers-reduced-motion`:** tampil statis, suasananya sesuai jam lokal pengunjung.
- **Tanpa JS:** suasana malam tampil statis.

## Memasang di website

Salin tiga bagian dari `index.html`:

1. **CSS** di dalam `<style>`, bagian `DAY-CYCLE BACKGROUND`. Bagian `DEMO KONTEN` opsional.
2. **Markup** `<section class="daycycle" data-daycycle …>…</section>`. Ganti isi `.dc-content` dengan konten Anda; elemen `.dc-bg` adalah background-nya.
3. **`<script>`** di akhir body.

## Konfigurasi

| Opsi | Default | Keterangan |
|---|---|---|
| `data-hold` | `3000` | Lama tiap suasana diam (ms) |
| `data-fade` | `9000` | Lama transisi (ms). Satu siklus = 4 × (hold + fade) = 48 detik |
| `data-fly` | `6000` | Lama burung melintasi jendela (ms), maksimal hold + fade |
| `data-start="clock"` | – | Mulai dari suasana sesuai jam lokal (pagi 05–11, siang 11–15, sore 15–18, malam 18–05) |
| `--dc-focus-x` | `.55` | Di layar sempit: bagian gambar yang dipertahankan. `.55` menjaga jendela di tengah. Di layar lebar gambar selalu rata kanan |

Event saat suasana berganti:

```js
document.querySelector('[data-daycycle]').addEventListener('daycycle:change', e => {
  console.log(e.detail.name); // 'malam' | 'pagi' | 'siang' | 'sore'
});
```

Atribut `data-scene` di `.daycycle` juga ikut berubah, jadi bisa dipakai di CSS, mis. `.daycycle[data-scene="siang"] { … }`.

## Dukungan browser

Memakai WebGL 1 (dengan fallback otomatis ke SVG), `@property`, container query units, `svh`, `aspect-ratio`, `mask-image`, dan WAAPI `commitStyles`: Chrome/Edge 105+, Safari 16+, Firefox 110+. Untuk browser lama ada fallback ke `vw`/`vh`.

Sudah diuji di Chromium pada ukuran desktop, laptop, tablet, ponsel portrait/landscape, dan ultrawide. Belum diuji langsung di Safari dan Firefox.
