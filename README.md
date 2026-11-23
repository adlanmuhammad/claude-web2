# Day Cycle Background

Background hero animasi **malam → pagi → siang → sore → malam**, dari 4 gambar ilustrasi jendela (sudah di-flip horizontal).

| File | Isi |
|---|---|
| `index.html` | Demo siap pakai, self-contained (SVG + CSS + JS inline, tanpa request tambahan) |
| `flipped/*.jpg` | 4 gambar asli yang sudah di-flip horizontal. Hanya referensi, **tidak dipakai** oleh halaman |

## Kenapa tidak pakai gambar

Keempat ilustrasi punya bentuk yang identik, hanya warnanya yang berbeda. Jadi gambar di-trace ulang menjadi vektor:

- **Bentuk disimpan sekali** (`<defs>`), lalu dipakai ulang oleh 4 layer suasana lewat `<use>`, masing-masing dengan palet warna/gradiennya sendiri.
- Hasilnya tajam di resolusi apa pun (termasuk 4K/retina) dan jauh lebih ringan:

| | Ukuran |
|---|---|
| 4 JPG asli | 420 KB |
| `index.html` (semua suasana + bulan + burung + CSS + JS + demo) | 81 KB, **≈26 KB gzip** |

## Cara kerja animasi

- **Transisi warna** = crossfade `opacity` antar layer suasana via Web Animations API, dengan kurva *ease-in-out sine* yang landai: transisi 9 detik, jeda diam hanya 3 detik, sehingga perubahan terasa mengalir. Animasi berjalan di compositor (GPU), tanpa repaint per frame.
- **Warna ruangan mengikuti suasana di luar jendela.** Dinding, tirai, kusen, dan lantai diwarnai cahaya luar: biru tengah malam (malam), ungu-plum fajar (pagi), biru segar (siang), ungu-plum senja dengan rim light jingga (sore). Semua warna ruangan digeser dengan delta yang sama di ruang warna OKLab, jadi warnanya tetap bersih dan kontras lipatan tirai terjaga. Teks putih tetap kontras di semua suasana.
- **Elemen pendukung ikut bergeser:** titik aksen dan latar teks di layar ponsel berubah warna sepanjang transisi, mulai bersamaan dengan crossfade.
- **Hemat memori:** saat diam hanya 1 layer yang dirender; saat transisi maksimal 2 layer. Layer lain `display:none`.
- **Bulan** adalah layer terpisah yang di-*mask* hanya ke area langit, sehingga bisa tertutup gunung, awan, dan kusen jendela. Lintasannya **setengah lingkaran dari kiri ke kanan**:
  - sore → malam: muncul dari balik gunung di kiri, lalu naik ke posisinya di gambar malam;
  - selama malam: terus bergerak pelan melintasi puncak busur;
  - malam → pagi: turun di kanan dan tenggelam di balik gunung.

  Kecepatannya (hampir) konstan. Sabit bulan tetap tegak sepanjang lintasan.
- **Bintang:** yang statis ada di layer malam, 5 bintang berkelip ada di layer kecil terpisah.
- **Burung:** 3 siluet elang terbang dari kanan ke kiri, setiap kali halaman dimuat dan sebelum tiap pergantian suasana. Siluet dan kepakan sayapnya di-trace dari video referensi: 14 frame vektor dalam satu sprite (≈5 KB), diputar dengan `transform` + `steps()`, jadi tetap dijalankan GPU tanpa repaint. Tiap burung mengepak dengan tempo dan fase berbeda. Burung terbang di depan awan, tapi tetap lewat di belakang kusen, tirai, dan gunung, dan warnanya menyesuaikan suasana.
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
| `--dc-focus-x` | `.82` | Bagian gambar yang dipertahankan saat ter-crop horizontal (layar portrait). `.82` menjaga jendela di tengah |
| `--dc-focus-y` | `.3` | Bagian yang dipertahankan saat ter-crop vertikal (layar ultrawide), supaya bulan tetap terlihat |

Event saat suasana berganti:

```js
document.querySelector('[data-daycycle]').addEventListener('daycycle:change', e => {
  console.log(e.detail.name); // 'malam' | 'pagi' | 'siang' | 'sore'
});
```

Atribut `data-scene` di `.daycycle` juga ikut berubah, jadi bisa dipakai di CSS, mis. `.daycycle[data-scene="siang"] { … }`.

## Dukungan browser

Memakai container query units, `svh`, `aspect-ratio`, `mask-image`, dan WAAPI `commitStyles`: Chrome/Edge 105+, Safari 16+, Firefox 110+. Untuk browser lama ada fallback ke `vw`/`vh`.

Sudah diuji di Chromium pada ukuran desktop, laptop, tablet, ponsel portrait/landscape, dan ultrawide. Belum diuji langsung di Safari dan Firefox.
